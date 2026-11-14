# Jarvis

A self-hosted voice assistant for the house. You talk to a small puck in each
room; an always-on PC transcribes what you said, runs a local LLM that can call
tools, and speaks the answer back in the room you asked from.

**Status: design only. Nothing here is built yet.**

- [SPEC.md](SPEC.md) is the specification: goals, components, interfaces, build phases and open questions.
- [The docs site](https://magna-nz.github.io/jarvis/) has the diagrams and a walkthrough of how a request moves through the system.

## The shape of it

| Part | What it does | Runs on |
|---|---|---|
| Voice PE puck, one per room | Microphones, wake word, echo cancellation, speaker output | The puck |
| Home Assistant | Manages the pucks, routes audio, controls devices, hosts Music Assistant | PC |
| Whisper and Piper | Speech-to-text and text-to-speech | PC |
| Orchestrator | The brain: takes the transcribed text, runs the LLM and its tools, returns a reply | PC |
| Ollama | Serves the local LLM behind an OpenAI-compatible endpoint | PC now, Mac later |
| Tool servers (MCP) | One per capability: devices, music, timers | PC |

The orchestrator and the tool servers are the code this repo will hold. The
rest is existing open-source software, configured.
