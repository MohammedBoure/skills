# Data Tables, Trees, Search Bars, Filters & Empty States (PySide6)

## 1. Unified Search & Filter Toolbar

Place search and filter controls inside the top area of the data view's `CardFrame`, separated from the table header by a subtle divider or spacing:

- **Debounced Search Input**:
  - Configure `QLineEdit` with a clear, specific placeholder (`"Search by name, ID, or tag..."`) and `setClearButtonEnabled(True)`.
  - Use a single-shot `QTimer` (`250ms`) on `textChanged` so proxy filtering or async queries run smoothly after typing pauses.
  - Connect `returnPressed` to cancel the timer and filter immediately; handle `Qt.Key_Escape` to clear the search text and restore focus.
- **Segmented Filter Pills**:
  - Group checkable `QPushButton`s (`objectName="FilterPill"`) inside an exclusive `QButtonGroup` for high-frequency status/category filters.
  - Display live counts in pill labels when helpful (e.g., `All (124)`, `Active (98)`, `Archived (26)`).
  - For more than 5 filter categories or multi-criteria filtering, use a compact `QComboBox` or expandable filter popover instead of wrapping dozens of pills.

## 2. `QTableView` / `QTreeView` Model-View Architecture

- **Strict Model/View Separation**: Use `QTableView` (or `QTreeView` for hierarchical data) backed by `QAbstractTableModel` / `QAbstractItemModel` and `QSortFilterProxyModel`.
- **Standard Table Ergonomics**:
  - Enable alternating row colors (`setAlternatingRowColors(True)`), single or multi-row selection (`SelectRows`), column sorting (`setSortingEnabled(True)`), `NoEditTriggers` (unless inline editing is explicitly intended), hidden vertical header, `38px` header height, and `>= 42px` row height.
- **Column Sizing Strategy**:
  - Assign `QHeaderView.ResizeMode.Stretch` to the primary descriptive column (e.g., `Title`, `Name`, `Description`) so the table fills 100% of its container width with zero dead margin.
  - Use `ResizeToContents` or `Fixed` with explicit minimum widths (`90px–140px`) on IDs, dates, numbers, status badges, and action columns.
- **Locale-Aware Alignment & Formatting**:
  - Return `Qt.AlignRight | Qt.AlignVCenter` in `TextAlignmentRole` for all numeric, currency, duration, and quantity columns so decimal points/units align vertically.
  - Format numbers, dates, and currencies using `QLocale` or the project's configured locale rather than hardcoding a single region's format.
- **Clean Row Interaction**:
  - Avoid embedding heavy interactive widgets (`QComboBox`, multi-button toolbars) in every row of large tables; prefer `QStyledItemDelegate` for badges, progress bars, and custom cells.
  - When embedding compact action buttons inside cells (`+`, `-`, `Trash`), assign a dedicated compact `objectName` (e.g., `CellIconBtn` with `padding: 0; min-height: 24px;`) so `36px` toolbar button padding does not crush button labels.
  - **Status Badge Design Discipline (`StatusBadgeDelegate`)**:
    - **Content-Fitted Pill Width**: Compute width dynamically via `min(option.rect.width() - 12, QFontMetrics(font).horizontalAdvance(text) + 22)`—never draw wide fixed-width capsules that leave short words (`Out`, `Paid`, `نفد`) swimming in empty space.
    - **Signal-Over-Noise (Quiet Healthy States)**: Avoid turning a status column into a loud wall of bright green pills. Render routine/healthy states (`In Stock`, `Paid`, `متوفر`, `مدفوعة`) with a subtle `7px` semantic dot + calm text, reserving soft tinted pills for warnings and critical alerts (`Low Stock`, `Expired`, `Out of Stock`).
    - **Selection & Divider Continuity**: In `paint()`, draw the row's normal/alternating/selected background across `option.rect` *and* preserve the bottom `1px` row hairline divider so the status cell blends smoothly with sibling columns instead of looking like an isolated sticker.
  - Bind `doubleClicked` (and `Enter` key) to open the primary detail/edit view, and provide a clean `QMenu` context menu (`customContextMenuRequested`) for secondary row actions.
  - If a row action button is rendered and disabled, set a `setToolTip(...)` explaining why.

## 3. Empty, Zero-Match & Loading States (`QStackedWidget`)

Wrap the data view inside a `QStackedWidget` with three dedicated states so users never stare at an unexplained blank grid:

1. **Active Data View**: The `QTableView`/`QTreeView` paired with an optional compact footer bar (row count, selection summary, or column totals).
2. **Empty / Zero-Match View**: Centered vertical layout (`min-height: 240px`) with a subtle icon, clear title (`"No matching results"` or `"No items yet"`), concise helper text, and one clear recovery button (`"Reset Filters"` or `"+ Create Item"`).
3. **Loading View**: Centered progress indicator (`QProgressBar` indeterminate or spinner) and `"Loading data..."` label while background workers fetch records.
