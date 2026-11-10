# Twelve-Lead Vector Lab

M1 cardiology teaching tool: real PTB-XL 12-lead ECGs, each paired with a 3D heart vector (Kors-reconstructed VCG), a lead inspector, a pattern picker, quiz mode, and simulated Wiggers / PV-loop hemodynamics.

Originally built as a claude.ai artifact (https://claude.ai/code/artifact/2ec6dff2-1916-40fb-8d4a-91dda030cf82) and exported here for local development.

## Structure

- `index.html` — the whole app: CSS, markup, a JSON manifest (`<script id="manifest">`, record metadata + forward matrix), and all JS. Some lines are very long (one is ~43 KB); read with offset/limit or grep rather than whole-file.
- `data/<category>.json` — tracings for each pattern category (normal, afib, lbbb, wpw, …), fetched on demand as `data/${r.c}.json`.
- `data/heart.json` — simplified Z-Anatomy / BodyParts3D heart mesh (~60k triangles).

## Measurements (PTB-XL+)

- `data/features.json` — per record (keyed by ecg_id) GE 12SL values from PTB-XL+ v1.0.1: `pr`, `qrs`, `qt`, `qtc` (ms), `pax`/`tax` (P/T frontal axes), `ha` (atrial rate), `R`/`S`/`ST` (mV, 12 leads in app order I…V6, S as a positive depth), `pA` (P amp in II, P' amp in V1), `pD` (P duration II), `f` (P on, P off, QRS on, QRS off, T end in ms into the median beat). Loaded at boot (`FEAT`, `feat(r)`).
- `data/medians.json` — 12SL median beats, base64 Int16 µV, 12 leads × 600 samples at 500 Hz, lead-major. Loaded on first use of the median-beat panel. Two records (21605, 21111) have no median in PTB-XL+. Note: the PTB-XL+ WFDB headers say mV, but after gain/baseline the values are µV.
- The manifest `hr` is the 12SL ventricular rate. For 16 records the original R-peak list (`r`) counted P or T waves (e.g. every complete heart block); those lists were re-detected from the VCG and checked against the 12SL rate.
- The 12SL QRS axis column is empty in PTB-XL+, so the manifest `ax` is computed as atan2(QRS_Area_aVF, QRS_Area_I) from the 12SL features (the original stored axes disagreed by more than 30° in 21 records). With it, three LAFB records (4444, 16678, 14123) measure −32° to −37°, short of the −45° LAFB criterion.
- Beat lists were audited against an independent detector (2026-11-10): non-QRS beats removed, missed beats added, 6685 and 8177 re-detected (their old lists sat on P waves, now used as their P lists), stale P lists rebuilt. Audit report: ecg-audit.md in the session folder.

## Explanations

- `MECH` (per category): a mechanism paragraph plus a diagram spec rendered by `heartSVG()` (frontal schematic: chambers, conduction system, blocks, regions, re-entry, ectopic focus, mean-QRS inset). Shown in the condition card via `catBody` → `mechHTML`, so it also appears in quiz reveals.
- "Mark findings" (`ANN`, `buildAnn`, `drawAnn` on the overlay canvas): per-category rules that box leads, bracket intervals on the rhythm strip and mark beats, using 12SL values from `data/features.json` (now also `QD`/`QA` Q-wave duration/depth, `TA` T amplitude, `PP` QRS peak-to-peak). Hidden while a quiz question is unanswered.

## Quiz modes

All three open in a focus view (`body.focus`: tracing + question, everything else hidden, View Transitions animation, reduced-motion respected) via `setQuiz(mode)`:
- `quick` — pick the diagnosis from four.
- `guided` — seven committed steps (`GS`, `renderGuide`). Rate, PR/QRS/QT and Sokolow–Lyon voltage graded against 12SL (20 ms PR/QRS, 40 ms QT, 4 mm or 15% voltage); rhythm, chambers, ischemia against the cardiologist statements; values hidden in the header until their step is committed (`gHide`).
- `vector` (button "Axis drill") — read the limb leads and set the frontal axis on a hexaxial dial (`startDrill`, `renderDrill`), graded within 20° of `ax`. The 3D heart stays hidden in the focus view since it would show the answer.
Per-step tallies (including `aim`) persist in `localStorage` (`tlvl.guided.v1`).

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
