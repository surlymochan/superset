# Superset Fix Handoff - 2026-04-11

## Current State

The current branch is `fix/patched-terminal-recovery`.

The active Superflow change is `desktop-memory-cpu-budget`.
It should now be treated as `blocked/partial`, not complete.

The desktop app has real improvements in place around renderer load, terminal
streaming, chat/pane rerender pressure, dev auth flow, and terminal restore
behavior. However, the final runtime is not cleanly reproducible from source in
this environment because the local desktop build chain is broken. During
recovery, parts of the running app were patched directly in `apps/desktop/dist`.

That means:

- some fixes exist in `src` but were never cleanly rebuilt into a fresh runtime
- some fixes exist in the currently runnable `dist` only
- regressions can appear to "come back" when the runtime switches between
  restored release assets and patched local assets

This change should not be considered fully archived as a finished optimization
pass. It is archived as a blocked checkpoint with durable lessons preserved.

## TL;DR

- Desktop dev login/CORS recovery was fixed in source.
- Multiple high-ROI renderer and terminal performance changes were implemented.
- The `vi`/alt-screen recovery path was repaired multiple times and partially
  mirrored into live `dist`.
- Sidebar spinner and chat input background were also repaired in live `dist`.
- The remaining blocker is not just app logic; it is the inability to produce a
  clean rebuilt desktop runtime from current source because the local
  `electron-vite` / `vite` / Bun dependency chain is damaged.
- Final status for this change: `blocked/partial archive`.

## Durable Facts

### What improved in source

- desktop renderer cloud auth/API in development now routes through same-origin
  logic instead of direct cross-origin cloud requests
- heavy markdown/code rendering paths were moved toward lazy loading
- terminal addon loading was reduced and WebGL made non-eager
- chat/message/pane rendering paths were optimized to reduce unnecessary
  rerenders
- terminal stream handling was changed to reduce main-thread pressure
- terminal restore logic was extended to handle alternate-screen recovery more
  carefully

### What was additionally repaired in the runnable dist

- restored a complete `apps/desktop/dist/renderer` from the packaged release
  app after an attempted asset prune deleted active route chunks
- patched live renderer chunks so the current runtime regained:
  - workspace spinner fallback when `workspaceRunState === "running"`
  - visible chat input background
  - improved `vi` alt-screen exit cleanup
  - terminal font stack and weights closer to iTerm style

### What is still not trustworthy

- a full clean rebuild of `apps/desktop` from source to runtime
- any claim that terminal restore regressions are permanently fixed until the
  source can be rebuilt and verified without live chunk patching
- any claim that the performance work is fully shipped rather than partially
  staged

## Main Blocker

The local desktop build chain is broken in a way that prevents reliable
`compile:app` or renderer-only rebuilds from producing a fresh runtime.

Observed failure modes included corrupted or inconsistent package behavior in:

- `electron-vite`
- `vite`
- `minimatch`
- `diff`
- `tsconfck`
- Babel runtime/config loading paths

Because of that, fixes were sometimes made in `src` but validated only by
patching live `dist` assets.

## Most Important Lessons

1. In this repo, mixing source changes with direct live `dist` patches is only
   acceptable as short-term recovery. It creates the exact "why did it regress"
   confusion the user experienced.
2. Do not prune or deduplicate `dist/renderer/assets` without a verified rebuild
   path. A first attempt deleted active route/layout chunks and forced runtime
   recovery from the packaged app.
3. Terminal `vi`/alternate-screen correctness must be validated in the actual
   runnable runtime, not only at source level, because this environment can
   diverge between `src` and `dist`.
4. The user's actual priority is interaction smoothness and correctness, not
   absolute RSS alone.

## Recommended Next Actions

1. Repair the local `apps/desktop` build chain so source changes can be rebuilt
   deterministically into `dist`.
2. Once rebuild works, remove the emergency live `dist` patch dependency by
   rebuilding from source and verifying:
   - `vi -> switch away -> switch back -> exit vi`
   - sidebar spinner
   - chat input background
   - terminal typing responsiveness
3. Only after runtime/source parity is restored, continue the next performance
   slice or resume unrelated feature work like `SkillHub`.

## High-Signal Paths

- [apps/desktop/src/renderer/screens/main/components/WorkspaceView/ContentView/TabsContent/Terminal/hooks/useTerminalRestore.ts](/Users/chenchao/Documents/workspace/superset-fix/apps/desktop/src/renderer/screens/main/components/WorkspaceView/ContentView/TabsContent/Terminal/hooks/useTerminalRestore.ts:1)
- [apps/desktop/src/renderer/screens/main/components/WorkspaceView/ContentView/TabsContent/Terminal/hooks/useTerminalStream.ts](/Users/chenchao/Documents/workspace/superset-fix/apps/desktop/src/renderer/screens/main/components/WorkspaceView/ContentView/TabsContent/Terminal/hooks/useTerminalStream.ts:1)
- [apps/desktop/src/renderer/screens/main/components/WorkspaceView/ContentView/TabsContent/Terminal/config.ts](/Users/chenchao/Documents/workspace/superset-fix/apps/desktop/src/renderer/screens/main/components/WorkspaceView/ContentView/TabsContent/Terminal/config.ts:1)
- [apps/desktop/src/renderer/lib/terminal/appearance/index.ts](/Users/chenchao/Documents/workspace/superset-fix/apps/desktop/src/renderer/lib/terminal/appearance/index.ts:1)
- [apps/desktop/dist/renderer/assets/page-5DzNZSH5.js](/Users/chenchao/Documents/workspace/superset-fix/apps/desktop/dist/renderer/assets/page-5DzNZSH5.js:21625)
- [apps/desktop/dist/renderer/assets/layout-CS5LjC11.js](/Users/chenchao/Documents/workspace/superset-fix/apps/desktop/dist/renderer/assets/layout-CS5LjC11.js:5038)
- [apps/desktop/dist/renderer/assets/constants-C6Q0QQE2.js](/Users/chenchao/Documents/workspace/superset-fix/apps/desktop/dist/renderer/assets/constants-C6Q0QQE2.js:23)
- [apps/desktop/dist/renderer/assets/config-CMOv1hQz.js](/Users/chenchao/Documents/workspace/superset-fix/apps/desktop/dist/renderer/assets/config-CMOv1hQz.js:9)

## Final Status

- change: `desktop-memory-cpu-budget`
- archive outcome: `blocked/partial`
- reason: meaningful improvements landed, but final runtime is still dependent on
  emergency `dist` recovery and cannot yet be cleanly reproduced from source
