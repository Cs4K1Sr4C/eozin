# Eozin Lustre Studio

A static, single-file THREE.js page that simulates **eozin** – the reduction-fired
metallic lustre glaze introduced by the Zsolnay manufactory in Pécs in 1893 – on a
parametrically generated ceramic vessel.

Open `index.html` in a browser with WebGL 2 and import maps (Chrome 93+, Firefox 108+,
Safari 16.4+). There is no build step. three.js r180 is imported from jsDelivr through an
import map, so the page needs a network connection on first load. The typefaces come from
Google Fonts and are optional: the page falls back to system fonts without them.

## The page

**Plate and catalogue.** The piece stands on a plate (the 3D stage, its caption and the
camera views); everything else is one catalogue column that scrolls by itself: the six
lustres, the eight shapes, *Surprise me*, the fired-film data, the reflectance chart and
a short text about eozin. Wide windows put the catalogue on the right, upright ones
(phones, upright tablets) below the plate; the page itself never scrolls, and the piece is
always fitted to the part of the stage that caption, views and masthead leave free.

**What a visitor can do.** Choose a lustre and a shape, press *Surprise me*, take one of
the five views, present the piece, switch the language, and drag or pinch the vessel.
There is nothing else on the page. Every parameter described below still exists and
travels with presets, pasted JSON and links, but the configurator that edits them one by
one is hidden (see *The configurator*).

**English and Hungarian.** The page opens in Hungarian for a browser whose language list
puts Hungarian before English, otherwise in English. The EN / HU switch in the masthead
changes every word at once, without a reload, and the choice is remembered.
`index.html?lang=hu` or `?lang=en` sets the language for one visit (a region tag such as
`hu-HU` or `en-GB` is read as its language). Numbers follow the
language (a decimal comma in Hungarian); stored parameter values and option names stay
English, so a parameter set copied in one language pastes in the other.

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
code runs on the CPU (the spectrum chart and the angle strip in the catalogue) and per
fragment in GLSL, where it replaces three.js's RGB iridescence approximation inside
`MeshPhysicalMaterial`, for direct and image-based light alike. The film answers in two
lobes: a sharp image of the lamps at the roughness of the glass surface, and under it a
broad glow from the metal colloids (*Colloid scatter*, `scatter`; 0 gives the single lobe
of earlier versions). The share of the glow follows the haze of the fired film, so the two
lobes together reflect what one did. The haze is the colloids': a film with no metal in
it (Gyöngyház) has none. Its clear bismuth film scatters by its own fine unevenness
instead and sends a fixed two fifths of its reflection into the broad lobe (`sheen` in
`deriveOptics`, in proportion to the bismuth; a film with metal has none of it). Toward
grazing the film's Fresnel factors are worked out at an angle that stops short of
grazing (its cosine never falls under 0.35), as for a layer that lies under a glassy skin
and is met by refracted light, while the path through each layer, which sets the hue, is
kept at the true angle. So a wall seen at a slant keeps its colour and slides along the
series, and only the last degrees before the silhouette go pale; before this the film
turned to a white mirror from about 65 degrees on and ribs near the silhouette read grey
with white lines. This is an approximation, not a third layer in the stack: no skin
reflection of its own is added. The glaze is coloured glass: seen at a slant the light
travels further through it and the colour deepens.

**Surface.** Film thickness and metal fill vary across the pot: glaze pooling in hollows
and breaking on ridges (driven by mesh curvature), gravity runs, flame flashing
(domain-warped noise with a flame-side bias), crackle with a per-cell mosaic shift,
pinholes, orange peel, and a second painted lustre that can follow the relief, with a
brushed edge. The flashing gathers into a few tongues with an edge (*Flame fronts*,
`front`); round the edge of a mark the cover film swells by some tens of nanometres and
wears a ring of the neighbouring interference colours (*Flame-mark halo*, `halo`), and
fine drawn-out streaks run through the film (*Film streaks*, `streak`). With all three at
0 the flashing is the even mottle of earlier versions. The kiln leaves two finer marks,
both small by default. *Kiln mottle* (`mottle`, 0–1, default 0.5): the depth and the metal
of the film also wander in soft clouds about 2 cm across and in a grain of about 3 mm
(a few per cent of the depth at the default; the two noises are ones the micro-relief
already works out, so nothing is added per pixel but arithmetic). *Kiln specks*
(`specks`, 0–1, default 0.35): sparse pinholes, 0.3 to 1 mm across, well under one per
square centimetre, where a bubble burst in the glaze or a crumb of the paste fell away:
no film there, over darker glass, so each is a dark point and never a glint; smaller than
a pixel they spread and fade. And the clear cover film now follows
the relief as the metal layer already did (*Lustre pooling*): a little thinner on a
ridge, thicker in a hollow, by a few per cent. Where a thin glaze breaks on an edge it stays glossy and goes darker and warmer than the wall:
the piece's own glaze colour stained halfway towards *Break colour*, never paler. Its width wanders slowly along
the edge, so it stays one line.

**Vessel.** A monotone-spline meridian with foot ring, recessed base, wall thickness,
rolled lip and glazed interior; lobes (sine, gadroon, flute), spiral twist, oval squash,
turned collar rings with coved roots, a waved petal rim, six relief motifs (leaves, scales,
vines, spiral ribs, hammered, cabochons), up to four pulled handles, an optional lid, and
an unglazed foot with a dripped glaze line that ends in a bead.

- The *foot* is turned (*Turned foot*, `footTurn`): the trimming tool leaves a cut line
  where the glaze stops, the ring below it stands a hair inside the wall, the bare clay
  carries fine turning marks, and the glaze gathers in a fatter bead on the cut. At 0 it
  is the plain foot of earlier versions.
- *Spiral ribs* are reeded: a narrow crest on a hollow flank, so each rib carries its own
  highlight and shadow. Vines are narrow cords, a little wider on the Draft mesh, which
  cannot carry the narrow one; cabochons sit in a higher collar. A saved piece with spiral
  ribs keeps its count and height and gets the new crest shape. The ribs run out in a long
  taper at both ends of their band (a blunt end held a dark drop of pooled glaze and shade).
