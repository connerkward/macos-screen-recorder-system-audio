# Using sck-record with Claude Code & AI agents

This tool also ships as a Claude Code skill, so an agent can record the screen
(with system audio) on its own. For the human walkthrough, see the
[README](../README.md).

## Install as a Claude Code plugin

Add this repo as its own marketplace:

```
/plugin marketplace add connerkward/macos-screen-recorder-system-audio
/plugin install macos-screen-recorder@macos-screen-recorder
```

Or pull it in as part of the full set:

```
/plugin marketplace add connerkward/connerkward-skills
```

Or just drop this repo's `SKILL.md` into your agent's skills directory.

## What the agent gets

Once installed, the skill triggers when a task needs a scripted screen recording
**with system sound** on macOS — demos, captures, recording a `say`/TTS voice
playing through the speakers — the case `screencapture -v` and QuickTime can't
cover without a virtual audio device.

The skill drives the same single command a human runs:

```
./sck-record <out.mp4> <seconds>
```

## Build note for automated setups

The compiled `sck-record` binary is gitignored. An agent (or a machine-setup
script) builds it once from source:

```sh
swiftc -O sck-record.swift -o sck-record
```

After that, the binary is reused. The first recording still needs the one-time
**Screen Recording** permission granted to whatever app shells out the command —
see [Permissions in the README](../README.md#permissions).

## Where it fits

`sck-record` is the raw capture primitive. A broader record-then-polish workflow
(auto-zoom, idle speed-up, cursor smoothing, vertical export) is a separate layer
that *calls* this tool rather than reimplementing capture. Keep this one narrow:
it captures, nothing more.
