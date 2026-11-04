# Orange Track Lab

A single-file web app that turns a box of Hot Wheels track into a set of buildable,
physics-checked layouts with a Lego-style interactive build guide.

Open `index.html` in a browser. No build step, no dependencies.

## The four stages

1. **Your sets** — tick the official sets you own; each seeds the parts bin.
2. **Your pieces** — photograph the pile and let the scan tally it, then correct
   anything it got wrong. Counts are always editable.
3. **Layouts** — nine layouts, filtered to what your bin can actually build, each
   drawn to scale as a drafting plan and run through an energy check.
4. **Build guide** — one step per piece. A step card states the job and the exact
   parts (with support heights), a zoomed detail inset shows the piece being placed
   with elevation callouts at both ends, and the full plan beneath shows placed
   track solid, the current piece haloed, and the rest ghosted in. The last step
   is a race-ready checklist generated from what the layout actually contains.

## How a layout is modelled

Every layout is an ordered list of **segments** walked by a turtle in centimetres
(`walkLane`). One segment is simultaneously three things:

- a **drawn piece** on the plan (`drawSheet`),
- a **build step** in the guide (`stepsFor`),
- a **physics element** in the energy budget (`runLane`).

That is why the step guide and the feasibility check never disagree with the picture —
they are all reading the same array.

Segment types: `str` (straight, with drop), `arc` (radius + signed sweep), `loop`,
`cork`, `jump`, `cross`, `split`. Lanes can be `pinTo` another lane (branches hanging
off a splitter) and `interlock` layouts slide one lane until its crossing piece meets
the other's, so a shared criss-cross really is shared.

## Making it solid

Elevated track sags and twists between props, and a sagging run-in is the usual
reason a loop that "should" work does not. `supportsFor` places a prop under the end
of every piece that sits off the floor and every 45 cm along long pieces (the clamp
holds the start; a loop has its own brace). Props are drawn on the plan as diamonds
with their height in cm, counted into the kit list, and named in each step.

Two-lane layouts draw the second lane in blue — Hot Wheels sells blue track too — so
the two builds never get confused on the plan. Joints are marked, and chevrons show
the direction of travel.

## The physics

Mass cancels for gravity and rolling resistance, so every number holds for any car in
good order — only bearing friction and wheel wear vary.

- Rolling model: `v² += 2(g·sinθ − µg·cosθ)·L`, with `µ ≈ 0.06` on clean plastic track.
- Curves cost ~4.5% of kinetic energy per 90°; corkscrews ~10%.
- A loop needs `v_entry² ≥ 5·g·r`, taken at a 1.35 safety factor for friction and
  entry misalignment. Below that the car leaves the roof.
- Jumps are projectile motion off a 24° lip. The air gap is an **output**, not an
  input: `compute()` runs the physics, then redraws the gap where the car actually
  lands, so the plan's dimension and the advice always agree.
- Two-lane layouts get a fairness readout — path-length difference, predicted finish
  gap in milliseconds, and the stagger that evens it up.

Margins under 15% are reported as tight rather than passing, because a dusty track
eats about that much.

## Photo scan

Stage 2 uses the published page's `sample` capability to read a photo of the pieces
and return counts as JSON. It degrades cleanly: where the capability is absent (a
local file, a view without it), the affordance says so and manual entry carries the
whole app. The scan proposes counts; it never silently overwrites the bin.

## Accuracy

Set contents are editable starting estimates, not a manufacturer parts list — confirm
them against your own boxes. Not affiliated with Mattel; Hot Wheels and Track Builder
are trademarks of Mattel, Inc.
