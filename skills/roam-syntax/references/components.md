# Roam components — the {{...}} family

Every example is shown in `code` so it stays literal — type it without the backticks to make it live.

## Components

- Invoke via `/name` or by typing `{{[[name]]: arg}}`.
- Video `{{[[video]]: url}}` — YouTube / Vimeo / Loom (paste a URL and click play to auto-convert). `/video`.
- Table `{{[[table]]}}`
  - Renders nested children as a grid: each first-level child is a row, and each deeper level within it is the next cell across (columns), so the first row reads as headers.
  - Cells can hold rich blocks ("complex cells") and columns are sortable.
  - Example (nesting depth = columns; first row reads as headers):
    ```
    {{[[table]]}}
        - Action
            - QWERTY
                - Colemak
        - Select Block Up
            - k
                - h
    ```
- Kanban `{{[[kanban]]}}`
  - 1st-level children = columns (can be `[[page]]` refs); their children = cards; deeper nesting = card detail.
  - Cards are directly editable; Enter splits a card; drag-and-drop moves cards and rewrites the nesting; the board can be opened in the sidebar.
- Mermaid `{{[[mermaid]]}}`
  - Diagram source in child blocks; the first child sets the type, and any diagram mermaid.js supports works (flowchart, sequenceDiagram, gantt, stateDiagram-v2, and so on).
  - Per-diagram theme: first line `%%{init: {"theme":"forest"}}%%`. Graph-wide theme: a `{{[[roam/css]]}}` block setting `--mermaidjs-theme` (reload required).
- Diagram `{{diagram: Title}}` — a 2D canvas; every node IS a real block and appears in that block's references (not canvas-only); nodes can be images or `{{[[video]]}}`. Has a legacy version.
- Mentions `{{[[mentions]]: [[Page]]}}` — inlines a page's linked + unlinked references (`/mentions`); `{{children-mentions: [[Page]]}}` inlines the children of those mentions.
- Calc `{{calc: 4 + 5}}` — block-ref args scrape the FIRST number from the referenced block (`3` from "My 3 apples"): `{{calc: ((uid)) + ((uid))}}`.
- roam/render `{{roam/render: ((codeUid))}}` — renders a referenced code block as a component (some take extra arg refs). JS/JSX must be ES5 (no `const`/imports/exports); full component docs live in Roam's dev graph.
- roam/css `{{[[roam/css]]}}` — a child fenced `css` block applies graph-wide styling. iframe `{{iframe: https://url}}` embeds a live webpage.
- Hiccup — a block starting with `:hiccup` renders a Clojure/Hiccup vector as HTML, e.g. `:hiccup [:iframe {:width "600" :height "400" :src "https://..."}]`. (Real Roam feature, but NOT in the help-graph export — verify against Roam's dev docs before relying.)
- More components
  - `{{[[video-timestamp]]: ((videoUid)) HH:MM:SS}}` — `/video timestamp` or `Cmd/Ctrl+Alt+t` under a video; newer ones embed a ref to the video so they stay linked, older bare ones still work but do not auto-update.
  - `{{character-count}}` / `{{word-count}}` — slash-inserted; count the containing block, not the page; word-count also sorts in All Pages.
  - `{{date}}` — `/date` calendar that inserts a date-page ref.
  - `{{encrypt:CIPHERTEXT}}` — a password-protected block; password unrecoverable and exports/backups are NOT encrypted; not graph-wide encryption.
  - `{{[[streak]]: [[Goal]]}}` — heatmap of how often a ref appears in daily notes; also takes query logic, e.g. `{{[[streak]]: {or: [[DONE]] [[Solutions]]}}}`.
  - `{{[[slider]]}}` (or `/` → Slider) — an inline rating control for polls; on Multiplayer graphs others can click to add their rating; used as an attribute value like `certainty:: {{[[slider]]}}` or in templates, and its state is captured/re-applied by templates.
