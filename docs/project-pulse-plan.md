# Project Pulse — Implementation Plan

## Summary

Project Pulse is a static, no-build, no-backend project dashboard living entirely under `app/`. It reads a local JSON file (`app/project-data.json`) client-side via `fetch`, renders a list of project cards into `app/index.html`, and is styled by `app/styles.css` with status badges and priority visual treatment. A `.vscode/launch.json` file lets the learner preview the dashboard directly from VS Code.

Because the app has no server and no build step, `fetch()` against a local JSON file will only work reliably when served over `http://` (not `file://`) in most browsers — this affects how the launch configuration and preview instructions must be defined (see Open Questions / Edge Cases). The plan assigns each of the four required files to exactly one agent, per the constraints in `coder.agent.md` (owns HTML structure, JS logic, data content, and `.vscode/launch.json`) and `designer.agent.md` (owns CSS visual/UX treatment, cannot touch files outside assigned scope).

The work splits into two sequential phases plus one parallel phase:
1. **Phase 0 (sequential, shared agreement):** Agree on the JSON data schema and the HTML element/class contract (IDs, class hooks) before either Coder or Designer builds against it.
2. **Phase 1 (parallel once contract is set):** Coder builds `index.html` structure + JS render logic + `project-data.json` content; Designer builds `styles.css` against the agreed class hooks. These can proceed in parallel only after the schema/markup contract from Phase 0 is fixed, since `styles.css` depends on class names Coder defines in `index.html`, and rendering JS depends on the JSON shape.
3. **Phase 2 (sequential, after Phase 1):** Coder creates `.vscode/launch.json` last, since it depends on `index.html` existing and being the correct entry file to open.

## File Assignments

| File | Owner | Rationale |
|---|---|---|
| `app/index.html` | **Coder** | HTML structure/semantics, script tag(s), fetch/render logic wiring, and DOM containers (e.g., `<div class="dashboard" id="dashboard"></div>`) are app logic/structure — Coder's domain. |
| `app/styles.css` | **Designer** | Visual design, layout, responsive behavior, status/priority visual treatment, and CSS hooks (`.dashboard`, `.project-card`, badge/priority classes) are UI/UX — Designer's domain per `designer.agent.md`. |
| `app/project-data.json` | **Coder** | Sample data content and its schema are app data/logic, not visual design — Coder owns it. |
| `.vscode/launch.json` | **Coder** | Explicitly called out in `coder.agent.md`: Coder creates this, strict JSON (no comments), `cwd` set to `${workspaceFolder}/app`, configured to open `index.html`. |

No file is jointly owned. Designer must not edit `index.html`, `project-data.json`, or `launch.json`. Coder must not edit `styles.css` unless explicitly reassigned.

## Designer Responsibilities

Scope: `app/styles.css` only.

1. Define `.dashboard` as the top-level grid/flex container for project cards — responsive (e.g., CSS grid with `auto-fill`/`minmax`, or flexbox wrap) so it reflows from multi-column to single-column on narrow viewports.
2. Define `.project-card` styling: rounded corners, subtle shadow/border, readable padding/spacing, clear typographic hierarchy (project name prominent, description/meta secondary).
3. Define visual treatment for **status** (e.g., `.status-badge`, with modifier classes such as `.status-active`, `.status-blocked`, `.status-done`, `.status-planned`) using color + shape (pill/badge) that also remains distinguishable without color alone (e.g., icon, label text, or pattern) for accessibility.
4. Define visual treatment for **priority** (e.g., `.priority-high`, `.priority-medium`, `.priority-low`) — border accent, left-edge color bar, or icon — again not relying on color alone.
5. Ensure sufficient color contrast (WCAG AA) for text on badges and card backgrounds.
6. Ensure responsive layout at common breakpoints (mobile ~375px, tablet ~768px, desktop ~1200px+), verified by resizing the preview.
7. Ensure focus states are visible for any interactive elements (if links/buttons are used) and that font sizes remain legible when zoomed.
8. Coordinate with Coder (via Orchestrator) on the exact class/data-attribute names before finalizing selectors — Designer should not invent class names unilaterally that conflict with what Coder wires into the DOM; propose hook names as part of Phase 0 agreement, then implement CSS against the agreed names in Phase 1.
9. Report back: which classes were used, how status/priority are visually differentiated, contrast choices, and how responsiveness was validated (breakpoints tested).

## Coder Responsibilities

Scope: `app/index.html`, `app/project-data.json`, `.vscode/launch.json`.

