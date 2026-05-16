# JS-to-TS Conversion Queue

This file tracks the queue of JavaScript files pending conversion to TypeScript.
The COORDINATOR agent manages this file; AGENT instances consume from it.

## Status Legend
- `PENDING` — not yet started
- `IN_PROGRESS` — an agent is actively working on this file
- `DONE` — conversion complete, PR ready
- `SKIPPED` — file excluded (e.g., auto-generated, third-party)
- `BLOCKED` — depends on another file that isn't converted yet

---

## Queue

| # | File | Status | Agent | Notes |
|---|------|--------|-------|-------|
| 1 | `superset-frontend/src/utils/common.js` | PENDING | — | Utility helpers, low complexity |
| 2 | `superset-frontend/src/utils/dates.js` | PENDING | — | Date formatting helpers |
| 3 | `superset-frontend/src/utils/urlUtils.js` | PENDING | — | URL parsing/building |
| 4 | `superset-frontend/src/components/Button/index.jsx` | PENDING | — | Simple presentational component |
| 5 | `superset-frontend/src/components/Label/index.jsx` | PENDING | — | Simple presentational component |
| 6 | `superset-frontend/src/components/ListView/index.jsx` | BLOCKED | — | Depends on #4, #5 |
| 7 | `superset-frontend/src/views/CRUD/utils.js` | PENDING | — | CRUD helpers |
| 8 | `superset-frontend/src/views/CRUD/chart/ChartList.jsx` | BLOCKED | — | Depends on #6, #7 |
| 9 | `superset-frontend/src/explore/exploreUtils.js` | PENDING | — | Explore view utilities |
| 10 | `superset-frontend/src/explore/actions/exploreActions.js` | PENDING | — | Redux actions |

---

## Completed

| # | File | PR | Completed At |
|---|------|----|--------------|
| — | — | — | — |

---

## Skipped

| File | Reason |
|------|--------|
| `superset-frontend/src/assets/stylesheets/**` | Not JS/JSX |
| `superset-frontend/src/setup/setupTests.js` | Test infrastructure, deferred |

---

## Notes

- Files are roughly ordered by dependency depth (leaves first).
- BLOCKED files should be re-queued as PENDING once their dependencies are DONE.
- Each agent should update its row's Status and Agent columns atomically.
- Run `yarn tsc --noEmit` after each conversion to verify no type regressions.
