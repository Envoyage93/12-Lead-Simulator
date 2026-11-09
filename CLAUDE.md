# Twelve-Lead Vector Lab

M1 cardiology teaching tool: real PTB-XL 12-lead ECGs, each paired with a 3D heart vector (Kors-reconstructed VCG), a lead inspector, a pattern picker, quiz mode, and simulated Wiggers / PV-loop hemodynamics.

Originally built as a claude.ai artifact (https://claude.ai/code/artifact/2ec6dff2-1916-40fb-8d4a-91dda030cf82) and exported here for local development.

## Structure

- `index.html` — the whole app: CSS, markup, a JSON manifest (`<script id="manifest">`, record metadata + forward matrix), and all JS. Some lines are very long (one is ~43 KB); read with offset/limit or grep rather than whole-file.
- `data/<category>.json` — tracings for each pattern category (normal, afib, lbbb, wpw, …), fetched on demand as `data/${r.c}.json`.
- `data/heart.json` — simplified Z-Anatomy / BodyParts3D heart mesh (~60k triangles).

## Running locally

The app `fetch()`es relative paths, so it must be served over HTTP (opening the file directly won't load data):

```
python3 -m http.server 8000
# then open http://localhost:8000
```

## Dependencies (CDN)

- three.js r128 + OrbitControls from cdn.jsdelivr.net
- Google Fonts: Archivo, Source Serif 4, IBM Plex Mono

## Republishing to claude.ai

The artifact host wraps the page in its own skeleton, so when republishing strip the outer `<!doctype html><html><head>…<body>` wrapper that was added at export, and publish `index.html` with every `data/*.json` file in `files`.

## Data / licensing

PTB-XL v1.0.3 (PhysioNet, CC BY 4.0); heart model Z-Anatomy / BodyParts3D (CC BY-SA 4.0). Teaching material, not for clinical use.