1. **`app/index.html`**:
   - Semantic HTML skeleton: `<head>` with title/meta/viewport tag, link to `styles.css`, and a `<body>` containing a header/title area and a `<div class="dashboard" id="dashboard"></div>` container that JS will populate.
   - Inline or separate `<script>` (e.g., `app/app.js` if the Orchestrator later approves an extra file, otherwise inline `<script>` at bottom of `index.html`) containing JS to `fetch('./project-data.json')`, parse JSON, and render one `.project-card` element per project into `#dashboard`.
   - Each rendered card must include: project name, status (mapped to a badge class per agreed schema), priority (mapped to a priority class), and any other schema fields (e.g., owner, due date, description).
   - Handle fetch/parse failure gracefully: render a visible error message in the dashboard container rather than a blank page or uncaught console exception.
   - Handle empty project array: render a friendly "No projects yet" empty state instead of nothing.
   - Use the exact class names agreed with Designer in Phase 0 (`.dashboard`, `.project-card`, badge/priority classes) so CSS applies without modification once merged.
2. **`app/project-data.json`**:
   - Define and document a clear schema, e.g.:
     ```json
     [
       {
         "id": "proj-001",
         "name": "Website Redesign",
         "status": "active",
         "priority": "high",
         "owner": "Jordan Lee",
         "dueDate": "2025-09-30",
         "description": "Refresh marketing site UI and content."
       }
     ]
     ```
   - Enumerate a fixed, agreed set of allowed `status` values (e.g., `active`, `blocked`, `done`, `planned`) and `priority` values (e.g., `high`, `medium`, `low`) so Designer can build a bounded, finite set of CSS modifier classes.
   - Provide 5–8 sample projects covering all status and priority combinations, plus at least one with a long name/description to exercise text overflow handling.
   - Ensure the JSON is syntactically valid (no trailing commas, correct quoting).
