# Plugin ideas — status & help wanted

The original ten-plugin wishlist, updated. **Eight are shipped** (in `plugins/`, built into Marksmith, verified live: real install → real render/conversion). Two remain, each blocked on something specific rather than unattempted.

## Shipped ✅

| Plugin | Fences / files | Notes |
| --- | --- | --- |
| **PlantUML** | ` ```plantuml `, ` ```puml ` | Private JRE + plantuml-mit.jar; Smetana layout (no Graphviz needed). |
| **Graphviz / DOT** | ` ```dot `, ` ```graphviz ` | Official 15.1.0 zips, checksum-pinned. Windows + macOS-Intel (upstream publishes no portable Linux build; macOS-ARM ships only as .pkg). |
| **D2 (Terrastruct)** | ` ```d2 ` | Single official binary, all platforms. |
| **Typst** | ` ```typst ` | Single official binary, all platforms (Linux/macOS unlocked by .tar.xz extraction support). |
| **Vega-Lite** | ` ```vega-lite `, ` ```vegalite ` | Official `vl-convert` binary, all platforms. |
| **LilyPond** | ` ```lilypond `, ` ```ly ` | Official 2.24.4, checksum-pinned on all platforms. **Deliberately not 2.26.0**: its Windows (mingw) build hard-crashes on compile (STATUS_STACK_BUFFER_OVERRUN, reproduced consistently). Uses the `{outputBase}` placeholder (LilyPond appends `.svg` itself). |
| **WaveDrom** | ` ```wavedrom ` | Private Node.js LTS runtime + `wavedrom-cli` from npm (pinned). Upstream's "single file" release asset is **not** self-contained — it requires its npm dependency tree, which is why the `npm` artifact source exists. |
| **Pandoc importer** | .rst .org .mediawiki .textile .docx .odt .rtf .epub | The first `type: "importer"` plugin — converts files to Markdown on open/drop. |

## Remaining 🔧

| Plugin | Blocker |
| --- | --- |
| **LaTeX / TikZ** (` ```tikz `, ` ```latex `) | Tectonic downloads TeX packages over the network *at first render*, violating the offline-after-install rule — needs either a disclosed network-at-render manifest flag or install-time package-cache warming. Also needs a multi-command `renderPipeline` (tectonic → xdv, dvisvgm → svg) and a reliable standalone `dvisvgm` distribution. All solvable; a real design decision about the offline promise comes first. |
| **Kroki local bridge** (` ```blockdiag ` etc.) | Needs a long-lived local-server render mode (start/health-check/stop lifecycle). Honest finding that lowered its priority: Kroki's standalone jar natively covers mostly languages Marksmith already renders (PlantUML, ditaa, ERD); the blockdiag/mermaid/etc. families require *additional companion services* beyond the jar, so the standalone bridge unlocks less than it appears to. |

## Engine capabilities (for authors)

Shipped this round: `runtime: {kind: "node"}` provisioner, `npm` artifact source, `.tar.xz` extraction, `{outputBase}` placeholder, `inputExtension`, per-OS `command` overrides (`commandWindows/Linux/Mac`), and the `type: "importer"` contract. Still open (PRs to the main app welcome):

- `renderPipeline` — run N commands in sequence (TikZ).
- A long-lived local-server render mode (Kroki).
- A disclosed network-at-render flag or install-time cache warming (TikZ).
- PNG output mode (`"output": "stdout-png"`) for tools that can't emit SVG.
