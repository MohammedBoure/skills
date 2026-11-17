# Navigation, Application Shell & Workspace Architecture

Choose the `QMainWindow` shell pattern that fits the application's workflow complexity rather than forcing one layout on every program.

## 1. Choosing the Right Shell Pattern

1. **Sidebar + `QStackedWidget` (Multi-Section Apps, Dashboards, Data Managers)**:
   - Left navigation rail (`200px–240px` expanded, optional `56px` icon-only collapsed mode) containing grouped navigation items (`QButtonGroup` with checkable `NavItem` buttons) and an optional bottom footer (Settings, Theme Toggle, User/Version).
   - Right main area: `QStackedWidget` hosting lazy-loaded or cached page views inside a `PageCanvas`.
2. **Tabbed Workspace (`QTabWidget` with `documentMode=True`) (Browsers, Multi-Document Editors, Query Tools)**:
   - Use closable/movable tabs when users work on multiple independent documents or sessions in parallel.
   - Provide an empty-state welcome screen when all tabs are closed.
3. **Tool / IDE / Studio Shell (`QMenuBar` + `QToolBar` + `QDockWidget`) (Editors, Scientific/Media/CAD Tools)**:
   - Use `QDockWidget` panels for inspectors, file trees, console logs, and property editors so users can dock, float, or toggle panels freely.
   - Keep `QToolBar` icons crisp, grouped with `addSeparator()`, and paired with tooltips showing keyboard shortcuts (e.g., `"Save (Ctrl+S)"`).

## 2. Multi-Pane Workspaces (`QSplitter`)

- Use `QSplitter` (`childrenCollapsible=False` unless intentional) for master-detail lists, side-by-side editors/previews, or log panes.
- Set sensible `setSizes([...])` ratios and minimum widths on panes so neither side collapses into an unreadable sliver.

## 3. Window & Layout State Persistence (`QSettings`)

- Persist and restore `QMainWindow.saveGeometry()` / `restoreGeometry()`, `saveState()` / `restoreState()` (for docks/toolbars), and `QSplitter.saveState()` / `restoreState()` using `QSettings` on close and startup.
- Always guard `restoreGeometry()` against disconnected/changed monitor setups so windows never open off-screen.

## 4. Canonical Action Placement (Zero Duplication)

- **Top-Right (`PageHeader`)**: Primary view-level creation/global actions (`+ New`, `Import`, `Export`).
- **Card / Table Toolbar**: Context-local search, filters, and view toggles.
- **Detail Drawer / Inspector**: Contextual actions for the currently selected item (`Edit`, `Duplicate`, `Delete`).
- **Modal Footer**: Dialog-level commit/cancel actions (`Cancel` on left/secondary, `Save` on far right).
- **Status Bar (`QStatusBar`)**: Passive, transient system state (connection status, item count, background task progress, zoom level)—never critical errors.
