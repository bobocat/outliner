# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Git workflow

Commit and push directly to the `main` branch. Do **not** create feature branches or open PRs for changes in this repo — work on `main` and push there.

## What this is

**Sift** is a tree-style note outliner that ships as a **single, self-contained HTML file** (`outliner.html`, ~58 KB, ~1300 lines). Open it in a browser and you get a collapsible outline tree on the left and a rich-text editor on the right. The entire document — every note and its text — saves to one human-readable `.otl` file (UTF-8 JSON).

There is **no build step, no dependencies, no server, no package manager, no test suite.** The app is HTML + CSS + JS inlined in one file.

## Running it

Just open `outliner.html` in a browser — double-click it, or `start outliner.html` (Windows). No install.

- Use **Chrome or Edge** for the full experience: in-place **Save** relies on the File System Access API (`window.showSaveFilePicker` / `showOpenFilePicker`), gated behind `supportsFSA`. Browsers without it (e.g. Firefox) fall back to a normal download.
- Google Fonts are fetched over the network the first time a non-system font is chosen; offline it uses system fonts.

## How to work on this file

Everything lives in `outliner.html`, in three sections:

1. **`<style>`** (lines ~29–379) — all CSS, organized by clearly labeled comment blocks (app frame, toolbar, split, sidebar, drag & drop, editor, status bar, format sheet, search, font selector, toast). CSS custom properties define the theme; keep to the existing variables.
2. **HTML body** (lines ~380–521) — the static markup. Elements are wired up by `id`; the JS looks them up with `$("#id")`.
3. **`<script>`** (lines ~523–end) — all app logic, plain vanilla JS, no framework.

When editing, **match the surrounding idiom**: terse single-line helpers, `$`/`$$` DOM selectors, short function names, section comments (`/* ---------- ... ---------- */`). No transpilation — write browser-native ES that runs as-is.

### Core data model

- The whole document is a single global `doc` object: `{ format, version, meta:{title,font}, tree:[...] }`.
- Each note node: `{ id, title, icon, marked, expanded, body, children:[] }`. `body` is a small HTML subset (paragraphs, `<br>`, bold, bullet lists, note links, web links) produced by the editor. Icons are one of `folder | doc | note | task` (see the `ICON` map).
- **Note links**: body text can link to another note as `<a class="notelink" data-note="<id>">text</a>`. Because links reference node ids, `loadOtlText` **preserves ids from the file** (minting fresh ones only for missing/duplicate ids and bumping `uid` past the highest `n<number>` seen) — don't reintroduce blanket id regeneration on load.
- `N(...)` constructs a node; `nid()`/`uid` generate ids.

### Key functions (all in the `<script>` block)

