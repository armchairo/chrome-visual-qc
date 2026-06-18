---
name: chrome-visual-qc
description: Drive Claude-in-Chrome to VISUALLY verify or QC a UI change, audit a live app, or diagnose a fault that shows in the browser — the merge-gate + visual-audit runbook. Use when verifying a UI PR before merge, auditing a deployed app for correctness and UX, or diagnosing something that only shows up at runtime in a real browser session.
---

# When to use this skill

The visual surface is the ground truth and the diff is not enough:

1. **UI PR merge-gate** — a frontend PR cannot be merged on code review alone; verify it renders correctly in the real app first.
2. **Live-app visual audit** — audit a deployed dashboard, cockpit, or feature tab for correctness, readability, and UX.
3. **UI-surfaced diagnosis** — the app misbehaves in a way that shows in the browser: silent 500s, empty states when data should be present, interaction bugs. Chrome MCP localises the fault without guessing.
4. **Design/UX QC** — heuristic and best-practice pass on a tab or flow before sign-off.

Not for headless logic with no visual surface (that is `verify`/tests) or backend-only changes.

# The discipline: verify against INTENT, not aesthetics

The reframe that makes this worth doing — it is not "does it look fine," it is **"does it match the spec / requirement / domain ground truth."** Frame every audit as a **gap analysis: current state vs expected**. Visual QC repeatedly catches what code review cannot:

- **Domain-correctness**, not just rendering: the chart shows the right data, the colour legend matches the metric, the computed value matches the source record. The pixels rendered fine; the *meaning* was wrong.
- **Encoding and UX flaws invisible in the diff**: 12 data series on an 8-colour palette; a filter that resets silently mid-flow; a label that truncates at a realistic string length.
- **Interaction bugs**: a lightbox unreachable after a mode switch; a form that pre-fills stale state; an event emitter wiped by a re-render that only shows up when you actually click.

Run the browser. Do not trust the artifact.

# Step 0 — Research best-practice for THIS app type first

Before auditing, ground in the conventions for the app type so findings are anchored, not vibes: data dashboard → exec-readability + tolerance-awareness; chart → encoding clarity (colour count, axis legibility); annotation tool → interaction conventions; form flow → validation and error states. A short pass on the app-type's heuristics turns a vague checklist into a *grounded* one.

# Step 1 — Drive modes

- **This-session Chrome MCP** — if `mcp__Claude_in_Chrome__*` is live (`/chrome` or `claude --chrome`), drive the open browser directly: navigate, screenshot, `read_page`, `find`, `javascript_tool`, `read_console_messages`, `read_network_requests`.
- **Handoff to a fresh `claude --chrome` session** — the canonical merge-gate when your current session cannot reach the dev server: write a self-contained handoff prompt → fresh CLI session drives the already-open (signed-in) browser → screenshots and measurements → returns a verdict. Posts it as a PR comment (the gate). Designed for the common case where headless agents cannot auth to a localhost dev server because the app uses OAuth — the fresh session rides your live browser cookies.
- **Fan out for many flows (scale)** — for comprehensive app QA, use a Workflow to verify many user flows in parallel rather than one page at a time. One agent per flow or route; dedup findings across parallel auditors. A single goal can cover hundreds of user flows this way.

# Step 2 — What to actually check

- **Prefer the accessibility tree over pixels.** Use `read_page` and `find` (text and ref-IDs) to locate and assert, not CSS-class or coordinate clicking — resilient to re-renders and restyles. Couple assertions to **user intent**, not DOM detail. Pixels are for the human-perception check (layout, spacing, colour encoding); the a11y tree is for "is the right thing there and interactable."
- **Representative sampling across states.** Do not verify one frame — cycle real variation: 3-4 representative cases (the happy path, an empty state, an error state, and at least one edge case). Screenshot each; the bug usually lives in a state, not the default.
- **Walk the user flows and hunt edge cases.** Click through, fill forms, trigger the empty, error, loading, and permission-denied states.
- **Multiple viewports.** Buttons and labels render differently at different sizes; check the responsive breakpoint and a narrow width.
- **Console and network are evidence.** `read_console_messages` and `read_network_requests` catch silent failures — 4xx/5xx responses, empty datasets, missing resources — that a screenshot alone misses.

# Step 3 — Mechanics and gotchas

- **Flaky native controls → set via JS.** Range sliders and `<select>`s often ignore synthetic clicks; use `javascript_tool` to set `.value` and dispatch `input` and `change` events.
- **Verifiable gate, not "looks good."** State the gate as a checkable claim: *"filter the table to status=active and confirm the row count matches the badge in the header."* PASS/FAIL per claim. Note partials honestly.
- **For pixel or regression goals, give yourself a measure** — build or use a screenshot-diff tool to compare frames. But do not rabbit-hole on pixel-perfection — keep the overall goal; hard constraints stop the agent burning the run chasing one stray pixel.
- **Screenshots cost context tokens** — take targeted shots; do not leave Chrome MCP streaming continuously; disable when not actively verifying.
- **Auth and subscription** — Claude-in-Chrome needs a direct Anthropic Pro/Max/Team subscription (NOT Bedrock/Vertex); Chrome or Edge only (not Arc/Brave); restart Chrome if the extension is not detected on first use.

# Step 4 — Output: severity-ranked findings, then remediate

- **PR verification** → post the verdict as a PR comment (PASS/FAIL per claim + screenshots). This is the merge gate.
- **Audit** → a findings file (e.g. `findings/<date>-qa-<feature>.md`), **severity-ranked (P0/P1/P2)**, then implement every finding in a continuous structured goal. Do not file and forget — the finding is only useful if it is acted on.
- **Consider an HTML report** for findings that will be shared — far more readable than markdown and can embed screenshots inline.

# Best-practice basis

- **Verify against intent, not surface**: gap-analysis (current vs expected); domain-correctness over rendering; representative sampling across real states; severity-rank findings then remediate in a goal.
- **bcherny (Claude Code)**: self-verify end-to-end; *"test the result e2e in a browser… especially look for edge cases and UI issues"*; verify user flows at scale via a workflow.
- **Research consensus**: couple verification to the **accessibility tree and user intent, not DOM/CSS selectors** (resilient to re-renders); multi-viewport (resolution sensitivity); filter the UI tree to interactable elements (reduces hallucination); requirement-driven verification (GUISpector). Sources: alexop.dev (Claude Code agent-browser QA 2026), arxiv GUISpector 2510.04791.
- **The /goal pattern**: verifiable criteria; build a self-measure tool (screenshot-diff); do not rabbit-hole on pixel-perfect.
- **HTML reports**: prefer HTML over markdown for audit findings that will be shared or read outside the session.
