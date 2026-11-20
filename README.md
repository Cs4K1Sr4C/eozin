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
pinholes, orange peel, and a second painted lustre that can follow the relief, with a
brushed edge. The break of a thin glaze on an edge is ragged and has its own colour
(*Break colour*).

**Vessel.** A monotone-spline meridian with foot ring, recessed base, wall thickness,
rolled lip and glazed interior; lobes (sine, gadroon, flute), spiral twist, oval squash,
turned collar rings with coved roots, a waved petal rim, six relief motifs (leaves, scales,
vines, spiral ribs, hammered, cabochons), up to four pulled handles, an optional lid, and
an unglazed foot with a dripped glaze line that ends in a bead.

- *Handles* are pulled, not extruded: heavier at the top where the pull began and thinner
  towards the lower joint (*Pulled taper*), with the ridges the fingers leave down the
  back (*Finger ribs*), and smeared onto the wall in a skirt of clay at both joints
  (*Joint blend*).
- *Lids* are a third turned part: a domed, flat or conical cover that rests on the rim
  with a bare-clay flange inside the mouth, and a finial (ball, bud, acorn or disc) on a
  short stem. A lobed mouth gets a lobed lid; a waved rim cannot carry one. Under a lid
  the upper joint of a handle stays below the lip, clear of the brim.

**The set.** The floor sweeps up into a round studio cove, so every view has a graded
backdrop and no horizon line; *Pool of light* gathers its brightness around the plinth
and lets it fall away outward and up the wall. The cove is as large as the view needs:
seen from further out than its floor reaches (a long lens, a small window, the wheel) it
grows about the piece, so the camera always stands inside it and the pool of light stays
where it is. *Backdrop: Void* brings back the plain
dark floor that dissolves into fog. The plinth is a turned stand (torus base, cavetto,
eased top edge) in lacquer, veined black stone, ebonised wood or patinated bronze.

**Light, shadow and reflections.** The piece stands in a small procedural studio:
softboxes with a hot centre, a few bare bulbs for pin highlights and a black flag, rendered
into a prefiltered environment.

- *One key light.* The spotlight that casts the shadow stands where the main softbox of
  the room is, so shadow and brightest reflection agree and turn together with *Light
  rotation*. The glaze takes its highlight from the reflected softbox, not from a point
  glint (*Spotlight glint* brings one back; *Spotlight at main lamp* off frees the aim).
- *Soft shadow.* The key shadow hardens towards contact: crisp at the foot and the plinth
  edge, wider and softer as it falls across the floor (*Shadow softness* is the angular
  size of the light). The shadow map is fitted to the piece and redrawn only when the
  piece or the light moves.
- *Room reflections.* The studio is captured again from the middle of the vessel, with the
  lit plinth, the floor and the shadow in it, and looked up with a parallax correction: the
  foot mirrors the plinth it stands on, the lower belly the floor, the shoulders the lamps.
- *Polished stand.* The plinth (or, without it, the floor) mirrors the vessel: sharp at the
  contact line and dissolving with distance. *Plinth polish* and *Floor polish* set how
  much.

**Camera and film.** The view is taken through a lens, not a pinhole.

- *Lens.* *Lens · mm* is a focal length on a 35 mm frame; changing it moves the camera so
  the piece keeps its size and only the perspective changes. *Aperture* and *Focus
  height* set the depth of field: the focus lies on the wall nearest the camera at that
  height. After the vessel is dragged, the focus stays on the wall at the height the
  camera is aimed at, until a shot is taken or *Focus height* is moved.
- *Depth of field.* While something moves, the picture is blurred by depth in one pass.
  Once it rests, the page draws a fixed number of further frames (*Still quality*: 16, 32
  or 64), each through another point of the aperture and a fraction of a pixel aside, and
  shows their mean: true lens blur, a mirror image on the plinth that defocuses with
  distance, and supersampled edges. Then it stops drawing. *Depth of field: Stills*
  keeps the blur for the picture at rest (the default on phones), *Off* removes it.
- *Finishing.* Bloom only on speculars, with *Halation* warming its outer glow to red; a
  vignette; *Lens fringing* towards the corners; a *Look* (Neutral, Print, Catalogue,
  Nocturne) graded after the tone curve; film grain; and a dither against banding in the
  dark backdrop. *Look: Neutral* with grain, fringing and halation at 0 is the ungraded
  picture, in which the lustre colours are exactly those of the label.
