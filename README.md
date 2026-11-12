# Three.js Web Viewer / Runtime — Rigged Character Preview

**Verified:** 2026-11-11 · three.js **r186** · part of Workstream D / the asset-generation workflow.

The **web (Three.js) target** for the pipeline's *auto-rig to game skeletons* stage. A rigged,
animated character exported as **GLB** (skinned mesh + animation clips) loads straight into the
browser — no conversion — via `GLTFLoader` + `AnimationMixer`. This is both a **client-facing preview**
and the seed of a **web/WebGL runtime** for a playable browser game.

> Why GLB (not FBX) for web: Three.js consumes glTF/GLB natively; skeleton + animation clips travel
> inside the GLB. (Tripo quad topology only survives in FBX, but runtime GPUs render triangles, so the
> triangulated GLB is the correct web deliverable.)

## Run it

No build step. Serve the folder over HTTP (ES modules + `fetch` need `http://`, not `file://`):

```bash
cd viewer/threejs
python3 -m http.server 8080      # or:  npx serve .
# open http://localhost:8080
```

By default it loads a small CORS-friendly sample rigged character (Khronos **Fox.glb**, clips
Survey / Walk / Run) so it works immediately. Buttons for each clip are generated from whatever
animations the loaded GLB actually contains.

## Load a pipeline asset

Two ways to point it at a real output:

1. **Query param:** `http://localhost:8080/?model=./models/character.glb`
2. **Edit the default:** change `DEFAULT_MODEL` in [`viewer.js`](viewer.js) to your exported GLB.

Drop exported GLBs into `viewer/threejs/models/` (git-ignored by the repo's asset rules; keep source
assets immutable and outputs versioned under `assets/outputs/`).

## What it does

- Loads a rigged GLB; **Draco-compressed** GLBs supported (decoder from CDN).
- Builds a clip switcher from `gltf.animations`; smooth cross-fade between clips (`fadeToAction`).
- Auto-frames the model (bounding box → camera + grid), drops feet to the ground plane.
- Orbit controls (drag to rotate, scroll to zoom), hemisphere + directional lighting.
- **Turntable auto-rotate** and **wireframe** toggles (top-right) for topology review and hands-free demos.
- Defaults to an **idle** clip when present (matches the pipeline's `idle / walk / run` presets).

## How it fits the pipeline

```
Tripo rig (spec=mixamo/unreal/unity/vrm) + retarget (idle/walk/run)
      └─ export GLB ──► viewer/threejs  (web preview + runtime seed)
                        FBX ──────────► Unity / Unreal
```

See [`../../pipelines/asset-generation-workflow.md`](../../pipelines/asset-generation-workflow.md)
(Stage 3 rig/animate, Stage 4 export) and [`../../research/tripo/research.md`](../../research/tripo/research.md).

## Dependencies & notes

- three.js **r186** loaded via import map (jsDelivr CDN) — pin the version for reproducibility.
- Verified APIs (2026-11-11): import map `three` + `three/addons/`, `GLTFLoader`, `DRACOLoader`,
  `OrbitControls`, `AnimationMixer` / `clipAction` / `fadeToAction`, `renderer.setAnimationLoop`.
- Offline/air-gapped: `npm i three`, then repoint the import map at local `node_modules/three`.
- This is a **preview/runtime scaffold**, not a game — physics, input, camera rig, and netcode are out
  of scope here and belong to a later game-integration milestone.
