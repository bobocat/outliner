# Sift — a single-file note outliner

Sift is a tree-style note outliner that runs entirely from **one HTML file**.
Open it in a browser and you get a collapsible outline on the left and a
rich-text page for each note on the right. The whole document — every note and
its text — is saved to a single human-readable `.otl` file you can version,
back up, or hand-edit.

No build step, no dependencies, no server. Just open `outliner.html`.

## Run it

Double-click `outliner.html`, or open it in a browser.

For the best experience (in-place Save — see below), use **Chrome** or **Edge**.

## Features

- **Outline tree** with expand/collapse arrows and per-note icons (folder, doc,
  note, task) plus a "marked" flag.
- **Rich-text editor** per note: bold (`Ctrl+B`), bullet lists, and
  indent/outdent with `Tab` / `Shift+Tab`.
- **Search** the outline by title *and* body text, with matches highlighted and
  a live match count.
- **Drag and drop** to rearrange notes: drop on a row's top/bottom edge to
  reorder, or on its middle to nest it inside (parent); drag up to a shallower
  node to unparent. A node can't be dropped into its own subtree.
- **Font selector** with a set of Google Fonts for the page (falls back to
  system fonts offline).
- **Single-file save/load** in the `.otl` format, with **Save** (overwrite the
  opened file in place), **Save As**, **Open**, and **New**.
- **Importers**:
  - **OPML / XML** — full import of titles, body text, and hierarchy.
  - **`.mpl`** (Absolute Database export) — best-effort salvage of note *titles*
    only; bodies and structure are stored compressed in that binary format and
    can't be recovered, so prefer an OPML/XML export when possible.

## Keyboard shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl+S` | Save (overwrite the opened file) |
| `Ctrl+Shift+S` | Save As |
| `Ctrl+B` | Bold (in the editor) |
| `Tab` / `Shift+Tab` | Indent / outdent (in the editor) |
| `Esc` | Clear search |
| Double-click a note title | Rename it in the tree |

## The `.otl` file format

An `.otl` file is a single UTF-8 JSON document — a nested tree of notes:

```json
{
  "format": "otl-outline",
  "version": 1,
  "meta": { "title": "barbq", "font": "system" },
  "tree": [
    {
      "id": "n1",
      "title": "code notes",
      "icon": "folder",
      "marked": false,
      "expanded": true,
      "body": "<p>rich text as an HTML subset…</p>",
      "children": [ /* … */ ]
    }
  ]
}
```

Note bodies are stored as a small HTML subset (paragraphs, `<br>`, bold, and
bullet lists), which is what the editor produces.

## Browser support and caveats

- **In-place Save requires the File System Access API** (Chrome / Edge). There,
  Open keeps a handle to your file and Save writes straight back to it in the
  same folder. In browsers without the API (e.g. Firefox), or in a sandboxed
  preview that blocks it, Save/Save As fall back to a normal download to your
  Downloads folder — the app tells you when this happens.
- For privacy, the browser exposes only the file **name**, not its full path,
  so the header shows `file.otl` rather than the directory.
- The **font selector** fetches fonts from Google Fonts over the network the
  first time a non-system font is chosen; offline it uses system fonts.
- Drag-and-drop is mouse-based (desktop); it doesn't respond to touch.
- The editor uses `document.execCommand` for formatting. It works in all current
  browsers but is a deprecated API; a future rework could replace it.

## Turning this into a Windows app

There's a separate plan (`windows-app-plan.md`, if included) for wrapping this
single file in a native shell (Tauri or Electron) to get real Open/Save dialogs,
`.otl` file association, offline fonts, and an installer. The UI and `.otl`
format carry over unchanged.

## Project layout

```
outliner/
├─ outliner.html     # the entire app (HTML + CSS + JS, ~58 KB)
├─ README.md
└─ .gitignore
```

## License

See `LICENSE` if present. Otherwise all rights reserved by the author.
