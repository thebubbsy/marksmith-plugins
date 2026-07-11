# Plugin manifest specification (`plugin.json`)

**Manifest version: 1.** This document is the authoring contract; the C# models it must stay in sync with live in Marksmith's `MdToPdf.Core/Plugins/PluginManifest.cs`.

A plugin is one JSON file. Comments and trailing commas are tolerated by the parser, but keep committed manifests clean-JSON so other tools can read them.

## Top level

| Field | Type | Required | Meaning |
| --- | --- | --- | --- |
| `manifestVersion` | int | yes | Always `1` for this spec. |
| `id` | string | yes | Lowercase, stable, unique. Doubles as the install folder name (`…\MdToPdf\Plugins\<id>\`). |
| `name` | string | yes | Display name shown in Settings → Plugins. |
| `description` | string | yes | One or two sentences shown under the name. Say what it renders **and** what it downloads (size, runtime) — users decide from this text. |
| `version` | string | yes | Your plugin's own version (semver recommended). |
| `homepage` | string | no | Upstream project URL. |
| `license` | string | no | License of the payload you download (e.g. `"MIT (plantuml-mit)"`). |
| `type` | string | yes | `"diagram"` is the only type today: fenced code block → SVG. The field exists so future types (importers, exporters, theme packs) can reuse this format. |
| `fenceLanguages` | string[] | for `diagram` | Code-fence languages you claim, lowercase (e.g. `["plantuml", "puml"]`). First come, first served within a running app: built-ins win over user plugins, and a language already claimed is skipped with a visible warning. |
| `runtime` | object | no | A managed runtime the host provisions for you. See below. |
| `artifacts` | object[] | no | Files to download at install. See below. |
| `render` | object | for `diagram` | How to invoke your tool. See below. |

## `runtime`

```json
"runtime": { "kind": "jre", "majorVersion": 17 }
```

- `"jre"`: the host downloads a private Eclipse Temurin JRE (via Adoptium's official API, correct OS/arch automatically) into `<plugin>/jre/`. Reference its `java` executable as `{java}`.
- `"node"`: the host downloads the current Node.js LTS (via nodejs.org's official dist index) into `<plugin>/node/`. Reference its `node` executable as `{node}`. (`majorVersion` is ignored — Node always provisions the newest LTS.)
- Never assume Java, Node, or anything else is already on the user's machine.

## `artifacts[]`

Each artifact is one downloaded file, optionally extracted.

| Field | Type | Meaning |
| --- | --- | --- |
| `name` | string | Target filename inside the plugin folder (or the archive's temp name when `extract: true`). |
| `os` | string? | `"windows"` / `"linux"` / `"mac"` — omit for all platforms. List per-OS variants side by side; each machine downloads only its own. |
| `arch` | string? | `"x64"` / `"aarch64"` — omit for all. |
| `source` | string | `"url"`, `"github-latest"`, or `"npm"`. |
| `url` | string | For `source: "url"` — direct download link. |
| `sha256` | string | Hex digest verified after download; mismatch aborts the install and deletes the file. **Required for registry-listed direct-URL artifacts.** |
| `repo` | string | For `github-latest` — `"owner/name"`. |
| `assetPattern` | string | For `github-latest` — regex matched against release asset names; first match wins. |
| `package` | string | For `npm` — the package name. Installed via the plugin's provisioned Node runtime's own npm into `<plugin>/npm/node_modules/<package>` (requires `runtime: { "kind": "node" }` and pulls the package's whole dependency tree — use for JS tools that aren't distributed self-contained, e.g. wavedrom-cli). |
| `packageVersion` | string | For `npm` — exact version. Strongly recommended: pins the tree. |
| `extract` | bool | Treat the download as a `.zip` / `.tar.gz` / `.tar.xz` and extract into the plugin folder. |
| `stripRoot` | bool | With `extract` — remove a single wrapping top-level directory (the common `tool-1.2.3/` layout). macOS AppleDouble junk (`._*`, `.DS_Store`) is ignored during extraction. |

On Linux/macOS, non-archive artifacts and extracted files referenced as the render command are marked executable automatically.

## `render`

```json
"render": {
  "command": "{java}",
  "args": ["-jar", "{dir}/plantuml.jar", "-tsvg", "-pipe"],
  "input": "stdin",
  "output": "stdout",
  "timeoutSeconds": 20,
  "wrap": { "prefix": "@startuml\n", "suffix": "\n@enduml", "unlessContains": "@start" }
}
```

| Field | Meaning |
| --- | --- |
| `command` | Executable to run. Placeholders: `{java}` / `{node}` (the provisioned runtime's executable), `{dir}` (the plugin's folder). On Windows, a command without an extension gets `.exe` appended automatically if that file exists — so `{dir}/d2` works cross-platform. |
| `args` | Argument list (each element is one argument — no shell, no quoting games). Placeholders: `{java}`, `{node}`, `{dir}`, plus `{input}` / `{output}` / `{outputBase}` (temp file paths, only meaningful with the file modes below; `{outputBase}` is `{output}` minus its `.svg` extension, for tools like LilyPond that take an output basename and append the extension themselves). |
| `input` | `"stdin"`: diagram source is written to your process's stdin. `"file"`: source is written to a temp file whose path replaces `{input}` in args. |
| `inputExtension` | With `input: "file"` — the temp file's extension, dot included (default `".txt"`). Tools like Typst (`.typ`) and D2 (`.d2`) enforce or sniff the extension. |
| `output` | `"stdout"`: your process prints the SVG to stdout. `"file"`: you write it to the path that replaces `{output}`. |
| `timeoutSeconds` | 1–120 (default 20). The process tree is killed on expiry and the render is treated as failed. |
| `wrap` | Optional convenience: prepend `prefix` / append `suffix` to the fence content before it reaches your tool, unless the source already contains `unlessContains`. Use for tools that demand delimiters users habitually omit (PlantUML's `@startuml`). |

### Output contract

- Produce a well-formed `<svg …>…</svg>`. The host extracts the first `<svg` to the last `</svg>` — leading log lines on stdout are tolerated but don't rely on it.
- `<script>` elements are always stripped from your output before it touches the document. Don't emit active content.
- Syntax errors in the user's diagram: exit non-zero or emit nothing. The app then keeps the plain code block and shows a "couldn't render" note. **Never** emit an SVG containing an error screenshot-of-text unless that's genuinely your tool's UX.

### Execution environment

- Working directory = the plugin's folder.
- No shell involved; your `command` is executed directly.
- One process per (plugin, unique-diagram-source); results are cached by content hash for the app session, so don't rely on per-invocation side effects.
- Renders can happen concurrently and repeatedly (live preview re-renders as the user types). Keep startup as cheap as your stack allows.

## `import` (for `type: "importer"`)

Importer plugins convert non-Markdown files to Markdown when the user opens or drops one — the
first non-diagram plugin type. Reference: `plugins/pandoc-import/plugin.json`.

```json
"type": "importer",
"import": {
  "extensions": ["rst", "org", "docx"],
  "command": "{dir}/pandoc",
  "commandLinux": "{dir}/bin/pandoc",
  "commandMac": "{dir}/bin/pandoc",
  "args": ["{input}", "-t", "gfm", "--wrap=none"],
  "timeoutSeconds": 120
}
```

| Field | Meaning |
| --- | --- |
| `extensions` | File extensions claimed, lowercase, no dot. First installed importer claiming an extension wins. `.md`/`.markdown`/`.txt` can't be claimed — they always read raw. |
| `command` | As in `render`. `commandWindows` / `commandLinux` / `commandMac` override it per-OS, for tools whose archive layout differs by platform (Pandoc: `pandoc.exe` at the zip root on Windows, `bin/pandoc` in the tarballs). |
| `args` | `{input}` is the user's **real file path** (no temp copy — tools like Pandoc infer the input format from the extension). Print Markdown to stdout; exit non-zero on failure (the app then falls back to reading the file raw). |
| `timeoutSeconds` | 1–300, default 60. |

## Lifecycle & expectations

1. **Discovery** — at app start, Marksmith loads built-in manifests, then scans `…\MdToPdf\Plugins\*\plugin.json`. Folder name must equal `id`; ids collide → the drop-in is skipped with a visible warning (built-ins can't be overridden).
2. **Install** — user clicks Install: runtime first, then artifacts in order, with progress. Any failure leaves the plugin "not installed"; installs must be idempotent (re-install overwrites).
3. **Render** — every matching code fence in preview and PDF export goes through your `render` spec. DOCX/PPTX/EPUB export does not yet embed plugin diagrams (roadmap).
4. **Uninstall** — the plugin folder is deleted recursively. Keep *all* your state in it.

## Versioning this spec

`manifestVersion` bumps only on breaking changes; additive optional fields keep version 1. Unknown fields are ignored by the host, so you may annotate manifests with `x-`-style extra keys at your own risk.
