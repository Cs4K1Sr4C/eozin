# Eozin Lustre Studio

A static, single-file THREE.js page that simulates **eozin** – the reduction-fired
metallic lustre glaze introduced by the Zsolnay manufactory in Pécs in 1893 – on a
parametrically generated ceramic vessel.

Open `index.html` in a current browser (WebGL 2). There is no build step. three.js
r180 is imported from jsDelivr through an import map and the typefaces come from
Google Fonts, so the page needs a network connection on first load.

## What is simulated

**Firing → film.** Kiln temperature, soak time, reducing atmosphere, re-oxidation on
cooling and the copper / silver / gold / bismuth content of the lustre paste are turned
into a film structure: thickness of the metal-bearing layer (Arrhenius ion diffusion),
metal volume fractions (species-dependent reduction), a clear cover film and its
refractive index, and colloid size.

**Film → colour.** The stack *air / cover film / metal–glass composite / glaze* is solved
per wavelength: Maxwell-Garnett effective medium with Johnson & Christy optical constants
for Cu, Ag and Au, then an Airy thin-film recursion for both polarisations, integrated
against the CIE 1931 observer. The same code runs on the CPU (the spectrum chart and the
angle strip in the label) and per fragment in GLSL, where it replaces three.js's RGB
iridescence approximation inside `MeshPhysicalMaterial`.

**Surface.** Film thickness and metal fill vary across the pot: glaze pooling in hollows
and breaking on ridges (driven by mesh curvature), gravity runs, flame flashing
(domain-warped noise with a flame-side bias), crackle with a per-cell mosaic shift,
pinholes, orange peel, and a second painted lustre that can follow the relief.

**Vessel.** A monotone-spline meridian with foot ring, recessed base, wall thickness,
rounded lip and glazed interior; lobes (sine, gadroon, flute), spiral twist, oval squash,
turned collar rings, six relief motifs (leaves, scales, vines, spiral ribs, hammered,
cabochons), up to four pulled handles, and an unglazed foot with a dripped glaze line.

## Controls

| Group | Parameters |
| --- | --- |
| Vessel form, Sculpting, Handles | profile radii and heights, wall, lobes, twist, relief, rings, handles |
| Base glaze | colour, clay body, thickness, pooling, breaking, runs, gloss, crackle, pinholes, glaze line and drips |
| Lustre chemistry | copper, silver, gold, bismuth flux, paste thickness, coverage |
| Reduction firing | temperature, soak, atmosphere, re-oxidation, diffusion barrier, flashing pattern |
| Film optics | refractive index, dispersion, thickness variation, direct trims, spectral sample count |
| Decoration | second-lustre pattern and how its film differs |
| Stage & light | lighting rig, key light, exposure, tone curve, bloom, turntable, mesh detail, resolution |

The shelf at the bottom holds six lustre presets (Zöld, Hajnal, Labrador, Ibolya, Bronz,
Gyöngyház) and six forms. A lustre preset can be linked directly with its hash, for
example `index.html#labrador`. *Copy parameters* puts the whole state on the clipboard as
JSON; pasting that JSON into the page restores it.

## Limits of the model

The firing model is a plausible reduced-order description, not a calibrated kiln
simulation: its constants are chosen so that ordinary lustre firings (roughly 550–700 °C,
10–60 minutes) give films in the range reported for historical lustres. Maxwell-Garnett
theory ignores particle-size effects and is stretched at the highest metal fractions.
Colours are computed for a D65-like white and are clipped to sRGB.
