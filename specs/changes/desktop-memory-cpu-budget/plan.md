# Plan

Status: active

## Target

Push renderer baseline down by removing eager heavyweight imports and verifying
with packaged/preview runs.

## Chosen Path

- lazy-load markdown code rendering dependencies
- lazy-load WebGL terminal addon code paths
- remeasure bundled startup behavior

## Scope

- renderer markdown/code path
- terminal addon loading path
- verification via tests, typecheck, build

## Task Slices

1. convert eager markdown/code imports to on-demand loading
2. stop eagerly loading WebGL addon code when DOM renderer is forced/default
3. run targeted tests and typecheck
4. rebuild desktop bundle and compare output/runtime shape

## Verify Commands

- `bun test apps/desktop/src/renderer/screens/main/components/WorkspaceView/ContentView/TabsContent/Terminal/helpers.test.ts`
- `bun test apps/desktop/src/renderer/screens/main/components/WorkspaceView/ContentView/TabsContent/Terminal/hooks/useTerminalRestore.test.ts`
- `bun test apps/desktop/src/renderer/screens/main/components/WorkspaceView/ContentView/TabsContent/Terminal/hooks/useTerminalLifecycle.test.ts`
- `cd apps/desktop && bun run typecheck`
- `cd apps/desktop && bun run compile:app`

## Main Risk

Dynamic-loading markdown and terminal renderer dependencies can introduce visual
fallback glitches or delayed first interaction if not staged carefully.
