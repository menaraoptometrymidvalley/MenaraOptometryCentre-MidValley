# MDV Customer Display — Handbook

**For:** Menara Optometry Centre — Mid Valley (LG-017)
**Built:** 20–22 May 2026
**Owner:** KNM (Hazwan)

Two-screen kiosk that runs all-day customer-facing promo display on the Philips screen, while keeping the AOC work screen clean for staff. Plays background music. Has a cute cat. Auto-launches on boot. Switches off at 10 PM nightly for natural reset.

---

## 1. What it does

| Layer | What |
|---|---|
| **Hardware** | DISPLAY2 = AOC 1080×1920 portrait (Work Screen). DISPLAY1 = Philips 768×1366 portrait (Customer Display). |
| **Content** | `display.html` + `promos.json` on GitHub Pages. Day-aware. Auto-expiring. |
| **Kiosk** | Edge `--kiosk` mode, dedicated profile, full-screen on Customer Display. |
| **Audio** | MP3 background music ("The Mountain" lounge, Pixabay). Toggle on/off via desktop button. |
| **Locks** | Two AutoHotkey scripts confine cursor + windows to Work Screen. Toggleable. |
| **Auto-start** | All 3 (kiosk + 2 AHK locks) launch on PC boot. |
| **Daily reset** | PC switched off at 10 PM → fresh start every morning. |

---

## 2. Desktop buttons (4 buttons)

| Icon | Button | Action |
|---|---|---|
| ✅ Green tick | **Test MDV Display** | Open kiosk on Customer Display. Respects music state. |
| ❌ Red X | **Stop MDV Display** | Close kiosk silently. |
| 🔒 Padlock | **Toggle Lock** | Flip cursor + window locks ON ↔ OFF. |
| 🔊 Speaker | **Toggle Music** | Flip music ON ↔ OFF. Kiosk restarts briefly (~1 sec). |

All silent — no confirmation popups.

---

## 3. The build journey — what we decided and why

### Phase 1 — Hardware fix (TN panel viewing angle)

**Problem:** Philips PHLC0CD has a TN panel. From one side, the screen washed out.

**Tried:**
- ❌ Rotated orientation — bad viewing angle just moved
- ❌ Increased brightness — helped slightly
- ✅ **Portrait Flipped** + physically rotated monitor 180° on arm → good side faces entrance

