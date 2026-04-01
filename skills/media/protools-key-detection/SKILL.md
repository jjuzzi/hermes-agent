---
name: protools-key-detection
description: Detect the musical key of audio files and generate Pro Tools-compatible markers, CSV reports, or JSON output. Uses the Krumhansl-Schmuckler algorithm with chroma CQT analysis. Includes Camelot wheel codes for harmonic mixing.
version: 1.0.0
author: community
license: MIT
platforms: [windows, linux, macos]
metadata:
  hermes:
    tags: [Audio, Music, Key Detection, Pro Tools, DAW, Windows, Mixing]
    related_skills: [songsee]
prerequisites:
  python_packages: [librosa, numpy]
---

# Pro Tools Key Detection

Detect the musical key of audio files (WAV, MP3, FLAC, AIFF) using chroma-based analysis with the Krumhansl-Schmuckler key-finding algorithm. Designed for integration with Avid Pro Tools on Windows but works cross-platform.

## When to Use

- User wants to detect the key/scale of an audio file or session
- User needs to tag tracks with key information for harmonic mixing
- User wants Pro Tools markers annotated with key data
- User needs batch key detection across session files
- User asks about Camelot wheel codes for DJ-style mixing workflows

## Prerequisites

Install Python dependencies:
```bash
pip install librosa numpy soundfile
```

On Windows, if loading MP3 files, also install FFmpeg and ensure it's on PATH:
```powershell
winget install Gyan.FFmpeg
```

## Quick Start

```bash
# Detect key of a single file
python key_detector.py track.wav

# Batch analyze all WAVs in a Pro Tools session Audio Files folder
python key_detector.py --batch "C:\Users\You\Documents\Pro Tools\Sessions\MySong\Audio Files\*.wav"

# Export as CSV for spreadsheets
python key_detector.py track.wav --format csv --output keys.csv

# Generate Pro Tools marker import file
python key_detector.py track.wav --format marker --output markers.txt

# JSON output for scripting
python key_detector.py track.wav --format json
```

## Output Formats

| Format   | Flag              | Description                                      |
|----------|-------------------|--------------------------------------------------|
| Text     | `--format text`   | Human-readable summary (default)                 |
| CSV      | `--format csv`    | Spreadsheet-compatible with key, Camelot, alts   |
| JSON     | `--format json`   | Full analysis data for scripting/automation       |
| Marker   | `--format marker` | Tab-delimited Pro Tools marker import format      |

### Example Text Output

```
  File:       vocals_chorus.wav
  Key:        Eb major
  Camelot:    5B
  Confidence: 0.8723
  Alternates: C minor, Bb major, G minor
```

## Pro Tools Integration on Windows

### Method 1: Analyze Session Audio Files Directly

Pro Tools stores audio in the session's `Audio Files` folder. Point the detector at it:

```powershell
cd "C:\path\to\key_detector.py"
python key_detector.py --batch "C:\Users\You\Documents\Pro Tools\Sessions\MySong\Audio Files\*.wav" --format csv -o keys.csv
```

### Method 2: Import Key Markers into Pro Tools

1. Generate a marker file:
   ```powershell
   python key_detector.py track.wav --format marker -o session_markers.txt
   ```
2. In Pro Tools, use **File > Import > Session Data** to bring in marker/memory location information, or manually create memory locations based on the output.

### Method 3: Windows Batch Script for One-Click Analysis

Create `detect_keys.bat` next to your Pro Tools session:

```batch
@echo off
echo Detecting keys for session audio files...
python "C:\Tools\key_detector.py" --batch "%~dp0Audio Files\*.wav" --format text
pause
```

Double-click it to analyze all audio in the session.

### Method 4: PowerShell Automation

```powershell
# Analyze and pipe to clipboard
python key_detector.py track.wav --format text | Set-Clipboard

# Watch a folder for new files and auto-detect
$watcher = [System.IO.FileSystemWatcher]::new("C:\Sessions\Audio Files", "*.wav")
$watcher.EnableRaisingEvents = $true
Register-ObjectEvent $watcher Created -Action {
    python C:\Tools\key_detector.py $Event.SourceEventArgs.FullPath
}
```

## How It Works

1. **Audio Loading** - Loads the file and converts to mono at 22050 Hz
2. **Harmonic Isolation** - Separates harmonic content from percussion using HPSS (improves accuracy on full mixes)
3. **Chroma Analysis** - Computes a 12-bin chroma vector using Constant-Q Transform
4. **Key Correlation** - Correlates the chroma vector against all 24 major/minor Krumhansl-Schmuckler key profiles
5. **Ranking** - Returns the best match with confidence score, plus top 3 alternates and Camelot wheel code

## CLI Reference

| Flag              | Default | Description                                    |
|-------------------|---------|------------------------------------------------|
| `files`           | —       | One or more audio file paths                   |
| `--batch`         | —       | Glob pattern for batch processing              |
| `--format`        | `text`  | Output format: text, csv, json, marker         |
| `--output` / `-o` | stdout  | Write results to file                          |
| `--sr`            | 22050   | Sample rate for analysis                       |
| `--no-harmonic`   | false   | Skip harmonic isolation (faster, less accurate)|

## Tips

- **Confidence > 0.75** generally indicates a reliable detection
- **Confidence 0.5-0.75** suggests the track may be ambiguous (modal, atonal, or key changes)
- Check the **alternates** — the relative minor/major is often the second result
- Use `--no-harmonic` for purely melodic/harmonic content (no drums) for a speed boost
- The **Camelot code** is useful for harmonic mixing: adjacent numbers on the wheel are compatible keys
