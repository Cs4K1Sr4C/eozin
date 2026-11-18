# Eozin Lustre Studio

A static, single-file THREE.js page that simulates **eozin** – the reduction-fired
metallic lustre glaze introduced by the Zsolnay manufactory in Pécs in 1893 – on a
parametrically generated ceramic vessel.

Open `index.html` in a browser with WebGL 2 and import maps (Chrome 89+, Firefox 108+,
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
rounded lip and glazed interior; lobes (sine, gadroon, flute), spiral twist, oval squash,
turned collar rings, a waved petal rim, six relief motifs (leaves, scales, vines, spiral
ribs, hammered, cabochons), up to four pulled handles, and an unglazed foot with a
dripped glaze line.

## Controls

| Group | Parameters |
| --- | --- |
| Vessel form, Sculpting, Handles | profile radii and heights, wall, lobes, twist, relief, rings, rim waves, handles |
| Base glaze | colour, clay body, thickness, pooling, breaking, runs, gloss, crackle, pinholes, glaze line and drips |
| Lustre chemistry | copper, silver, gold, bismuth flux, paste thickness, coverage |
| Reduction firing | temperature, soak, atmosphere, re-oxidation, diffusion barrier, flashing pattern |
| Film optics | refractive index, dispersion, thickness variation, direct trims, spectral sample count |
| Decoration | second-lustre pattern and how its film differs |
| Stage & light | lighting rig, key light, exposure, tone curve, bloom, turntable, mesh detail, pixel ratio, adaptive quality |

Sizes of surface patterns (flame size, run width, variation scale, crackle cell) are in
centimetres. Controls that cannot do anything in the current state are greyed out, for
example the lobe settings while the lobe count is zero.

The shelf at the bottom holds six lustre presets (Zöld, Hajnal, Labrador, Ibolya, Bronz,
Gyöngyház) and six forms. A preset restores every parameter of its groups. A lustre
preset can be linked directly with its hash, for example `index.html#labrador`; the hash
is dropped again as soon as the glaze is edited. *Copy parameters* puts the whole state
on the clipboard as JSON; pasting that JSON into the page restores it.

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