**Long-term:** Replace with IPS monitor (~RM 400–700 for 22"). Not urgent.

### Phase 2 — Display content (HTML chosen)

**Considered:** PowerPoint loop, photo slideshow, HTML, digital signage tools.

**Chose: HTML on GitHub Pages** because:
- User already maintains `ops.html` on same repo
- Day-aware logic
- Auto-expiring promos
- `git push` = live update
- No subscription fees

**Trade-off accepted:** Longer initial setup, much easier maintenance afterwards.

### Phase 3 — Background music (longest journey)

**v1 — Random pentatonic synthesis** → sounded eerie / creepy
**v2 — Slow C–Am–F–G piano chords** → too quiet, too slow
**v3 — Upbeat lounge with walking bass + chord stabs** → felt right BUT
**v3.1 — Music started dying after a few minutes** → CPU exhausted from too many per-note oscillator allocations

**v4 — Rebuilt with persistent oscillators** + AudioContext lookahead scheduler → stable but sounded "ambient/spa", lost the rancak feel

**v5 — Hybrid: persistent foundation + rhythmic one-shot stabs** → upbeat feel back, no lag

**v6 — Master compressor + lowpass + high-shelf EQ** → still didn't sound right on user's cheap **Soniear speaker** (no-brand mono)

**v7 — Cheap-speaker tuning: sawtooth bass, boosted pads, softer stabs** → still sounded like one high-pitched tone on Soniear

**v8 (FINAL) — Gave up on procedural; downloaded real MP3** ("The Mountain — Lounge" from Pixabay)
- Loops seamlessly via HTML5 `audio.loop = true`
- Sounds great on cheap speakers (mixed/mastered properly)
- 3.6 MB file in `assets/audio/`
- Added Toggle Music desktop button + `?music=off` URL param

**Why procedural failed:** cheap mono speakers can't reproduce sub-bass (< 150 Hz). Real mastered music handles this; synthesized tones don't.

### Phase 4 — Video slides

- Built `type: "video"` support in `promos.json`
- Added **2 videos**: Hadiah Utama (Proton e.MAS5 reveal) + Thursday CL (Into Your Eyes)
- Both initially `unmute: true` → volume spike → reverted to muted (background music plays through instead)
- Use HTML5 `ended` event to play full natural duration (not cut at 8.5 sec)
- 35-second safety cap

### Phase 5 — Cursor and window locks

**Why:** Staff kept accidentally dragging cursor / windows onto Customer Display.

**Two AutoHotkey scripts:**
1. **CursorLock_to_Display1.ahk** — uses Win32 `ClipCursor` API (native, no polling, no jitter)
2. **WindowLock_to_Display1.ahk** — polls every 0.6 sec, snaps non-kiosk windows back

**Iterated:**
- ❌ Polling-based cursor confinement → jittery → switched to `ClipCursor`
- ❌ Window-lock identified kiosk by title → snapped kiosk Edge during page load (before title appeared) → switched to identifying ALL `msedge.exe` windows as kiosk
- ❌ Snap-if-any-pixel rule → broke maximize (window borders extend 8px past screen) → switched to "only if window CENTER on Customer Display" → too lenient (windows could bleed)
- ✅ **Final: snap if window intrudes >50px into Customer Display** — allows maximized borders, catches real drags

### Phase 6 — Desktop buttons (settled after iterations)

- 2 toggles → 1 combined → 1 mode switcher → **3 single-purpose buttons** → finally **4 buttons** (added Toggle Music)
- Each button = one job, no thinking required

### Phase 7 — Naming cleanup

- Windows quietly swapped `DISPLAY1`/`DISPLAY2` labels overnight
- Renamed all UI to **"Work Screen"** and **"Customer Display"**
- Code uses coordinates (1080,0), not Windows labels

---

## 4. Bug log

| Bug | Cause | Fix |
|---|---|---|
| Kiosk Edge ran but invisible | `shell.Run cmd, 0` = hidden window flag inherited | Changed to `shell.Run cmd, 1` |
| Music sounded creepy | Random pentatonic notes (no harmony) | Real chord progression C-Am-F-G |
| Music lagged + stopped | Per-note oscillator creation accumulated | Persistent oscillators + lookahead |
| Stop button WMI error popup | `SELECT ProcessId` returned partial object (can't call Terminate) | `SELECT *` + `On Error Resume Next` |
| Cat head separated from body | CSS transform overrode SVG position attribute | Restructured with stable rig group |
| Maximize un-maximized by window-lock | Any-pixel-bleed rule caught maximize border (8px overhang) | Changed to >50px intrusion threshold |
| Music monotone on cheap speaker | Synthesis can't reproduce on Soniear | Switched to real MP3 file |
| Ghost desktop icons | Explorer icon cache after script deletes shortcuts | Restart `explorer.exe` |
| PowerPoint deck looked sparse | First built with PowerShell COM, basic layouts | Rebuilt with pptxgenjs + visual QA |

---

## 5. Current architecture

### Files in the GitHub repo
```
menaraoptometrycentre-midvalley/
├── display.html          ← The kiosk page (vanilla JS, ~1700 lines)
├── promos.json           ← Content data (edit to add promos)
├── HANDBOOK.md           ← This document
├── ops.html              ← Internal ops dashboard
├── index.html            ← Public homepage
└── assets/
    ├── brand/menara-logo.jpg
    ├── promos/           ← 11 portrait JPGs
    ├── videos/           ← 2 MP4 videos
    ├── audio/            ← 1 MP3 (The Mountain - Lounge)
    └── cat/              ← Empty (cat is inline SVG)
```

### Files on the PC

**`C:\Users\User\MDV_Display\`**

| File | Purpose |
|---|---|
| `Test_MDV_Display.vbs` | Test button — launches kiosk, respects music state |
| `Stop_MDV_Display.vbs` | Stop button — closes kiosk |
| `Toggle_Display2_Access.vbs` | Toggle Lock button — flips cursor + window locks |
| `Toggle_Music.vbs` | Toggle Music button — flips music on/off via URL param |
| `MDV_Display_Launch.bat` | Auto-launch on boot (Startup folder shortcut) |
| `CursorLock_to_Display1.ahk` | Cursor lock AHK script |
| `WindowLock_to_Display1.ahk` | Window lock AHK script |
| `music.state` | Stores last music on/off choice |
| `edge-profile/` | Edge profile (auto-created) |
| `trigger/` | IPC folder for lock toggles |
| `Build_Handbook_PPT.ps1` | Generates PowerPoint via Word COM (older script) |
| `build_handbook_pptx.js` | Generates PowerPoint via pptxgenjs (current) |

**`Desktop\`**

| File | Icon |
|---|---|
| Test MDV Display.lnk | ✅ Green tick |
| Stop MDV Display.lnk | ❌ Red X |
| Toggle Lock.lnk | 🔒 Padlock |
| Toggle Music.lnk | 🔊 Speaker |
| MDV Display Handbook.md | Markdown handbook |
| MDV Display Handbook.docx | Word handbook |
| MDV Display Handbook.pptx | PowerPoint deck |

**Windows Startup folder:**

| Shortcut | Auto-launches |
|---|---|
| MDV_Display.lnk | Edge kiosk (8 sec delay after login) |
| MDV_CursorLock.lnk | Cursor lock |
| MDV_WindowLock.lnk | Window lock |

### Live URL

https://menaraoptometrymidvalley.github.io/menaraoptometrycentre-midvalley/display.html?kiosk=1

---

## 6. Daily operation

### Morning (PC boots)
1. 8 sec after login → kiosk auto-launches
2. Cursor + window locks engage
3. Today's promos + music start

### During the day
- Promos cycle ~10 slides every 100 sec
- Music plays continuously (single MP3 loop)
- Page auto-reloads every 1 hour for fresh content

### Show customer something on Customer Display
1. Click **Toggle Lock** → locks turn OFF
2. Drag window onto Customer Display
3. Click **Toggle Lock** again → back to kiosk mode

### Restart kiosk
1. Click **Stop MDV Display**
2. Click **Test MDV Display**

### Switch off music for radio time
1. Click **Toggle Music** → kiosk restarts briefly, music goes silent
2. Play radio
3. Click **Toggle Music** again → music returns

### Night (10 PM)
- Just power off PC normally
- Everything auto-launches when you power it back on tomorrow

---

## 7. How to add or edit a promo poster

### Step 1 — Prepare image
- **Format:** JPG or PNG
- **Aspect ratio:** Portrait 9:16 (e.g., 1080×1920)
- **Size:** Under 300 KB
- **Naming:** `lowercase-with-hyphens.jpg`

### Step 2 — Drop into repo
```
C:\Users\User\Documents\GitHub\menaraoptometrycentre-midvalley\assets\promos\
```

### Step 3 — Add to promos.json
Find `activePromos` array. Add:
```json
{
  "id": "merdeka-2026",
  "title": "Merdeka Family Promo",
  "subtitle": "20% off family eye exams",
  "image": "assets/promos/merdeka-2026.jpg",
  "category": "Merdeka",
  "validFrom": "2026-08-25",
  "validUntil": "2026-09-16",
  "priority": 9
}
```

### Step 4 — Publish
Open PowerShell, navigate, push:
```
cd C:\Users\User\Documents\GitHub\menaraoptometrycentre-midvalley
git add .
git commit -m "Add Merdeka 2026 promo"
git push
```

Live within 1 hour. Or Stop+Test to see immediately.

---

## 8. How to add a video slide

### Format requirements
- **MP4 (H.264 video + AAC audio)**
- **Portrait 9:16 aspect**
- **10–30 sec** ideal (45 sec max)
- **Under 25 MB**

### Drop into
```
assets/videos/
```

### JSON entry (use `type: "video"` instead of `image`)
```json
{
  "id": "varilux-promo",
  "type": "video",
  "video": "assets/videos/varilux.mp4",
  "category": "Multifocal",
  "validFrom": "2026-06-01",
  "validUntil": "2026-08-31",
  "priority": 8
}
```

Add `"unmute": true` if you want video audio (background music auto-ducks).

---

## 9. How to add background music tracks

Currently: 1 MP3 ("The Mountain" lounge) on loop.

### To add more tracks
1. Get free MP3 from **Pixabay Music** (https://pixabay.com/music/) — no attribution needed
2. Drop into `assets/audio/`
3. Add to `promos.json` → `audio.tracks` array:
   ```json
   "audio": {
     "volume": 0.45,
     "shuffle": true,
     "loop": true,
     "tracks": [
       { "title": "The Mountain", "src": "assets/audio/the-mountain-lounge.mp3" },
       { "title": "Cafe Vibes",   "src": "assets/audio/cafe-vibes.mp3" }
     ]
   }
   ```
With multiple tracks, the player rotates through them shuffled.

---

## 10. The 7DP weekly themes (auto day-aware)

| Day | Theme | Pillar |
|---|---|---|
| Mon | Cyber Monday — Eyes aging faster? | DES (Digital Eye Strain) |
| Tue | MyopiaCARE Tuesday | MyopiaCARE (kids 4-17) |
| Wed | Wow Wednesday — 20% off RM500+ | General loyalty |
| Thu | Look Fresh Thursday + CL Aftercare | Contact Lens |
| Fri | Blessed Friday — Show up yourself | Brand love |
| Sat | MyopiaCARE Saturday (same as Tue) | MyopiaCARE |
| Sun | Sun-Day Shield — Bring your parents | UV / Cataract |

Today's 7DP is auto-shown first slide of the deck. No editing needed.

To change content, edit `weekly7DP` array in `promos.json`.

---

## 11. Hotkeys (alternative to desktop buttons)

| Hotkey | Action |
|---|---|
| `Win + Alt + →` | Unlock cursor only |
| `Win + Alt + ←` | Lock cursor only |
| `Win + Alt + L` | Toggle cursor lock |
| `Win + Alt + W` | Toggle window lock |

---

## 12. Troubleshooting

### Kiosk not appearing on Customer Display
1. Check Customer Display monitor powered + cable connected
2. Click **Test MDV Display** (might be closed)
3. If stuck, restart PC

### Music stopped playing
- HTML5 `audio.loop = true` should handle continuous playback
- Worst case: **Stop → Test** to relaunch kiosk

### Cursor / windows escaping to Customer Display
- Click **Toggle Lock** (might have been left unlocked)
- Check tray icons for AHK lock states
- If AHK scripts not running → double-click them in `C:\Users\User\MDV_Display\`

### A promo isn't showing on the right date
- Check `validFrom` / `validUntil` in `promos.json`
- Display uses local Malaysia time (MYT)

### Ghost / old desktop icons
- Right-click desktop → Refresh (F5)
- Or restart `explorer.exe`

### Maximizing Chrome / Claude unmaximizes
- Should be fixed (window-lock uses >50px intrusion threshold)
- If recurs, the lock rules need tuning — tell me

### Music sounds different than I remember
- Music was changed multiple times during build (3 procedural attempts → final MP3)
- Current: "The Mountain (Lounge)" from Pixabay, looping forever

---

## 13. The 30 Tahun anniversary slide

Always the LAST slide of every cycle — hardcoded in `display.html`. Won't move regardless of new promos.

To change: swap the image at `assets/promos/30-tahun-brand.jpg`.

---

## 14. Performance notes

- Page reloads every **1 hour** (clears DOM/JS memory)
- PC switches off **10 PM nightly** (full reset)
- Combined: multi-day stability is fine

If anything ever feels laggy or stuck, just **Stop → Test** to relaunch.

---

## 15. Update log (chronological)

### 20 May 2026 — Day 1 setup
- DISPLAY2 rotation fix (Portrait Flipped + 180° physical rotate)
- Cloned repo, built initial `display.html` + `promos.json`
- Added 11 portrait promo images
- Built kiosk launcher (.bat) + auto-start shortcut
- Cursor lock AHK script
- Added Beli & Menang (4 slides), Essilor Varilux, HOYA SUPEREADER, Sudoku template, CL Aftercare, Menara logo header
- Added chibi tuxedo cat (inline SVG, CSS animated)
- Built Sudoku as HTML template with live QR code + voucher tiers

### 22 May 2026 — Day 2 polish + iterations
- **Date filter fixes**: `todayISO()` switched to local MYT (was UTC, would hide expired promos 8h late); added filter to `dayBonus`
- **Music v1**: random pentatonic — eerie
- **Music v2**: slow piano chords — too quiet
- **Music v3**: upbeat lounge — felt right BUT laggy
- **Music v4**: persistent oscillators — fixed lag, lost rancak feel
- **Music v5**: hybrid persistent + rhythmic stabs — upbeat back
- **Music v6**: added compressor + EQ for cheap speakers
- **Music v7**: tuned for Soniear — still sounded monotone
- **Music v8 (FINAL)**: gave up on procedural, downloaded "The Mountain" Pixabay MP3
- Added `?music=off` URL parameter support
- Added Toggle Music desktop button (4th button)
- Video slide architecture: added support, integrated 2 WhatsApp videos
- Videos initially unmuted (volume spike) → reverted to muted
- Fixed video play-to-end via `ended` event (was cut at 8.5s)
- **Window-lock script** (new): snaps non-kiosk windows back from Customer Display
- Window-lock iterations: title match → command-line match → trust all Edge → intrusion-based (50px tolerance)
- **Desktop buttons consolidation**: 2 toggles → 1 mode switcher → 3 single-purpose → 4 (added Toggle Music)
- Removed popup confirmations from all VBS scripts
- Cleaned up unused scripts (4 old VBS deleted)
- Naming cleanup: Customer Display / Work Screen (no more DISPLAY1/2 numbers)
- Slide duration 8.5s → 10.5s
- **Bug**: VBS hidden window (`shell.Run cmd, 0`) → kiosk launched but invisible → fixed to `1`
- **Bug**: WMI SELECT ProcessId returning partial object → fixed with `SELECT *` + error handling
- **Built MD handbook** (this document)
- **Built PowerPoint deck**: first via PowerShell COM (sparse layouts) → rebuilt via Node.js + pptxgenjs (~25 polished slides with visual QA)
- **Built Word handbook** (.docx)

---

*Last updated: 22 May 2026*
