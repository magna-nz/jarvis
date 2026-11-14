# Jarvis feature tracker

What we are building, in the order we are building it. Tick a box here when
its issue closes, which means its "done when" line is true.

[SPEC.md](SPEC.md) says how each part works. This file is the map. The sub-work
for each item lives in its GitHub issue, as sub-issues, and nowhere else.

## How this is organised

- A **component** is a piece of the system that has to exist (the orchestrator, a tool server).
- A **feature** is something you can say to Jarvis and have happen. A feature is done only when it works end to end.
- A **slice** is the set of components and features built together, and is a GitHub milestone. One slice at a time.
- Sub-work is planned in detail only for the current slice and the next one. Later items have an issue but no sub-issues until we get there, because several depend on tests we cannot run yet.

Markers: 🔌 needs the puck · 🔊 needs a powered speaker · ❓ blocked on an open question in the spec

Labels on GitHub: [`component`](https://github.com/magna-nz/jarvis/labels/component) · [`feature`](https://github.com/magna-nz/jarvis/labels/feature) · [`task`](https://github.com/magna-nz/jarvis/labels/task) · [`needs-puck`](https://github.com/magna-nz/jarvis/labels/needs-puck) · [`needs-speaker`](https://github.com/magna-nz/jarvis/labels/needs-speaker) · [`blocked-question`](https://github.com/magna-nz/jarvis/labels/blocked-question)

## Slices

| Slice | Contents | State |
|---|---|---|
| [1. Typed end to end](https://github.com/magna-nz/jarvis/milestone/1) | C1, C2, C3, C4, C5 · F1 (typed), F2 (demo lights) | Next |
| [2. Reminders and model choice](https://github.com/magna-nz/jarvis/milestone/2) | C6, C7 · F3 (without speech) | Planned |
| [3. First room](https://github.com/magna-nz/jarvis/milestone/3) | C8 · F1 (spoken), F2 (real device), F3 (spoken), F4 | Waiting for the puck |
| [4. Music](https://github.com/magna-nz/jarvis/milestone/4) | C9 · F5 | Waiting for the puck and a speaker |
| [5. Whole house](https://github.com/magna-nz/jarvis/milestone/5) | F6, F7 | Later |
| [6. Later](https://github.com/magna-nz/jarvis/milestone/6) | F8, F9, F10, F11 | Not planned |

A feature's issue sits in the slice where it is finished. Its sub-issues sit in the slice where each one is built.

## Components

- [ ] **C1. Platform** ([#1](https://github.com/magna-nz/jarvis/issues/1)) · slice 1
  Done when one command brings the services up on the Mac, and a typed question in Home Assistant's Assist box gets an answer from the stock pipeline.
- [ ] **C2. Orchestrator core** ([#2](https://github.com/magna-nz/jarvis/issues/2)) · slice 1
  Done when `POST /v1/converse` with a plain question returns an answer from the LLM, and the tests pass with a fake LLM.
- [ ] **C3. Tool calling** ([#3](https://github.com/magna-nz/jarvis/issues/3)) · slice 1
  Done when a fake tool server's tool is offered to the LLM, called, and its result is used in the reply, under test.
- [ ] **C4. Jarvis agent for Home Assistant** ([#4](https://github.com/magna-nz/jarvis/issues/4)) · slice 1
  Done when Jarvis can be chosen as the conversation agent in a voice pipeline, and text typed into Assist reaches the orchestrator with the right room.
- [ ] **C5. Timing and logs** ([#5](https://github.com/magna-nz/jarvis/issues/5)) · slice 1
  Done when each turn writes one log line with time spent in the LLM, in each tool, and in total.
- [ ] **C6. Scheduler and state** ([#6](https://github.com/magna-nz/jarvis/issues/6)) · slice 2
  Done when a reminder created before a restart still fires after it.
- [ ] **C7. Model evaluation** ([#7](https://github.com/magna-nz/jarvis/issues/7)) · slice 2
  Done when a set of real requests has been run against the candidate models, and one is chosen with its accuracy and speed written down.
- [ ] **C8. Room hardware** ([#8](https://github.com/magna-nz/jarvis/issues/8)) · slice 3 · 🔌
  Done when the puck is on the network, set up in Home Assistant, and using the Jarvis pipeline.
- [ ] **C9. Music stack** ([#9](https://github.com/magna-nz/jarvis/issues/9)) · slice 4 · 🔌 🔊 ❓
  Done when Music Assistant can play a Spotify track on the puck.

## Features

- [ ] **F1. Ask a question** ([#10](https://github.com/magna-nz/jarvis/issues/10)) · typed in slice 1, spoken in slice 3 🔌
  Done when a spoken question from across the room is answered through the orchestrator, with timings inside the latency target.
- [ ] **F2. Control devices** ([#11](https://github.com/magna-nz/jarvis/issues/11)) · demo light in slice 1, real device in slice 3 🔌
  Done when "turn off the lamp" switches off the lamp in the room the request came from.
- [ ] **F3. Reminders** ([#12](https://github.com/magna-nz/jarvis/issues/12)) · fires in slice 2, spoken in slice 3 🔌
  Done when "remind me at six to ring Mum" is spoken in the room at six.
- [ ] **F4. Timers** ([#13](https://github.com/magna-nz/jarvis/issues/13)) · slice 3 · 🔌
  Done when "set a timer for ten minutes" rings on the puck, without reaching the orchestrator.
- [ ] **F5. Play music** ([#14](https://github.com/magna-nz/jarvis/issues/14)) · slice 4 · 🔌 🔊 ❓
  Done when "play some jazz" plays in the room, and "stop" is heard over it.
- [ ] **F6. More rooms** ([#15](https://github.com/magna-nz/jarvis/issues/15)) · slice 5 · 🔌
  Done when a second puck works, and each request acts in its own room.
- [ ] **F7. Follow-up without the wake word** ([#16](https://github.com/magna-nz/jarvis/issues/16)) · slice 5 · 🔌
  Done when Jarvis can ask a question and hear the answer.

### Later, not planned

- [ ] **F8. Knowledge tools** ([#17](https://github.com/magna-nz/jarvis/issues/17)): weather, calendar, news
- [ ] **F9. Memory across conversations** ([#18](https://github.com/magna-nz/jarvis/issues/18))
- [ ] **F10. Mac migration** ([#19](https://github.com/magna-nz/jarvis/issues/19))
- [ ] **F11. Push-to-talk phone app** ([#20](https://github.com/magna-nz/jarvis/issues/20))
