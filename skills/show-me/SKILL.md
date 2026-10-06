---
name: show-me
description: Replace prose-heavy explanations with compact visual artifacts such as Mermaid diagrams, component trees, call stacks, state flows, file layouts, pseudocode, type signatures, diffs, and live HTML explainers. Use when the user invokes /show-me or $show-me, asks to see work visually, wants a route or feature previewed, or needs a visual explanation of code, architecture, behavior, or a change. Works across Codex, Claude Code, T3 Code, Cursor, and similar agent hosts by selecting the host's provider adapter.
---

# Show Me

Use visuals as the primary conversation surface. Keep surrounding prose short, concrete, and conversational.

## Operating model

1. Apply the provider-neutral rules below.
2. Identify the actual host and provider. T3 Code is a host/orchestrator; choose the underlying provider first, then apply the T3 overlay.
3. Follow exactly one provider adapter. Use only capabilities that are actually exposed in the current session.
4. Report the evidence level honestly: `code-shaped`, `browser-verified`, or `screenshot proof`.
5. When the request names or mentions a browser, preview, localhost route, or running UI, use the available browser/preview skill first. Treat an explicit browser mention as the provider choice. For an existing application, preview the real route and follow the authentication guidance below; authentication is not a reason to substitute a mockup. If browser capability is unavailable, an inline explanation may supplement the work, but it does not fulfill a request to verify the real UI.

## Provider-neutral rules

### Start with the smallest useful visual

Lead with one visual, then add only the context needed to read it:

- **Architecture or ownership:** a shallow file layout or component tree with one responsibility per node.
- **Component refactors:** a component tree that keeps the important state hooks, effects, async boundaries, and module boundaries; omit unchanged leaves.
- **Runtime behavior:** a Mermaid sequence, state, or flow diagram; quote labels that contain punctuation.
- **Backend or orchestration:** a call stack or call tree showing ordered calls, ownership, data/status transitions, and external boundaries.
- **Data or API design:** TypeScript-like interfaces and function signatures before implementation details.
- **Algorithmic behavior:** concise pseudocode with inputs, branching, mutation, and termination visible.
- **A focused change or review:** diff syntax showing what moves, appears, disappears, or changes state.
- **UI or interaction design:** a lightweight HTML mockup, HTML diagram/explainer, or the actual running page in the available preview surface.

Use real names from the code or request. Mark assumptions and unknowns directly in the visual. Prefer one strong visual over several decorative ones. Do not wrap a diagram in a wall of prose.

### Choose the smallest shape that answers the question

Default to lightweight inline visuals—trees, call stacks, Mermaid, types, pseudocode, and diff syntax. Escalate to HTML when interaction, layout, visual hierarchy, or a live preview is the thing being evaluated.

Use this menu:

- **“Where does this live?”** → shallow file layout with one-line responsibilities.
- **“What renders or owns state?”** → component tree with state hooks and module boundaries.
- **“What calls what?”** → call stack for one path; call tree for branching orchestration.
- **“What happens over time?”** → sequence diagram for participants; state diagram for lifecycle states.
- **“What is the code shape?”** → interfaces, types, signatures, and pseudocode.
- **“What changed?”** → diff syntax, including component-tree diffs, call-tree diffs, file-layout diffs, and state/control-flow diffs.
- **“What will it look or feel like?”** → HTML mockup or live page.
- **“What concept needs explaining?”** → HTML diagram/explainer when a static tree or Mermaid diagram is not enough.

For diff syntax, show only the changed shape with `+` and `-` markers even when most of the implementation is unchanged. Keep call stacks and trees shallow enough to scan; expand only the boundary that matters.

### Use `/show-me` for alignment before and after coding

Use the active request and recent work when `/show-me` has no named target; do not ask the user to repeat context that is already available. Before implementation, show the types, signatures, component tree, call stack, or state shape that the agent is about to build. After implementation, use the same shape to explain the result and use a diff-shaped visual to focus a large change or review.

### Keep the conversation visual

Make progress updates one to three lines long. Show the shape of a complex plan before implementation, then show the final visual state or screenshot after the work. Use text only for decisions, caveats, and evidence the visual cannot carry.

### Live preview contract

Use this workflow whenever the user asks to preview a route, UI, HTML explainer, local app, or finished visual artifact:

1. Identify the exact preview target and URL. Reuse the project's existing dev server and route when one exists. If no app exists, create a small self-contained artifact under `work/show-me/` and serve it over HTTP. Never use a `data:` URL.
2. Use the browser or preview provider explicitly named by the user when one is mentioned; otherwise use the current host's native browser or preview surface. Do not substitute an external browser, a guessed tool name, a different provider's API, or web search.
3. Navigate to the target, wait for the page's meaningful ready signal, inspect the rendered result, and verify the requested route, key text, controls, layout, and relevant console errors.
4. Take a final screenshot of the stable, user-facing state when the host supports capture. Save it under `work/show-me/`, include it in the final response with an absolute-path Markdown image link, and say what it proves.
5. Leave the finished preview, server, or session open when the host supports persistence. Never close the deliverable just before handoff.
6. If the host lacks browser or screenshot capability, do not fabricate browser verification or a screenshot. Return the strongest inline visual available and state the missing capability.

### Authenticate to the real application

