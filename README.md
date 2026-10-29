# rea-replica

One plugin that combines two MIT-licensed projects:

- **REA** (github.com/morluto/rea) — reverse engineer native binaries,
  Electron/JavaScript apps, Android packages, .NET assemblies, firmware,
  browser and network captures, with evidence behind every conclusion.
- **Replica** (github.com/Jakeschincariol/replica-skill) — turn what you learn
  into your own clean-room rebuild: recon map, architecture, design tokens,
  build and test, parity scoring, review mining, rebrand, launch and deploy.

## Install

In Claude Code:

```
/plugin marketplace add jk4rk2dvwg-lab/REA
/plugin install rea-replica@rea-replica
```

Or download `rea-replica.plugin` from the releases and install it in the Claude app.

## What's inside

- `skills/reverse-engineer-anything/` — the REA skill (v55) plus two added
  guides: `references/rebuild-workflow.md` and `references/launch-and-deploy.md`.
- `skills/reverse-engineer-anything/tools/` — six standard-library Python
  3.8+ tools, no network access: `parity.py`, `imgdiff.py`, `contrast.py`,
  `reviews.py`, `sweep.py`, `listing.py`.
- `skills/reverse-engineer-anything/templates/` — recon map, feature matrix, architecture, design tokens, test
  plan, bug report, Playwright example, store listing, deploy checklist.
- `.mcp.json` — starts the REA analysis server with `npx -y rea-agents@6.8.0 mcp`.

## Requirements

- Node.js and npm (for the REA server), Python 3.8+ (for the tools).
- Native decompilation also needs an existing Ghidra, IDA, Hopper or Kuna
  install. JavaScript/Electron analysis needs none.
- Don't install this alongside the official `rea` plugin; both register a
  server named `rea`.

## Ground rules

Rebuild what an app does, never what it owns. Decompiled code, extracted
modules, assets and copy are evidence, not material to ship. Public sources
and your own account only. No fake reviews or testimonials. Nothing deploys
until the rebrand sweep is clean and you say go. Not legal advice.

## Licenses

`LICENSE-rea` and `LICENSE-replica` (both MIT).
