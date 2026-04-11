# AHA

- [2026-04-11] `apps/desktop` source truth and runnable truth can diverge badly
  when the local Electron/Vite build chain is damaged. Once live `dist` patches
  enter the loop, "regressions" may simply be runtime/source mismatch rather
  than logic rollback. Repair rebuild determinism before trusting further UI or
  terminal fixes.
