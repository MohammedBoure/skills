# Design System, QSS Architecture & Structural Primitives

## 1. Centralized Theme & QSS Strategy

- **Single Source of Truth**: Keep color tokens and QSS rules in one centralized theme module (or load `./assets/base-theme.qss` when starting a brand-new project) and apply them once at the `QApplication` level using `app.setStyle("Fusion")` for consistent cross-platform rendering.
- **Light & Dark Theme Readiness**: Define semantic color tokens (`canvas`, `surface`, `subtle`, `border`, `text_primary`, `text_secondary`, `accent`) in a dictionary or dataclass and generate/swap the QSS from those tokens rather than hardcoding hex values inside widget classes.
- **Prevent QSS Selector Leakage**:
  - Never apply broad descendant rules like `QFrame { border: 1px solid ...; }` on container widgets, or they will leak into child dialogs, popups, and combo-box dropdowns.
  - Scope styles using explicit `setObjectName(...)` roles (`PageCanvas`, `Card`, `SubtlePanel`, `PageTitle`, `PageSubtitle`, `SectionTitle`, `FieldLabel`, `ErrorText`, `PrimaryButton`, `DangerButton`, `FilterPill`) and strict child/ID selectors (`QFrame#Card`, `QPushButton#PrimaryButton`).
- **Dynamic State Properties**: For runtime states such as validation errors (`widget.setProperty("invalid", True)`) or active selection (`widget.setProperty("active", True)`), always force Qt to recalculate styles:
  ```python
  widget.style().unpolish(widget)
  widget.style().polish(widget)
  ```
- **HiDPI & Crisp Icons**: Enable Qt's default HiDPI scaling (built-in in PySide6) and prefer SVG/vector icons rendered at `devicePixelRatio` over fixed-size raster bitmaps.
- **Slim Modern Scrollbars**: Style vertical/horizontal `QScrollBar` handles (`8px` wide/tall, rounded `4px`, subtle handle, zero arrow buttons) so dense tables, trees, and lists stay uncluttered.

## 2. Reusable Structural Primitives

Compose screens from predictable, reusable building blocks:

- **`PageHeader`**:
  - Horizontal bar at the top of a view.
  - Left side: Vertical stack of `PageTitle` (`20px` Bold) and optional `PageSubtitle` (`13px` secondary text).
  - Right side: Horizontal action group (`8px` spacing) with secondary actions (`Refresh`, `Export`, `Settings`) and at most **one** `PrimaryButton` (`+ New Item`).
- **`CardFrame`**:
  - `QFrame` (`objectName="Card"`) with elevated surface fill, `1px` hairline border, `10px` radius, `16px` inner padding, and `12px` vertical spacing.
- **`StatCard` / `KpiCard` Strip**:
  - Row of 3 to 4 equal-stretch summary cards (`min-height: 84px`) for overview/dashboard views.
  - Top line: Uppercase muted label (`11px` Bold).
  - Bottom line: Bold tabular metric (`20px–24px` Bold) paired with an optional trend/status badge.
- **`StatusBadge`**:
  - Compact capsule `QLabel` (`border-radius: 11px`, `padding: 3px 10px`, `font-size: 11px`, `font-weight: 700`) mapped to semantic tokens (Success, Warning, Danger, Info, Neutral).

## 3. Spatial & Alignment Discipline

- **Margins & Gaps**: Use `20px–24px` outer page margins and `16px` vertical gaps between major sections (`PageHeader`, `StatCard` strip, main `CardFrame`).
- **No Floating Orphan Controls**: Every button, search box, or filter must belong to a structured `PageHeader`, toolbar, or `CardFrame`.
- **Visual Hierarchy**: Differentiate primary, secondary, and destructive actions clearly—never place two filled primary-colored buttons side by side in the same action group.
