# macos-screen-recorder

**macos-screen-recorder-system-audio** is a tiny Swift CLI that records the macOS main display **with system audio** via ScreenCaptureKit — no BlackHole/loopback driver, no sudo, just the standard Screen Recording permission. Ships a Claude Code skill too.

> **What this is:** A new tool with one narrow, truthful differentiator: system audio from the CLI with zero install. Not 'better than OBS/Screen Studio' — it fills the headless-CLI-with-system-audio gap.

## Install (Claude Code plugin)

```
/plugin marketplace add connerkward/macos-screen-recorder-system-audio
/plugin install macos-screen-recorder@macos-screen-recorder
```

Or install the whole set: `/plugin marketplace add connerkward/connerkward-skills`. Or drop this repo's `SKILL.md` into your agent's skills directory.

## License

MIT © Conner K Ward

---

# sck-record — macOS screen recorder with system audio (no BlackHole)

**Record your macOS screen *with system audio* — no virtual audio driver, no BlackHole, no Loopback, no sudo.** A single-file Swift CLI built on [ScreenCaptureKit](https://developer.apple.com/documentation/screencapturekit), Apple's native capture framework.

macOS's built-in `screencapture -v` records video only (its `-g` flag captures the *microphone*, not what your Mac is playing). The usual workaround is installing a virtual audio device (BlackHole, Loopback, Soundflower) and rerouting your output through it. ScreenCaptureKit makes all of that unnecessary: since macOS 13 it can tap system audio directly, with nothing but the standard Screen Recording permission. This tool is the smallest possible wrapper around that capability — one Swift file, one binary, one command.

## What it does

- Records the **main display** (cursor included) and the **system audio** your Mac is outputting — app sounds, browser tabs, `say` voices, music, everything.
- Writes a standard **H.264 / AAC `.mp4`** you can share immediately (8 Mbps video, 48 kHz stereo 160 kbps AAC).
- Captures at retina resolution, capped at 2560 px wide to keep file sizes sane, at up to 30 fps.
- No configuration, no daemons, no kernel extensions, no audio rerouting. Your sound output is untouched while recording.

## Requirements

- **macOS 13 (Ventura) or later** — the system-audio capture API (`SCStreamConfiguration.capturesAudio`) was introduced in macOS 13.
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
| `--no-cursor` | off | Omit the system cursor from the capture (for post-production tools that overlay a smoothed synthetic cursor). |

Examples:

```sh
./sck-record                          # 20s recording to ~/Desktop/sck.mp4
./sck-record demo.mp4 5               # 5s recording to ./demo.mp4
./sck-record /tmp/clip.mp4 12.5       # 12.5s recording
```

The tool prints the output path on completion and exits. Recording stops automatically after the requested duration.

## Permissions

The first run triggers macOS's **Screen Recording** permission prompt for the app that launched the binary (your terminal). Grant it under **System Settings → Privacy & Security → Screen & System Audio Recording**, then re-run. This is the same permission `screencapture` uses — no additional audio permission is needed, because ScreenCaptureKit delivers system audio under the screen-recording grant.

No microphone permission is requested; the microphone is never touched.

## How it compares

| Approach | System audio | Setup | Notes |
|---|---|---|---|
| **sck-record** | ✅ native | none | One binary, one permission. |
| `screencapture -v` (built-in) | ❌ | none | `-g` records the **mic**, not system output. |
| BlackHole / Loopback / Soundflower | ✅ via reroute | install driver, create multi-output device, switch output | Audio rerouting affects what *you* hear; driver installs may need security approval. |
| OBS | ✅ via plugin/driver | install app, configure scenes + audio capture | Powerful, but heavyweight for "give me an mp4 of the next 10 seconds". |
| QuickTime screen recording | ❌ (mic only) | none | Needs a virtual driver for system audio, same as above. |

## How it works

ScreenCaptureKit's `SCStream` is configured with `capturesAudio = true`, which yields system-audio sample buffers alongside the screen frames. Both streams are muxed in real time into an `.mp4` by `AVAssetWriter` (H.264 video + AAC audio). Audio/video stay in sync because both inputs share the stream's presentation timestamps; writing starts on the first complete video frame.

The whole implementation is ~90 lines — read [sck-record.swift](sck-record.swift).

## Limitations

- Records the **main display** only (no window/region selection — keep it simple).
- Fixed duration set up front; no interactive stop key. `Ctrl-C` aborts without finalizing the file.
- Output codec/bitrate are hardcoded (H.264 8 Mbps / AAC 160 kbps). Edit the source if you want HEVC or different rates — it's one dictionary.

## License

[MIT](LICENSE)
