# Plugin ideas — status & help wanted

The original ten-plugin wishlist, updated. Five are **shipped** (in `plugins/`, built into Marksmith, verified live: real install → real render). Five are **blocked on spec capabilities** — each is a well-scoped contribution to the host engine; claim one by opening an issue.

## Shipped ✅

| Plugin | Fences | Notes |
| --- | --- | --- |
| **PlantUML** | ` ```plantuml `, ` ```puml ` | Private JRE + plantuml-mit.jar; Smetana layout (no Graphviz needed). All platforms. |
| **Graphviz / DOT** | ` ```dot `, ` ```graphviz ` | Official 15.1.0 zips, checksum-pinned. Windows + macOS-Intel (upstream publishes no portable Linux build; macOS-ARM ships only as .pkg). |
| **D2 (Terrastruct)** | ` ```d2 ` | Single official binary, all platforms via github-latest. |
| **Typst** | ` ```typst ` | Single official binary. Windows-only until the installer supports `.tar.xz` (Linux/macOS release format — see gaps below). |
| **Vega-Lite** | ` ```vega-lite `, ` ```vegalite ` | Official `vl-convert` binary, all platforms. JSON chart spec in, SVG chart out. |

## Blocked on spec/engine gaps 🔧

| # | Plugin | Fences | What it needs |
| --- | --- | --- | --- |
| 1 | **WaveDrom timing diagrams** | ` ```wavedrom ` | A `runtime: { kind: "node" }` provisioner (upstream ships `wavedrom-cli.js`, a single bundled JS file — trivial once Node can be provisioned like the JRE is). |
| 2 | **LilyPond sheet music** | ` ```lilypond ` | An `{outputBase}` placeholder — LilyPond takes an output *basename* and appends `.svg` itself. Official binaries exist for all platforms. |
| 3 | **ABC music notation** | ` ```abc ` | A reliable binary distribution of `abcm2ps` (upstream publishes no release binaries), or a build-from-source story. |
| 4 | **LaTeX / TikZ** | ` ```tikz `, ` ```latex ` | Tectonic downloads TeX packages over the network *at first render*, violating the offline-after-install rule — needs either a "network allowed at render" manifest flag (with UI disclosure) or a pre-warmed package cache at install. Also needs `dvisvgm` as a second pipeline stage (`renderPipeline` — multiple commands). |
| 5 | **Kroki local bridge** | ` ```blockdiag `, ` ```seqdiag `, … | A long-lived local-server render mode with lifecycle management (start on first render, health check, stop on app exit). Unlocks ~25 diagram languages in one plugin. |
| 6 | **Pandoc importer** | n/a | A second manifest `type: "importer"` (file-drop → Markdown conversion) — the manifest format was designed to extend this way; the app-side contract needs defining. |

## Engine gaps these surface (PRs to the main app welcome)

- `runtime: { kind: "node" }` — managed Node.js provisioner mirroring the JRE one (#1).
- `.tar.xz` extraction support — unblocks Typst on Linux/macOS (and many Rust-tool release archives).
- `{outputBase}` placeholder (#2).
- `renderPipeline` — run N commands in sequence, not just one (#4).
- A long-lived local-server render mode (#5).
- `type: "importer"` contract (#6).
- PNG output mode (`"output": "stdout-png"`) for tools that can't emit SVG, rasterized into the document.
