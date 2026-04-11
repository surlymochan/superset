# Proposal

Status: frozen

## Chosen Path

Reduce desktop memory and CPU usage within the existing Electron + React stack by
cutting renderer baseline cost first, then trimming terminal-specific overhead.

The first execution slice is:

1. remove eager loading of heavyweight renderer dependencies that are not needed
   at cold start
2. keep terminal rendering on the DOM path without eagerly loading WebGL addons
3. verify on packaged/preview builds instead of relying on dev-only memory data

## Rejected Paths

- Full Rust rewrite
  Rejected because it does not directly solve Chromium renderer baseline cost and
  would delay measurable improvement.
- Broad architectural refactor before measurement
  Rejected because the bundle already exposes clear eager-load targets.
- Dev-only tuning as the main fix
  Rejected because the user wants product-level resource reduction, not just a
  lighter local development loop.

## Scope

### In

- Renderer baseline reduction
- Terminal runtime/addon eager-load reduction
- Packaged/preview verification workflow

### Out

- Rust/Tauri migration
- Feature work such as SkillHub
- Rewriting the desktop routing/data model

## Acceptance Intent

- Lower cold-start renderer baseline materially from the current packaged/dev
  behavior
- Avoid loading heavy markdown/code/terminal renderer dependencies before they
  are actually used
- Preserve existing behavior for markdown rendering and terminal search

## Consistency Check

Checked against current repo truth:

- `HANDOFF.md`: aligns with the stated need to get a clean memory baseline before
  resuming unrelated feature work
- current user direction: explicitly stay on the existing stack and optimize
  memory/CPU rather than rewrite
- no `specs/product.md`, `specs/architecture.md`, `specs/workflow.md`,
  `DESIGN.md`, or `AHA.md` currently exist, so no conflicting canonical spec was
  found
