---
name: show-me
description: Replace prose-heavy explanations with compact visual artifacts such as Mermaid diagrams, component trees, call stacks, state flows, file layouts, pseudocode, type signatures, diffs, and live HTML explainers. Use when the user invokes /show-me or $show-me, asks to see work visually, wants a route or feature previewed, or needs a visual explanation of code, architecture, behavior, or a change. For live previews, use Codex's built-in in-app Browser, take and show screenshots, and leave the final preview tab open.
---

# Show Me

Use visuals as the primary conversation surface. Keep surrounding prose short, concrete, and conversational.

## Start with the smallest useful visual

Lead with one visual, then add only the context needed to read it:

- **Architecture or ownership:** a shallow file tree or component tree with one responsibility per node.
- **Runtime behavior:** a Mermaid sequence, state, or flow diagram; quote labels that contain punctuation.
- **Backend or orchestration:** a call stack or typed pseudocode showing the important boundaries.
- **Data or API design:** TypeScript-like interfaces and function signatures before implementation details.
- **A focused change:** a diff-shaped summary showing what moves, appears, disappears, or changes state.
- **UI or interaction design:** a self-contained HTML explainer or the actual running page in the browser.

Use real names from the code or request, and mark assumptions and unknowns directly in the visual. Prefer one strong visual over several decorative ones. Do not wrap a diagram in a wall of prose.

If the user invokes `/show-me` without naming a target, use the active request and recent work as the target. Restate the problem simply and show its shape; do not ask the user to repeat context that is already available.

## Keep the conversation visual

During work, make progress updates one to three lines long. Show the shape of a plan before a complex implementation, then show the final visual state or screenshot after the work. Use text only for decisions, caveats, and evidence that the visual cannot carry.

## Live preview workflow

Use this workflow when the user asks to preview a route, UI, HTML explainer, local app, or finished visual artifact. A live preview is a required part of the result in those cases.

1. Identify the exact preview target and URL. Reuse the project's existing dev server and route when one exists. If an HTML explainer is needed and no app exists, create a small self-contained artifact under the workspace's `work/show-me/` directory and serve it from a localhost HTTP server. Do not use a `data:` URL; Codex's in-app Browser blocks it.
2. Read and follow the `control-in-app-browser` skill before any browser action. Use only Codex's built-in in-app Browser (`iab`) through the browser-client runtime and the Node REPL. Do not substitute standalone Playwright, Playwright MCP, Computer Use, Chrome, an external browser, or web search.
3. In a fresh browser runtime, select the in-app Browser exactly as the browser skill specifies and read its complete documentation. When the user asked to see or watch the preview, make the Browser visible:

   ```js
   await (await iab.capabilities.get("visibility")).set(true)
   ```

4. Prefer claiming an already-open in-app tab only when its visible URL exactly matches the preview target. Otherwise create one with `await iab.tabs.new()`. Store and reuse the `iab` and tab bindings across REPL calls; recover a stale tab by creating a fresh tab, not by selecting another browser.
5. Navigate once to the target. Wait for `domcontentloaded`, then inspect a fresh DOM snapshot. For local apps, reload after code or build changes before taking the final snapshot or screenshot. Use an explicit DOM wait for the page's meaningful ready signal; do not wait on `networkidle` for ordinary local development pages.
6. Verify the visible result. Check the requested route, key text, controls, layout, and console errors as relevant. Fix actionable failures before presenting the preview. Do not claim browser verification from source inspection alone.
7. Take at least one final screenshot of the stable, user-facing state. Save it to an absolute path under `work/show-me/` (or the task's existing output directory), and emit the same bytes inline through `nodeRepl.emitImage(...)` so the user can see it during the turn:

   ```js
   var showMeFs = await import("node:fs/promises")
   var showMePath = await import("node:path")
   var showMeScreenshotPath = showMePath.resolve(nodeRepl.cwd, "work", "show-me", "final.png")
   await showMeFs.mkdir(showMePath.dirname(showMeScreenshotPath), { recursive: true })
   var showMePng = await showMeTab.screenshot({ fullPage: false })
   await showMeFs.writeFile(showMeScreenshotPath, showMePng)
   await nodeRepl.emitImage(showMePng)
   nodeRepl.write(showMeScreenshotPath)
   ```

   In the final response, include the saved image with an absolute-path Markdown link:

   ```md
   ![Final preview](/absolute/path/to/show-me-final.png)
   ```

   Take additional screenshots only for a meaningful second state, responsive breakpoint, or before/after comparison.
8. Keep the final preview tab open and visible. Never call `showMeTab.close()` for the deliverable. Make `tabs.finalize(...)` the final browser action, keeping the finished tab as a `deliverable`:

   ```js
   await iab.tabs.finalize({
     keep: [{ tab: showMeTab, status: "deliverable" }],
   })
   ```

   Do not perform another browser action after finalizing. Omit intermediate tabs from `keep`. Use `handoff` instead of `deliverable` only when the task is intentionally unfinished and the user should continue from the live page.

## Screenshot and handoff rules

- If the user asked for screenshots or a website test, include the screenshots in the final Markdown response, not merely in tool output or as bare links.
- State what each screenshot proves in one short caption or sentence.
- Report the exact live URL and whether the tab was left open. Say “tab left open” only after the finalization call succeeds.
- Separate evidence levels: `code-shaped`, `browser-verified`, and `screenshot proof`. Do not call source-only inspection completed visual verification.
- If the built-in in-app Browser is unavailable, do not silently fall back to another browser. For a code-only explanation, continue with an inline visual; for a requested live preview, state the blocker clearly and preserve any artifact that was created.
- If authentication blocks the target, ask the user to sign in in the in-app Browser and tell you when it is ready. Do not bypass login or switch browsers.

## Compact response shape

Use this shape unless the user requests another format:

1. The visual artifact first.
2. One or two sentences explaining the key relationship or change.
3. A short “Proof” line with the URL, screenshot, and verification level.
4. At most three follow-up bullets for risks, assumptions, or next actions.

When there is no live page to preview, do not fabricate a browser screenshot. Use Mermaid, a tree, pseudocode, types, or a diff directly in the response and keep the explanation concise.
