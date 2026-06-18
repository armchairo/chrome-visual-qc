# chrome-visual-qc

**Verify a UI change against its intent in a real browser — not by reading the diff.**

The diff tells you what changed. The browser tells you whether it's right. A chart can render without error and still show the wrong data. A form can pass tests and still corrupt state on the third submit. A palette can look sensible in isolation and still collapse when 12 series share 8 colours.

chrome-visual-qc is a Claude Code plugin that gives your agent a disciplined runbook for visual verification: how to drive Claude-in-Chrome, what to actually check, how to turn observations into a verifiable verdict, and how to fan out across hundreds of user flows when a single pass is not enough.

```
SCOPE → RESEARCH APP-TYPE → DRIVE BROWSER → CHECK STATES + FLOWS → VERDICT
        grounded heuristics  screenshot/tree   representative sample   gap analysis
```

## When to use it

- **PR merge-gate** — a frontend PR cannot merge on code review alone; verify it in the real app first.
- **Live-app audit** — dashboard, cockpit, or feature tab: is the data correct, the encoding clear, the interactions sound?
- **Fault diagnosis** — the app misbehaves in a way that shows in the browser. Console and network evidence, not guesswork.
- **Design/UX QC** — heuristic pass on a tab or flow before sign-off.

Not for headless logic (use tests) or backend-only changes.

## How it works

The skill has four stages:

1. **Research the app type first.** Ground in conventions before auditing — data dashboard, annotation tool, form flow — so findings are anchored, not vibes.
2. **Choose a drive mode.** In-session Chrome MCP for fast iteration; handoff to a fresh `claude --chrome` session for the merge-gate (it rides your already-authenticated browser); fan-out Workflow for comprehensive coverage across many flows.
3. **Check states, flows, and edges.** Prefer the accessibility tree over pixel-clicking. Sample representative cases. Walk user flows. Check multiple viewports. Read console and network output.
4. **Severity-rank findings, then remediate.** Post the verdict as a PR comment (the gate) or file a findings doc. Implement every finding in a continuous structured goal — do not file and park.

## What it is not

Not a screenshot tool. Not an aesthetics judge. Not a replacement for unit or integration tests. It is the last check before merge that answers: *does this actually work for a real user, in the real app, against the real requirement?*

See [`docs/pattern.md`](docs/pattern.md) for the research basis and failure modes. The skill itself is in [`skills/chrome-visual-qc/SKILL.md`](skills/chrome-visual-qc/SKILL.md).

## Install

```
/plugin marketplace add armchairo/chrome-visual-qc
/plugin install chrome-visual-qc
```

## Requirements

- Claude Code CLI
- Claude-in-Chrome browser extension (Pro/Max/Team subscription)
- Chrome or Edge (not Arc/Brave)