Login is part of the real preview workflow. Prefer an existing authenticated tab or session in the selected browser. Reuse available shared/test credentials or existing session cookies when authorized for the target application and supported by the browser tools; use the application's normal login flow when a fresh login is needed. Do not ask the user to sign in again when usable authorized access already exists.

Keep credentials and session cookies in the supported authentication/session mechanism. Never expose them in skill files, memory, source files, screenshots, logs, or chat. Do not disable authentication, forge an identity, or change access controls to make a preview work.

If available authorized access is insufficient, ask the user to complete login, MFA, or account selection in the selected browser and resume the real route afterward. Report verification as pending while access is blocked. Do not replace the application with a mockup, fake data, or an alternate implementation to claim completion. Mockups remain useful when the user asks for a proposed design or concept, rather than verification of an existing application.

## Provider adapters

### Named Browser Plugin

- If the user explicitly mentions a browser plugin or browser tab, use that plugin's exposed native browser tools for navigation, accessibility inspection, interaction, screenshots, and tab handoff.
- For the OpenAI bundled Browser when its tools are exposed, prefer the `mcp__playwright__browser_*` operations. Keep the deliverable tab open and do not switch to `web.run`, standalone automation, or computer-use controls.
- If the target requires authentication, follow the shared authentication guidance above: reuse authorized sessions, cookies, or shared/test credentials through supported browser mechanisms, or sign in normally.

### Codex App

- Read and follow the `control-in-app-browser` skill before any browser action.
- Use only Codex's built-in in-app Browser (`iab`) through the browser-client runtime and Node REPL. Do not use standalone Playwright, Playwright MCP, Computer Use, Chrome, an external browser, or web search as a substitute.
- In a fresh browser runtime, make the Browser visible with `await (await iab.capabilities.get("visibility")).set(true)`.
- Prefer a tab whose visible URL exactly matches the target; otherwise create one with `await iab.tabs.new()`. Navigate once, wait for `domcontentloaded`, then use explicit DOM waits. Do not use `networkidle` for ordinary local pages.
- Save and emit screenshots with `nodeRepl.emitImage(...)`, then include the saved absolute path in the final Markdown response.
- Make `tabs.finalize({ keep: [{ tab, status: "deliverable" }] })` the final browser action. Do not close the deliverable tab or perform another browser action afterward.

### Claude Code

- In Claude Code Desktop, use the built-in Browser/app-preview pane for running apps and localhost routes. Keep the pane visible for handoff.
- In Claude Code CLI, cloud, or remote sessions, use a configured browser or preview MCP tool when one is available. Discover its actual navigation, inspection, screenshot, and keep/open operations; do not copy Codex's `iab` calls into Claude.
- Use the available native capture operation, save the bytes under `work/show-me/`, and include an absolute-path Markdown image in the final response. If the current Claude surface can show the preview but cannot export a screenshot, say so explicitly rather than claiming screenshot proof.
- Leave the preview pane, dev server, and session available when possible. Report the exact URL and whether the preview was left open. Do not invent a tab-finalize API.

### Cursor

- Use Cursor's native browser/preview or browser tool when exposed in the current Agent session. Use its actual tool names and screenshot operation; never assume Codex's `iab` runtime exists.
- Keep the preview visible/open when the Cursor surface supports it, save screenshots under `work/show-me/`, and include them in the final response.
- For Cloud Agents, treat the checked-in project skill and the remote preview URL as the source of truth; do not rely on a local machine-only browser tab.

### T3 Code

- T3 Code is a control plane that can run different providers. Identify whether the thread is using Codex, Claude Code, Cursor, OpenCode, or another provider, then follow that provider adapter.
- If T3 exposes its own preview/browser automation to the agent, prefer that surface for navigation, inspection, screenshots, and the visible handoff. Use the actual tools exposed by the current thread; do not invent a generic T3 API.
- Keep the T3 project/thread and its preview surface running. Report the preview URL, the underlying provider, the screenshot path, and whether the preview remains open.
- If T3 does not expose preview automation in the current thread, fall back to the underlying provider adapter or an inline visual. Do not claim browser verification just because T3 is displaying a project.

### Other hosts

- Use the host's native preview/browser and screenshot capabilities if present.
- If the host has no live browser, produce a self-contained inline visual or served HTML artifact and label the result `code-shaped` rather than `browser-verified`.
- Never silently switch providers or browsers to manufacture a stronger evidence level.

## Screenshot and handoff rules

- If the user asked for screenshots or a website test, include screenshots in the final Markdown response, not merely tool output or bare links.
- State what each screenshot proves in one short caption or sentence.
- Report the exact live URL and whether the preview/tab/session was left open. Say “tab left open” only after the host's handoff/finalization operation succeeds.
- Separate `code-shaped`, `browser-verified`, and `screenshot proof`. Source inspection alone is never browser verification.
- If authentication blocks the target, first try available authorized sessions or credentials as described above. Ask for user login assistance only when needed, keep verification pending, and resume the real application once access is ready. Preserve the selected browser/provider unless the user authorizes a change.

## Compact response shape

Use this shape unless the user requests another format:

1. The visual artifact first.
2. One or two sentences explaining the key relationship or change.
3. A short `Proof` line with the URL, screenshot, and verification level.
4. At most three follow-up bullets for risks, assumptions, or next actions.

When there is no live page to preview, do not fabricate a browser screenshot. Use Mermaid, a tree, pseudocode, types, or a diff directly in the response and keep the explanation concise.
