# Mantra Cosmosis: project guide

Read this first. It is the hand-off from the session that built the project.

## What this is

A single-file web art piece (`index.html`). Eight devotional figures are drawn as particles on a canvas and chant their mantras in deep space. Lips move syllable by syllable. Voices, drones, bells and drums are synthesised live with Web Audio (no recordings). Live artifact (private) the first version was built in: https://claude.ai/artifact/7vHA2XneEdfuM5dBQsvg7j

Figures and mantras (mode key in brackets):

| Key | Figure | Mantra |
|---|---|---|
| `shiva` | Shiva (still, monumental yogic bust, inspired by traditional iconography; never name a specific statue in user-facing text) | Mahamrityunjaya, or Om Namah Shivaya (toggle) |
| `kali` | Kali (crimson/black) | Kreem |
| `bhairav` | Bhairav (black sun, amber eyes, fire) | Om Bhram Bhairavaya Namah |
| `monk` | Buddhist monk | Om Mani Padme Hum (+ optional Tibetan overtone voice) |
| `gayatri` | Bearded sage + rising sun | Gayatri Mantra |
| `ganesha` | Ganesha (trunk, ears, tusk, crown) | Om Gam Ganapataye Namah |
| `tara` | Green Tara as a goddess | Om Tare Tuttare Ture Soha |
| `hanuman` | Hanuman (heroic vanara, Ram tilak, gada) | Om Ham Hanumate Namah, with Ham written हँ (owner's choice) |

A Sufi figure existed and was **deliberately removed** at the owner's request. Do not re-add it.

## Hard constraints

- **One file, no build step.** Everything is inline in `index.html`. No bundler, no framework, no npm dependencies.
- **Only external resource: Google Fonts** (Cinzel, Tiro Devanagari Hindi, Noto Serif Tibetan). Do not add other external hosts.
- Must work on phones first (about 360x640 up to desktop), no horizontal scroll, safe-area insets respected.
- Sound may only start after a user tap. Keep the iOS silent-switch workaround (`unlockIOS`).
- Respectful depiction: these are deities and mantras of Hindu and Buddhist traditions. Keep meaning/gyan text restrained and accurate: no health claims, no supernatural promises, no fear language about Kali or Bhairav.
- Do not impersonate real people. (An early request to do a real neuroscientist was declined; that older chat prototype is not part of this repo.)

## Code map (all inside the one `<script>`)

1. **Mantra data**: `WORDS_S` (Mahamrityunjaya), `WORDS_N` (Namah Shivaya), `WORDS_K`, `WORDS_B`, `WORDS_M`, `WORDS_GA`, `WORDS_GN`, `WORDS_T`, `WORDS_H`. Each word: `{d: Devanagari, l: Latin label, tts?: spoken text, s: [syllables]}`. A syllable is `[text, vowel(a e i o u m), mouth openness 0-1, lip rounding (negative = spread), starts-with-lip-closure 0/1, seconds]`. `MODES` holds per-figure `words, pause, title, sub, begin, meaning, gyan`. `buildTL` turns words into a timeline. `SHIVA_VARS` holds the two Shiva chants. `MODEKEYS` and `MODE_BTNS` list modes and their strip buttons.
2. **Audio**: `initAudio` builds the graph: detuned saws, three vowel formant filters, vibrato, sub-bass, choir oscillators, drone, generated reverb. `VOICEP` (filter/sub/vibrato/Q per mode), `BASEP` (pitch per mode), `DRONES`, `DRUMI` (drum interval) tune each figure. `setVowel/setPitch/applyVoice` drive it, `bowl/drum/burst/whoosh/boom` are effects. Spoken Hindi (`pickVoice`, `startWord`, `spokenKick`) is optional and off by default.
3. **Canvas and cosmos**: `layout`, nebula textures (`makeNebula`, `NEBS`), stars, spiral galaxy (`GAL`, `GALC`, `GALF`, `GALD`), embers (`EMB`).
4. **Figures**: Shiva uses its own kinds (`faceM`, `eyesS`, `noseS`, `jaw`, `cobra2`, `ganga`, `rudra`) and Hanuman `faceH`, `browH`, `snout`, `tilak`, `sindoor`, `gada`; both share `pecs`. Skin kinds are in the `SKIN` set. `layout()` fits the figure between the top bar and the controls (uses the panel height `ph`); `FADE0` marks where the lower bust dissolves before the Begin button (the face and neck are never faded). `layer(kind, draw, stepX, stepY)` draws a shape on a hidden canvas and samples it into particles of that `kind`. Per-kind data: `COLS[mode][kind]` colours, `SIZE`, `PRI` (draw order), `GLOW` (additive kinds), and visibility sets `SHIVA_K, KALI_K, ...` collected in `SETS`. Mouth parts (`lip`, `inner`, `tongue`, `fang`) are re-aimed each frame from the chant state.
5. **State and loop**: `frame()` does the chant clock and mouth targets, cosmos drawing, per-mode atmosphere (halo, eclipse, sun, lotus, fire), then the figure particle loop. `WT` holds smooth per-mode blend weights (`mix3` blends colours). `scatter()` / `formed()` implement the optional gather effect. A quality governor lowers draw density on slow devices (`?q=0|1|2` forces a level). The loop pauses when the tab is hidden.

Mode theming in CSS is via `:root[data-mode="..."]` variables (`--gold`, `--glow`, `--bg`, `--line`, `--dim`, `--fg`, `--panel`).

UI: top bar has a large Chant button and a Settings tray (Mantra, Voice, Overtone, Choir, Gather, Drone, Meaning, Gyan, My recording). The swipeable mode strip and mantra panel sit at the bottom. Panel height is published as `--ph` so the strip and Begin button position above it.

## Adding a new figure (checklist)

1. Add `WORDS_X` and a `MODES.x` entry (title, sub, begin, meaning, gyan, pause).
2. Add `'x'` to `MODEKEYS` and a `['mX','x']` pair to `MODE_BTNS`, plus a chip in `#modes`.
3. Add `x` to `WT`, a `:root[data-mode="x"]` CSS block, and entries in `VOICEP`, `BASEP`, `DRONES` (and `DRUMI` if it has a drum).
4. Add `COLS.x` (missing kinds fall back to the face colour), a visibility set (`X_K`) and an entry in `SETS`.
5. Add any new figure shapes with `layer(...)` plus `SIZE`/`PRI` entries, and a `mk()` depth rule if needed.
6. Add atmosphere in `frame()` (background arrays in the `mix3` calls, nebula in `NEBS`, `GALC`/`GALF`, `RIP`, `BLOOM`, optional custom glow block).
7. Screenshot it at phone size and check there are no console errors.

## Advisor team

Six read-only advisor lenses live outside the repo in the owner's `~/pantheon/roster/` (`mc-vfx-artist`, `mc-visual-designer`, `mc-esoteric-master`, `mc-aghori-baba`, `mc-mantra-sadhak`, `mc-tantra-sadhak`), with Bindu for the final design pass. They are lenses written for the owner, not real practitioners; they do not replace the native-speaker review.

## Testing

Playwright is the quick check (no test framework in the repo). Minimal pattern:

```js
const {chromium}=require('playwright');
const b=await chromium.launch({args:['--autoplay-policy=no-user-gesture-required']});
const p=await b.newPage({viewport:{width:390,height:844}});
p.on('pageerror',e=>console.log('ERR',e.message));
await p.goto('file://'+process.cwd()+'/index.html'); await p.waitForTimeout(3000);
await p.click('#setBtn'); await p.click('#gather'); await p.click('#setBtn'); // gather off for speed
await p.click('#mK'); await p.click('#chant'); await p.waitForTimeout(3000);
await p.screenshot({path:'out.png'});
```

Mode chip ids: `mS mK mB mM mGa mGn mT mH`. Always check: no `pageerror`/console errors in every mode, no horizontal scroll, face and lips not covered by UI, and screenshots at 390x844, 360x640 and desktop.

## Owner preferences learned

- The **original synthesised chant voice is preferred** over the device's spoken Hindi voice (which sounded robotic). Keep synthesised as the default.
- Intensity matters: do not tone down the Kali / Bhairav / Hanuman looks or the gather effect. The **Gather** effect stays but must remain optional (default on).
- Tara must read as a goddess (halo, jewels, lotuses), not a monk.
- Keep the page clean: one big Chant button, settings in the tray, mode strip swipeable.

## Known limits and open items

- Voices are synthesised imitations, not recordings. A real recorded chant layer would be a big upgrade (the "My recording" button lets the lips follow any audio file by loudness only).
- The quality governor never steps back up until reload. Hiding the tab pauses visuals but not audio.
- Ganesha's ear fans run off the sides on narrow phones. Hanuman's gada is closer to the figure now but can still touch the left edge on the narrowest phones.
- Shiva's phone panel is four rows tall, so on a 360x640 phone the figure is small; Kali's skull garland still dims behind the Begin button on desktop.
- **Native-speaker review still needed** for the Hindi gyan lines and a few spellings: see `CONTENT_REVIEW.md` (हूँ vs हूं, Tibetan padme spelling, Tara and Bhairav notes).
- `LICENSE` is an all-rights-reserved notice written for the owner; swap it if she picks a standard license later.
- Ideas the owner liked: more figures (Lakshmi, Durga, Krishna, Saraswati, Nam Myoho Renge Kyo), recorded/sampled chant layers, shareable links to a figure, a meditation timer.

## Git

Default branch `main`. Remote: https://github.com/aashna-builds/mantracosmosis. Commit messages: short imperative summary, then a sentence of why. The page can be hosted as-is on any static host (for GitHub Pages: Settings, Pages, source `main` / root).
