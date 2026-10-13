# sketch: ink

An interactive audiovisual toy in the style of a Chinese ink painting. Press keys to play synthesized traditional Chinese instruments and paint with a virtual brush, and walk a little ink-drawn sage around the page with `W` `A` `S` `D` and `Space`.

Everything is in a single file, [`index.html`](index.html). There is no build step, no dependencies and no network access needed. Open the file in a modern browser and press keys. (The browser only starts audio after the first key press, so don't worry if the page is silent until then.)

----

## Contents

1. [Controls](#controls)
2. [The sage](#the-sage)
3. [Instruments (sound)](#instruments-sound)
4. [Painting effects (visuals)](#painting-effects-visuals)
5. [How the code is organized](#how-the-code-is-organized)
6. [Customizing](#customizing)
7. [Browser support and notes](#browser-support-and-notes)

----

## Controls

| Key | Sound | What happens on screen |
|---|---|---|
| `W` `A` `S` `D` | guzheng / dizi notes (see below) | Walk the sage up / left / down / right |
| `Space` | wooden fish knock + high guzheng note | The sage jumps |
| `Q` `E` `R` `T` `Y` `U` `I` `O` `P` | guzheng (plucked zither) | A painting effect (table below) |
| `F` `G` `H` `J` `K` `L` | dizi (bamboo flute) | A painting effect (table below) |
| `Z` `X` `C` `V` | gong, wooden fish, war drum, small cymbals | A painting effect |
| `B` `N` `M` | erhu (bowed fiddle), three different notes | A painting effect |

Notes:
- Every mapped key makes a sound, including the movement keys. Pressing `W` `A` `S` `D` plays the same note it would have played before the sage was added, but paints nothing, because the sage moves instead.
- Holding a movement key keeps the sage walking. Held keys do not re-trigger the sound (key repeat is ignored), only the first press does.
- `Ctrl`, `Alt` and `Cmd` combinations are ignored so browser shortcuts still work.
- Unmapped keys do nothing.

----

## The sage

A small master with a topknot, a long beard, a white robe and a staff topped with a gourd. He is drawn as SVG on its own layer, above the paintings, so ink never covers him.

- **Walking:** `A` / `D` move him left and right and turn him to face the direction of travel. `W` / `S` move him up and down the page. Diagonals are normalized so moving diagonally isn't faster. Vertical movement is slower than horizontal (60%) so the page feels like a scroll painting, and he is kept in the lower two thirds of the page.
- **Depth:** the further up the page he stands, the smaller he is (a scale of about 0.9 near the top of his range up to about 1.5 at the bottom).
- **Walk animation:** his body bobs, his feet shuffle, his sleeve and staff swing, all driven by a single `phase` value that advances while he moves. When idle he breathes with a very slight bob.
- **Footprints:** every ~42 pixels walked he leaves a small, faint ink dot where his foot lands, alternating left and right. These dissolve like the other ink drops.
- **Jumping:** `Space` launches him with a simple physics model (initial velocity, gravity). While airborne his feet spread, the sleeve and staff swing up and his shadow shrinks. You can only jump from the ground, so pressing `Space` mid-air plays the sound but doesn't launch a second jump. When he lands, a small ring of ink spreads under him.

----

## Instruments (sound)

All sounds are synthesized live with the [Web Audio API](https://developer.mozilla.org/docs/Web/API/Web_Audio_API). No audio files are loaded, which keeps the page self-contained, but it also means they are imitations of the real instruments, not recordings.

Everything melodic is tuned to the **pentatonic scale** (C D E G A, the traditional gong–shang–jue–zhi–yu), so any combination of keys sounds consonant.

All instruments go through a shared master volume and a gentle hall-style reverb (a convolution reverb built from decaying noise).

| Instrument | Keys | How it's synthesized |
|---|---|---|
| **Guzheng** (plucked zither) | `Q W E R T Y U I O P` | Karplus–Strong plucked-string synthesis: a burst of noise recirculated through a short averaging delay line. Notes run C4, D4, E4, G4, A4, C5, D5, E5, G5, A5. The string waveforms are cached per pitch. |
| **Dizi** (bamboo flute) | `A S D F G H J K L` | A sine wave plus a quiet triangle octave, with ~5 Hz vibrato and a band-passed noise "breath" layer that fades after the attack. Notes run C5, D5, E5, G5, A5, C6, D6, E6, G6. |
| **Erhu** (bowed fiddle) | `B` `N` `M` | A sawtooth wave with a slow attack, a small slide up into the pitch, vibrato that grows over time, a low-pass filter and a resonant peak around 1.4 kHz for the nasal tone. Notes are G3, D4 and G4. |
| **Luo / gong** | `Z` | Six sine partials at inharmonic ratios, each with its own decay, plus a slight downward pitch glide. About 4 seconds long. Fundamental 110 Hz. |
| **Muyu** (wooden fish) | `X` | A short sine with a fast downward pitch sweep and a filtered noise click. |
| **Tanggu** (war drum) | `C` | A deep sine thump sweeping from 160 Hz down to 55 Hz, with a short low-passed noise "slap". |
| **Bo** (small cymbals) | `V` | High-passed noise plus four high square-wave partials, decaying over ~1.6 seconds. |
| **Jump sound** | `Space` | A wooden fish knock layered with a high guzheng note (E6). |

----

## Painting effects (visuals)

The page is white "paper" with black ink. The palette is strictly black, white and greys. Every effect fades and removes its own elements when it finishes, so the page never fills up or slows down over time.

| Effect | What it looks like |
|---|---|
| **Ink drop** (`drop`) | A dark core with two translucent halos and a few small splatters appears at a random spot. It bleeds outward while its edges blur, then dissolves into the paper over 2–3 seconds. |
| **Brush stroke** (`stroke`) | One sweeping curved calligraphy stroke. The width swells as the brush presses down and tapers as it lifts. A few thin, dry "flying white" hairs run through it. It is drawn progressively, then fades. |
| **Ensō** (`enso`) | The Zen brush circle, painted in a single stroke that doesn't quite close. The gong draws a larger one. |
| **Bamboo** (`bamboo`) | A jointed stalk grows upward segment by segment, then slender leaves sweep out from the top joints. |
| **Plum blossom** (`blossom`) | A dark branch is brushed in from the left or right edge, then five-petal flowers pop open along it. |
| **Misty mountains** (`mountain`) | Three overlapping, blurred ranges of diluted ink drift in along the bottom of the page and fade out. |
| **Paper wash** (`wash`) | The entire sheet briefly takes on a faint grey tone, then returns to white. |

Which key triggers which effect:

| Keys | Effects (in order) |
|---|---|
| `Q E R T Y U I O P` | stroke, blossom, stroke, enso, stroke, bamboo, blossom, stroke, enso (`W` is the movement key, so it is skipped) |
| `F G H J K L` | wash, drop, mountain, stroke, drop, wash (`A S D` are movement keys, so they are skipped) |
| `Z` | large ensō |
| `X` | drop |
| `C` | extra-large drop |
| `V` | wash |
| `B` | stroke |
| `N` | ensō |
| `M` | bamboo |

### How the brush works

The brush stroke is the core of most effects (`brush()` in the source). It is given a function `pt(t)` that returns the centreline of the stroke for `t` from 0 to 1. The function samples that curve, works out the direction at each point, and builds a filled polygon whose width follows a sine-shaped pressure curve with a little random wobble. A requestAnimationFrame loop redraws the polygon with more of the curve each frame, so the stroke appears to be painted, eased so the brush starts fast and slows towards the end. It returns a promise that resolves when the stroke is complete, which is how effects like bamboo and plum blossom chain further drawing after the first stroke.

----

## How the code is organized

`index.html` has three parts: some CSS, two SVG layers, and one script. The script reads top to bottom as follows (each section has a banner comment):

1. **Helpers**: `rand`, `pick`, `el()` (creates SVG elements) and `dissolve()` (fades a node out and removes it).
2. **Sound**: `initAudio()` creates the audio context lazily on the first key press (browsers require a user gesture), then one function per instrument (`pluck`, `dizi`, `erhu`, `gong`, `muyu`, `drum`, `cymbal`) and `penta()`, which turns a base pitch and a step number into a pentatonic frequency.
3. **Visuals**: `blot()`, `drop()`, `brush()`, `curve()`, `stroke()`, `enso()`, `bamboo()`, `blossom()`/`flower()`, `mountain()` and `wash()`.
4. **The sage**: the SVG drawing, `pose()` (updates his transform each frame), `footprint()`, `land()` and the `tick()` game loop (movement, jumping physics).
5. **Keys**: the `keymap` and the `keydown` / `keyup` handlers.

Layers: `#stage` (the paintings) sits below `#actor` (the sage). Both are fixed full-window SVGs that ignore the mouse.

----

## Customizing

**Add or change a key.** Everything is in the `keymap` object near the bottom of the script. An entry is `{ sound, fx }`: `sound` is a function that plays something, and `fx` is a function that draws something (or `null` for no visual).

```js
keymap.q = { sound: () => gong(220), fx: bamboo }
```

**Change the notes or scale.** `penta(base, i)` returns the frequency `i` pentatonic steps above `base` (in Hz). To use a different scale, edit the `steps` array inside it, which holds semitone offsets (currently `[0, 2, 4, 7, 9]`).

**Tweak how the sage feels.** Near the top of the sage section: `SPEED` (walking speed in pixels per second, currently 230), `JUMP` (jump launch velocity, 620) and `GRAVITY` (1900). A higher `JUMP` or lower `GRAVITY` gives higher, floatier jumps.

**Make the ink look wetter, drier or darker.** The blur amounts and opacities in `blot()` control how the ink drops spread and fade. The brush's width, wobble and dry hairs are set in `brush()`. The `maxW` argument each effect passes sets the stroke thickness.

**Add a new effect.** Write a function that draws into `stage` (use `el()` to create SVG nodes and `dissolve()` to remove them afterwards), then put it in a `keymap` entry.

----

## Browser support and notes

- Tested in headless Chromium. It should work in current Chrome, Edge, Firefox and Safari, but the audio has not been listened to in every browser. CSS `filter` blur on SVG elements and the Web Animations API are required.
- Audio only starts after the first key press (a browser rule). If the page is silent, click the page and press a key again.
- Keyboard layout: keys are matched by the character they produce, so on non-QWERTY layouts the keys will be in different physical positions.
- The effects are intentionally layered, so pressing many keys at once produces a dense, overlapping painting. That's by design, but a very long run of key mashing can briefly slow older machines.

----

◕ ◞ ◕ This project was made by MazterJeff using https://netnet.studio
