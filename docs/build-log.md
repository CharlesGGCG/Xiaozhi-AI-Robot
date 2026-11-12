# Build log

One month, from an empty toolchain to a handheld device that answers and acts. Dates come from the
internship log; every phase below was completed.

## Timeline

| Target | Phase | Window | Status | Issues hit |
|---|---|---|---|---|
| by the 18th | Program environment | Jun 17 – Jun 18 | Done | none |
| by the 20th | Parts | Jun 19 – Jun 21 | Done | reshipment; microphone dead; OLED dead |
| by the 25th | Flash firmware + assemble | Jun 20 – Jun 27 | Done | voice module would not flash; cold solder joint; flashing program unusable |
| by the 7th | Backend, local deployment | Jul 1 – Jul 7 | Done | npm download errors; front-end file permission |
| by the 9th | LLM deployment + firmware config | Jul 8 – Jul 14 | Done | Anaconda environment not recognised |
| by the 12th | Full local deployment | Jul 8 – Jul 12 | Done | — |
| by the 14th | Documentation | Jul 10 – Jul 15 | Done | — |

## What each phase actually meant

### 1. Environment (Jun 17–18)

Source tree, toolchain, and the local machine prepared. The important decision here was to treat
the project as a **system**: firmware, server, console and models were going to run together, so the
environment had to be reproducible before any hardware arrived.

### 2. Parts (Jun 19–21)

Ordered the ESP32-S3 board, microphone, OLED, speaker and wiring. Two parts arrived dead — the
microphone and the display — and replacements had to be shipped. This is where the project learned
its first real lesson: **a “silent microphone” is not always a software bug.**

### 3. Flash + assemble (Jun 20–27)

Firmware flashing, wiring, and the first end-to-end attempt. Three separate failures stacked on top
of each other: the voice module would not flash, the main board had a cold solder joint, and the
flashing tool itself was misbehaving. Each had to be isolated before the next could be seen.

### 4. Backend, local (Jul 1–7)

Docker for MySQL and Redis, the Java `manager-api`, and the Vue `manager-web` console. Two
environment problems: npm could not download packages reliably, and the front-end build hit file
permissions. Both were local machine problems, not project code.

### 5. Models + firmware config (Jul 8–14)

Configured the model provider in the console, wrote the OTA and WebSocket endpoints into the
firmware config, and started the Python voice server. The Anaconda environment was not recognised
at first, so it was rebuilt cleanly.

### 6–7. Full local run + documentation (Jul 8–15)

With the whole stack on one laptop, the device could bind, connect over WebSocket, and execute
commands. The last days went into screenshots, the issue list, and this write-up.

## Issue table

| # | Symptom | Layer | Resolution |
|---|---|---|---|
| 1 | Microphone captured nothing | Hardware | replaced (parts reshipment) |
| 2 | OLED stayed dark | Hardware | replaced |
| 3 | Voice module would not flash | Firmware / toolchain | re-flashed after fixing the toolchain |
| 4 | Main board had a cold solder joint | Hardware | re-soldered |
| 5 | Flashing program unusable | Toolchain | reinstalled / reconfigured |
| 6 | npm downloads failed | Front-end toolchain | switched registry / retried |
| 7 | Front-end file permission error | OS / front-end | fixed permissions |
| 8 | Anaconda environment not recognised | Python | recreated the environment |

## What I would do differently

- **Test the parts on arrival.** A five-minute microphone check would have saved a day of
  debugging the wrong layer.
- **Write the environment down as you build it.** Reconstructing “which Python, which JDK, which
  port” later is slower than keeping a one-page runbook from day one.
- **Keep a failure log in the repo, not in chat.** This file is the result of that lesson.
