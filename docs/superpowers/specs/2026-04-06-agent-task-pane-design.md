# Per-Agent Task Pane — Design Spec

## Problem

The dashboard has no task visibility. 24 tasks exist in the work-server but operators can't see them. Showing all tasks in a flat list would be noisy and unactionable. Operators need per-agent task context when investigating a specific agent.

## Solution

A slide-in right panel triggered by clicking an agent card. Shows only tasks assigned to the clicked agent. The rest of the dashboard shifts left to accommodate.

## Interaction

1. **No card selected (default):** Dashboard renders full-width, no pane visible.
2. **Click agent card:** Pane slides in from the right (~380px wide). Dashboard content gets `margin-right: 380px` with a CSS transition. Clicked card gets an accent border highlight.
3. **Click same card or X button:** Pane closes, dashboard returns to full width.
4. **Click different card:** Pane content switches to the new agent's tasks (no close/reopen animation). Previous card loses highlight, new card gains it.

## Data Source

- **Endpoint:** `GET /tasks?assignee={actor_id}` on the work-server.
- **Auth:** Cookie auth (`credentials: "same-origin"`) when embedded, Bearer token when standalone — same pattern as the status poll.
- **Fetch timing:** On card click only. No auto-refresh or polling. Re-clicking the active card re-fetches.
- **AbortController:** Cancel any in-flight task fetch when a new card is clicked (same pattern as status polling).

## Pane Layout

```
+---------------------------------------+
| [X]  {Role} — Tasks (N)              |
+---------------------------------------+
| +-----------------------------------+ |
| | Title of task                     | |
| | [assigned] [high]                 | |
| | First line of description...      | |
| +-----------------------------------+ |
| +-----------------------------------+ |
| | Another task title                | |
| | [pending] [medium]               | |
| | First line of description...      | |
| +-----------------------------------+ |
|                                       |
| (click task card to expand desc)      |
+---------------------------------------+
```

- **Header:** Close button (X), agent role capitalized, "Tasks (N)" count.
- **Task card:** Title (bold), status badge (colored: green=completed, blue=assigned, gray=pending), priority badge, single-line description truncated with ellipsis.
- **Expand:** Click a task card to expand its full description. Click again to collapse.
- **Empty state:** "No tasks assigned" centered in the pane body.
- **Loading state:** Simple "Loading..." text while fetch is in-flight.
- **Error state:** "Failed to load tasks" with a retry link.

## CSS

- Pane: `position: fixed; right: 0; top: 0; bottom: 0; width: 380px; background: var(--bg); border-left: 1px solid var(--border); z-index: 10; overflow-y: auto; transform: translateX(100%); transition: transform 0.2s ease;`
- Pane open: `transform: translateX(0);`
- Dashboard shift: `#dashboard { transition: margin-right 0.2s ease; }` — set `margin-right: 380px` via JS when pane is open.
- Selected card: `border-color: var(--accent);` (reuse existing accent color).
- Responsive: At viewport widths below 900px, pane overlays instead of shifting (no margin-right change, pane gets `box-shadow` for depth).

## Status Badges

| Task status | Color | Badge text |
|---|---|---|
| assigned | `var(--blue)` / `#3b82f6` | assigned |
| pending | `var(--text-dim)` | pending |
| completed | `var(--green)` | done |

## Priority Badges

| Priority | Style |
|---|---|
| high | Red text |
| medium | Amber text |
| low | Dim text |
| (missing) | No badge |

## Implementation Constraints

- All CSS and JS inline in `dashboard.html` — no external files (per CLAUDE.md).
- Vanilla JS only — no frameworks.
- Reuse existing `el()` helper for DOM construction.
- Reuse existing CSS variables for colors/spacing.
- Add pane HTML structure as a sibling to `#dashboard`, not inside it.
- Task fetch uses the same auth pattern (cookie vs Bearer) as status polling.

## Files Modified

- `dashboard.html` — CSS additions (~50 lines), HTML pane container, JS click handlers + fetch + render (~100 lines).

## Out of Scope

- Tasks created by the agent (only assigned-to).
- Auto-refresh / polling of tasks.
- Task mutation (create, assign, complete) from the dashboard.
- Showing tasks in the main dashboard area.