3. **`.vscode/launch.json`**:
   - Strict JSON, no comments.
   - `cwd` set to `${workspaceFolder}/app`.
   - Configuration should open `index.html` (e.g., using a "Live Preview"/"Open with Live Server" type task, or an `"type": "node-terminal"`/`"pwa-chrome"` launch with `"file": "${workspaceFolder}/app/index.html"` or a `url` pointing at a local server, depending on what extensions are assumed available in this repo's devcontainer — verify against `.vscode/` and `.devcontainer/` existing config before choosing type).
   - Use deterministic, descriptive names (e.g., `"Preview Project Pulse"`).
   - Must actually work given whatever browser-preview / live-server tooling is already available in this repo (check `.vscode/` and `.devcontainer/` for existing extensions/config before authoring — see Open Questions).
4. Validate: open the dashboard via the launch config, confirm cards render, confirm no console errors, confirm empty/malformed-data fallback behavior.

## Dependencies

1. **Schema agreement (Coder proposes, Designer reviews) → before `index.html` and `styles.css` are finalized.** The JSON field names (`status`, `priority`, etc.) and their allowed value sets must be fixed before Designer can enumerate CSS modifier classes and before Coder can write the JS mapping logic.
2. **`index.html` DOM/class contract (Coder proposes container/card class names) → before `styles.css` selectors are written.** Designer needs to know the exact class names (`.dashboard`, `.project-card`, `.status-badge.status-active`, etc.) Coder will emit from JS, otherwise CSS will not match anything.
3. **`project-data.json` existing (even as a draft/sample) → before Coder finishes the render logic in `index.html`**, so the JS can be tested against real sample data.
4. **`index.html` existing and finalized as the entry point → before `.vscode/launch.json` is written**, since the launch config must reference the correct file path (`${workspaceFolder}/app/index.html`) and correct `cwd`.
5. **`styles.css` link tag present in `index.html` `<head>` → before Designer's styling becomes visible**, so Coder must include `<link rel="stylesheet" href="styles.css">` in Phase 0/1 even before Designer's CSS content is complete.

## Parallel Work

- **Can run in parallel (Phase 1, after Phase 0 contract agreement):**
  - Coder: `app/index.html` (structure + JS) and `app/project-data.json` (sample data) — same owner, sequential within Coder's own work but independent of Designer.
  - Designer: `app/styles.css` — non-overlapping file scope with Coder's files, and no data dependency once the class-name/schema contract from Phase 0 is fixed.
  - These can genuinely run side by side because file scopes never overlap (`index.html`/`project-data.json` vs `styles.css`) and both sides only need the *contract* (names/schema), not each other's finished file content.

- **Must run sequentially:**
  - Phase 0 (contract/schema agreement) must complete before Phase 1 starts — this is a shared-understanding step, not a file-editing step, but skipping it risks class-name mismatches requiring rework.
  - `.vscode/launch.json` (Coder) must come after `index.html` exists and is stable, since it references that file/path — Phase 2 after Phase 1.
  - Within Coder's own scope, `project-data.json` sample data should exist (at least a draft) before the JS fetch/render logic in `index.html` is finalized and tested, though both are the same owner so this is an internal ordering choice, not a cross-agent blocker.

## Validation Expectations

1. Launch the app using the `.vscode/launch.json` configuration (`cwd = ${workspaceFolder}/app`) and confirm `index.html` opens directly (not a directory listing).
2. Confirm the dashboard renders one `.project-card` per entry in `project-data.json`, with correct name, status badge, and priority treatment matching the JSON values.
3. Confirm the status badge and priority indicator are visually distinct for each defined value (e.g., active vs. blocked vs. done all look different) and pass basic color-contrast sanity check.
4. Confirm no errors appear in the browser DevTools console during load (check fetch success, JSON parse success).
5. Resize the browser/preview window to mobile, tablet, and desktop widths and confirm the `.dashboard` grid reflows without horizontal scroll or overlapping cards.
6. Temporarily test with an empty `project-data.json` array (`[]`) and confirm the empty-state message renders instead of a blank page (manual/learner-driven check, not automated).
7. Temporarily test with malformed JSON (if learner wants to exercise this) and confirm the JS catches the fetch/parse error and shows a visible error message rather than a silent blank dashboard.
8. Visually confirm the first view "looks like a dashboard" per Designer's brief — not a bare unstyled HTML list.

## Edge Cases

- **Empty project list**: `project-data.json` is `[]` — dashboard must show a friendly empty state, not nothing or a JS error.
- **Malformed JSON**: syntax error in the file — `fetch().then(r => r.json())` will reject; JS must `.catch()` and render a visible error message.
- **Missing fields**: a project object missing `status` or `priority` — JS should apply a default/fallback class (e.g., `.status-unknown`) rather than throwing or rendering `undefined`.
- **Unexpected status/priority values**: JSON contains a value not in the agreed enumerated set (e.g., `status: "archived"` when only active/blocked/done/planned were agreed) — JS/CSS should degrade gracefully (fallback badge style) rather than break the layout.
- **Long project names/descriptions**: at least one sample project should have a long name/description to verify text wraps or truncates properly rather than breaking card layout (Designer must handle via `overflow-wrap`/`text-overflow`, Coder must include this sample case in data).
- **Many projects**: sample data set (5–8) should be enough to verify grid wrapping to multiple rows; consider noting in report if learner wants to stress-test with 50+ entries later.
- **Small screens**: verify at ~320–375px width there is no horizontal scrollbar and card content remains legible/tappable.
- **`file://` vs `http://` protocol**: some browsers block `fetch()` of local JSON files opened directly via `file://` due to CORS/security restrictions on XHR to local files — this is a real risk for a "no backend" static app and must be resolved via the launch configuration (serving via a local dev/preview server) rather than opening `index.html` directly by double-click.
- **Missing `.devcontainer`/`.vscode` browser-preview tooling**: if no live-server-type extension is preconfigured in this repo's devcontainer, the launch config approach needs adjustment (see Open Questions).

## Open Questions

1. **How will `.vscode/launch.json` actually serve the app given the `file://` fetch restriction?** The repo constraints say `cwd` = `${workspaceFolder}/app` and "open index.html," but plain `file://` fetch of a local JSON file is blocked or unreliable in several browsers. The Orchestrator/Coder should confirm whether the devcontainer already provides a live-server/browser-preview extension (check `.devcontainer/` and existing `.vscode/settings.json` if any) so the launch config can target `http://localhost:<port>/index.html` instead of a raw file path. If no such tooling exists, Coder may need to note this as a known limitation for the learner, or the Orchestrator may need to scope in a minimal serving mechanism (still no "build step," but possibly a documented `npx serve` instruction) — this needs a decision before Phase 2.
2. **Should `index.html`'s JS live inline or in a separate `app/app.js` file?** The requirements only name four files; a separate JS file isn't in the required list. Recommend inline `<script>` in `index.html` to stay within the exact four files specified, unless the Orchestrator explicitly wants to add `app/app.js` as a fifth assigned file.
3. **Exact enumerated status/priority values** are not specified by the learner exercise — Coder should propose a small, fixed set (e.g., status: active/blocked/done/planned; priority: high/medium/low) during Phase 0 for Designer sign-off, but this is not yet confirmed by any existing spec in the repo.
4. **Existing `.vscode/` contents**: this repo already has a `.vscode` directory — its current contents should be reviewed before Coder adds `launch.json` to avoid overwriting unrelated existing configuration (I did not enumerate its contents in this pass; Orchestrator should have Coder inspect it first).
