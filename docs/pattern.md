# chrome-visual-qc — pattern and rationale

## Why visual verification is a distinct discipline

Code review and automated tests verify what the code does. Visual verification answers a different question: does the running app, in a real browser, against real data, do what a user needs?

The gap is real. An encoding bug (wrong colour scale, truncated label, palette collision) passes every test. A domain-correctness error (chart shows revenue but the metric is margin; the computed value does not match the source) passes every test. An interaction bug that only surfaces after a specific click sequence — also invisible to tests.

Visual verification is not "checking it looks nice." It is a gap analysis: current state vs expected state, verified against the intent of the change, not against aesthetics.

## The three failure modes this skill is designed to catch

**1. Domain-correctness errors**
The pixels render fine; the meaning is wrong. A value is computed correctly for the happy-path record but rolls up incorrectly when the filter is changed. A chart legend says one thing; the tooltip says another. These only surface when you look at real data in the running app.

**2. State-dependent bugs**
The bug lives in a state, not the default. A loading state shows stale data. An empty state shows nothing where something should be. An error state does not communicate clearly. A permission-denied state reaches a crash instead of a message. The default frame passes; everything else breaks. Representative sampling catches these.

**3. Interaction-chain bugs**
The fault only appears when you actually click things in sequence: a re-render that wipes an event emitter; a form that pre-fills stale state from a previous submission; a modal that becomes unreachable after a mode switch. These are invisible in static review and only detectable by walking the flow.

## The accessibility-tree preference

Use `read_page` and `find` to locate and assert against elements — not CSS selectors or screen coordinates. The a11y tree is resilient to restyling and re-renders; coordinates are not. More importantly, it couples assertions to user intent (can a user find and interact with this control?) rather than implementation detail. Pixels are still useful for the human-perception check — layout, spacing, colour encoding — but the a11y tree is the evidence layer.

## Research-first discipline

Before auditing any app type, ground in that type's conventions:
- Data dashboards: executive-readability conventions, tolerance awareness, colour encoding standards
- Annotation tools: interaction modes (view/annotate/review), keyboard shortcut conventions
- Form flows: validation timing, error message placement, recovery from bad state
- Charts: colour count vs series count, axis legibility, legend clarity

A finding anchored to convention ("12 series on an 8-colour palette is a known encoding failure above 7") carries more weight and is easier to act on than "the chart looks confusing."

## Fan-out at scale

For comprehensive coverage — a full app audit rather than a single PR gate — use a Claude Code Workflow to run one agent per user flow in parallel. Each agent gets a specific flow to walk (e.g. "add a new user → verify the confirmation email is queued" or "filter by date range → verify the totals update"). Findings are collected and deduplicated. This approach can cover large apps systematically without sequential bottlenecks.

## Failure modes

- **Verifying aesthetics instead of intent**: stopped by framing every assertion as a checkable claim against the requirement.
- **Only checking the default state**: stopped by explicitly sampling empty, error, loading, and edge states.
- **Coordinate-clicking that breaks on restyle**: stopped by preferring `read_page`/`find` over coordinates.
- **Not reading console/network**: stopped by treating `read_console_messages` and `read_network_requests` as mandatory evidence sources, not optional.
- **Screenshot-flooding the context**: stopped by targeted shots and disabling Chrome MCP when not actively verifying.
- **Filing findings and not acting**: stopped by immediately converting findings to a remediation goal.

## Research basis

- bcherny (Claude Code): self-verify end-to-end in a browser; verify user flows at scale via workflows; "especially look for edge cases and UI issues."
- alexop.dev (2026): practical Claude Code agent-browser QA patterns.
- arxiv GUISpector 2510.04791: requirement-driven GUI verification; filtering the UI tree to interactable elements reduces hallucination.
- qaskills.sh / shiplight.ai (2026): agent-native QA emerging discipline: multi-viewport, a11y-tree preference, user-intent coupling.
