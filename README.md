<p align="center">
  <img src="docs/assets/social-preview.png" alt="conv" width="100%">
</p>

# conv

> Mac-native universal file and media converter for developers who want one command, adaptive defaults, and benchmark-backed ffmpeg flags instead of hand-tuned conversion scripts.

[![macOS](https://img.shields.io/badge/macOS-11%2B-000000?logo=apple&logoColor=white)](https://www.apple.com/macos/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![ShellCheck](https://github.com/Supersynergy/conv/actions/workflows/shellcheck.yml/badge.svg)](https://github.com/Supersynergy/conv/actions/workflows/shellcheck.yml)
[![Smoke](https://github.com/Supersynergy/conv/actions/workflows/smoke.yml/badge.svg)](https://github.com/Supersynergy/conv/actions/workflows/smoke.yml)

```bash
conv input.any output.any
```

`conv` routes video, audio, image, and document conversions through the fastest available local tool for your Mac. It detects Apple Silicon vs Intel, RAM tier, VideoToolbox support, and safe parallelism, then applies defaults learned from 55+ local benchmarks.

[🚀 Quickstart](#quickstart) · [🎬 Try it](#try-it) · [🗺️ Tool map](#tool-map) · [⚙️ Profiles](#adaptive-profiles) · [🔒 Limits](#limits)

## Quickstart

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Supersynergy/conv/main/install.sh)"
source ~/.zshrc
conv_info
```

Expected result: `conv_info` prints your detected profile and `conv --help` is available in the current shell.

Prefer to inspect first:

```bash
git clone https://github.com/Supersynergy/conv
cd conv
sed -n '1,140p' install.sh
```

Uninstall:

```bash
curl -fsSL https://raw.githubusercontent.com/Supersynergy/conv/main/uninstall.sh | bash
```

## Try It

```bash
conv movie.mkv movie.mp4        # remux to MP4 when copy is safe
conv photo.heic photo.jpg       # native HEIC to JPEG through sips
conv voice.wav voice.mp3        # speech-friendly MP3 defaults
conv design.png design.jxl      # JPEG XL compression
conv notes.md notes.pdf         # Markdown to PDF through pandoc/typst
```

Typical benchmark examples from the included M4 Max run:

<details>
<summary><strong>📋 Show 5 entries</strong></summary>

- **MKV to MP4 remux**<br>
  Result: `0.18s`

- **HEIC to JPEG**<br>
  Result: `0.20s`

- **WAV voice to MP3 V2**<br>
  Result: `0.21s`, 95% smaller

- **PNG to JXL q85**<br>
  Result: `0.49s`, 98.4% smaller

- **Typst to PDF**<br>
  Result: `0.22s`

</details>


See [BENCHMARKS.md](BENCHMARKS.md) for the full data and caveats.

## Why It Exists

Raw `ffmpeg` is powerful, but the defaults are easy to get wrong. On the benchmark input, this one missing bitrate flag changed the output size by an order of magnitude:

```text
ffmpeg -c:v h264_videotoolbox            1791% output ratio
ffmpeg -c:v h264_videotoolbox -b:v 6000k  744% output ratio
ffmpeg -c:v h264_videotoolbox -b:v 4000k  496% output ratio
ffmpeg -c:v h264_videotoolbox -b:v 2000k  277% output ratio
```

`conv` keeps the useful part of ffmpeg and removes the repeated decision work: format routing, hardware profile, bitrate, encoder preset, image optimizer, and batch safety.

## Tool Map

<details>
<summary><strong>📋 Show 7 entries</strong></summary>

- **`conv <input> <output>`**<br>
  Use: Convert one file with automatic routing

- **`conv_info`**<br>
  Use: Show detected CPU, RAM, VideoToolbox, and defaults

- **`convall <src> <dst>`**<br>
  Use: Batch convert files in the current directory

- **`optimg <files...>`**<br>
  Use: Optimize PNG/JPG files in place

- **`optall`**<br>
  Use: Recursively optimize images in parallel

- **`smartencode <video> [low/med/high/lossless/auto]`**<br>
  Use: Encode video with named quality targets

- **`resize <image> <width>`**<br>
  Use: Resize through `vips` or `sips` depending on profile

</details>


Powered by `ffmpeg`, `vips`, `gifski`, `oxipng`, `pngquant`, `jpegoptim`, `cjxl`, `avifenc`, `cwebp`, `typst`, `pandoc`, `sox`, `sips`, `ImageMagick`, `ab-av1`, and `mediainfo`.

## Adaptive Profiles

`conv` initializes once per shell and exports conversion defaults for your machine.

<details>
<summary><strong>📋 Show 5 entries</strong></summary>

- **Ultra**<br>
  Hardware: M-series Ultra<br>
  H.264: `6000k`<br>
  H.265: `5000k`<br>
  AV1: `p6`<br>
  AVIF: `s4`<br>
  Parallel: 8<br>
  Resize: `vips`

- **Max**<br>
  Hardware: M-series Max<br>
  H.264: `4000k`<br>
  H.265: `3000k`<br>
  AV1: `p8`<br>
  AVIF: `s6`<br>
  Parallel: 6<br>
  Resize: `vips`

- **Pro**<br>
  Hardware: M-series Pro<br>
  H.264: `4000k`<br>
  H.265: `3000k`<br>
  AV1: `p8`<br>
  AVIF: `s6`<br>
  Parallel: 4<br>
  Resize: `vips`

- **Base**<br>
  Hardware: M1/M2/M3/M4<br>
  H.264: `3000k`<br>
  H.265: `2500k`<br>
  AV1: `p10`<br>
  AVIF: `s8`<br>
  Parallel: 2<br>
  Resize: `sips`

- **Intel**<br>
  Hardware: Intel Mac<br>
  H.264: `libx264 -crf 23`<br>
  H.265: `libx265 -crf 28`<br>
  AV1: `p12`<br>
  AVIF: `s9`<br>
  Parallel: 2<br>
  Resize: `sips`

</details>


Overrides:

```bash
CONV_H264_BR=8000k conv in.mov out.mp4
CONV_SOUND=0 conv in.wav out.mp3
CONV_KONAMI=1 conv in.mkv out.mp4
```

## Repository Layout

<details>
<summary><strong>📋 Show 6 entries</strong></summary>

- **[converter.sh](converter.sh)**<br>
  Purpose: Main shell implementation

- **[install.sh](install.sh)**<br>
  Purpose: Homebrew dependency install and shell wiring

- **[uninstall.sh](uninstall.sh)**<br>
  Purpose: Removes installed files and shell hook

- **[skill/universal-media-converter.md](skill/universal-media-converter.md)**<br>
  Purpose: Agent skill for Claude Code / GG Coder style workflows

- **[BENCHMARKS.md](BENCHMARKS.md)**<br>
  Purpose: Benchmark table and Pareto findings

- **[CONTRIBUTING.md](CONTRIBUTING.md)**<br>
  Purpose: Benchmark and PR guide

</details>


## Agent Integration

The installer copies the bundled skill into `~/.gg/skills/` when that directory exists. Agents can then call `conv` instead of inventing raw ffmpeg commands.

Skip skill installation:

```bash
CONV_SKILL_DIR=/dev/null /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Supersynergy/conv/main/install.sh)"
```

Agent instructions for this repository live in [AGENTS.md](AGENTS.md).

## Development

```bash
brew install shellcheck ffmpeg pandoc imagemagick webp libavif jpeg-xl
shellcheck converter.sh install.sh uninstall.sh
source ./converter.sh
conv_info
conv --help
```

The GitHub Actions suite runs ShellCheck on Ubuntu and smoke conversions on macOS.

## Limits

- macOS only.
- Benchmarks are from one M4 Max system; your files and hardware will differ.
- The CatBoost note is about ranking tested configs, not predicting universal outcomes.
- `pipe | bash` is convenient but always deserves inspection. Read [install.sh](install.sh) if you are unsure.

## Contributing

The highest-value contribution is a benchmark from a real Mac. M1 base, M2 Pro, M3 Ultra, Intel, and unusual media files are especially useful. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

[MIT](LICENSE) © Supersynergy.
