# MDV Customer Display — Handbook

**For:** Menara Optometry Centre — Mid Valley (LG-017)
**Built:** 20–22 May 2026
**Owner:** KNM (Hazwan)

Two-screen kiosk system that runs all-day customer-facing promo display on the Philips screen, while keeping the AOC work screen clean for staff.

---

## 1. What it does

> A second-display "digital signage" that shows today's promos, plays calming background music, has a cute animated cat in the corner, and surfaces a Sudoku QR for waiting customers — all running automatically when the PC boots, no daily intervention required.

| Layer | What |
|---|---|
| **Hardware** | DISPLAY2 = AOC 1080×1920 portrait (Work Screen). DISPLAY1 = Philips 768×1366 portrait (Customer Display). |
| **Content** | `display.html` + `promos.json` on GitHub Pages. Day-aware. Auto-expiring. |
| **Kiosk** | Edge `--kiosk` mode, dedicated profile, full-screen on Customer Display. |
| **Locks** | Two AutoHotkey scripts confine cursor + windows to Work Screen. |
| **Audio** | Procedural ambient music (Web Audio API) — no copyright issues. |
| **Auto-start** | All 3 (kiosk + 2 AHK locks) launch on PC boot. |
| **Daily reset** | PC switched off at 10 PM → fresh start every morning. |

---

## 2. The build journey — what we decided and why

### Phase 1 — Hardware fix (DISPLAY2 visibility)

**Problem:** Philips PHLC0CD has a TN panel. From one side, the screen washed out / looked black.

