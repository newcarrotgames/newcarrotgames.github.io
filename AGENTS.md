# newcarrotgames.github.io

Personal GitHub Pages site (user site, served from `main` at the repo root). Pushing to `main` deploys in about a minute.

## Layout

- `galaxy-rotation/index.html` — Galaxy Rotation Lab, served at https://newcarrotgames.github.io/galaxy-rotation/.
  One self-contained page: three.js r128 from cdnjs, Google Fonts, everything else inline.
- `socket-gems/index.html` — Dregora Socket Gems, a player guide to the Socketed gems on the Dregora Parasited
  Minecraft server, served at https://newcarrotgames.github.io/socket-gems/. Google Fonts, everything else inline.
- No build system and no landing page at the root (the bare domain 404s).

## Dregora Socket Gems: source and publishing

The working source is `D:\dregora\nightmare\player_guides\socket_gems.html` (Windows machine), an HTML fragment.
`socket-gems/index.html` wraps it like the Galaxy page, plus a small reset the fragment relies on
(`[hidden]{display:none!important}` is required, or the gem filters stop hiding cards):

```bash
{ printf '<!doctype html>\n<html lang="en">\n<head>\n<meta charset="utf-8">\n<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">\n<style>body{margin:0}img{max-width:100%%}[hidden]{display:none!important}</style>\n</head>\n<body>\n'; cat /d/dregora/nightmare/player_guides/socket_gems.html; printf '\n</body>\n</html>\n'; } > socket-gems/index.html
```

The gem data is hand-written from the pack's `config/socketed/gemtypes/*.json` and `gemcombinations/*.json`
(Oct 21 2026 #2 release). Re-check it against those files after a pack update.

## Galaxy Rotation Lab: source and publishing

The working source is `/home/dave/dark/galaxy-rotation.html`, an HTML fragment with no doctype/head/body.
`galaxy-rotation/index.html` is that fragment wrapped in a minimal document. Edit the source, then rebuild:

```bash
{ printf '<!doctype html>\n<html lang="en">\n<head>\n<meta charset="utf-8">\n<meta name="viewport" content="width=device-width, initial-scale=1">\n</head>\n<body>\n'; cat /home/dave/dark/galaxy-rotation.html; printf '\n</body>\n</html>\n'; } > galaxy-rotation/index.html
```

Keep the two in sync; never edit only `index.html`.

## Code structure (inside the one `<script>`)

- `galaxyModel()` — Milky Way mass model shared by the page and the worker: bulge, gas, Einasto halo, MOND,
  rotation curves, escape speed, starting disk motions. **Must stay self-contained**: the worker is built from its source.
- `physicsWorker()` — runs in a Web Worker made from `galaxyModel.toString() + physicsWorker.toString()`.
  Holds the particle-mesh self-gravity solver (128×128 grid, FFT), the leapfrog integrator, and trail sampling.
  **Must not reference anything outside itself except `galaxyModel`.**
- Main IIFE — generates the star field from the seeded RNG (`mulberry32(20261001)`), draws with three.js,
  sends the worker one `advance` at a time, and draws whatever positions come back. `gen` counts restarts so
  stale replies are dropped. The RNG stream is shared with the halo cloud, so changing how many random numbers
  the star setup draws changes the picture.
- The Bullet Cluster section further down is independent of the galaxy simulation.

## Physics notes worth knowing before changing things

- Disk stars carry the stellar disk's mass; bulge, gas and halo are smooth and fixed. Tracer stars are massless.
- Each galaxy's grid is rebuilt every `PM.DT` (4 Myr) from positions drifted ahead to mid-interval. Without that
  drift, the frozen field lags the rotating bar and drains angular momentum (disks shrank by about half in 2 Gyr).
  At 8 Myr results are similar but further from a 10× finer reference run.
- Starting disk motions target Toomre Q = 1.5 with asymmetric drift; random draws are capped at 2σ so no star starts unbound.
- With self-gravity on, all three disks form bars; the Newton-only disk spreads and loses a few stars even in
  "Compare speeds". This is physical (Ostriker & Peebles 1973) and the page text says so.
- Numbers quoted in the page text (curve peaks, MOND shortfall, halo mass) come from the model. Recompute them after changing the model.

## Data sources

- Measured circular speeds `MW_VC`: Ou et al. 2024, MNRAS 528, 693, Table 1 (arXiv:2303.12838).
- Visible matter: de Salas et al. 2019, as listed in Ou et al. Table 2.
- Halo: Einasto best fit from Ou et al. Table 3 (M0 = 0.62e11, rs = 3.86 kpc, alpha = 0.91).

## Testing

No test suite. Changes were checked in headless Chromium driven by `playwright-core`
(browser at `~/.cache/ms-playwright/chromium_headless_shell-1243/`, Node via `~/.nvm/versions/node/v22.23.1/bin`),
loading a scratch copy of the page with a debug hook added. Headless WebGL renders in software at about 8 fps,
so frame-rate numbers from it are meaningless; time the worker's replies instead.
Check: no page errors, chart draws, both modes and the self-gravity toggle work, phone width (390 px) has no horizontal scroll.

## Git

Commits are authored as `newcarrotgames <newcarrotgames@users.noreply.github.com>` (repo-local config), not the
global work identity. Push over SSH; `gh` is installed at `~/.local/bin/gh` and logged in as newcarrotgames.
