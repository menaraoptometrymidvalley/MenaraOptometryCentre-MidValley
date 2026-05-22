# Background Music for the Display

Drop **MP3 files** in this folder, then list them in `promos.json` under the `audio.tracks` array. The display loops them with shuffle by default.

## Recommended free / royalty-free music sources

All free for commercial use. Search for instrumental, no lyrics (less distracting in a retail setting):

| Source | Notes |
|---|---|
| **Pixabay Music** — https://pixabay.com/music/ | Free for commercial use, **no attribution required**. Best starting point. |
| **YouTube Audio Library** — https://studio.youtube.com (sign in → Audio Library) | Massive catalog, free, mostly no attribution. |
| **Free Music Archive** — https://freemusicarchive.org/ | Mix of Creative Commons; check each track's license. |
| **Bensound** — https://www.bensound.com/ | Quality tracks, free with attribution. |

## What kind of music for an optometry shop?

Look for:
- **Instrumental** (no lyrics — they fight with promo text)
- **Calm / ambient / lo-fi / soft jazz / acoustic**
- **Slow to mid tempo** (around 60–100 BPM)

Avoid:
- Pop/rock with vocals
- Loud/aggressive tracks
- Anything sad or melancholic

## How to add tracks

1. Download MP3 files (or convert other formats to MP3)
2. Rename clearly, e.g. `01-pixabay-ambient-piano.mp3`, `02-bensound-acoustic.mp3`
3. Drop into this folder
4. Edit `promos.json` → `audio.tracks` array:

```json
"audio": {
  "volume": 0.25,
  "shuffle": true,
  "tracks": [
    { "title": "Ambient Piano",   "src": "assets/audio/01-pixabay-ambient-piano.mp3" },
    { "title": "Acoustic Café",   "src": "assets/audio/02-bensound-acoustic.mp3" },
    { "title": "Lo-fi Study",     "src": "assets/audio/03-pixabay-lofi.mp3" }
  ]
}
```

5. `git push` — the display picks them up within the hour, or refresh Edge to play immediately.

## Volume

`volume: 0.25` = 25% of max. Good for retail (audible but not intrusive). Range: 0 (silent) to 1 (full blast). Suggest staying ≤ 0.4.

## What happens if a video slide has audio

When a video has `"unmute": true`, the background music auto-ducks to 15% so the video's voice/sound takes priority. Resumes full volume after the video.

## Empty folder?

If `audio.tracks` is empty, the display runs silent. No errors, no audio indicator shown.

## Autoplay note

The Edge kiosk launcher (`MDV_Display_Launch.bat`) already includes `--autoplay-policy=no-user-gesture-required` so the music starts on its own without anyone clicking. **If you open the display URL in regular Chrome/Edge for testing**, autoplay may be blocked — that's normal browser behavior, only the kiosk launcher bypasses it.
