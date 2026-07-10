# Marksmith Plugins

Community plugin registry for [Marksmith](https://github.com/thebubbsy/marksmith) — the app that turns AI-chat Markdown into polished PDF/DOCX/PPTX/EPUB documents.

Plugins add **optional, separately-downloaded capabilities** to Marksmith. Nothing in this repo is bundled with the app: each plugin's payload (renderer binaries, runtimes) downloads only when a user clicks **Install** in *Settings → Plugins*, into its own isolated folder.

## Available plugins

| Plugin | Fences | Platforms |
| --- | --- | --- |
| [PlantUML](plugins/plantuml/plugin.json) | ` ```plantuml `, ` ```puml ` | Windows, Linux, macOS |
| [Graphviz / DOT](plugins/graphviz/plugin.json) | ` ```dot `, ` ```graphviz ` | Windows, macOS (Intel) |
| [D2](plugins/d2/plugin.json) | ` ```d2 ` | Windows, Linux, macOS |
| [Typst](plugins/typst/plugin.json) | ` ```typst ` | Windows |
| [Vega-Lite](plugins/vega-lite/plugin.json) | ` ```vega-lite `, ` ```vegalite ` | Windows, Linux, macOS |

All five ship built into Marksmith's *Settings → Plugins* list (payloads still download only on install); the copies here are the canonical manifests. Every listed plugin is verified: real install, real render, on at least Windows. See [IDEAS.md](IDEAS.md) for what's next and where help is wanted.

## What a plugin is

A plugin is a single `plugin.json` manifest — no C#, no compilation. The manifest declares:

1. **Identity** — id, name, description, version, license, homepage.
2. **What it claims** — which fenced-code-block languages it renders (e.g. ` ```plantuml `).
3. **What it downloads** — artifacts (binaries/jars/archives, per-OS variants side by side) and optionally a managed runtime (currently a private Java JRE, provisioned automatically from [Adoptium](https://adoptium.net)).
4. **How to run it** — a subprocess command template: diagram source goes in (stdin or temp file), SVG comes out (stdout or temp file).

Read the full authoring spec in [SPEC.md](SPEC.md). The machine-readable schema is [schema/plugin.schema.json](schema/plugin.schema.json).

## Installing a plugin manually

1. Create a folder: `%LOCALAPPDATA%\MdToPdf\Plugins\<plugin-id>\` (Windows) or `~/.local/share/MdToPdf/Plugins/<plugin-id>/` (Linux/macOS — the app's local-app-data equivalent).
   The folder name **must equal** the manifest's `id`.
2. Drop the plugin's `plugin.json` into it.
3. Restart Marksmith. The plugin appears in *Settings → Plugins*; click **Install** to fetch its payload.

## Writing your own

Start from [`templates/my-diagram-tool/plugin.json`](templates/my-diagram-tool/plugin.json) — a commented skeleton. The canonical worked example is [`plugins/plantuml/plugin.json`](plugins/plantuml/plugin.json), which is byte-for-byte the manifest Marksmith ships built-in for PlantUML.

Rules of the road (details in SPEC.md):

- **Offline after install.** Everything your renderer needs must be a declared artifact or the managed runtime. Never assume anything on the user's PATH, and never phone home at render time.
- **SVG out.** Your tool must produce an `<svg>…</svg>` document. `<script>` elements are stripped by the host regardless.
- **Fail cleanly.** Bad diagram syntax → non-zero exit or empty output; the app falls back to showing the code block. Renders are killed after `timeoutSeconds` (default 20).
- **Pin your bytes.** Direct-URL artifacts in registry-listed plugins must carry a `sha256`. (`github-latest` artifacts float by design — the tradeoff for auto-tracking upstream releases — so prefer pinned URLs when the upstream project versions its releases sanely.)

## Submitting to the registry

Open a PR adding `plugins/<your-id>/plugin.json` plus a line in [registry.json](registry.json). Checklist:

- [ ] `plugin.json` validates against the schema
- [ ] Folder name == manifest `id`; id is lowercase, unique across this repo
- [ ] Direct-URL artifacts have `sha256`
- [ ] Renderer + all artifacts are redistributable under their stated licenses
- [ ] You've run it locally end-to-end (install → render → uninstall)

## Plugin ideas — help wanted

See [IDEAS.md](IDEAS.md) for a curated list of plugins we'd love to exist, with suggested implementation notes for each.

## Security model (read before installing anything)

A plugin is **arbitrary code execution by design** — its manifest tells Marksmith to download files and run them as a subprocess whenever a matching code fence renders. Only install plugins from sources you trust. Registry PRs are reviewed, but review is not a sandbox: treat third-party plugins with the same suspicion as any software you install.
