# Jarvis specification

Status: draft. Nothing is built. Items marked **(verify)** are assumptions that
have not been tested and are tracked in [Open questions](#11-open-questions).

## 1. Summary

Jarvis is a self-hosted voice assistant for one household. A small microphone
puck sits in each room. When it hears the wake word it streams audio to an
always-on PC, which transcribes it, hands the text to a local LLM that can call
tools, and speaks the reply back through the puck in the room the request came
from.

The system reuses existing open-source parts for everything that is already a
solved problem (far-field audio, wake word, speech-to-text, text-to-speech,
device control). The code written for this project is the **orchestrator** (the
brain that runs the LLM and its tools) and the **tool servers** it calls.

## 2. Goals and non-goals

### Goals

1. Ask a question from anywhere in a room and get a spoken answer.
2. Set timers and have them ring in the room they were set from.
3. Control smart-home devices by voice.
4. Play music from Spotify through speakers in the room.
5. Add a new capability by adding a tool server, without changing the orchestrator.
6. Run entirely on the local network. No cloud service is needed for the core loop.
7. Move the LLM, or the whole stack, from the PC to a Mac by changing configuration.

### Non-goals, for now

- Building custom microphone hardware or firmware.
- A phone app. A push-to-talk app is a possible later addition.
- Recognising who is speaking.
- Access from outside the home network.
- Conversations that continue without the wake word.

## 3. Decisions made

| Decision | Choice | Why |
|---|---|---|
| Room hardware | Home Assistant Voice PE, one per room | Solves far-field microphones, wake word and echo cancellation in one device, with no Raspberry Pi per room |
| Hub | Home Assistant, used as plumbing | The pucks are built to talk to it, and it already integrates Tapo and most other devices |
| Brain | Own orchestrator service | This is the part worth owning: prompts, tools, memory, behaviour |
| Tools | MCP servers | A standard contract; Home Assistant already ships an MCP server |
| LLM serving | Ollama, behind an OpenAI-compatible endpoint | Runs on both the PC and the Mac; the orchestrator does not care which |
| Server | PC with an RTX 3080 Ti (12 GB), on 24/7 | Fast enough for speech and an 8–12B model together |
| Music | Music Assistant with Spotify, played through the puck's 3.5mm output | Audio the puck plays is audio it can cancel, so it still hears "stop" |

## 4. Architecture

### 4.1 Components

| Component | Responsibility | Built or reused |
|---|---|---|
| Voice PE puck | Wake word, microphones, echo cancellation, audio out | Reused |
| Home Assistant | Puck management, the voice pipeline, device integrations, announcements | Reused |
| Whisper (Wyoming) | Speech-to-text | Reused |
| Piper (Wyoming) | Text-to-speech | Reused |
| Jarvis conversation agent | Thin Home Assistant integration that forwards transcribed text to the orchestrator | **Built** (small) |
| Orchestrator | Turn loop: context, LLM calls, tool calls, reply. Owns reminders and other scheduled speech | **Built** |
| Ollama | Serves the LLM | Reused |
| Tool servers | One per capability, speaking MCP | Mix: Home Assistant's is reused, others **built** |
| Music Assistant | Streams Spotify to players | Reused |

### 4.2 One turn

1. The puck matches the wake word locally and starts streaming audio to Home Assistant.
2. Home Assistant sends the audio to Whisper and gets text back.
3. Home Assistant first tries its built-in intents ("prefer handling commands locally"). Timers and simple commands stop here.
4. Anything else goes to the Jarvis conversation agent, which posts the text, the room and a session id to the orchestrator.
5. The orchestrator builds the prompt, calls the LLM with the list of available tools, runs any tool calls it asks for, and loops until the LLM produces a reply.
6. The reply text returns to Home Assistant, which sends it to Piper and plays the audio on the puck.

### 4.3 Proposed repository layout

```
jarvis/
  orchestrator/                 the brain
  tools/                        one folder per MCP tool server
  homeassistant/
    custom_components/jarvis/   the thin conversation agent
  deploy/                       compose files and example config
  docs/                         the GitHub Pages site
  SPEC.md
```

## 5. Interfaces

### 5.1 Home Assistant to orchestrator

One HTTP endpoint. Home Assistant calls it once per utterance.

`POST /v1/converse`

```json
{
  "text": "what's the weather doing tomorrow",
  "session_id": "01JCX4...",
  "satellite_id": "assist_satellite.kitchen",
  "room": "kitchen",
  "language": "en"
}
```

Response:

```json
{
  "reply": "Showers in the morning, clearing by lunch.",
  "session_id": "01JCX4...",
  "continue_listening": false
}
```

- `session_id` ties follow-up utterances to the same conversation. The orchestrator expires a session after a short idle period.
- `room` and `satellite_id` come from Home Assistant, never from the LLM.
- `continue_listening` asks the puck to reopen the microphone for an answer to a question Jarvis just asked.
- A streaming variant of this endpoint is expected later so that speech can start before the full reply exists **(verify)**.

### 5.2 Orchestrator to LLM

The OpenAI-compatible chat completions API with tool calling, at a base URL
from configuration. Ollama provides this on both the PC and the Mac. Nothing in
the orchestrator may depend on Ollama-specific behaviour.

### 5.3 Orchestrator to tools

The orchestrator is an MCP client. At start it connects to every tool server
listed in configuration, reads their tool lists, and offers those tools to the
LLM.

Rules for tool servers:

1. One server per capability, runnable and testable on its own.
2. Tool names and descriptions are written for the LLM: short, concrete, one job each.
3. A tool that acts in a room takes a `room` argument. The orchestrator fills it from the request context; the LLM is not asked to guess it.
4. A tool returns a short result the LLM can turn into speech. It does not return audio.
5. A failing tool returns an error message, and the orchestrator lets the LLM explain the failure aloud.

Initial tool servers:

| Server | Tools | Source |
|---|---|---|
| `home-assistant` | Device control and state | Home Assistant's MCP Server integration |
| `music` | Play, pause, skip, volume, by room | Built; calls Music Assistant through Home Assistant |
| `reminders` | Create, list and cancel reminders | Built; uses the orchestrator's scheduler |

### 5.4 Orchestrator to a room

Speech that is not a reply to a request (a reminder firing, a long-running task
finishing) is pushed. The orchestrator calls Home Assistant's
`assist_satellite.announce` action with the target puck and the text. Home
Assistant synthesises it and plays it in that room.

This is the only path by which the server starts speaking on its own.

## 6. Behaviours

### 6.1 Questions

Answered by the LLM from its own knowledge. Replies are written for the ear:
one or two sentences, no lists, no markup.

### 6.2 Timers

Handled by Home Assistant's built-in intents before the orchestrator is
involved. The timer runs on the puck that heard the request and rings there.
This needs no code.

### 6.3 Reminders

Owned by the orchestrator, because they carry text and can outlive a puck
restart. Stored on disk, fired through the announce path in 5.4.

### 6.4 Device control

The LLM calls a tool on the `home-assistant` server. "The lamp" is resolved
using the room the request came from.

### 6.5 Music

The LLM calls the `music` tool with a query and the room. The tool asks Music
Assistant to play it on that room's player, which is the puck with a powered
speaker on its 3.5mm output **(verify)**. Audio never passes through the
orchestrator or the LLM.

## 7. Latency

Target: under 2 seconds from the end of speech to the first spoken word. These
are targets to measure against, not measurements.

| Stage | Budget |
|---|---|
| Detecting the end of speech | 0.5–0.7 s |
| Speech-to-text | 0.3 s |
| LLM, to the end of the first sentence | 0.5–0.8 s |
| Text-to-speech, to first audio | 0.2 s |

Every turn is logged with per-stage timings from the first day, so regressions
are visible.

## 8. Deployment and portability

Everything runs on the PC to begin with. Each part is a network service with
its own address, so moving a part means changing a host name.

| Service | Default port | Notes |
|---|---|---|
| Home Assistant | 8123 | |
| Whisper (Wyoming) | 10300 | Uses the GPU |
| Piper (Wyoming) | 10200 | |
| Ollama | 11434 | Uses the GPU |
| Orchestrator | 8700 | Proposed |
| Music Assistant | 8095 | |

Moving to the Mac:

- Containers on macOS cannot use the GPU. Ollama and Whisper run natively there; everything else can stay in containers.
- Parts can move one at a time. The LLM on the Mac with speech still on the PC is a valid arrangement.
- The pucks only know Home Assistant's address and are unaffected.

## 9. Hardware per room

| Item | Purpose |
|---|---|
| Home Assistant Voice PE | Microphones, wake word, echo cancellation |
| Powered speaker with a 3.5mm input | Replies and music, at a quality the puck's own speaker cannot manage |
| USB power | For the puck |

No add-ons (Home Assistant Green, Zigbee or Z-Wave radios) are needed.

## 10. Build phases

Each phase ends with something that works and can be lived with.

| Phase | What gets built | Done when |
|---|---|---|
| 0. Bench | Home Assistant, Whisper, Piper and Ollama running on the PC, tested from a browser | A typed and a spoken question both get a spoken answer |
| 1. One room | One puck on the stock pipeline with local intent handling | Questions and timers work from across the room |
| 2. Own brain | The orchestrator and the conversation agent; the `home-assistant` tool server connected | A device can be controlled by voice through the orchestrator, with timings logged |
| 3. Music | Music Assistant, Spotify, the `music` tool, a powered speaker | "Play some jazz" plays in the room, and "stop" is heard over it |
| 4. Whole house | More pucks; reminders and announcements | A reminder set in one room can be delivered in another |
| 5. Later | Mac migration, memory across conversations, a push-to-talk phone app | To be specified |

## 11. Open questions

1. **Echo cancellation on the 3.5mm output.** Does the puck still hear the wake word with music playing loudly through an external speaker? Test with the first puck before buying more speakers.
2. **Music Assistant targeting the puck.** Confirm the Voice PE works as a Music Assistant player, including ducking the music while Jarvis speaks.
3. **Streaming replies.** Can a custom conversation agent stream partial text into Home Assistant's text-to-speech, or does the reply have to be complete first? This decides whether the latency target is reachable for long answers.
4. **Which model.** It must fit in 12 GB beside Whisper and call tools reliably. Needs a small evaluation set of real requests rather than a guess.
5. **Orchestrator language.** The conversation agent must be Python. The orchestrator can be anything with an MCP client and an HTTP server.
6. **Spotify access.** Premium is required for playback control, and Spotify's developer terms have tightened. Check what Music Assistant needs at the time.
7. **PC operating system.** Windows with WSL2 or Linux changes how the GPU reaches the containers.
8. **Knowledge beyond the model.** Weather, news and "what's on my calendar" each need a tool. Which come first?

## 12. Risks

| Risk | Effect | Mitigation |
|---|---|---|
| Small models call tools unreliably | Wrong or missing actions | Keep tools few and sharply described; let Home Assistant handle common commands locally; evaluate models against real requests |
| Latency above target | Feels slow, gets abandoned | Per-stage timing from day one; streaming; a smaller model for simple turns |
| Wake word misses over music | "Stop" is not heard | Route music through the puck; test early (question 1) |
| The PC is off or busy gaming | The assistant is down or slow | Accept for now; the Mac migration path exists for this |
