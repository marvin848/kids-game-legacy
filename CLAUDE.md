# kids-game-legacy

A self-contained kids' learning game ("Avalyn's Learning Games"), deployed as a
static site on Netlify and played mostly on iPads.

## What's here

| File | Role |
|------|------|
| `index.html` | **The entire app.** All markup, CSS, JavaScript, and artwork. |
| `sw.js` | Service worker. Offline cache, network-first. |
| `manifest.webmanifest` | PWA metadata so it installs to the iPad home screen. |

There is no build step, no dependency install, and no package manager. There
are no external fonts, scripts, or image files -- the shapes artwork is inline
SVG and the C-lesson artwork is base64 WebP embedded in `index.html` -- so the
game works with no internet connection once loaded. To preview locally, open
`index.html` in a browser or serve the folder with any static file server.

Two games live inside `index.html`: **Shapes & Sizes** (6 levels) and
**Letter Sounds: C** (4 lessons). They're defined in the `GAMES` array.

### The two artwork styles are deliberate

**Shapes & Sizes uses flat inline SVG and must stay that way.** Its questions
ask "which one is the same?", which only works if two pictures are identical
except for the single attribute being tested. Generated or hand-drawn art
varies between pictures, so a child who spots a real difference gets marked
wrong. Keep drawing these from the `P` primitives.

**Letter Sounds: C uses colored-pencil pictures** matching Avalyn's Unit 1 (C)
workbook, held in the `IMG` map as base64 WebP (420px, quality 80, ~11 KB
each). Letter sounds have no same/different constraint -- each picture only
has to be recognizable -- so richer art is safe here.

To add a picture: generate it in the same style (colored pencil and soft
watercolor, thin brown outline, isolated on white, **no text of any kind**),
resize to 420px, `cwebp -q 80`, base64 it into `IMG`, then add the word to
`CW` (begins with C) or `XW` + `FIRST` (doesn't).

### Why every picture is spoken aloud

On the paper workbook, several of her misses were naming gaps, not sound gaps:
she knows /k/, but if she calls the cookie jar "jar" or the cream carton
"milk", the right phonics reasoning still scores wrong. The game says each
word before she answers and lets her re-tap any picture to hear it again, so
that trap can't cost her a point. Keep that property in any new lesson.

Wrong answers say *why* -- "Basket begins with the b sound, not the c sound" --
which is what the `FIRST` map is for. Don't add a word without its sound.

## Bump the version on EVERY change

iPads cache this game aggressively. If the version isn't bumped, kids keep
playing the old build even after a successful deploy. Two places must change
together, and they must match:

1. **`sw.js` line 2** — the cache name:
   ```js
   const CACHE = 'kids-game-legacy-v10';   // -> 'kids-game-legacy-v11'
   ```
   Changing this string is what forces the service worker to reinstall and drop
   the previous cache. This is the one that actually matters.

2. **`index.html`** — the version label on the home screen (search for
   `version 10`):
   ```html
   <small style="color:#9aa3ad;font-size:12px">version 10</small>
   ```
   Bump to `version 11`. This is how you confirm on the iPad that the new build
   actually landed — read the number under "Pick a game to play."

Bump both, in the same commit, every time. `v10 -> v11 -> v12`, no skipping.

> Note: line 7 of `index.html` has a stale `<!-- v5 -->` comment left over from
> an earlier version. It is not used by anything. Either keep it updated too or
> delete it, but don't mistake it for the real version label.

## Deploying

Deploys are automatic. Push to `main` and Netlify rebuilds:

```sh
git add -A
git commit -m "Version 11"
git push
```

- GitHub: https://github.com/marvin848/kids-game-legacy
- Netlify: https://app.netlify.com/projects/kids-game-legacy
- Build command: *(empty — nothing to build)*
- Publish directory: `/`

After a deploy, hard-refresh on the iPad (or close and reopen the installed
app) and check the version label reads the new number.
