# Plugin ideas — help wanted

Ten plugins we'd love to see. Each is a good fit for the manifest format today (subprocess in, SVG out); implementation notes point at the easiest known path. Claim one by opening an issue.

| # | Plugin | Fences | Payload & notes |
| --- | --- | --- | --- |
| 1 | **Graphviz / DOT** | ` ```dot `, ` ```graphviz ` | Official Graphviz release zips (windows) / tarballs. `dot -Tsvg` reads stdin, writes stdout — the textbook manifest. The single most-requested diagram language after Mermaid/PlantUML. |
| 2 | **D2 (Terrastruct)** | ` ```d2 ` | Single static Go binary per platform from `terrastruct/d2` GitHub releases. `d2 {input} {output}` (file/file). Modern look, great for architecture diagrams. |
| 3 | **Typst math & snippets** | ` ```typst ` | Single Rust binary from `typst/typst` releases. `typst compile --format svg {input} {output}`. Beautiful typeset math/figures without a LaTeX install. |
| 4 | **Vega-Lite charts** | ` ```vega-lite `, ` ```vegalite ` | `vl-convert` single binary (`vega/vl-convert` releases): JSON chart spec → SVG. Turns AI-generated chart specs into real data viz — very on-theme for Marksmith. |
| 5 | **WaveDrom timing diagrams** | ` ```wavedrom ` | `wavedrom-cli` (needs a Node runtime — motivates a `"runtime": { "kind": "node" }` addition to the spec, mirroring the JRE provisioner). Digital/hardware timing diagrams. |
| 6 | **LilyPond sheet music** | ` ```lilypond ` | Official LilyPond binaries; `lilypond -dbackend=svg`. Markdown → engraved scores. |
| 7 | **ABC music notation** | ` ```abc ` | `abcm2ps -g` (tiny C binary) → SVG. Much lighter than LilyPond for simple tunes. |
| 8 | **LaTeX / TikZ** | ` ```tikz `, ` ```latex ` | Tectonic (single-binary LaTeX engine) + `dvisvgm`. The heaviest payload here but unlocks the entire TikZ ecosystem. |
| 9 | **Kroki bridge (local)** | ` ```blockdiag `, ` ```seqdiag `, ` ```nwdiag `, … | A locally-run Kroki container/binary exposes ~25 diagram languages behind one endpoint. Needs a "long-lived local server" render mode — a spec extension worth designing. |
| 10 | **Pandoc importer** | n/a (importer) | Not a diagram renderer: `type: "importer"` converting .rst/.org/.mediawiki/.docx → Markdown on file-drop. The first exercise of a second plugin `type` — spec design welcome. |

## Spec gaps these surface (PRs welcome)

- `runtime: { kind: "node" }` — a managed Node.js provisioner (needed by #5, useful for many JS-based tools).
- A long-lived local-server render mode with lifecycle management (needed by #9).
- A `type: "importer"` contract (needed by #10).
- PNG output for tools that can't emit SVG (`"output": "stdout-png"`), rasterized into the document.