- *Moves.* The camera never jumps: a new shape, a shot, Reset and the first view are all
  eased moves, and any drag on the vessel stops a move at once. With *reduced motion*
  set in the system, moves are cuts, the opening move and the grain animation are off
  and the view does not follow edits.

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
  pot, and leaves fine throwing rings in the glaze, strongest inside, their pitch and
  depth wandering up the wall. At 0 all of it is absent.
- *Micro-relief.* Orange peel, slump waves, throwing rings, crackle grooves and pinholes
  are one analytic height field whose slope is evaluated per pixel in the pot's own space,
  on a lattice turned off the pot's axes. Detail too fine for a pixel is not dropped but
  handed to the roughness, so the gloss does not change as the camera moves in.
- *Pooling at two scales.* The cavity that thickens the glaze is the mesh curvature at
  the scale of a relief cut and again over about a third of a centimetre of wall, so
  glaze also gathers in the lap of a ring or the root of a neck.
- *What lies on the surface.* Dust settles on what faces up, as a faint haze and as
  single motes, on the piece and on the plinth; hands leave a duller bloom where a pot is
  held; the bare foot shows coarse grog, grains of kiln sand and a rubbed foot ring. All
  of it is for close-ups and fades before it could shimmer.

## Controls

| Group | Parameters |
| --- | --- |
| Shape, Sculpting | profile radii and heights, wall, *Hand of the maker* (hand-made irregularity), lobes, twist, relief, rings, rim waves |
| Handles & lid | handle count, joints, reach, arch, thickness, flatness, *Joint blend* (how far a handle flares and spreads into the wall), *Pulled taper* (`handleTaper`, 0–1), *Finger ribs* (`handleRibs`, 0–3); *Lid* (`lid`: None, Domed, Flat, Conical), *Lid rise* (`lidRise`), *Lid overhang* (`lidOverhang`, cm), *Finial* (`finial`: None, Ball, Bud, Acorn, Disc), *Finial size* (`finialSize`, cm) |
| Glaze | colour, clay body, thickness, dark in hollows, thin on edges and its *Break colour* (`breakTone`), runs, gloss, surface ripple, *Slump waves* (`wave`), crackle, pinholes, *Dust* (`dust`), *Handling bloom* (`wear`), *Foot grit* (`grit`), bare foot and drips |
| Lustre metals | copper, silver, gold, bismuth, paste coat, coverage |
| Firing | temperature, soak, smoke (reduction), air on cooling (re-oxidation), glaze resistance (diffusion barrier), flame marks |
| Film optics | refractive index, dispersion, thickness variation, direct trims (layer depth, metal amount, clear skin) |
| Painted decoration | second-lustre pattern and how its film differs |
| Lighting | lighting rig (Atelier softboxes, North window, Vitrine spots, Kiln glow, and the low-key *Chiaroscuro*), room light, spotlight (aim, shadow softness, glint), *Backdrop* (`backdrop`: Cove, Void), *Backdrop tone* (`backdropTone`), *Pool of light* (`pool`), plinth on/off, *Plinth material* (`plinthMat`: Lacquer, Black stone, Ebonised wood, Bronze), plinth and floor polish, *Contact shading* (ambient occlusion and the contact shadow) |
| Camera & film | *Lens · mm* (`focal`, 24–135, default 45), *Aperture · f/* (`fstop`: 1.4 to 16, default 4), *Depth of field* (`dof`: Off, Stills, Always), *Focus height* (`focusLevel`, a level, default 0.5), *Still quality* (`stillSamples`: 16, 32, 64), *Look* (`look`: Neutral, Print, Catalogue, Nocturne), *Film grain* (`grain`), *Lens fringing* (`chroma`), *Halation* (`halation`), exposure, tone curve, bloom, vignette, *Presentation frame* (`frame`: Free, 2.39:1, 16:9, 1:1) |
| Rendering | turntable, room reflections and their detail, mesh detail, pixel ratio, adaptive quality, colour sample count |

Positions along the piece are *levels*: a share of the height, 0 at the foot and 1 at the
rim. Sizes of surface patterns (flame size, run width, variation scale, crackle cell) are
in centimetres.

*Dust*, *Handling bloom* and *Foot grit* belong to the one piece on the stand, not to a
glaze recipe: lustre presets leave them alone. A parameter set saved before handles had
a taper is restored with even, smooth straps, as it was drawn then.

### Views

The third row of the shelf holds five shots, worked out from the piece as it is, and
*Present*.

- **Shots** (keys 1 to 5). *Hero*: three-quarter view, 45 mm at f/4. *Profile*: side-on
  and level, 105 mm at f/8. *Lip*: from above into the mouth, or onto the finial of a
  lidded piece, 70 mm at f/8. *Foot*: low, on the foot ring and its reflection, 60 mm at
  f/2.8. *Detail*: close on the middle of the relief band, or of the body where there is
  no relief, 85 mm at f/4. A shot sets lens, aperture and focus height as one undo step
  and flies the camera there; the three close shots hold the turntable until the vessel
  is dragged or another view is chosen (a plain click or tap leaves the shot as it is).
  The camera pose itself is not a parameter: undoing a shot puts the lens values back,
  releases the shot and leaves the camera where it is.
- **Present** (key P; Esc or P returns). The panels leave, the piece is fitted to the
  whole stage or to the *Presentation frame* with bars around it, and a caption gives
  the lustre, the form and its height, and the firing. Moving the pointer, pressing a
  key or, on a touch screen, tapping the stage brings the shelf back for a moment. While
  presenting, the shelf also has *Save* (key S): the picture as presented, cropped to
  its frame. On an upright phone the wide frames are ignored.
- **Follow edits.** While a control that belongs to one part of the piece is changed
  (the rim, the foot, a handle, the lid, the relief band, the glaze surface…), the
  camera eases to that part and focuses there, and about a second and a half after the
  last change it eases back. Dragging the vessel meanwhile keeps the view where it is
  put. *Follow edits* in the ⋯ menu switches this off; the choice is remembered. Preset
  chips, undo, paste and Reset do not move the view this way.

### Using the panel

- **Simple / All.** The panel opens in *Simple*, with the thirty or so controls that change
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
  text box for touch screens and for frames without clipboard access. *Save PNG* takes
  an exposure: the turntable waits, the still is gathered in full (at least 32 lens
  samples) without the page layout's offset, and the picture is saved at the size of the
  canvas, cropped to the frame while presenting in one (*Save* on the shelf, or key S,
  starts it there); Esc cancels. *Follow edits* and
  *Reset everything* are here too.
- **Keys.** In a number field, PageUp and PageDown move ten steps and Home and End go to
  the ends of the range. 1 to 5 take the shots, P presents and S saves the presented
  picture, except while a field is being typed in. Escape closes the menu, or else cancels an exposure, leaves the
  presentation, and on small screens closes the open panel. On phones the panel is a
  sheet that shows one folder at a time.

The shelf at the bottom holds eight shapes (two of them covered: *Ginger jar* and *Covered urn*) and six lustre presets (Zöld, Hajnal, Labrador,
Ibolya, Bronz, Gyöngyház). A preset restores every parameter of its groups, and its chip
stays pressed exactly while all of them have the preset's values. A lustre preset can be
linked directly with its hash, for example `index.html#labrador`; the hash is dropped as
soon as the glaze is edited and comes back when the edit is undone.

**Adaptive quality** keeps the frame rate usable on slower GPUs: it first renders fewer
pixels, then gives up stage extras one at a time (the plinth mirror at lower resolution
with fewer blur taps and half the lens samples of a still; the mirror and the blur of
moving frames; a smaller shadow map and 8 lens samples; a smaller studio capture), then
falls back to a lighter mesh, and tries the finer setting again from time to time. The
frames that refine a still are not counted as a measure of speed. A scene in which
nothing moves is not redrawn at all once its still is complete.

## Limits of the model

The firing model is a plausible reduced-order description, not a calibrated kiln
simulation: its constants are chosen so that ordinary lustre firings (roughly 550–700 °C,
10–60 minutes) give films in the range reported for historical lustres. Maxwell-Garnett
theory ignores particle-size effects and is stretched at the highest metal fractions.
At those fractions (Hajnal and Bronz, for instance) the lustre layer is opaque, so its
thickness and the glaze colour beneath only show where the film thins: on relief edges,
in flame-lean patches and in pinholes. The label says whether the film is opaque.
Colours are computed for a D65-like white and are clipped to sRGB.
