# Single-File Static App

[English](README.md) | [简体中文](README.zh-CN.md)

> Build, verify and deliver a single-file static application with Chart.js inlined — offline-capable, no build toolchain.

## What this is

This is not a guide to bundling a framework. It is a workflow for the specific case where the deliverable must be one HTML file that opens on a machine with no network and no tooling installed.

That constraint rules out a lot of the usual approaches and creates a particular set of problems — which is why the workflow is written down rather than rediscovered each time.

## Why we open-sourced it

We produce internal tools this way on a regular basis: a little dashboard, a tracker, a calculator. The single-file constraint comes from the delivery environment, not from preference.

It is a build recipe with no business content in it, so publishing it costs nothing and saves the next person the same afternoon.

## What it does

- **Template with a placeholder.** `src/app.html` holds a marked slot where the charting library gets injected at build time, so the source stays editable.
- **One-command build.** The build step inlines the library and writes `index.html`, the only file that ships.
- **The `file://` storage trap, documented.** Browsers restrict `localStorage` under the `file://` protocol in ways that break the obvious design. The workaround is written down.
- **Automated verification.** A browser automation pass opens the built file and checks that it actually works, rather than assuming the build succeeded.
- **Delivery steps standardised.** Opening the containing folder and presenting the artifact are part of the workflow, not an afterthought.

## Requirements

Python 3 · Node (optional, for browser automation verification)

## Installation

```bash
git clone https://github.com/<your-username>/single-file-static-app.git
```

Then place the folder into your agent skills directory:

| OS | Path |
|---|---|
| Windows | `C:\Users\<you>\.workbuddy\skills\single-file-static-app\` |
| macOS / Linux | `~/.workbuddy/skills/single-file-static-app/` |

There is no build step. Restart the session and it takes effect.

## How to use

Describe the tool you want:

- "A single HTML file, offline, with a chart"
- "A local tracker I can just double-click"
- "Make it one file, everything inlined"

The skill sets up the template, builds with the library inlined, verifies in a browser, then delivers.

## Files

```
└── SKILL.md
```

## Notes and limits

1. **Inlining has a size cost.** A charting library adds a few hundred kilobytes to the HTML file. For a local tool that is acceptable; for a public page it may not be.
2. **`file://` is a restricted context.** Not every browser API is available. Storage, fetch and some clipboard operations behave differently.
3. **Verification matters more here.** A single-file build has fewer moving parts, which makes a silent failure easier to miss, not harder.
4. **No framework.** If the app needs routing, state management or a component tree, this is the wrong starting point.

## License

MIT
