# SpotiRec

A desktop music app that recognizes songs from a short audio clip, Shazam-style, and lets you search and stream music from YouTube in a built-in player.

Song recognition is implemented from scratch with audio fingerprinting: no third-party recognition API is used.

> Team project, originally created by [Ilia Arkania](https://github.com/Naviamold1). This fork includes my UI and visual changes.

## Features

- **Song recognition**: record a few seconds from the microphone (or pick a WAV file) and get the best matches with a confidence score
- **Music player**: search YouTube, stream tracks, play/pause, seek, next/previous, shuffle, repeat, volume and playback speed
- **Phone recording over LAN**: a small FastAPI web page lets you record on your phone and send the audio to the desktop app over a WebSocket
- Dark-themed PyQt6 interface with an animated visualizer

## How recognition works

1. **Spectrogram**: the audio is converted to mono and transformed with a short-time Fourier transform (`scipy.signal.stft`).
2. **Peak finding**: local maxima above an amplitude threshold are picked from the spectrogram, giving a sparse "constellation map" that survives noise.
3. **Combinatorial hashing**: each peak is paired with nearby peaks; the two frequencies plus the time gap form a hash.
4. **Indexing**: hashes and their time offsets are stored in SQLite with an index on the hash column.
5. **Matching**: hashes from the recording are looked up, and matches are grouped by song and time offset. The song with the most hashes aligned at the same offset wins.

## Tech stack

Python 3.13 · PyQt6 · NumPy / SciPy / librosa · SQLite · FastAPI + WebSockets · yt-dlp · uv

## Project structure

```
backend/
  main.py         fingerprinting: spectrogram, peak finding, hashing
  db.py           SQLite storage and hash lookup
  recognizer.py   matching and ranking of candidate songs
  web.py          FastAPI app for recording from a phone
frontend/
  app.py          main window
  shazam.py       recognition screen
  player.py       YouTube search and player
  workers.py      background threads for recognition, search and streaming
scripts/
  add_songs.py    download songs and add their fingerprints to the database
  serve_web.py    serve the web recorder on the local network
tests/            fingerprinting tests
```

## Getting started

Requires Python 3.13+, [uv](https://docs.astral.sh/uv/) and FFmpeg.

```bash
git clone https://github.com/data2222222/shazam.git
cd shazam
uv sync
```

Put the songs you want to recognize as WAV files in an `assets/` folder (named `<id> - <title>.wav`), then build the fingerprint database and start the app:

```bash
uv run python -m scripts.add_songs
uv run python main.py
```

`scripts/add_songs.py` also has a `download_songs()` helper that uses yt-dlp and FFmpeg to fetch a playlist as mono WAV files.

Run the tests with:

```bash
uv run pytest
```