- **Tree traversal:** `walk`, `find`, `pathTo`, `parentArr`, `isSelfOrDescendant`, `descendants`.
- **Rendering:** `renderTree` → `buildLevel`, `renderCrumbs`, `refreshStatus`, `paintState`.
- **Selection / editing:** `select`, `flushEditor`, `beginRename`, `cmd` (wraps `document.execCommand`), `syncToolbar`, `updateWords`.
- **Note & web links:** `linkCommand` (Ctrl+K / toolbar — inserts a link via the picker, or removes the one under the caret), `openLinkPicker` / `buildLpList` / `chooseLink` (the searchable target picker; if the input is a URL per `urlFrom`, it offers a single web-link row instead), `linkAtSelection`, `removeLink`; an editor `click` listener follows `a[data-note]` to its note and opens `a.extlink` (`<a class="extlink" href="…">`, http/https/mailto only) in a new tab.
- **Lists & checklists:** `checkCommand` (toolbar ☑ — toggles `class="check"` on just the items in `selectedItems()`: the caret's item, or those whose own text the selection touches; a paragraph becomes a list item first), `rollChecks` (run on every editor `input` and on `select`: migrates older bodies' `ul.checklist`/`ul.bullets` to per-item classes; an unmarked item whose `parentItem` is a checkbox becomes one, unless it was switched off there, which marks it `class="plain"`; a checkbox item with checkbox children gets `locked` and its `done` is derived from them), `kidLists`/`kidItems`/`parentItem` (a child list is either inside the `<li>` or — as Chrome's indent produces — the `<ul>` sibling right after it), `liAtSelection`. The checkbox is the `<li>`'s `::before`; an editor `mousedown` left of the item's box toggles `done` (locked items just toast). Folding: `rollFolds` marks items with child lists `haskids` (drawn arrow = the `<li>`'s `::after`, further left than the checkbox) and hides the child lists of `folded` items via `class="infold"`; `toggleFold` is hit by the same `mousedown` (`foldHit` / `boxHit` split the left margin in ems). `tidyLists` = `rollChecks` + `rollFolds` and is what editor `input` / `select` call. `updateWords` counts on a clone, since `innerText` would skip folded items.
- **Search:** `computeSearch`, `hiTitle`, `updateSearchCount`, `clearSearch` — searches titles *and* body text, highlights matches, keeps ancestors expanded.
- **Dirty tracking:** `markDirty` / `dirty` flag / `paintState` drive the saved/unsaved indicator.
- **Save / load:** `otlText` (serialize), `loadOtlText` (parse), `doSave` / `doSaveAs`, `writeToHandle`, `downloadText`. In-place save keeps a `FileSystemFileHandle`.
- **Importers:** `importOpml` (OPML/XML → full titles + bodies + hierarchy via `DOMParser`), `importMpl` / `extractMplTitles` (Absolute Database `.mpl` — best-effort **titles only**; bodies/structure are compressed and unrecoverable).
- **Drag & drop:** `tree` listeners (`dragstart`/`dragover`/`dragleave`/`drop`/`dragend`) + `moveNode`, `clearDropMarks`, `endDrag`. Drop on a row's top/bottom edge reorders; middle nests as a child. A node can't be dropped into its own subtree.
- **Misc:** `applyFont` / `loadGoogleFont`, `showToast`, `togglePanel` (the slide-in `.otl` format reference).

### Conventions & gotchas

- **Formatting uses `document.execCommand`** — deprecated but works in all current browsers. The README notes a future rework could replace it; don't casually rip it out.
- Escape user/HTML content with `esc()` where building markup by hand.
- Drag-and-drop is **mouse-only** (no touch).
- Keyboard shortcuts: `Ctrl+S` save, `Ctrl+Shift+S` save as, `Ctrl+B` bold, `Ctrl+K` note/web link, `Tab`/`Shift+Tab` indent/outdent in the editor, `Esc` clears search, double-click a title to rename.

## Verifying changes

There are no automated tests. Verify by opening `outliner.html` in Chrome/Edge and exercising the affected flow by hand: create/nest/delete notes, edit and reformat text, search, drag to rearrange, and round-trip a **Save → Open** of an `.otl` file. For importer changes, test with a real OPML/XML (or `.mpl`) file.

## The `.otl` format

Single UTF-8 JSON document — a nested tree of notes. `meta` holds document title and font; `tree` is the array of root nodes. See README.md for a full example. When changing the schema, bump `doc.version` and keep `loadOtlText` tolerant of older files.

## Repo layout

```
outliner/
├─ outliner.html   # the entire app (HTML + CSS + JS)
├─ README.md       # user-facing docs (features, shortcuts, format, caveats)
├─ CLAUDE.md       # this file
└─ .gitignore      # ignores *.otl / *.mpl (personal data) and future tooling dirs
```

Note `.gitignore` excludes `*.otl` and `*.mpl` — personal outline data and import sources are **not** committed. Adjust if you intentionally want to track sample files.

## Possible future direction

README mentions wrapping this single file in a native shell (Tauri or Electron) for real Open/Save dialogs, `.otl` file association, offline fonts, and an installer — the UI and `.otl` format would carry over unchanged. No such code exists yet.
