# Cat Corner — Lottie animations

The customer display (`display.html`) looks in this folder for Lottie JSON files. Drop them in here and the cat will appear automatically.

## File naming

Use these exact filenames. The display rotates through whichever states it finds (skips missing ones):

| Filename | Cat state |
|---|---|
| `cat-walking.json` | Walking around |
| `cat-sleeping.json` | Curled up, sleeping |
| `cat-playing.json` | Playing / pouncing |
| `cat-eating.json` | Eating from a bowl |

If you only have 1 or 2 of these, no problem — the cat will still appear, just won't cycle states.

## Where to find tuxedo cat Lottie animations

1. Go to **https://lottiefiles.com/**
2. Search for: `tuxedo cat`, `black white cat`, `cat walking`, `cat sleeping`, etc.
3. Filter by **Free** (left sidebar) — make sure it's free for commercial use
4. Click an animation → **Download** → choose **Lottie JSON**
5. Rename to one of the filenames above
6. Drop it into this folder
7. `git push` — the display picks it up within the hour, or refresh Edge to see it now

## Recommended creators for tuxedo cats on LottieFiles

- Search "**cat by Marcin Treder**" — clean, modern
- "**cat by Filip Ariola**" — playful
- "**cat animation pack**" — often comes with multiple states in one bundle

## Want me to remove the cat entirely?

Just leave this folder empty (no .json files) — the cat corner stays hidden, no errors, no impact on the display.
