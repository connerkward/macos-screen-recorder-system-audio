# sck-record — macOS screen recorder with system audio

**Record your macOS screen *with the sound your Mac is playing* — no virtual audio
driver, no BlackHole, no Loopback, no sudo.** It's a single Swift file built on
[ScreenCaptureKit](https://developer.apple.com/documentation/screencapturekit),
Apple's native capture framework.

macOS's built-in `screencapture -v` records video only — its `-g` flag captures the
*microphone*, not what your speakers are playing. The usual fix is installing a
virtual audio device and rerouting your output through it. ScreenCaptureKit makes
that unnecessary: since macOS 13 it can tap system audio directly, with nothing but
the standard Screen Recording permission. This is the smallest possible wrapper
around that capability — one file, one binary, one command.

## What it does

- Records the **main display** (cursor included) plus the **system audio** your Mac
  is outputting — app sounds, browser tabs, `say` voices, music, all of it.
- Writes a standard **H.264 / AAC `.mp4`** you can share immediately (8 Mbps video,
  48 kHz stereo 160 kbps AAC).
- Captures at retina resolution, capped at 2560 px wide to keep files sane, up to
  30 fps.
- No daemons, no kernel extensions, no audio rerouting. **What you hear is
  untouched while recording.**

## Requirements

- **macOS 13 (Ventura) or later** — the system-audio capture API
  (`SCStreamConfiguration.capturesAudio`) arrived in macOS 13.
- Xcode Command Line Tools (for `swiftc` / `swift`): `xcode-select --install`

## Build

```sh
swiftc -O sck-record.swift -o sck-record
```

Or skip the build and run it directly as a script:

```sh
swift sck-record.swift out.mp4 10
```

## Usage

```
sck-record [output.mp4] [seconds] [--no-cursor]
```

| Argument | Default | Description |
|---|---|---|
| `output.mp4` | `~/Desktop/sck.mp4` | Output file path. Overwritten if it exists. |
| `seconds` | `20` | Recording duration in seconds (fractional values allowed). |
| `--no-cursor` | off | Omit the system cursor (for tools that overlay a smoothed synthetic one). |

```sh
./sck-record                          # 20s recording to ~/Desktop/sck.mp4
./sck-record demo.mp4 5               # 5s recording to ./demo.mp4
./sck-record /tmp/clip.mp4 12.5       # 12.5s recording
```

The tool prints the output path and exits; recording stops automatically after the
requested duration.

> Want an AI agent (Claude Code) to drive this for you? It ships as a skill —
> see **[docs/agents.md](docs/agents.md)**.

## Permissions

The first run triggers macOS's **Screen Recording** permission prompt for the app
that launched the binary (your terminal). Grant it under **System Settings →
Privacy & Security → Screen & System Audio Recording**, then re-run. This is the
same permission `screencapture` uses — no separate audio permission is needed,
because ScreenCaptureKit delivers system audio under the screen-recording grant.

No microphone permission is requested; the microphone is never touched.

## How it compares

| Approach | System audio | Setup | Notes |
|---|---|---|---|
| **sck-record** | ✅ native | none | One binary, one permission. |
| `screencapture -v` (built-in) | ❌ | none | `-g` records the **mic**, not system output. |
| BlackHole / Loopback / Soundflower | ✅ via reroute | install driver, multi-output device, switch output | Rerouting changes what *you* hear; driver installs may need security approval. |
| OBS | ✅ via plugin/driver | install app, configure scenes + audio | Powerful, but heavyweight for "give me an mp4 of the next 10 seconds". |
| QuickTime screen recording | ❌ (mic only) | none | Needs a virtual driver for system audio, same as above. |

The honest pitch: this isn't "better than OBS or Screen Studio." It fills one
narrow gap they don't — **system audio from a headless CLI with zero install.**

## How it works

ScreenCaptureKit's `SCStream` is configured with `capturesAudio = true`, which
yields system-audio sample buffers alongside the screen frames. Both are muxed in
real time into an `.mp4` by `AVAssetWriter` (H.264 video + AAC audio). Audio and
video stay in sync because both inputs share the stream's presentation timestamps;
writing starts on the first complete video frame. The whole thing is ~90 lines —
read [sck-record.swift](sck-record.swift).

## Limitations

- Records the **main display** only — no window or region selection.
- Fixed duration set up front; no interactive stop key. `Ctrl-C` aborts without
  finalizing the file.
- Output codec/bitrate are hardcoded (H.264 8 Mbps / AAC 160 kbps). Want HEVC or
  different rates? It's one dictionary in the source.

## License

[MIT](LICENSE) © Conner K Ward