**Tried:**
- ❌ Rotated screen orientation (didn't fix — TN's bad viewing angle just moves to a different side)
- ❌ Increased brightness (helped a bit, didn't solve)
- ✅ **Portrait Flipped** orientation + physically rotated monitor 180° on its arm → good viewing side now faces the entrance

**Long-term recommendation:** Replace with an IPS panel (~RM 400–700 for 22"). Will give ~178° viewing angle from all sides. Not urgent — current setup works.

### Phase 2 — Display content system

**Considered:** PowerPoint loop, Windows photo slideshow, HTML, free digital signage tools.

**Chose:** **HTML on GitHub Pages**, because:
- User already maintains `ops.html` on same repo — natural extension
- Day-aware logic (auto-show today's 7DP)
- Auto-expiring promos (Mother's Day disappears June 1 without manual edit)
- Updates push instantly via `git push`
- No subscription fees
- Lower long-term maintenance than PowerPoint (no monthly slide swap)

**Tradeoff accepted:** Initial setup takes longer than dropping PNGs in PowerPoint. Maintenance afterwards is much easier.

### Phase 3 — Content schema (promos.json)

**Designed:** A typed structure that supports:
- `weekly7DP` (Mon–Sun themed promos)
- `dayBonus` (extra content per weekday — e.g., Thursday CL Aftercare)
- `campaigns` (multi-slide sequences, e.g., Beli & Menang)
- `activePromos` (dated single-card promos with validity windows)
- `evergreen` (always-on — 30 Tahun anniversary, Sudoku)
- `audio` (background music config)

**Why this shape:** Lets the display.html JS curate ~10 slides per day based on what's relevant TODAY, not show all 17 every time.

### Phase 4 — Video slides

**Considered:** Just images, or animated GIFs, or videos.

**Built:** **MP4 video support** with `"type": "video"` in promos.json. Web-native `<video>` tag with autoplay+muted+loop.

**Iterated on:**
- ❌ Added 3 videos (Beli & Menang intro, Hadiah Utama, Thursday CL) → intro was redundant
- ✅ Settled on 2 videos (Hadiah Utama, Thursday CL)
- ❌ Videos cut off at 8.5 sec default → switched to `ended` event for full natural play
- ❌ Videos initially `unmute: true` (spike in volume vs music) → reverted to muted, background music plays through

### Phase 5 — Background music

**Iterated three times:**
- ❌ **v1: Random pentatonic notes** → sounded eerie / scary
- ❌ **v2: Slow piano chord progression (C-Am-F-G, 7-sec bars)** → too quiet + too slow
- ✅ **v3: Upbeat lounge (4-sec bars, walking bass, chord stabs, melody)** → felt right

**Then a performance issue:**
- ❌ Music stopped after a few slides + lagged the whole display
- ✅ **Rebuilt with persistent oscillators** (no per-note node creation) + AudioContext lookahead scheduler

**Then resilience:**
- ✅ Added **watchdog** (auto-restart if music stalls) + **15-min preventive refresh** (silent fade)

**Result:** Music plays continuously for hours without dropouts.

### Phase 6 — Cursor and window locks

**Why:** Staff kept accidentally dragging cursor/windows onto the customer display.

**Two AHK scripts:**
1. `CursorLock_to_Display1.ahk` — uses Win32 `ClipCursor` API (not polling, no jitter) to confine cursor to Work Screen
2. `WindowLock_to_Display1.ahk` — polls every 0.6 sec, snaps any non-kiosk window back to Work Screen

**Iterated:**
- ❌ Polling-based cursor confinement → jittery / circular cursor → switched to `ClipCursor`
- ❌ Window-lock identified kiosk Edge by title → snapped kiosk Edge during page load (before title appeared) → switched to identification by command-line `--user-data-dir` path
- ✅ Final: trust ALL `msedge.exe` windows on Customer Display (simpler, more robust)

### Phase 7 — Desktop buttons

**Iterated several times** (apologies for the back-and-forth):
- 2 separate toggles → 1 combined → 1 mode switcher → settled on **3 single-purpose buttons**

**Final:**
- **Test MDV Display** (green tick) — start the kiosk
- **Stop MDV Display** (red X) — stop the kiosk
- **Toggle Lock** (padlock) — flip cursor + window locks

**Why 3 not 1:** Each button = one job, no thinking required. Combined toggles were confusing.

### Phase 8 — Bugs squashed

| Bug | Cause | Fix |
|---|---|---|
| Kiosk Edge ran but invisible | VBS used `shell.Run cmd, 0, False` (hidden window flag) — Edge inherited hidden state | Changed to `shell.Run cmd, 1, False` (normal window) |
| Music droned eerily | Random pentatonic notes | Chord progression (C-Am-F-G) with bass + melody |
| Music lagged + stopped | Per-note oscillator creation accumulated | Persistent oscillators + lookahead scheduler |
| Stop button popup error | WMI SELECT ProcessId returned partial object | `SELECT *` + `On Error Resume Next` |
| Ghost desktop icons | Cached Explorer icons after script deletes | Restart `explorer.exe` |
| Windows swapped DISPLAY1/DISPLAY2 labels overnight | Windows quirk after reboot | Use coordinates (1080,0) not labels |

---

## 3. Current architecture (final state)

### Files in the GitHub repo
```
menaraoptometrycentre-midvalley/
├── display.html          ← The kiosk page (one file, vanilla JS)
├── promos.json           ← Content data (edit this to add promos)
├── ops.html              ← Internal ops dashboard (unchanged)
├── index.html            ← Public homepage (unchanged)
├── HANDBOOK.md           ← This document
└── assets/
    ├── brand/
    │   └── menara-logo.jpg
    ├── promos/           ← Portrait promo JPGs
    │   ├── 7dp-mon-des.jpg
    │   ├── 7dp-tue-myopia.jpg
    │   ├── ... (10 more)
    ├── videos/           ← MP4 promo videos
    │   ├── beli-menang-02-hadiah-utama.mp4
    │   └── 7dp-thu-cl.mp4
    ├── audio/            ← Optional MP3 background tracks (empty by default — uses procedural)
    │   └── README.md
    └── cat/              ← Empty (cat is inline SVG in display.html)
```

### Files on the PC
```
C:\Users\User\MDV_Display\
├── Test_MDV_Display.vbs           ← Launches kiosk
├── Stop_MDV_Display.vbs           ← Closes kiosk
├── Toggle_Display2_Access.vbs     ← Toggles cursor + window locks
├── MDV_Display_Launch.bat         ← Auto-launch on boot (Startup folder shortcut)
├── CursorLock_to_Display1.ahk     ← Cursor lock script
├── WindowLock_to_Display1.ahk     ← Window lock script
├── edge-profile/                   ← Edge profile (auto-created)
└── trigger/                        ← IPC for AHK toggles

Desktop\
├── Test MDV Display.lnk           ← Green tick
├── Stop MDV Display.lnk           ← Red X
└── Toggle Lock.lnk                ← Padlock

Windows Startup\
├── MDV_Display.lnk                ← Auto-launches kiosk (8 sec delay)
├── MDV_CursorLock.lnk             ← Auto-launches cursor lock
└── MDV_WindowLock.lnk             ← Auto-launches window lock
```

### Live URL
```
https://menaraoptometrymidvalley.github.io/menaraoptometrycentre-midvalley/display.html?kiosk=1
```

---

## 4. Daily operation

### Morning (PC boots)
1. 8 sec after login → Kiosk Edge launches fullscreen on Customer Display
2. Cursor + Window locks engage automatically
3. Today's promo cycle starts playing with music
4. Nothing to do

### During the day
- Promos cycle ~10–12 slides every 100 sec
- Music plays continuously (refreshes every 15 min silently)
- Page auto-reloads every 1 hour to pick up any new promos

### To temporarily use the Customer Display (e.g., show customer something)
1. Click **Toggle Lock** on desktop → cursor + window unlock
2. Drag your window onto Customer Display
3. When done, click **Toggle Lock** again → back to kiosk mode

### To restart the kiosk during the day
1. Click **Stop MDV Display** → kiosk closes
2. Click **Test MDV Display** → kiosk reopens

### Night (10 PM shutdown)
- Just power off the PC normally
- Everything auto-launches when you turn it back on tomorrow

---

## 5. How to add or edit a promo poster

### Step 1 — Prepare the image
- **Format:** JPG or PNG
- **Aspect ratio:** Portrait 9:16 (e.g., 1080×1920) — matches the screen perfectly
- **Size:** Optimize to under 300 KB (use any online image compressor or keep JPG quality at 85%)
- **Naming:** descriptive, lowercase, hyphens-not-spaces (e.g., `merdeka-2026.jpg`)

### Step 2 — Drop the file into the repo
Open File Explorer:
```
C:\Users\User\Documents\GitHub\menaraoptometrycentre-midvalley\assets\promos\
```
Drag-and-drop your new JPG here.

### Step 3 — Add it to `promos.json`
Open `C:\Users\User\Documents\GitHub\menaraoptometrycentre-midvalley\promos.json` in Notepad or VS Code.

Find the `activePromos` array. Add a new entry. Example:

```json
{
  "id": "merdeka-2026",
  "title": "Merdeka Family Promo",
  "subtitle": "20% off family eye exams — 25 Aug to 16 Sep",
  "image": "assets/promos/merdeka-2026.jpg",
  "category": "Merdeka",
  "partner": "Menara",
  "validFrom": "2026-08-25",
  "validUntil": "2026-09-16",
  "priority": 9
}
```

**Required fields:** `id`, `image`
**Recommended:** `title`, `validFrom`, `validUntil`, `priority`
**Optional:** `subtitle`, `category`, `partner`

### Step 4 — Publish
Open PowerShell, navigate to the repo, push:
```powershell
cd C:\Users\User\Documents\GitHub\menaraoptometrycentre-midvalley
git add .
git commit -m "Add Merdeka 2026 promo"
git push
```

Within ~1 hour the kiosk picks it up automatically (or **Stop → Test** to see immediately).

### To EDIT an existing promo
- Just edit the entry in `promos.json` and `git push`
- Or replace the JPG file (keep the same filename)

### To REMOVE a promo permanently
- Delete the entry from `promos.json` and `git push`
- (Don't need to delete the JPG file — it just won't be referenced)

---

## 6. How to add a video slide

### Step 1 — Prepare the video
- **Format:** MP4 with H.264 video + AAC audio (WhatsApp videos already use this)
- **Aspect ratio:** Portrait 9:16
- **Length:** 10–30 sec ideal (45 max)
- **Size:** Under 25 MB
- **Naming:** lowercase-with-hyphens

### Step 2 — Drop into the repo
```
C:\Users\User\Documents\GitHub\menaraoptometrycentre-midvalley\assets\videos\
```

### Step 3 — Add to `promos.json`
In `activePromos` or `campaigns.sequence`:
```json
{
  "id": "varilux-promo-video",
  "type": "video",
  "video": "assets/videos/varilux-promo-video.mp4",
  "category": "Multifocal",
  "validFrom": "2026-06-01",
  "validUntil": "2026-08-31",
  "priority": 8
}
```

**Key difference from image entries:**
- `"type": "video"` instead of relying on `image`
- `"video": "path"` instead of `"image": "path"`
- Optional `"unmute": true` if you want the video's audio to play (background music will auto-duck)

### Step 4 — Push
Same git workflow as posters.

---

## 7. How to add background music tracks

Default: procedural lounge music (auto-generated, no copyright concerns).

To use real MP3 tracks instead:

### Step 1 — Get tracks
Free sources (commercial OK, no attribution required):
- **Pixabay Music**: https://pixabay.com/music/
- **YouTube Audio Library**: https://studio.youtube.com → Audio Library

Pick instrumental tracks (no lyrics), calm to mid-tempo.

### Step 2 — Drop into repo
```
C:\Users\User\Documents\GitHub\menaraoptometrycentre-midvalley\assets\audio\
```

### Step 3 — Update `promos.json`
```json
"audio": {
  "volume": 0.4,
  "shuffle": true,
  "tracks": [
    { "title": "Ambient Piano", "src": "assets/audio/track-01.mp3" },
    { "title": "Lo-fi Café",   "src": "assets/audio/track-02.mp3" }
  ]
}
```

### Step 4 — Push
Once `tracks` array has entries, the kiosk uses MP3 mode instead of procedural.

---

## 8. The 7DP weekly themes (auto day-aware)

Today's 7DP shows automatically — no editing needed. Just know what each day shows:

| Day | Theme | Pillar |
|---|---|---|
| Mon | "Cyber Monday" — Eyes aging faster? | DES (Digital Eye Strain) |
| Tue | MyopiaCARE Tuesday | MyopiaCARE (kids 4–17) |
| Wed | "Wow Wednesday" — 20% off RM500+ | General loyalty |
| Thu | "Look Fresh Thursday" + CL Aftercare bonus | Contact Lens |
| Fri | "Blessed Friday" — Show up yourself | Brand love |
| Sat | MyopiaCARE Saturday (same as Tue) | MyopiaCARE |
| Sun | "Sun-Day Shield" — Bring your parents | UV / Cataract |

To change the 7DP content, edit `weekly7DP` array in `promos.json`.

---

## 9. Desktop buttons reference

| Icon | Button | Action | When to use |
|---|---|---|---|
| ✅ Green tick | **Test MDV Display** | Opens kiosk on Customer Display | Kiosk closed and you need to restart |
| ❌ Red X | **Stop MDV Display** | Closes kiosk | Maintenance / shop closing early |
| 🔒 Padlock | **Toggle Lock** | Flips cursor + window locks ON ↔ OFF | When you need to show customer something on Customer Display |

All silent (no confirmation popups).

---

## 10. Hotkeys (alternative to desktop buttons)

| Hotkey | What |
|---|---|
| `Win + Alt + →` | Unlock cursor only |
| `Win + Alt + ←` | Lock cursor only |
| `Win + Alt + L` | Toggle cursor lock |
| `Win + Alt + W` | Toggle window lock |

---

## 11. Troubleshooting

### Kiosk not appearing on Customer Display
1. Check Customer Display monitor is powered on + cable connected
2. Click **Test MDV Display** (might be stopped)
3. If kiosk Edge process is running but window invisible → restart PC

### Music stopped playing
- The 15-min watchdog should auto-recover within 10 sec
- If still silent for >1 min → page is auto-reloading at hourly mark, should restart
- Worst case: **Stop → Test** to relaunch

### Cursor or windows escaping to Customer Display
- Click **Toggle Lock** (might have been left unlocked)
- Check tray icons for the AHK lock states (padlock + window icons)
- If AHK scripts not running → double-click them in `C:\Users\User\MDV_Display\`

### A promo isn't showing on the right date
- Check `validFrom` / `validUntil` in promos.json
- Check today's actual date matches the validity window
- The display uses local Malaysia time (MYT)

### Ghost/old desktop icons
- Right-click desktop → **Refresh** (F5)
- If still stuck → restart explorer.exe (Task Manager → Restart Windows Explorer)

---

## 12. The 30 Tahun anniversary slide

Always the LAST slide of every cycle — hardcoded in `display.html`. Won't move regardless of new promos.

To change this slide, swap the image at:
```
assets/promos/30-tahun-brand.jpg
```

---

## 13. Performance notes

- Page reloads every **1 hour** (clears DOM/JS memory)
- Music refreshes every **15 min** (silent fade, fresh oscillators)
- PC switches off **10 PM nightly** (full reset for next morning)
- These three layers together make multi-day stability a non-issue

If anything ever feels laggy or stuck, just **Stop → Test** to relaunch the kiosk. Or wait for the next hourly auto-reload.

---

*Last updated: 22 May 2026*
