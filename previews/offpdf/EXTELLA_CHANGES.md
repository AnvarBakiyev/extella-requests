# OffPDF inside the Extella window: what Extella changed (PRIVATE TEST)

OffPDF 0.4.0 by Mehmetcan Kul and the OffPDF contributors, MIT, https://github.com/McanKul/offpdf
(tag v0.4.0, commit f3ba7f2a7c066f688945f7b67037ac845eb241d3).

## Not changed
- OffPDF's Rust backend: src-tauri/src/commands, error.rs, models.rs, pdf_engine, utils are compiled
  byte-for-byte as published into `offpdf-companion`.
- OffPDF's frontend source (React, all 23 tools, every component and store) is built as published.
- The official signed and notarized `OffPDF_0.4.0_aarch64.dmg` is shipped unchanged; only the tools inside it
  (qpdf, poppler pdftoppm/pdftotext, tesseract with its language data) are run. The app itself is not started.

## Added by Extella (MIT, Chariot Technologies Lab)
- `offpdf-companion` server part (companion/src/main.rs): HTTP on 127.0.0.1 that calls OffPDF's command
  functions by name; key, Host, Origin and path checks; uploads, downloads, progress events.
- Stand-in crates for `tauri`, `tauri-plugin-dialog`, `tauri-plugin-opener` (companion/shims), so OffPDF's
  backend compiles without a webview: events go to the window, pickers and "open" are done by the window.
- Stand-ins for the four `@tauri-apps/api` modules OffPDF imports (window/shim): `invoke` -> HTTP to the
  companion; native file picker -> the browser's file input (the file is copied to the companion);
  "open file / folder" -> download; native drag-and-drop -> browser drag-and-drop.
- Start-up script (window/boot/offpdf-window.js): finds the companion, gets its key from the product's expert,
  shows Install / Start when needed; then loads OffPDF.
- installer (package/install.py) and two experts (offpdf_inside_setup, offpdf_inside_where).

## Edits to OffPDF's own files (window build only, applied by window/patch_tree.py)
1. index.html: the Content-Security-Policy also allows the companion on 127.0.0.1 and Extella
   (OffPDF's allowed only the Tauri IPC); the Extella scripts are added before OffPDF; title "OffPDF".
2. src/lib/pdfjs.ts: pdf.js font and cmap folders `./pdfjs/...` instead of `/pdfjs/...`.
3. built index.html: OffPDF's entry module is loaded by the start-up script, after the companion answered.

## Behaviour differences in the window
- Output folder: each window session has its own folder under ~/.extella/offpdf-inside/work/out; results are
  delivered by download ("Open output file" / "Open containing folder" download the file or a .zip).
- Files are copied into ~/.extella/offpdf-inside/work/in when added; work files are removed after 24 hours.
- "Open With" from Finder, the About page's mail link and Settings' "open temp folder" do nothing in the window.

## Windows build of the companion only
- heif-rs (OffPDF's vendored HEIC decoder) links Microsoft-toolchain static libraries that a MinGW cross build in
  Docker cannot link; companion/build_windows.sh swaps in companion/shims/heif-unsupported, so the Windows companion
  refuses HEIC/HEIF photos with a clear message. macOS keeps OffPDF's own heif-rs.
