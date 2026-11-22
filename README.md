# TwinFinder

Electron app that scans a folder (and its sub-folders) for duplicate **photos and videos**.

## Run it

```bash
npm install
npm run dev      # hot-reloading development window
npm start        # build the UI, then run the app
npm run dist     # installer for your OS (output in ./release); Windows setup uses a guided wizard
```

Needs Node 20+. `sharp`, `ffmpeg-static` and `ffprobe-static` ship their own binaries — nothing else to install.

## Icon

`build/icon.svg` is the master design. `npm run icons` renders it to `build/icon.png` (1024 px) and `build/icon.ico`
(16–256 px); the installers and the window use those. Edit the SVG and re-run the command to change the look.

## Languages

English, Nederlands, Deutsch, Français and Español. The app starts in your system language (falling back to
English) and remembers the language you pick in the selector (top right). Plurals, number formats and dates follow
the chosen language.

To add a language: copy `src/i18n/locales/en.js` to e.g. `it.js`, translate the values (keep the `{placeholders}`
and the `_one` / `_other` plural pairs), then add it to `LOCALES` and `LANGUAGES` in `src/i18n/index.jsx`.
Missing keys fall back to English, so a partial translation never shows blank text.

## How detection works

| Layer | Finds | How |
| --- | --- | --- |
| Exact duplicates | Byte-identical files | size → first/last 64 KB fingerprint → full SHA-256 (most files drop out early, so big videos are rarely read in full) |
| Look-alike photos | Same picture, different size / format / compression | 64-bit dHash of a 9×8 greyscale thumbnail, hashes ≤ N bits apart match |
| Look-alike videos | Same footage, re-encoded or scaled | length within ±2 %, then dHash of 5 frames (10/30/50/70/90 %) compared |

Byte-identical files are analysed only once and fanned back out to all their copies.

## Safety

- Nothing is removed without a confirmation dialog, and files go to the **system Trash** (restorable).
- At least one file of every group always survives, enforced in the UI and re-checked before trashing.
- The window can only show / open / trash media inside folders you scanned (`twin://` protocol allow-list in `electron/main.js`).

## Layout

```
electron/
  main.js        window, IPC, twin:// protocol, trash/reveal/open
  preload.js     the small, explicit API exposed to the page
  scanWorker.js  runs the scan on a worker thread
  scanner.js     discovery, hashing, clustering (the core logic)
  thumbs.js      cached thumbnails (sharp for photos, ffmpeg for videos)
src/             React + Tailwind + framer-motion interface (src/i18n = translations)
build/           app icon (SVG master + generated PNG / ICO)
scripts/         make-icons.js
```

## Known limits

- HEIC/HEIF files are found as exact duplicates but can't be compared as look-alikes (sharp's prebuilt binaries can't decode them).
- Solid-colour images are skipped for look-alike matching (they'd all "match" each other).
- Cropped or mirrored copies are not detected as look-alikes.
- `asar` is disabled in the packaged build so the worker thread and native modules load without unpacking tricks.
