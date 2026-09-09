# plane-moving-targets (aligned, asymmetric oscillation)

A Voronoi Tessellation Population (VTP) agent simulation with moving targets —
a MATLAB model and a dependency-light browser port of the same dynamics.
This is a variant of the aligned-targets version where the one oscillating
target moves **asymmetrically**: it goes down at one rate and comes back up
at a scalar multiple of that rate.

**[Live site →](#)** (enable GitHub Pages, see below)

## What it is

Agents ("cells") move under three local forces computed from their Delaunay
neighborhood:

- **repulsion** from the nearest neighboring agent
- **alignment** toward neighbors heading the same direction
- **homing** toward whichever target is currently closest

Each step is capped by an estimate of the agent's own Voronoi cell size, so a
crowded agent can never leap past its neighbors. Two constants shape that
balance and are live-editable on the *live simulation* tab (and in the MATLAB
control panel):

- **ν** — alignment strength (weight of the alignment force relative to
  repulsion + homing)
- **L** — interaction length scale (sets the distance at which repulsion/
  homing hand off, and caps how far a crowded agent can move per step)

### Target motion

You choose a number of straight-line targets (0–5), each on its own
fixed-height horizontal lane. One more target is always added: it oscillates
in y, but **asymmetrically** —

- it **descends** at angular frequency **q**
- it **rises back up** at **n·q** — n times faster than it went down

Both halves are genuine half-cosine curves (still sinusoidal, not a linear
ramp); they just run at different rates, joined continuously at the top and
bottom of the swing. `n` is itself a live-editable slider, alongside q, the
oscillation's height (center) and amplitude.

Every target — including the oscillating one — shares a single horizontal
speed and a single shared x-coordinate: there's only one `x(t)` in the whole
system, so all targets are aligned (exactly the same x) at every instant, not
just when they start.

## Structure

```
index.html              site shell — tabs for about / live simulation / matlab version
assets/style.css        site styling
assets/sim.js           the simulation engine (Delaunay neighbor graph via d3-delaunay, force law, aligned straight-line + asymmetric-oscillation target motion, canvas rendering)
assets/app.js           page wiring — tabs, sliders, MATLAB source viewer
assets/matlab_src.json  bundled MATLAB source (for the in-page code viewer)
matlab/                 original MATLAB implementation
```

## Running the web version

No build step. Either open `index.html` directly, or serve the folder:

```
python3 -m http.server 8000
```

then visit `http://localhost:8000/`.

## Running the MATLAB version

Requires base MATLAB only (`delaunayTriangulation`, `polyshape` — no extra
toolboxes):

```
cd matlab
matlab -r dynamics
```

or open `matlab/dynamics.m` in the MATLAB editor and run it. You'll be
prompted for the number of straight-line targets (0–5); one
asymmetrically-oscillating target is always added on top. An interactive
figure then opens with a control panel (agent speed, ν, L, shared horizontal
speed, each straight target's height, and the oscillating target's
height/amplitude/q/n — Apply / Pause / Reset Targets).

## Publishing to GitHub Pages

1. Create a new GitHub repository and push this folder to it (see commands
   below).
2. In the repo, go to **Settings → Pages**.
3. Under **Build and deployment**, set **Source** to `Deploy from a branch`,
   branch `main`, folder `/ (root)`.
4. Save — the site will be published at
   `https://<your-username>.github.io/<repo-name>/` within a minute or two.

```bash
cd plane-moving-targets-aligned-asymmetric
git init
git add .
git commit -m "Initial commit: VTP aligned moving-targets site (asymmetric oscillation)"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

## Implementation note: the down/up state machine

Both `dynamics.m` and `sim.js` implement the asymmetric swing as a small
state machine per oscillating target: a `goingDown` flag and a `localPhase`
that runs from `0` to `π` within the *current* half-swing, advancing at `q`
while going down and at `n·q` while going up, switching (and resetting
`localPhase` to `0`) whenever it reaches `π`.

The one subtlety worth calling out: position must be computed from whichever
half is **current after** any switch that happens that step — not the
pre-switch half — or the join between the two halves jumps by a full swing
(bottom straight to top) instead of connecting smoothly. Both
implementations separate "advance and possibly switch" from "compute
position/velocity from the current state" for exactly this reason.

## Note on the JS port

The browser port implements the same force law as `dynamics.m` (repulsion +
alignment + homing, weighted by the `expReciprocal` transition function over
the Delaunay graph), with the same live-editable ν and L, and the identical
down/up state machine for the oscillating target (ported line-for-line from
MATLAB to JS). One piece is approximated for simplicity: the MATLAB version
caps an agent's step by ray-casting its intended direction onto the exact
boundary of its Voronoi cell (`voronoiProjectToBoundary.m`); the JS version
approximates that cap as half the distance to the nearest Delaunay neighbor.
Visually and qualitatively the dynamics match; if you need the exact cap,
port `voronoiProjectToBoundary.m` into `assets/sim.js`.
