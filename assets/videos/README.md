# Video Slides for the Display

The display supports video slides. Drop **MP4** files in this folder and reference them in `promos.json`.

## Recommendations

- **Format:** MP4 with H.264 video + AAC audio (the universal browser-safe combo). WhatsApp videos already use this — they work as-is.
- **Aspect ratio:** Portrait 9:16 (matches the screen perfectly). Landscape videos will have letterboxing.
- **Length:** 10–30 seconds for promo loops. Up to 60s for educational content.
- **File size:** Keep under 25 MB each. GitHub limits files to 100 MB.
- **Resolution:** 720×1280 or 1080×1920 is more than enough for the Philips screen (768×1366).

## Two flavors of video slide

### A. **Looping background promo** (muted, like a moving poster)

Default behavior — autoplay, loop, no sound. Perfect for atmospheric or product showcase videos.

```json
{
  "id": "raya-haji-video",
  "type": "video",
  "video": "assets/videos/raya-haji-promo.mp4",
  "category": "Raya Haji",
  "validFrom": "2026-05-15",
  "validUntil": "2026-06-15",
  "priority": 9
}
```

### B. **Educational video with sound** (unmuted)

When `"unmute": true`, the background music ducks to 15% so the video's audio takes priority.

```json
{
  "id": "myopia-explained",
  "type": "video",
  "video": "assets/videos/myopia-care-explained.mp4",
  "unmute": true,
  "category": "Education",
  "evergreen": true
}
```

## Where to put video entries

Put video entries in any of these JSON sections — same place as image entries:

- `activePromos` — dated promos with `validFrom`/`validUntil`
- `campaigns` — multi-slide sequences
- `evergreen` — always-on
- `dayBonus` — day-specific extras

Just add `"type": "video"` and `"video": "assets/videos/filename.mp4"` instead of `"image": "..."`.

## Existing WhatsApp videos you have

Located at `C:\Users\User\Desktop\MDV PROMOTION\`:

- `WhatsApp Video 2026-05-20 at 19.02.27.mp4` (18s, 3.6 MB)
- `WhatsApp Video 2026-05-20 at 19.02.27 (1).mp4` (21s, 4.2 MB)
- `WhatsApp Video 2026-05-20 at 19.02.27 (2).mp4` (54s, 11.1 MB)
- `WhatsApp Video 2026-05-11 at 15.56.29.mp4` (30s, 3.3 MB)

When you decide which to use:
1. Copy chosen MP4(s) into `assets/videos/`
2. Rename to something descriptive (`raya-haji-2026.mp4`, etc.)
3. Add entry to `promos.json`
4. `git push`

## Autoplay note

The Edge kiosk launcher uses `--autoplay-policy=no-user-gesture-required` so videos start automatically. In normal Chrome/Edge browsing, the FIRST video might wait for a user click — that's normal browser behavior.