- The glaze is pulled thin on the crest of the *lip*: with *Thin on edges* above zero a
  fine, continuous, glossy line of the break runs round the rim where it faces straight up,
  wider as *Thin on edges* is raised. That line stains the glass only; the lustre on it
  is left as on the wall (thinned there, it turned the crest's highlight grey-cream). The
  highlight on the crest itself stays: it is the lamp overhead, not the glaze. The rounded lip as a whole is no longer treated as a
  ridge by the curvature field (it used to break over a centimetre of wall on either side,
  a pale matte band); that field is faded out over the lip and back in down the wall.

- *Handles* are pulled, not extruded: heavier at the top where the pull began and thinner
  towards the lower joint (*Pulled taper*), with the ridges the fingers leave down the
  back (*Finger ribs*), and luted onto the wall at both joints (*Joint blend*). Close
  to the wall the strap spreads into a quarter-round fillet of clay (0.42 of the strap's
  thickness at full blend) that runs out along the wall; its shading normal turns into
  the wall's own over half a strap's thickness and is the wall's at the join, so a
  highlight crosses from strap to wall without a crease; and a fine line of pooled glaze
  follows the join on the strap's side, over the broader pooling on the wall around it.
  The root used to swell into a collar half as thick again as the strap that met the
  wall at about 27 degrees, with a visible edge; the collar is now a quarter as thick
  again, and the fillet takes its place. At blend 0 the joint is the plain butt it was.
  The fillet is not made everywhere: where a strap leaves the wall at a shallow angle (a
  short reach: the strap needs more than about a third further along its spine to stand
  clear of the wall by the fillet than the straight way out) and on a wide flat strap
  (*Strap flatness* over about 1.2, none from 1.6) the joint fades back, per joint, to
  the skirt and collar it had before, because the fillet tore and folded there (ragged
  glints and a dark slit at the lower joint at reach 1 cm, and at any reach with
  flatness 1.8). The default handles (flatness 0.78) keep the full fillet. And where
  the turned normals of a triangle would look away from its own face its corners keep
  their own: a per-triangle count of such faces on the Standard handle mesh is 0 on
  the default Amphora and Covered urn at Draft, Standard and Fine (it was 18 to 34
  with the fillet as first made, 0 before it), and at the extremes it is at the level
  of the version before the fillet (Amphora, reach 1, flatness 1.8: 40 at thickness
  1.5 and 34 at 2.4, where it was 41 and 36 before the fillet and 78 and 88 with it;
  reach 10, flatness 0.5, thickness 2.4: 8, before 0).
- *Lids* are a third turned part: a domed, flat or conical cover that rests on the rim
  with a bare-clay flange inside the mouth, and a finial (ball, bud, acorn or disc) on a
  short stem. A conical lid is turned to a rounded apex, and the cap sweeps up into the
  stem in a small fillet. The lid rides a hair above the lip, which reads as a dark seat
  line, and with *Hand of the maker* above zero it is set down a fraction of a millimetre
  off centre and a fraction of a degree out of level (seeded). A lobed mouth gets a lobed
  lid; a waved rim cannot carry one. Under a lid the upper joint of a handle stays below
  the lip, clear of the brim.
- At its buried tips a wide strap narrows to what the round wall can hide, so the root of
  a very wide, thick handle no longer folds where it leaves the wall. Checked over 162
  handle builds on the Standard mesh (Amphora and Covered urn; reach 1, 4.4 and 10 cm;
  flatness 0.5, 0.78 and 1.8; thickness 0.5, 1.5 and 2.4 cm; blend 0, 0.85 and 1): in
  none does an end of a strap show outside the wall. Faces turned inside out where they
  stand clear of the wall (a quad that looks at its own strap's centre line) remain in 8
  of the 162, 219 quads in all, every one at reach 1 cm with flatness 1.8 and blend on
  (37 to 57 a build with the 2.4 cm strap, 4 to 12 with the 1.5 cm one). That is where
  the version before the fillet stood (8 builds, 237 quads, the same corner): the
  fillet as first made had brought it to 4 builds and 84 quads but tore visibly there,
  and falling back to the old skirt gave the count back. Seen at four times
  magnification on the Amphora those joints are the clean, heavy collar they were
  before. The test knows the turned wall only, not the warp of *Hand of the maker*.

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
into a prefiltered environment. A softbox is a lamp with a shape, not a glow: a crisp
face with rounded corners, shaded by its own lip, the inner baffle showing through the
diffuser, on a black stand; at *Reflection detail: High* the seams of its panels show
too. Each face is sized to send the same light as the feathered panel of earlier
versions, so the exposure of every rig is unchanged. The *Atelier softboxes* rig has two low fills, warm and
cool, so the underside of a belly and a foot reflect lamps and not a black floor, and a
key softbox with a gentler core that keeps colour in its reflection. The close shots
(Lip, Foot, Detail) open up the exposure by a fraction of a stop while they are held;
that is part of the view, not of the parameters, and the exposure returns with Hero or
when the camera is moved.

- *One key light.* The spotlight that casts the shadow stands where the main softbox of
  the room is, so shadow and brightest reflection agree and turn together with *Light
  rotation*. The glaze takes its highlight from the reflected softbox, not from a point
  glint (*Spotlight glint* brings one back; *Spotlight at main lamp* off frees the aim).
- *Soft shadow.* The key shadow hardens towards contact: crisp at the foot and the plinth
  edge, wider and softer as it falls across the floor (*Shadow softness* is the angular
  size of the light). The shadow map is fitted to the piece and redrawn only when the
  piece or the light moves. In the picture at rest the key lamp is a true area light:
  from the second lens sample on, each sample is lit from another point of a disc of
  that angular size with a hard shadow, and their mean is the penumbra without the
  grain of the moving frame's filter.
- *Lamp-by-lamp shade on the stand.* Most of the light on plinth and floor is image
  light, which the key shadow alone does not block. Plinth and floor therefore ask, for
  each of the four strongest lamps of the rig and for the dome, how much of it the
  silhouette of the piece (body, lid and the reach of the handles, as twelve stacked
  rings) and the plinth hide, with a penumbra as wide as that lamp is large, and dim
  their light lamp by lamp. It is worked out in their shaders, with no extra pass, and
  the room capture below sees it. *Contact shading* scales it; at 0 it is off.
- *Lustre bounce.* The lit belly, and on a tall form the other swelling too, throws a
  faint light of the lustre's own colour back onto the plinth and a rough glow onto the
  floor, strongest on the side of the main lamp (*Lustre bounce*, `bounce`; 0 is off).
- *Room reflections.* The studio is captured again from the middle of the vessel, with the
  lit plinth, the floor and the shadow in it, and looked up with a parallax correction: the
  foot mirrors the plinth it stands on, the lower belly the floor, the shoulders the lamps.
  In that capture the dark floor is a veil that glows a little and thins out towards its
  rim: the low fills show through it, so a wall that looks down (the lower belly in
  *Detail*, the outside of a bowl) carries glaze colour and not black, and no hard floor
  edge is mirrored in the glaze. While *Detail* or *Foot* is held, that floor and the
  plinth glow more, as if a white card lay in front of the piece: the lower half of a
  close view of a dark lustre is no longer black. The card lies around the plinth and
  fades out well before the floor's horizon, so it leaves no line in the glaze; it is laid
  out in world units, and the capture takes the floor at the cove's size on either
  backdrop: the *Void*'s floor, five times as wide, met the room at a horizon which the
  picture itself hides in fog, a straight crease across the belly. It belongs to the view, not to the parameters, and goes when the
  shot is left. A piece whose wall leans out over the floor (an open bowl) gets a weaker
  card in every view, by how much of its outer wall looks down: under a dark floor such a
  wall mirrors nothing else and its lustre went out. Upright forms are lit as before. The
  card is not one even sheet: mirrored in a wall, an even field of light reads as matte
  paint (the outside of a Bronz bowl was a flat olive-khaki). Most of its light lies in
  eight sheets of unequal width set around the stand like spokes, with the floor between
  them left dim, scaled so that the floor sends up as much light as the even card did;
  the wall then mirrors lit shapes with edges, as the upper walls mirror the softboxes.
  Outside the close views 20 % of the card stays even. In *Detail* and *Foot* the card is
  the even sheet it was before: a tightly curved rib squeezes each sheet into a hard pale
  line down its crest (seen on the Amphora's spiral ribs in *Detail*), and neither softer
  edges, nor wider sheets, nor a softer near end removed it in the four variants tried,
  so the sheets were taken out of those two views. The same ribs on a Bowl in *Hero*
  showed no such line (one shot). The sheets are fixed to the floor and do not turn
  with *Studio rotation*. A piece without a card in a view without one is drawn exactly
  as before. The floor's capture material is the same program with or without a card, so
  entering *Detail* or *Foot* compiles nothing.
- *Self-reflections, traced.* The glaze also mirrors the piece itself (*Self-reflections*,
  `selfRefl`). For every pixel one ray is followed along the mirror direction through a
  height field of the piece: outer radius and cavity radius over angle and height, with
  their slopes and occlusion, built from the meshes whenever the shape changes (lobes,
  twist, relief, a waved rim and the lid as it sits are in it). Handles are not in it: the
  piece does not mirror its handles (their image in a wall was a hairline or a
  hard-edged patch, never a handle), though a handle's own glaze mirrors the body. A ray that starts in
  the cavity is tested against the cavity wall alone, one that starts outside against the
  outer wall alone, so each march looks for a single crossing, which is then bisected and
  closed with a secant step. The ray follows the smooth wall's mirror direction, not the
  micro-relief. No border of a reflected region is a yes or a no; each is a weight that
  runs out: under the lip of an open mouth (the image of the far rim), where the ray only
  clips an edge or pierces a rim thinner than the march's stride, where it grazes what it
  hit, and where the image is squeezed past what a pixel resolves (a shoulder mirrored in
  a waisted neck is given up where a pixel takes in most of it, sooner the flatter the
  ray meets it: under a lid's brim the far edge of that image was a dark band). Where the ray meets the piece, the room's reflection gives way to
  the light of that spot: its body colour under the room's light and the key lamp, and
  its own mirror image of the room where that is open to it (through the mouth, for a
  spot inside), of more wall where it is not. So the wall inside a bowl mirrors the
  opposite wall, a waisted neck the lip and the shoulder, and the mirror lobe no longer
  sees lamps through the pot. Where the ray is not believed (it grazes its own wall, or
  the hit is too near to place) the mirror lobe is dimmed by the profile's occlusion, as
  it is without tracing. While the picture moves the ray takes 12 steps, the same for
  every pixel, so a moving frame has no grain; each sample of a still takes 24, starts at
  another offset and leaves in another direction of the glaze's lobe.
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
  picture, in which the lustre colours are exactly those of the fired-film data.
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
  and cabochon collars are slopes instead of staircases. The Draft mesh, which is shown
  while a slider is dragged and where the page has given up triangles, used to be left
  unfiltered; it now takes the mean of the motif at the four corners of each vertex's
  patch (as many motifs worked out as before, so dragging costs no more), and its edges
  are the same slopes drawn softer. Standard and Fine build exactly the vertices they
  did.
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

## Parameters

The groups of the parameter model, as the configurator shows them:

| Group | Parameters |
| --- | --- |
| Shape, Sculpting | profile radii and heights, wall, *Hand of the maker* (hand-made irregularity), *Turned foot* (`footTurn`, 0–1), lobes, twist, relief, rings, rim waves |
| Handles & lid | handle count, joints, reach, arch, thickness, flatness, *Joint blend* (how far a handle flares and spreads into the wall), *Pulled taper* (`handleTaper`, 0–1), *Finger ribs* (`handleRibs`, 0–3); *Lid* (`lid`: None, Domed, Flat, Conical), *Lid rise* (`lidRise`), *Lid overhang* (`lidOverhang`, cm), *Finial* (`finial`: None, Ball, Bud, Acorn, Disc), *Finial size* (`finialSize`, cm) |
| Glaze | colour, clay body, thickness, dark in hollows, thin on edges and its *Break colour* (`breakTone`), runs, gloss, surface ripple, *Slump waves* (`wave`), crackle, pinholes, *Dust* (`dust`), *Handling bloom* (`wear`), *Foot grit* (`grit`), bare foot and drips |
| Lustre metals | copper, silver, gold, bismuth, paste coat, coverage |
| Firing | temperature, soak, smoke (reduction), air on cooling (re-oxidation), glaze resistance (diffusion barrier), flame marks, *Kiln mottle* (`mottle`), *Kiln specks* (`specks`) |
| Film optics | refractive index, dispersion, thickness variation, direct trims (layer depth, metal amount, clear skin) |
| Painted decoration | second-lustre pattern and how its film differs |
| Lighting | lighting rig (Atelier softboxes, North window, Vitrine spots, Kiln glow, and the low-key *Chiaroscuro*), room light, spotlight (aim, shadow softness, glint), *Lustre bounce* (`bounce`, 0–2, default 1), *Backdrop* (`backdrop`: Cove, Void), *Backdrop tone* (`backdropTone`), *Pool of light* (`pool`), plinth on/off, *Plinth material* (`plinthMat`: Lacquer, Black stone, Ebonised wood, Bronze), plinth and floor polish, *Contact shading* (ambient occlusion, the contact shadow and the lamp-by-lamp shade on the stand) |
| Camera & film | *Lens · mm* (`focal`, 24–135, default 45), *Aperture · f/* (`fstop`: 1.4 to 16, default 4), *Depth of field* (`dof`: Off, Stills, Always), *Focus height* (`focusLevel`, a level, default 0.5), *Still quality* (`stillSamples`: 16, 32, 64), *Look* (`look`: Neutral, Print, Catalogue, Nocturne), *Film grain* (`grain`), *Lens fringing* (`chroma`), *Halation* (`halation`), exposure, tone curve, bloom, vignette, *Presentation frame* (`frame`: Free, 2.39:1, 16:9, 1:1) |
| Rendering | turntable, room reflections and their detail, *Self-reflections* (`selfRefl`, on by default; needs room reflections), mesh detail, pixel ratio, adaptive quality, *Upscaling* (`upscale`: Off, Auto, Native, Quality, Balanced, Performance; default Auto), *Sharpening* (`sharpen`, 0–1, default 0.4; needs Upscaling), colour sample count |

Positions along the piece are *levels*: a share of the height, 0 at the foot and 1 at the
rim. Sizes of surface patterns (flame size, run width, variation scale, crackle cell) are
in centimetres.

*Dust*, *Handling bloom* and *Foot grit* belong to the one piece on the stand, not to a
glaze recipe: lustre presets leave them alone. A parameter set saved before handles had
a taper is restored with even, smooth straps, as it was drawn then. A whole set saved
before the film had flame fronts, halo rings, fine streaks and the scatter lobe is
restored without them, and one saved before the kiln left mottle and specks has neither
(`mottle` 0, `specks` 0). A whole set saved before the foot was turned keeps its plain foot
(`footTurn` 0) and its own glaze line and drips.

### Views

Along the foot of the plate are five views, worked out from the piece as it is, and
*Present*.

- **Shots** (keys 1 to 5). *Hero*: three-quarter view, 45 mm at f/4. *Profile*: side-on
  and level, 105 mm at f/8. *Lip*: from above into the mouth, or onto the finial of a
  lidded piece, 70 mm at f/8. *Foot*: low, on the foot ring and its reflection, 60 mm at
  f/2.8. *Detail*: close on the middle of the relief band, or of the body where there is
  no relief, 85 mm at f/4; on a wall that leans out (a bowl) it takes in the whole band
  from low down, turned so that two repeats of the motif stand side by side. A shot sets lens, aperture and focus height as one undo step
  and flies the camera there; the three close shots hold the turntable until the vessel
  is dragged or another view is chosen (a plain click or tap leaves the shot as it is).
  The camera pose itself is not a parameter: undoing a shot puts the lens values back,
  releases the shot and leaves the camera where it is.
- **Present** (key P; Esc or P returns). The catalogue leaves, the piece is fitted to the
  whole window or to the *Presentation frame* with bars around it, and the caption gives
  the lustre, the form and its height, and the firing. Moving the pointer, pressing a
  key or, on a touch screen, tapping the stage brings the views back for a moment. While
  presenting they include *Save* (key S): an exposure of the picture as presented,
  cropped to its frame; a note counts the lens samples and Esc cancels. On an upright
  phone the wide frames are ignored.
- **Follow edits** (configurator only). While a control that belongs to one part of the piece is changed
  (the rim, the foot, a handle, the lid, the relief band, the glaze surface…), the
  camera eases to that part and focuses there, and about a second and a half after the
  last change it eases back. Dragging the vessel meanwhile keeps the view where it is
  put. *Follow edits* in the ⋯ menu switches this off; the choice is remembered. Preset
  chips, undo, paste and Reset do not move the view this way.

### Presets, Surprise me and undo

- **Lustre and shape tiles.** A tile restores every parameter of its group and stays
  pressed exactly while all of them have its values. The caption on the plate names
  what is on show; a varied piece is named after the lustre it came from.
- **Surprise me** is described below. Shift-click keeps the shape.
- **Undo.** Each preset and each surprise shows a short note with *Undo*; Ctrl+Z and
  Ctrl+Shift+Z (Ctrl+Y) step through the whole history.
- **Paste.** Parameter JSON pasted anywhere on the page is applied.

### The configurator

The panel with all 154 controls is not part of the page an ordinary visitor sees, and
nothing on the page leads to it. It is fetched and built only on request:

- `index.html?configurator=1` opens it as a drawer over the left of the stage;
- `eozin.configurator()` in the browser console does the same, `eozin.configurator(false)`
  hides it again.

Without it, the same actions are on the console handle `window.eozin`:
`eozin.applyParams({...})`, `eozin.undo()`, `eozin.redo()`,
`eozin.actions['Copy parameters']()` (logs the JSON when there is no clipboard),
`eozin.actions['Save PNG']()`, `eozin.actions.Reset()`, `eozin.menu['Restore shape']()`,
`eozin.menu['Restore lustre']()`, `eozin.setFollow(false)`, `eozin.takeShot('foot')`,
`eozin.setPresent(true)`, `eozin.setLang('hu')`, `eozin.makePiece(seed)`.

The rest of this section describes the drawer once it is open.

- **Simple / All.** The panel opens in *Simple*, with the thirty or so controls that change
  the piece most. *All* shows every parameter; the choice is remembered. Presets and
  pasted parameter sets also set the controls that Simple hides, and the line at the end
  of the list says how many hidden controls differ from the preset.
- **Help strip.** Pointing at a row, tapping it or tabbing into it shows what it does, its
  range and its preset value under the list, in the language of the page. Controls that cannot do anything in the
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
- **Surprise me** makes a curated random piece: one of the eight shapes with its
  proportions varied together, a relief that suits it, and one of the six lustres fired
  a little differently (metals, kiln schedule, glaze tone, flame side and seed, all inside
  bounds). A firing that comes out dull, black or far from its family is thrown away and
  another tried, and two presses in a row never give the same shape or lustre family.
  With Shift held it keeps the shape and changes the firing only. A surprise piece is a
  function of its seed and can be linked as `index.html#piece-<seed>`; such a link names
  a piece of this version of the generator and may give another piece after an update.
  Editing, a preset, undo or Reset drop the piece link.
- **⋯ menu.** *Copy parameters* puts the whole state on the clipboard as JSON. Pasting
  that JSON anywhere on the page restores it, as does *Paste parameters…*, which opens a
  text box for touch screens and for frames without clipboard access. *Save PNG* takes
  an exposure: the turntable waits, the still is gathered in full (at least 32 lens
  samples) without the page layout's offset, and the picture is saved at the size of the
  canvas, cropped to the frame while presenting in one (*Save* among the views, or key S,
  starts it there); Esc cancels. *Follow edits* and
  *Reset everything* are here too.
- **Keys.** In a number field, PageUp and PageDown move ten steps and Home and End go to
  the ends of the range. 1 to 5 take the shots, P presents and S saves the presented
  picture, except while a field is being typed in. Escape closes the menu, or else cancels an exposure, or leaves the
  presentation. On phones the drawer covers the stage and shows one folder at a time.

The catalogue holds eight shapes (two of them covered: *Ginger jar* and *Covered urn*) and six lustre presets (Zöld, Hajnal, Labrador,
Ibolya, Bronz, Gyöngyház). A preset restores every parameter of its groups, and its tile
stays pressed exactly while all of them have the preset's values. A lustre preset can be
linked directly with its hash, for example `index.html#labrador`; the hash is dropped as
soon as the glaze is edited and comes back when the edit is undone. The six are fired as
coloured films over dark glass rather than as metal plate: Zöld emerald, Hajnal copper red
(its tile says so now; it said ruby red, and the piece renders copper to orange-red)
(a shallow copper layer, about 96 nm, under a lean cover film of about 46 nm, fired cooler
than before so the colloids stay fine: the film's haze is 0.02 where it was 0.07, so less of
the reflection goes to the broad glow; a dark ruby face-on, sliding to orange-gold toward
grazing and where the film runs thin, to rose where it lies deep; its motif is the same
film a seventh thinner, gold on ruby. It does not reach green: in this model a copper film
whose cover is deep enough to turn green toward grazing turns magenta wherever it is a
little thicker, which is the candy colour the preset had before, so the travel stops at gold),
Labrador deep blue breaking towards cyan where the film is thick (its glaze pools and breaks less, so rims keep their lustre; its motif is a thinner, fuller film, plum-violet, where a deeper one fell between two orders and read slate grey), Ibolya a plum blue-violet, Bronz a bronze-gold with no gold in the paste:
copper alone under a cover film a little short of the depth where its gold turns rose, so
it goes green-gold toward grazing and where the film runs thin and takes a rose blush
where it lies deep (gloss 0.9, where it was a satin 0.8; the film is a little leaner in
metal than it was first re-fired, because the fuller one read as polished lemon-gold
brass on the tall shapes). Gyöngyház is the
exception: no metal at all, only a clear bismuth film (about 210 nm) on a pale ivory,
glossy, lightly crackled glaze. It is a white glossy pot first, with the colour in the
reflection: rose at the film's mean depth, gold and then sea-green where it runs thin,
lilac and blue where it lies deep, each sliding toward gold with the angle. Three fifths
of the film's reflection is the sharp image of the lamps and two fifths a broad sheen.
(Before the re-firing the long hot firing gave it the largest haze of the six and sent
73 % into the broad lobe, which read as matte plaster. The first re-firing sent none,
kept the film's depth even at a third of its unevenness and laid the veining on 6 %
deeper. As a pure mirror the film showed its colour only inside the images of the lamps
and the lit floor, so on a bowl the wall carried khaki, cream and pink fields with
cut-out edges: mirror images of lamps, and the veining's step in depth across a steep
part of the colour series. With the film even, nothing reached lilac or blue.) Now the
film is as uneven as it was at first (thickness variation 1), a little deeper (cover
trim 1.36, was 1.3 and 1.27), and the veining is 3 % deeper with the softest edge;
flame fronts and streaks stay turned down. Its fields still have edges that can be made
out on a bowl, about as at first. A first re-firing with a thinner film (165 nm) on a grey-blue glaze
had more gloss contrast and a wider palette, but the pot read pewter, not pearl, and
on open forms its blue-grey and mustard fields read as stains; it was taken back. The crackle is fainter and no longer gives each cell its own tint. Under so clear a film the
body would return exactly the complement of the reflection and cancel its colour, so
there (and only with no metal in the film) the glaze is tinted by the film's mean
transmission alone: an approximation that keeps the pearl's play of colour. The painted second
lustre is the same film laid on about a quarter thicker (on Hajnal a seventh thinner), so the motif reads tone on
tone, in the next colours of the series; on Bronz it is only a few per cent thicker and
leaner in metal; a quarter more would carry the gold
into magenta, and on Gyöngyház it is a soft veining a little (3 %) thicker. A lustre tile's swatch
shows what the film reflects plus what the glaze returns through it.

**Measured for the re-fired presets**, against the version before them, on completed
stills in headless Chromium on a software rasteriser (480 by 480 pixels, pixel ratio 1,
Standard mesh, 32 samples, grain 0, the page's Hero and Detail shots, baluster and bowl,
one pose, one seed, one lighting set; the target bands are judgement, not measurements of
real pieces). *Gyöngyház* (three figures each: before the re-firing, the first re-firing, now; the
ginger jar's first figure is from an earlier run of the same set-up): the body's median
luminance is 0.53, 0.51, 0.51 on the baluster Hero, 0.53, 0.52, 0.52 on the ginger jar
Hero, 0.39, 0.39, 0.37 on the bowl Hero and 0.43, 0.43, 0.43 on the bowl Detail.
Brightest (99.5 %) to median luminance 1.63, 1.74, 1.70 on the baluster, 1.64, 1.70,
1.68 on the ginger jar, 2.19, 2.25, 2.25 on the bowl Hero: about half of the first
re-firing's small gain in gloss contrast on the closed forms is kept, and the lamp
images are still mirror images over a sheen, not a satin glow alone. Median saturation
of the body on the bowl Hero 0.20, 0.25, 0.22 (the khaki of the first re-firing was
the 0.25). Share of the bowl Hero's body that is coloured (saturation over 0.10) and
pink 0.31, 0.19, 0.38; gold 0.61, 0.78, 0.55; blue and violet together 0.015, 0.000,
0.021; mint 0.003, 0.000, 0.001: the cool colours are back in about the measure they
had, the mint is not. On the baluster Hero pink 0.22, 0.16, 0.26, gold 0.57, 0.76,
0.57, blue and violet 0.011, 0.000, 0.008. A stepped, hard edge on the neck of the
baluster and the collar of the ginger jar in the first re-firing was the mirror image
of two softboxes that stand one above the other on the left (the side lamp and the low
fill): it stayed with self-reflections off, on the Fine mesh, with a larger capture of
the room, without depth of field and with every field of the film switched off, and
went when those two lamps were put out. The sheen dilutes it to a soft band; it is not
removed. Not measured again: the two random pearl pieces (0.40 and 0.48 median
luminance at the first re-firing). *Hajnal*: mean hue 345 to
9 degrees, magenta-pink area 0.37 to 0.05, orange-gold 0.005 to 0.12 on the baluster Hero
(0.008 to 0.06 on the bowl), saturation of the brightest 1 % 0.26 to 0.71 (the lamp image
is orange-gold, not washed pink), median luminance 0.030 to 0.021. Its centre-to-rim shift
is 5 degrees toward orange and its hue span (90 %) 45 degrees: red to gold, no green, and
the clipped share of the baluster Hero is unchanged at 0.055 (0.092 on the Detail). Its share of
strongly saturated pixels (saturation over 0.8 with value over 0.3) **rose**: 0.23 to
0.37 on the baluster Hero, 0.23 to 0.42 on the bowl Hero, 0.43 to 0.61 and 0.73 to 0.87
on the two Details, 0.26 to 0.42 on the amphora and 0.37 to 0.39 on the tulip; with the
candy pink gone the red is a purer red. A trial with less smoke (reduction 0.85) only
moved the baluster Hero from 0.37 to 0.34 and made the piece darker, so it was not
kept. The motif is still a flat gold fill inside a hard outline. *Bronz* (as it is now,
with the leaner film): yellow-olive plus green area 0.007 to 0.05 on the baluster Hero
(0.09 with the fuller film of the first re-firing), red-rose 0.15 to 0.25, the
centre-to-rim shift 12 to 15 degrees toward green (rim hue 40), the highlight edge 3.1 to
2.5 %, the hue span (90 %) 40 to 50 degrees, mean hue 30 as before, median saturation
0.59 to 0.62. The share of strongly saturated pixels, 0 before, was 0.03 on the baluster,
0.10 on the covered urn and 0.09 on the amphora with the fuller film and is 0.00, 0.01
and 0.01 now. The bowl Hero with the film as it is now: median luminance 0.038, share of
strongly saturated pixels 0.00, mean hue 32 degrees. The look into the bowl (the Lip
shot) was measured with the fuller film only (olive and green 0.006 to 0.06, and a hue
span of 25 to 37 degrees) and not again.
*The other three.* Zöld, Labrador and Ibolya were not retuned, but the glassy skin, the
mottle and the specks are in the shader for every preset, so they moved too: on the
baluster Hero the median luminance went 0.057 to 0.051 (Zöld), 0.020 to 0.016 (Labrador)
and 0.010 to 0.008 (Ibolya, the darkest before and darker now; the share of its body
under 0.02 went 0.70 to 0.77), and the saturation in the rim band rose (Zöld 0.45 to
0.61, Labrador 0.69 to 0.84, Ibolya 0.52 to 0.72, Bronz 0.32 to 0.48): the pale line at
the silhouette is gone, where a real glaze keeps a pale rim in its last degrees. The
default picture (Zöld, 400 by 360) is 33.7 dB from the one before.
*Ribs at the silhouette*
(Zöld, Detail, 360 by 360, 16 samples): in the band 2 to 10 pixels inside the outline the
mean saturation went 0.19 to 0.30 on the amphora and 0.25 to 0.36 on the covered urn, and
the share of that band paler than 0.2 went 0.55 to 0.19 and 0.35 to 0.08; the band is
still paler than the interior (0.71), part of which is the lens blurring it into the
backdrop. *Cost*: the glaze frame, 12 interleaved pairs twice, 300 by 290, traced
reflections on: the median of the pair ratios was 0.98 and 1.01 on the default piece, 1.02
and 1.00 on the bowl. A default load still links 32 programs with 23 live (33 and 24 at
reduced scale) and makes the same requests; CPU time to the third frame over three
alternating loads each was 82.5, 68.3 and 82.8 s before and 83.2, 82.7 and 79.7 s after,
which is inside the spread of the runs. At 460 by 420 the time to the curtain lifting,
measured in the page over nine alternating loads each, had a median of 6.69 s before
(5.83 to 7.96) and 7.11 s after (6.52 to 7.63): the first four pairs separated, the
next five did not, so a slowdown of about 5 % there is neither shown nor excluded; at
390 by 700 (four loads each) it was 6.34 s before and 6.23 s after. The glaze frame was
compared at 300 by 290 only; on a large stage (923 by 800) at Quality (0.77) a moving
frame measured 2 to 4 % dearer than before (pair medians 1.026 and 1.022, n = 3 and 8),
with Balanced and Performance level (0.99 to 1.01, n = 8). *Mesh build*: the handle
fillet and the Draft relief filter are mesh code. The Standard build (the least of five
rebuilds, two alternating runs, 220 by 220 page) took 193 ms on the Amphora and 181 ms on
the Covered urn against 204 and 194 ms before, and 153 against 151 ms on the baluster,
which has no handles: no slower within the spread of the runs (single figures vary by
20 % and more between runs).

**Adaptive quality** keeps the frame rate usable on slower GPUs: it first draws fewer
pixels (with *Upscaling* on Auto, the default, the scene is drawn at 0.77, then 0.67,
then 0.5 of the canvas, which keeps its resolution, and the picture at rest is rebuilt
in full: see *Reconstruction*; with Upscaling Off the canvas itself shrinks in steps of
0.82 down to half and stays stretched, as it always did; the fixed modes skip this
step), then gives up stage extras one at a time (1: the plinth mirror at lower
resolution with fewer blur taps and half the lens samples of a still; 2: the mirror, the
blur and the traced self-reflections of moving frames, and two of the four lamps that
shade the stand; 3: a smaller shadow map, 8 lens samples, no traced self-reflections and
no area-light shadow in stills; 4: a smaller studio capture, and the stand shaded by the
dome alone, without bounce), then
falls back to a lighter mesh, and tries the finer setting again from time to time, in
the reverse order. With Upscaling on Auto a lowered scale makes the picture at rest take
longer to finish than the shrunk canvas did (more frames: see *Reconstruction*). Fewer
pixels buy time only where the pixels are what a frame costs, so each step of the drawn
scale is checked: if the next eight moving frames are not at least a tenth shorter than
the eight before the step, the step is taken back and the ladder goes on with the stage
extras from the scale above it (until *Quality* or *Upscaling* is chosen anew, or the
canvas grows by more than a fifth in pixels). Intervals enter that mean cut off at 10 s;
the cut-off used to be 2 s, which made every step look useless where frames take longer
than that. On a slow device bound by its triangles the ladder therefore reaches the
stage extras sooner than it did: in one run in the software rasteriser (460 x 420,
turntable running) rung 3 after about four minutes, where the earlier ladder was still
lowering the scale and its frames had grown slower. In the
software rasteriser this page is tested in that is what happens at the first step: see
the frame times under *Reconstruction*. A change of the drawn scale links no program, and rungs 1 to 3 are
uniform values only. Rung 4 is not, although this file used to say so: the smaller
studio capture changes the defines of the materials that read it, and nine programs
are linked again on the way down and nine more on the way back (measured: 32 linked
after load, 41 after rung 4, 50 after returning; the number of live programs does not
change). That was so before upscaling existed and is recorded here, not fixed. On a
phone-sized screen the traced
self-reflections are kept for the picture at rest from the start. The
frames that refine a still are not counted as a measure of speed. A scene in which
nothing moves is not redrawn at all once its still is complete.

**Start-up.** A preconnect to the CDN and the import map sit in the head, followed by
thirteen `modulepreload` links: the whole three.js module graph of the page, flat, so that it is
fetched in one wave while the HTML is still arriving, not in two (the second used to wait
for the first to be parsed). The list has to follow the three.js version; a browser
without `modulepreload` ignores it, and the configurator's lil-gui is still fetched on
demand. The caption is written before the first layout measurement, so the first frame
is drawn into the final view and no refit follows it. The programs of the picture are
handed to the driver before the mesh is built (`warmPrograms`: one `renderer.compile` for
the picture and one for the state of the room capture, the glaze against a stand-in of
the capture's shape), so a driver that compiles on its own threads works while the main
thread makes the geometry. The room capture parks the fog beyond its far plane where it
used to remove it, which saved compiling the stand and the unseen piece a second time:
a load links 32 programs where it linked 34, and 23 are live where 25 were. The loading
curtain lifts when the GPU reports the first complete frame finished (a fence sync
behind it; two drawn frames count as well, should a driver never signal), not after up
to seven frames, and a still's samples are drawn in sight, not behind the curtain: the
tick that lifts the curtain draws nothing itself, so the browser can paint the lift
before the next frame is worked on, and the opening move is clocked from the tick after
it.
Measured only in headless Chromium on a software rasteriser (460 by 420 pixels): one
frame drawn before the curtain where there were seven, the curtain up after about 9 s
where it took about 29 s (the time of the change in the page; the next animation frame
followed 5 ms later, where a frame of several seconds used to come first). On a real GPU the gain is expected to be about one frame;
no real-GPU timings exist.

Two changes in the glaze shader are exact, not approximations. What the film optics
need per colour sample and not per pixel (the squared indices of glass and cover,
4π/λ, and the Clausius-Mossotti term of each metal in the glass) is worked out once on
the CPU when the material changes, in single precision and in the shader's own order of
operations, which removes three complex divisions per colour sample and pixel. And
noise whose result is multiplied by zero or lands in a saturated step is not asked:
the two glaze-line noises farther than 0.16 cm from the line, the streak noise when
streaks are off or the pixel is too coarse to show them, each orange-peel octave once
it has faded out, and the run, thickness and flame fields when no parameter that uses
them is non-zero. With both, the resting picture was identical pixel for pixel to the
one before in every comparison made (default view, a view turned 45 degrees, flame,
streaks and runs each alone, all off, all on; Cove and Void). When the room is captured
again and the shadow map is not due in the same frame, the piece is left out of the
capture, not drawn unseen through its six faces; an edit from a control asks for the
shadow map as well, so this applies only to a capture that was put off to a later frame.

### Reconstruction

The page can draw the scene at a reduced scale while the canvas keeps its full
resolution, and rebuild the full picture from several frames. This is a temporal
super-resolution with a sharpening step, written for this page: jittered frames, a
history at canvas resolution, reprojection from depth. No neural network and no
downloaded weights are involved, and it is not DLSS, FSR or any vendor's upscaler and
does not match one. Its limits on the lustre in motion are measured below.

Two parameters in the configurator's Rendering group control it: *Upscaling* (`upscale`)
and *Sharpening* (`sharpen`, 0 to 1, default 0.4; greyed out with Upscaling Off or
Native, where it can do nothing, and without effect in Auto until the ladder has
lowered the scale).

| `upscale` | Scale the scene is drawn at | Adaptive ladder | Picture at rest |
|---|---|---|---|
| Off | 1; a lowered rung shrinks the canvas itself, as before | resolution of the canvas (steps of 0.82 down to half), stage extras, mesh | exactly as before this feature |
| Auto (default) | starts at 1; the ladder lowers it to 0.77, 0.67, 0.5 | drawn scale first, then stage extras, then mesh | at scale 1 as Native; below it rebuilt at full resolution |
| Native | 1 always | stage extras, then mesh | with lens blur off: finished with up to 16 jittered samples (the still's sample count if that is lower; none from stage rung 3, where it is a single frame again); with lens blur on: the still as before |
| Quality / Balanced / Performance | 0.77 / 0.67 / 0.5, fixed | stage extras, then mesh | rebuilt at full resolution |

With the defaults (Auto, lens blur on) nothing changes until the ladder lowers the
scale: the completed still is the same picture, pixel for pixel, as before the feature
existed, and the history targets are not even allocated. A device that cannot draw
into the history targets falls back to Off for the session, with one console warning:
the page then behaves as with Off whatever `upscale` holds, the parameter itself is left
as it was (so a copied parameter set does not carry one device's fallback to another),
and the configurator greys out both controls and says that upscaling is not available
on this device.
The gathering pass is one more program, linked the first time the scene is drawn below
scale 1 (24 live and 33 linked where a default load has 23 and 32); a load with the
defaults links none.

**At rest.** Every frame is moved aside by a part of a drawn pixel: in motion by a
short Halton cycle, in a still by the points of a (0,2)-sequence in base 2 (Sobol'),
none used twice, so the frames of a still cover the drawn pixel evenly at every count.
At half scale, where each drawn pixel is exactly four canvas pixels, the frames go in
fours: one place within the canvas pixel, given to each of the four canvas pixels in
turn, so every canvas pixel gathers the same 16 evenly spread places. At a reduced scale each canvas pixel keeps a running mean
of the nearest samples that fell on it, each weighted by how close it fell
(exp(-4 d²), d in canvas pixels; exponents of 6, 8 and 12 were measured at 0.67 and
0.5, and 1.5, 2.2 and 3 at 0.5 only: 6 gained 0.07 dB at 0.67 and lost 0.27 at 0.5, 3
was level at 0.5, the others were worse). The still takes as many more frames as there are fewer
pixels in each (27, 36 or 64 frames for 16 full frames' worth at 0.77, 0.67, 0.5; with
lens blur on, the still's own sample count divided by the square of the scale, at most
64), with the lens samples, the area-light shadow and the traced self-reflections of a
still as before, and then nothing is drawn. Two details keep such a still clean and
repeatable. A canvas pixel is given its sample by one frame in a few, so a lamp that
moved with every frame showed each canvas pixel another part of itself, and the soft
shadow came out with a pattern from pixel to pixel (first a two-pixel comb, then, with
the lamp on Halton bases of its own, a fainter dither). The area light now keeps its
point for as many frames as it takes to serve every canvas pixel: the four frames of a
place at half scale, where each canvas pixel then gathers exactly the 15 lamp points a
full-resolution still does, and two frames half a drawn pixel apart at the other
scales, where the canvas pixels have no fixed place in the drawn one and the cure is
partial (in a test penumbra, 320 x 280, shadow softness 4, the floor without the stand, the spread between the canvas-pixel phases of the high-passed luminance fell from 1.30 to 0.25 levels of 255 at 0.5, where a full-resolution still shows 0.23; at 0.67 from 2.20 to 1.65 against 0.72, with the row and the column patterns alone at the full-resolution figure; at 0.77 from 3.02 to 2.11 against 1.23; the shadow's PSNR against the full-resolution still rose from 42.1 to 45.8, from 42.0 to 45.1 and from 42.1 to 45.0 dB). Four frames to a lamp point at 0.67 removed the pattern too, but
left 8 lamp points, and the penumbra showed them as contour lines; it was not kept.
And what was
gathered before the still began (the moving picture, or the previous still) is weighted
down by 0.25 on the first frame at rest and, together with the four frames gathered on
top of it, to a sixteenth on the fifth, so a 64th of its weight is left: the picture
goes over into the still without a hard cut,
and the completed still is the same whatever preceded it (measured: at 0.5, 320 x 280, at most 1/255 apart after a reset and after an orbit out and back, on 496 and 723 of 89,600 pixels). The first
four frames of the still count for little in return; they take the last places of the
sequence, so the frames that count are its first. A redraw that is asked for while the view, the pose of the piece, every parameter
and every uniform of the glaze and of the finishing pass are as the complete still was
gathered with (a parameter edit that changes nothing, a script's redraw, the tab shown
again) puts the still back on the canvas as it is: one frame in which nothing of the
scene is drawn, not counted into the still (measured: at 0.67, one frame, the still's count unchanged, 0 of 144,000 pixels different, also after a lost and restored context; a changed exposure, a moved camera and a uniform written from the console each started the still anew). Before, such a redraw
showed one coarse frame and gathered the still again. Any real change starts it anew, as before.

The price of the full-resolution rest picture is time. On a lowered rung the canvas
used to shrink and the still took its usual number of frames, each cheaper; now it takes
1.7 to 4 times as many frames of about that drawn size, each followed by a gathering
pass at full resolution, so a slow device reaches its finished picture, and an idle
GPU, correspondingly later. Upscaling Off restores the short still.

At scale 1 with lens blur Off, a resting
picture used to be a single frame; with Upscaling on (Auto, Native) it is now finished
like a still without an aperture through the existing accumulation: 16 samples, or the
still's own count where the ladder has lowered that below 16 (8 with Still quality 16
on stage rungs 1 and 2), and not at all from stage rung 3, where a resting picture is a
single frame as before. An
exposure (Save PNG) is always drawn at full scale. Away from scale 1 the glaze's
footprint estimate is scaled by the drawn scale on the frames of a still, so the still
resolves the canvas pixel, not the drawn one.

**In motion.** There is no velocity buffer: each pixel's world position comes from the
depth buffer (nearest of 3×3) and is carried to the previous frame, inside the piece's
bounding cylinder through the turntable's previous pose as well. History is rejected
where the depth it was seen at disagrees (what was hidden a frame ago) and outside the
frame, clipped to the colour range of the new frame's 3×3 neighbourhood (mean ± 1.25
standard deviations, in YCoCg of a reversibly tone-mapped colour), and blended with the
new frame by how near its sample fell. Away from the piece the history may grow to 8
frames' worth. On the piece it may only grow to the number of frames the view of the
piece (camera and turntable together) takes to turn by 0.1 degrees, which at any
visible speed is a fraction of one frame: the lustre's colour depends on the view, and
a history carried across a turn is a picture of another colour. The outline of the
piece is the exception. Where the drawn pixels around a canvas pixel are part piece and
part something at least a few per cent farther away, the colour is mostly a matter of
how much of each a drawn pixel caught, and a drawn pixel whose centre fell off the
piece is shaded from beyond its edge, pale: single frames at half scale showed a beaded,
stepped outline. There the history may grow to 4 frames' worth. It is looked up twice,
once carried with the surface and once as if the piece had not turned about its axis
against the camera (the outline of a turned form stands still while its surface passes
under it), and of the two the one nearer the new frame's colour is taken, so a handle's
edge goes with its surface. In the frames examined the beads went and the outline is even (figures below); what it costs is two more history lookups on those pixels only. No trail was seen on the handles of a turning two-handled piece in the two frames looked at (turntable at 1.2° and 0.2° a frame), which is a look, not a measurement.

**Sharpening** is contrast-adaptive (the RCAS lobe), applied in the output pass to the
rebuilt moving picture only, and clamped channel by channel, in scene-linear values, to
the range of the five texels it reads, so edges steepen with next to no overshoot (in
the one moving frame examined, 9 of 144,000 displayed pixels left the local range of
the unsharpened frame by more than 2/255, by 4/255 at most: the tone curve mixes the
channels after the clamp; that frame's PSNR was 37.2 dB sharpened and 37.1 dB not). It
fades out over the first
half of a still and is zero in the completed picture, in Off and at scale 1; it is never
written into the history.

**Measured** in headless Chromium on a software rasteriser, 400 by 360 pixels in
presentation mode, the default piece on the Draft mesh with traced self-reflections on,
no grain, turntable stopped. PSNR against a reference of 16 jittered full-resolution
frames without aperture at the same pose; "lustre" is a box on the piece's wall only.
One piece, one view, one size: these are not general figures, and no real-GPU timings or
pictures exist.

| Picture | PSNR |
|---|---|
| one full-resolution frame (the resting picture before, lens blur off) | 42.0 dB (38.7 before the lustre change) |
| Auto or Native at scale 1, lens blur off, at rest | identical to the reference |
| Off with the canvas lowered to 0.67 and stretched by the browser | 36.1 dB |
| Quality (0.77) at rest, 27 frames | 46.3 dB (45.6 with the Halton places) |
| Balanced (0.67) at rest, 36 frames | 45.8 dB (44.0 before the lustre change; then 43.4 with the Halton places and the eight-frame fade, same run) |
| Performance (0.5) at rest, 64 frames | 41.3 dB (39.2 before the lustre change) |
| Performance, one frame upscaled bilinearly (no history) | 35.4 dB (33.9 before the lustre change) |
| Performance, 8 frames at rest after an orbit of 8 degrees | 37.4 dB (36.4 with the eight-frame fade, same move; an earlier round measured 38.1 after another move) |
| Balanced at rest, Void backdrop / plinth hidden (floor mirror) | 43.2 / 42.8 dB (one frame: 38.4 / 39.0); measured with the Halton places, not again |

The four rows with two figures were measured again after the lustre change (the glassy
skin, mottle and specks alter the default piece: it has less of the pale rim highlight
and is easier to rebuild), same procedure, one run; the file before it reproduces the
earlier figures exactly (44.03, 39.21, 38.71, 33.90). Every other row of this table and
all of the moving table below are from before that change and were **not** measured
again, so they are about 1.5 to 3 dB conservative if the gain carries over, which is
not known.

| Moving (sharpening off) | rebuilt | one frame, bilinear |
|---|---|---|
| orbit 0.25°/frame, 8 frames, Performance, whole picture | 34.9 dB | 33.9 dB |
| the same, Balanced | 37.6 dB | 35.8 dB |
| the same, Balanced, lustre only | 34.4 dB | 34.3 dB |
| orbit 1°/frame, 8 frames, Performance, lustre only | 31.3 dB | 31.0 dB |
| turntable 0.2°/frame, 16 frames, Balanced, whole picture | 37.3 dB | 35.9 dB |
| the same, lustre only | 34.4 dB | 34.0 dB |

The rows above were measured before the outline kept a history and were not all
measured again. With it, in one run at half scale (10 frames at 1°/frame and 8 at
0.25°/frame, ending on the reference pose; "outline" is a strip 35 pixels wide over the
left and over the right edge of the piece):

| Moving, Performance, sharpening off | before | with the outline's history | one frame, bilinear |
|---|---|---|---|
| orbit 1°/frame, whole picture | 34.2 dB | 34.5 dB | 34.0 dB |
| the same, left / right outline | 30.0 / 32.3 dB | 30.4 / 33.0 dB | 29.9 / 31.8 dB |
| the same, lustre only | 29.0 dB | 29.0 dB | 29.0 dB |
| orbit 0.25°/frame, whole picture | 34.6 dB | 34.9 dB | 34.0 dB |
| the same, left / right outline | 30.6 / 32.6 dB | 30.9 / 33.5 dB | 29.9 / 31.8 dB |
| turntable 0.2°/frame, 16 frames, Balanced, whole picture | 37.2 dB | 37.4 dB | 36.7 dB |
| the same, left / right outline | 34.4 / 34.6 dB | 34.7 / 35.2 dB | 32.6 / 33.1 dB |
| the same, lustre only | 32.9 dB | 33.1 dB | 32.1 dB |

**Frame time in motion**, the figure the reduced scale exists for, was not measured
before. Same software rasteriser, default piece on the default (Standard) mesh, lens
blur off, sharpening 0.4, an orbit of half a degree per frame; each mode in turn, the
order reversed every round, seconds per moving frame as the mean of the rounds (and
their range):

| Window (stage) | scale 1 | 0.77 | 0.67 | 0.5 |
|---|---|---|---|---|
| 460 x 420 (460 x 235; moving frames not traced), 4 rounds of 2 frames | 3.50 (3.26 to 3.88) | 3.80 (3.41 to 4.46) | 3.86 (3.65 to 4.17) | 4.13 (4.00 to 4.40) |
| 1300 x 800 (923 x 800; moving frames traced, 12 steps), 3 rounds of 1 frame | 5.49 (5.37 to 5.69) | 4.87 (4.28 to 5.39) | 4.20 (4.13 to 4.32) | 4.06 (3.91 to 4.33) |

**In this rasteriser a reduced scale is not cheaper on the small window: it is 9, 11
and 18 % dearer**, because a frame there is bound by the mesh's triangles and the
processor, and the gathering pass at canvas resolution comes on top. On the large
window it is 11, 24 and 26 % cheaper, far less than the pixel counts (59, 45, 25 % of
the pixels) suggest, and the step from 0.67 to 0.5 buys 3 %. A GPU that is bound by
its fragment work should gain more; none has been measured. This is why the ladder
now checks each step of the scale and takes back one that did not shorten the frame
by a tenth (see *Adaptive quality*): in a run with adaptive quality on (400 x 360, Draft mesh, turntable running) the ladder stepped to 0.77 after its first eight frames at 1.48 s a frame, found 1.43 s, went back to scale 1 and went on with stage rungs 1 to 4. The rule has not run on a real GPU, and its tenth is a choice, not a measurement.

**Fewer triangles in motion (built, measured, switched off).** The small window is bound
by triangles because a turned mesh has cells well under a drawn pixel there. The page
can now make coarser index lists over the same vertices (every 2nd, 4th and 8th
column, rows merged up to the same spacing, never across the edge of foot, wall, lip or
interior) and draw a frame in motion at a reduced scale from the coarsest whose cells
stay under 1.25 drawn pixels where the wall comes nearest the camera. The frames of a
still, an exposure, the shadow map, the room capture and every frame at scale 1 keep
the full list; the lists are made only when the first such frame asks for them, so a
mesh costs nothing more to build. It is **off** in this file (`sr.lod`;
`eozin.sr.lod = true` in the console switches it on for a session), because it missed
one of the three conditions it was to be accepted on:

- *Cheaper than scale 1 in motion: met on the small window.* 460 x 420 (stage 460 x 235),
  default piece, Standard mesh, lens blur and adaptive quality off, half a degree of
  orbit per frame. The page as it loads, where the masthead and the caption leave the
  piece about 65 pixels tall: interleaved pairs on two pages, 8 pairs each, medians:
  scale 1 3.09 s a frame against 0.21 s at 0.5, and 3.20 s against 0.21 s at 0.67;
  without the lists 0.5 took 3.52 s and 0.67 3.25 s (the file as it was against this
  one, same procedure). The wall is drawn from every 8th column there, 15,600
  triangles of 583,440, and a frame draws 65,135 in all passes. With the catalogue
  hidden so that the piece fills the stage (about 170 pixels tall), one page, lists
  switched on, off and on again, 4 to 6 frames each: scale 1 2.6 s (2.47 to 2.70);
  0.67 with the lists 0.95 s (0.90 to 1.05), without 2.6 s; 0.5 with the lists 0.89 s
  (0.86 to 0.97), without 2.8 s; triangles per frame in all passes 1,200,815 without,
  286,135 at 0.67 and 239,855 at 0.5 with (the wall from every 2nd column, 190,320,
  and the plinth mirror from a coarser list still). On 1300 x 800 (stage 923 x 800,
  catalogue hidden) the cells are not under a drawn pixel and nothing changes at 0.67
  (3.57 s with, 3.64 s without); at 0.5 only the mirror takes a coarser list (3.08 s
  with, 3.11 s without). A Bowl that fills the small stage keeps the full list at
  0.67. Software rasteriser only; no GPU has been measured.
- *No jump of the outline above a pixel on coming to rest: met as far as measured.* With the piece
  drawn as a flat mask, a frame in motion with the lists and one without, same view,
  differ in 58 to 181 pixels along the outline of a mask of 9,000 to 16,000 (Baluster
  at 0.5 and 0.67, Covered urn at 0.5; 7 comparisons), with one 2 x 2 block of differing
  pixels in one of the seven and none in the rest. A Bowl at 0.67, where both frames
  use the full list, differs by as much (160 pixels). Against the still that follows,
  a frame in motion with the lists differs in 92 to 164 pixels with no 2 x 2 block
  (4 comparisons), one without them in 88 to 147 with two such blocks in one of two.
  Six frames after a 5 degree move, though, the frame with the lists was 284 to 380
  pixels from the still with 1 to 3 such blocks (3 runs), and no frame without the
  lists was taken at that moment to set beside it (a Bowl on the full list: 306 pixels,
  no block). So the outline moves by about a pixel on coming to rest with the lists or
  without, which is the gathering pass's own play; whether the lists add a second pixel
  in a few places right after a fast move is not settled by this test.
- *A still at rest identical to the pixel with the lists on or off: not met.* Two runs
  with the same frames before and after (460 x 420, scale 0.5, a 5 degree move, then 64
  frames at rest) differ by one 8-bit step in 112 of 193,200 pixels, because a still
  gathered at a reduced scale keeps a faded trace of the last picture in motion (see
  above), and that picture was drawn from other triangles. Two runs with the lists on
  are identical to the pixel when the frames before them are the same, and differ by
  one step in 225 pixels when they are not, so the stills already depend on their
  history by more than on the lists; but the condition was identity, and it fails.

The pictures in motion with and without the lists are 40.7 dB apart (one frame after a
5 degree move, stage only); the decibel figures in the tables above were all measured
without them. The ladder's rule for taking back a step of the scale is unchanged, and
with the lists off so are all the frame times above.

So at rest the method works (Balanced ends well above a single full-resolution frame;
Performance was about level with one, and is 0.7 dB below one on the current shader), and in motion it helps silhouettes, the stand
and the backdrop by one to two decibels. **On the lustre in motion it does not
reconstruct anything**: it is level with a plain upscale of the current frame, by
design. A longer history there was tried and is worse: with 0.75 degrees of turn allowed,
the turntable's lustre fell to 31.3 dB against 34.0 for the plain upscale. Fine relief
and thin-film fringes therefore still shimmer between jittered low-resolution frames
while the piece turns, and settle only when it rests. Of the designers' two bounds missed on
this piece, one is now met and one is not known: Performance at rest was 39.2 dB where
39.5 was asked and is 41.3 dB on the current shader; 8 rest frames after a move were
37.4 dB where 39 was asked, measured before the lustre change and not again. What separates the half-scale still
from the reference lies on the lustre, along the borders of highlights and of the
painted fields (99 % of the squared error is inside the piece's bounding box, 30 % in a
box on its wall), not on outlines or the backdrop; neither the places of the samples
nor the width of the gathering kernel moves it by more than a tenth of a decibel. The
bloom and the stand's mirror are drawn at the reduced scale and were not examined as
causes.

Other known limits. The stand's mirror image of the piece moves with the mirrored piece
while the stand's surface stands still, and only the colour clip protects it (the stand
is rough, so the image is soft). Bloom is computed from one jittered low-resolution
frame and can flicker on pin highlights. The lens fringing reads its red and blue
unsharpened; where the fringe is under a pixel they take the sharpening of the texel
they are drawn at, farther out they stay unsharpened. Lens blur *Always* in motion at a
reduced scale was compared with the full-resolution moving frame once (35.7 dB, no
trails seen) and not otherwise examined. A specular anti-aliasing of the mirror lobe on
moving frames (roughness widened by the screen-space change of the normal) was built
and left out: it lowered the frame-to-frame difference on the lustre by 1.4 % where
10 % was the bar, and cost 0.33 dB. What is in instead is narrower: on the lip alone, a
frame that is not a sample of a still takes the film's response at the mean viewing
angle of the pixel (from the screen-space change of the cosine), because the crest of a
far rim drawn thinner than a pixel otherwise breaks into pale dashes. On the one frame
measured (Bowl, Bronz, Hero, 440 x 480, software rasteriser) the dashes went (in a second
check, *Lip* view at 380 x 420, two pale dashes were left at the top of the far crest,
the difference is plain only under magnification, and the stair-steps of the crest are
unchanged), 423 of
211,200 pixels changed by more than 8 levels, all in the rim's rows, and on the Baluster
184, all on its lip; the crest is then dimmer than the finished still draws it (RMS
difference to the still in the rim's band 6.7 before, 8.8 after), so the rim brightens a
little as the picture comes to rest. Samples of a still do not use it. With *Upscaling*
Off and lens blur Off (a combination only the configurator offers) the picture at rest
is itself such a single frame and keeps the dimmer crest. Silhouettes away
from the lip, lid seats and handles are not filtered and can still show a beaded pale
edge in a single frame. Index lists of lower detail for moving frames were
not built. Firefox, Safari and every real GPU are untested.

## Limits of the model

The firing model is a plausible reduced-order description, not a calibrated kiln
simulation: its constants are chosen so that ordinary lustre firings (roughly 550–700 °C,
10–60 minutes) give films in the range reported for historical lustres. Maxwell-Garnett
theory ignores particle-size effects and is stretched at the highest metal fractions.
At those fractions (Bronz, for instance) the lustre layer is opaque, so its
thickness and the glaze colour beneath only show where the film thins: on relief edges,
in flame-lean patches and in pinholes. The fired-film data say whether the film is opaque.
Colours are computed for a D65-like white and are clipped to sRGB.

The traced self-reflection is one bounce against a height field of body and lid:
undercuts are filled in, relief is as fine as the map (128 x 256
cells, 64 x 256 at Draft), and the spot that is
hit is shaded with the film colour of the spot that reflects it, not its own. Handles
are not mirrored at all: a neck beside a handle shows the room where the handle's image would be; a
thin overhanging rim is mirrored at rest sooner than in motion; a second bounce is an
estimate, not a ray. The shade
on the stand treats the piece as a body of revolution (an oval squash is followed and
turns with the piece; lobes and lean are not), so two handles shade it as a ring
would. Softbox shapes, the card and the bounce are stage dressing, not measurements.
Relief on a lower wall that mirrors the stand and the dark floor (the vines low on a
Ginger jar in Labrador) stays a dull, even colour in the Hero view: nothing lit and shaped
lies where it looks, and no card is laid for an upright form. At Draft mesh detail the
edge of a relief is now filtered like Standard's (see *Relief filtering*) and reads as a
softer slope, but a few steps remain along the shaded upper edge of a cabochon. At
Standard the thin pale line of a cabochon's collar on a bowl is still finely serrated
when seen close: the relief's edge is carried by the mesh alone, not by the shader.
