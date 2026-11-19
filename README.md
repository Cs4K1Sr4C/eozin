# Eozin Lustre Studio

A static, single-file THREE.js page that simulates **eozin** – the reduction-fired
metallic lustre glaze introduced by the Zsolnay manufactory in Pécs in 1893 – on a
parametrically generated ceramic vessel.

Open `index.html` in a browser with WebGL 2 and import maps (Chrome 93+, Firefox 108+,
Safari 16.4+). There is no build step. three.js r180 is imported from jsDelivr through an
import map, so the page needs a network connection on first load. The typefaces come from
Google Fonts and are optional: the page falls back to system fonts without them.

## What is simulated

**Firing → film.** Kiln temperature, soak time, reducing atmosphere, re-oxidation on
cooling and the copper / silver / gold / bismuth content of the lustre paste are turned
into a film structure: thickness of the metal-bearing layer (Arrhenius ion diffusion),
metal volume fractions (species-dependent reduction, fed by the paste as a metal
reservoir), a clear cover film and its refractive index, and colloid size.

**Film → colour.** The stack *air / cover film / metal–glass composite / glaze* is solved
per wavelength: Maxwell-Garnett effective medium with Johnson & Christy optical constants
for Cu, Ag and Au, then an Airy thin-film recursion for both polarisations, integrated
against the CIE 1931 observer. Each spectral sample stands for a band of wavelengths, so
thick films wash out to their phase-averaged reflectance instead of aliasing. The same
code runs on the CPU (the spectrum chart and the angle strip in the label) and per
fragment in GLSL, where it replaces three.js's RGB iridescence approximation inside
`MeshPhysicalMaterial`, for direct and image-based light alike.

**Surface.** Film thickness and metal fill vary across the pot: glaze pooling in hollows
and breaking on ridges (driven by mesh curvature), gravity runs, flame flashing
(domain-warped noise with a flame-side bias), crackle with a per-cell mosaic shift,
pinholes, orange peel, and a second painted lustre that can follow the relief.

**Vessel.** A monotone-spline meridian with foot ring, recessed base, wall thickness,
rolled lip and glazed interior; lobes (sine, gadroon, flute), spiral twist, oval squash,
turned collar rings with coved roots, a waved petal rim, six relief motifs (leaves, scales,
vines, spiral ribs, hammered, cabochons), up to four pulled handles that flare into the
wall, and an unglazed foot with a dripped glaze line that ends in a bead.

**Mesh realism.** Everything below is computed from the profile; there are no textures.

- *Contact shading.* Ambient occlusion is traced in the meridian plane and stored per
  vertex: the interior, the foot, the coves under rings and the crotch of a handle joint
  receive less of the studio light. The same trace darkens a soft contact shadow on the
  plinth around the foot.
- *Relief filtering.* Motifs with a hard outline are averaged over each vertex's patch of
  wall, and every quad is split along the diagonal that follows the relief, so scale rims
  and cabochon collars are slopes instead of staircases.
- *Hand of the maker.* A seeded, smooth warp makes the piece very slightly oval, leaning
  and uneven at the rim (the foot stays flat), lets the motifs wander and swell around the
  pot, and leaves fine throwing rings in the glaze, strongest inside. At 0 all of it is
  absent.

## Controls

| Group | Parameters |
| --- | --- |
| Shape, Sculpting, Handles | profile radii and heights, wall, *Hand of the maker* (hand-made irregularity), lobes, twist, relief, rings, rim waves, handles and their *Joint blend* (how far a handle flares and spreads into the wall) |
| Glaze | colour, clay body, thickness, dark in hollows, thin on edges, runs, gloss, crackle, pinholes, bare foot and drips |
| Lustre metals | copper, silver, gold, bismuth, paste coat, coverage |
| Firing | temperature, soak, smoke (reduction), air on cooling (re-oxidation), glaze resistance (diffusion barrier), flame marks |
| Film optics | refractive index, dispersion, thickness variation, direct trims (layer depth, metal amount, clear skin), colour sample count |
| Painted decoration | second-lustre pattern and how its film differs |
| Light & view | lighting rig, room light, spotlight, exposure, tone curve, bloom, *Contact shading* (ambient occlusion and the contact shadow), turntable, mesh detail, pixel ratio, adaptive quality |

Positions along the piece are *levels*: a share of the height, 0 at the foot and 1 at the
rim. Sizes of surface patterns (flame size, run width, variation scale, crackle cell) are
in centimetres.

### Using the panel

- **Simple / All.** The panel opens in *Simple*, with the two dozen controls that change
  the piece most. *All* shows every parameter; the choice is remembered. Presets and
  pasted parameter sets also set the controls that Simple hides, and the line at the end
  of the list says how many hidden controls differ from the preset.
- **Help strip.** Pointing at a row, tapping it or tabbing into it shows what it does, its
  range and its preset value under the list. Controls that cannot do anything in the
  current state are greyed out, for example the lobe settings while the lobe count is
  zero; the strip then says what they wait for.
- **Colour tracks.** The sliders of the lustre metals, the firing schedule and the film
  trims are painted with the face-on lustre colour each value would give with everything
  else as it is, so the track shows where a colour lies before the slider is moved.
- **Changed marks and Restore.** A dot marks every row that differs from the preset it
  started from, and the folder title counts them. Double-click a row's name, or use
  *Restore* in the help strip, to put one value back; *Restore shape* and *Restore
  lustre* in the ⋯ menu put a whole group back.
- **Undo.** Every change is an undo step, presets and Reset included: the ↶ ↷ buttons, or
  Ctrl+Z and Ctrl+Shift+Z (Ctrl+Y). The camera is not part of the history.
- **Surprise me** picks a preset shape, varies it and fires a random lustre on it; with
  Shift held it keeps the shape and changes the lustre only.
- **⋯ menu.** *Copy parameters* puts the whole state on the clipboard as JSON. Pasting
  that JSON anywhere on the page restores it, as does *Paste parameters…*, which opens a
  text box for touch screens and for frames without clipboard access. *Save PNG* and
  *Reset everything* are here too.
- **Keys.** In a number field, PageUp and PageDown move ten steps and Home and End go to
  the ends of the range. Escape closes the menu, and on small screens the open panel. On
  phones the panel is a sheet that shows one folder at a time.

The shelf at the bottom holds six shapes and six lustre presets (Zöld, Hajnal, Labrador,
Ibolya, Bronz, Gyöngyház). A preset restores every parameter of its groups, and its chip
stays pressed exactly while all of them have the preset's values. A lustre preset can be
linked directly with its hash, for example `index.html#labrador`; the hash is dropped as
soon as the glaze is edited and comes back when the edit is undone.

**Adaptive quality** keeps the frame rate usable on slower GPUs: it first renders fewer
pixels, then falls back to a lighter mesh, and tries the finer setting again from time to
time. A scene in which nothing moves is not redrawn at all.

## Limits of the model

The firing model is a plausible reduced-order description, not a calibrated kiln
simulation: its constants are chosen so that ordinary lustre firings (roughly 550–700 °C,
10–60 minutes) give films in the range reported for historical lustres. Maxwell-Garnett
theory ignores particle-size effects and is stretched at the highest metal fractions.
At those fractions (Hajnal and Bronz, for instance) the lustre layer is opaque, so its
thickness and the glaze colour beneath only show where the film thins: on relief edges,
in flame-lean patches and in pinholes. The label says whether the film is opaque.
Colours are computed for a D65-like white and are clipped to sRGB.
