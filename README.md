````markdown
# TS Data Tracker - Repository Index

## Repository Map

```text
/
├─ Class_Modules/
│  ├─ clsPickerItem
│  └─ clsTestNode
│
├─ Forms/
│  └─ frmPicker
│
├─ Modules/
│  ├─ modCatalog
│  ├─ modPicker
│  ├─ modPositioning
│  ├─ modRecordSearch
│  ├─ modRibbon
│  ├─ modRouting
│  ├─ modSheetView
│  ├─ modState
│  ├─ modTestHeirarchy
│  ├─ modTestTree
│  ├─ modTheme
│  └─ modUITheme
│
├─ TS_Data_Workbook
└─ customUI14(TS_DB).xml
````

---

# Navigation

| Task                                 | Primary Files                                                 |
| ------------------------------------ | ------------------------------------------------------------- |
| Understand worksheet entry behavior  | `TS_Data_Workbook`, `modState`                                |
| Change custom picker behavior        | `Forms/frmPicker`, `Class_Modules/clsPickerItem`, `modPicker` |
| Change picker appearance             | `modUITheme`, `modTheme`, `clsPickerItem`                     |
| Change semantic worksheet colors     | `modTheme`, `tbl_Theme` in workbook                           |
| Change hierarchy generation          | `modTestHeirarchy`                                            |
| Query hierarchy nodes                | `modTestTree`, `clsTestNode`                                  |
| Change legacy routing                | `modRouting`, `modCatalog`                                    |
| Change Ribbon UI                     | `customUI14(TS_DB).xml`                                       |
| Change Ribbon behavior               | `modRibbon`                                                   |
| Change record search/filter behavior | `modRecordSearch`, `modRibbon`                                |
| Change picker screen positioning     | `modPositioning`                                              |
| Investigate Sheet View behavior      | `modSheetView`, `modRecordSearch`                             |
| Recover stale workbook state         | `modRibbon`, `modRecordSearch`                                |

---

# Root Files

## `TS_Data_Workbook`

**Purpose:**
Code-behind for the `TS Data` worksheet.

**Contains:**

* `Worksheet_Change` processing.
* Watches UUT and Test hierarchy entry fields.
* Synchronizes hidden Node IDs after visible edits.
* Reconciles downstream hierarchy fields.
* Clears invalid descendants after parent changes.
* Applies semantic worksheet themes.
* Handles bulk clears without repeatedly reconciling the same row.
* `Worksheet_BeforeDoubleClick` entry point for custom picker fields.
* Maps worksheet columns to logical field names.

**Primary dependencies:**

* `modPicker`
* `modState`
* `modTheme`

**Typical flow:**

```text
User changes TS Data cell
    ↓
Worksheet_Change
    ↓
SyncNodeIdentityForField
    ↓
ReconcileAfterChange
    ↓
ApplyThemeToCell
```

---

## `customUI14(TS_DB).xml`

**Purpose:**
RibbonX definition for the workbook-specific `Troubleshooting` Ribbon tab.

**Contains:**

* Search group.
* Free-text record-search edit box.
* Clear Search button.
* LRU preset toggle buttons:

  * BMPG
  * RMPG
  * SIM
  * OCP
  * OPU
* Test preset toggle buttons:

  * Functional
  * Thermal
  * ATP
* Navigation group.
* TS Data navigation button.
* Debug group.
* Reset State button.
* Callback mappings into `modRibbon`.

**Behavior implementation:**
`Modules/modRibbon`

Do not implement business logic in the XML. Ribbon XML defines controls and callback bindings only.

---

# Forms

## `Forms/frmPicker`

**Purpose:**
Modeless custom selector used in place of native Excel dropdowns for controlled entry fields.

**Contains:**

* Picker initialization.
* Public picker load/commit interface.
* Search textbox behavior.
* Search placeholder.
* Breadcrumb/context display.
* Dynamic item generation.
* String catalog item support.
* `clsTestNode` hierarchy item support.
* Dynamic picker width and height.
* Dynamic breadcrumb wrapping.
* Scrollable item panel.
* Mouse hover handling.
* Keyboard navigation.
* Up/down wrap-around.
* Enter-to-select.
* Escape-to-close.
* Current-value/current-Node-ID selection detection.
* Semantic picker theming.
* Hidden Node ID write before visible value commit.

**Important behavior:**

For tree-backed fields:

```text
Write hidden Node ID
    ↓
Write visible value
    ↓
Worksheet_Change executes with correct identity already present
```

**Primary dependencies:**

* `clsPickerItem`
* `clsTestNode`
* `modUITheme`
* `modTheme`
* `modPositioning`
* `modState`

---

# Class Modules

## `Class_Modules/clsPickerItem`

**Purpose:**
Event-enabled wrapper around one dynamically generated picker item.

**Contains:**

* `WithEvents` item Label.
* Accent/selection rail reference.
* Item display value.
* Internal item key / Node ID.
* Parent `frmPicker` reference.
* Baseline themed appearance.
* Selected state.
* Hover state.
* Hover color transformation.
* Click forwarding to `frmPicker.CommitValue`.
* Mouse movement forwarding to `frmPicker.HoverItem`.

**Important behavior:**

* Captures the already-themed card appearance during initialization.
* Hover and selection effects are derived from that baseline.
* Selection and hover use `modUITheme` color utilities.

---

## `Class_Modules/clsTestNode`

**Purpose:**
Data object representing one generated Test hierarchy node.

**Contains:**

* Test
* Node ID
* Parent ID
* Label
* Depth
* Sort Order

**Used by:**

* `modTestTree`
* `modPicker`
* `frmPicker`
* `modState`

Contains data only. No hierarchy traversal logic.

---

# Standard Modules

## `Modules/modCatalog`

**Purpose:**
Read controlled-entry values from Excel catalog tables.

**Contains:**

* Resolve a table/column to its data range.
* Return nonblank catalog values as a `Collection`.
* Resolve legacy route targets through `tbl_Routes`.
* Locate catalog tables in the workbook.
* Catalog-engine diagnostic tests.

**Primary consumers:**

* `modPicker`
* Legacy routing paths.

**Distinction:**

* `modCatalog` reads values.
* `modRouting` decides which catalog should be used.

---

## `Modules/modPicker`

**Purpose:**
Determine which picker data source belongs to a selected `TS Data` cell and launch `frmPicker`.

**Contains:**

* UUT picker routing.
* Test picker routing.
* Test Step root-node routing.
* Sub Step hierarchy routing.
* Tree-backed selection preference.
* Legacy `tbl_Routes` fallback.
* Parent Node ID lookup.
* Picker breadcrumb construction.
* Picker load, positioning, and display.

**Supported visible fields:**

```text
B = UUT
D = Test
E = Test Step
F = Sub Step 1
G = Sub Step 2
H = Sub Step 3
I = Sub Step 4
```

**Important transitional behavior:**

```text
Tree hierarchy exists
    → use tbl_TestHierarchy

Tree hierarchy unavailable
    → fall back to tbl_Routes
```

Thermal and ATP currently depend on legacy fallback until migrated.

---

## `Modules/modPositioning`

**Purpose:**
Position `frmPicker` near the selected worksheet cell while respecting Excel window boundaries and DPI scaling.

**Contains:**

* Windows `GetDpiForWindow`.
* Pixel-to-point conversion.
* Cell screen-coordinate lookup.
* Preferred right-side picker placement.
* Left-side fallback when insufficient space exists.
* Vertical and horizontal Excel-window boundary constraints.
* DPI fallback.

**Important distinction:**

* Excel screen-location APIs return pixels.
* UserForm positioning uses points.

---

## `Modules/modRecordSearch`

**Purpose:**
Record-search engine and private Sheet View lifecycle for `TS Data`.

**Contains:**

* Free-text multi-word search.
* LRU preset filter state.
* Test preset filter state.
* Search state accessors.
* Search-owned hidden-row tracking.
* In-memory record matching.
* Temporary Sheet View creation.
* Temporary Sheet View reuse.
* Default-view restoration.
* Search clearing.
* Hard recovery/reset support.
* Record-search test harness.

**Search semantics:**

```text
Free text:
term1 AND term2 AND term3
```

Each term may occur anywhere in the same record.

Preset filters:

```text
LRU  → exact match against UUT
Test → exact match against Test
```

Combined criteria are ANDed.

Example:

```text
Query: 112
LRU: RMPG
Test: Functional

Result:
record contains 112
AND UUT = RMPG
AND Test = Functional
```

**Sheet View behavior:**

* Searches operate in a Temporary Sheet View to avoid disrupting other viewers.
* Record Search tracks only row blocks it hides.
* Normal clearing restores only Record Search-owned row hiding.
* Hard reset intentionally restores all populated record rows.

**Known Excel behavior:**

* Exiting an unsaved Temporary Sheet View displays `Keep this view`.
* `Application.DisplayAlerts = False` does not suppress that prompt.
* `Alt+N` selects `Don't Keep`.
* Working implementation queues `%n` through `Application.SendKeys` immediately before `ExecuteMso "ExitSheetView"`.

---

## `Modules/modRibbon`

**Purpose:**
VBA callback layer for `customUI14(TS_DB).xml`.

**Contains:**

* Ribbon initialization.
* Search edit-box callbacks.
* Clear Search callbacks.
* LRU toggle callbacks.
* Test toggle callbacks.
* Toggle pressed-state callbacks.
* TS Data navigation.
* Debug Reset State.
* Deferred preset-filter execution.
* Ribbon invalidation/refresh.
* `Application.OnTime` scheduling.
* Cancellation of pending preset searches.

**Important RibbonX behavior:**

Toggle button rendering can remain visually stale when filtering executes directly inside `onAction`.

Working pattern:

```text
Ribbon click
    ↓
Update logical VBA filter state
    ↓
Invalidate toggle controls
    ↓
Schedule filtering with Application.OnTime
    ↓
Return from Ribbon callback
    ↓
Excel repaints controls
    ↓
Deferred search executes
```

Do not casually replace this with synchronous filtering or `DoEvents` inside toggle callbacks.

**Search editBox quirk:**

* `getText` can correctly return an empty string while the live Ribbon editBox continues displaying previously entered text.
* Logical search state is authoritative.
* Debug Reset State is the recovery mechanism for stale UI state.

---

## `Modules/modRouting`

**Purpose:**
Legacy parent-to-child catalog routing engine.

**Primary data source:**
`tbl_Routes`

**Contains:**

* Route lookup by:

  * Parent Field
  * Parent Value
  * Child Field
* Routed List Table / List Column output.
* Duplicate-route detection.
* Route-to-range resolution.
* Case-insensitive trimmed comparisons.
* Workbook table lookup.
* Routing test harness.

**Current role:**
Compatibility layer for Test hierarchies not yet migrated to `tbl_TestHierarchy`.

Do not remove until Thermal and ATP migration is complete.

---

## `Modules/modSheetView`

**Purpose:**
Diagnostic test module for Excel Sheet View capabilities used by Record Search.

**Contains:**

* Test creation of a Temporary Sheet View through `ExecuteMso`.
* Test Sheet View exit.
* Test silent Temporary View discard using queued `Alt+N`.
* Command enabled-state verification.

**Production behavior:**
Production Sheet View handling is in `modRecordSearch`.

This module exists primarily for capability/regression testing.

---

## `Modules/modState`

**Purpose:**
Maintain valid row state across dependent Test hierarchy fields.

**Contains:**

* `ChildState` enumeration:

  * Blank
  * Routed
  * N/A
* Tree-backed child-state resolution.
* Legacy-route child-state fallback.
* Downstream reconciliation after parent changes.
* Descendant clearing.
* Automatic `-` propagation for hierarchy leaves.
* Hidden Node ID synchronization.
* Visible-field hierarchy mapping.
* Visible-column mapping.
* Hidden Node-ID-column mapping.
* State-engine test harness.

**Visible hierarchy:**

```text
Test
  ↓
Test Step
  ↓
Sub Step 1
  ↓
Sub Step 2
  ↓
Sub Step 3
  ↓
Sub Step 4
```

**Hidden identity mapping:**

```text
Test Step  → AA
Sub Step 1 → AB
Sub Step 2 → AC
Sub Step 3 → AD
Sub Step 4 → AE
```

**Important behavior:**

* Changing an upstream field invalidates descendants.
* Tree leaf selections populate remaining subordinate fields with `-`.
* `-` never carries a Node ID.
* Manually edited visible values invalidate stale hidden Node IDs.

---

## `Modules/modTestHeirarchy`

**Purpose:**
Build `tbl_TestHierarchy` from the indentation-based source outline in `tbl_TestOutline`.

**Contains:**

* Source/output table lookup.
* Exact column mapping.
* Existing Node ID collection.
* Duplicate Node ID detection.
* Stable generated Node IDs.
* Two-pass rebuild.
* Outline parsing.
* Indentation validation.
* Parent resolution.
* Sort-order generation.
* Source-row tracking.
* Active branch stack by hierarchy depth.
* Generated hierarchy table rewrite.
* Excel event/screen-state preservation.

**Input table:**

```text
tbl_TestOutline
```

Important columns:

```text
Test
Node ID
Raw Line
```

**Output table:**

```text
tbl_TestHierarchy
```

Important columns:

```text
Test
Node ID
Parent ID
Label
Depth
Sort Order
Source Row
```

**Outline syntax:**

* Depth 0 has no leading spaces and no bullet.
* Each leading ASCII space adds one hierarchy level.
* Indented lines use `-` after indentation.
* Tabs are rejected.
* Depth may increase by at most one level between adjacent nodes.

**Generated ID format:**

```text
TN-000001
TN-000002
...
```

Existing IDs are preserved across rebuilds.

---

## `Modules/modTestTree`

**Purpose:**
Read/query the generated Test hierarchy as `clsTestNode` objects.

**Contains:**

* Root-node lookup by Test.
* Direct-child lookup by Parent Node ID.
* Child-existence test.
* Node lookup by Node ID.
* Hierarchy-row → `clsTestNode` construction.
* Hierarchy table lookup.
* Tree-engine test harness.

**Primary consumers:**

* `modPicker`
* `modState`
* `frmPicker`

**Important distinction:**

* `modTestHeirarchy` builds the hierarchy.
* `modTestTree` reads/traverses the hierarchy.

---

## `Modules/modTheme`

**Purpose:**
Semantic data-theme lookup and worksheet-cell theme application.

**Primary data source:**

```text
tbl_Theme
```

**Contains:**

* Theme-rule lookup.
* Exact matching.
* Starts With matching.
* Match specificity scoring.
* Hex color parsing.
* Worksheet-cell theme application.
* Worksheet-cell theme reset.
* Theme table lookup.
* Theme-engine test harness.

**Matching precedence:**

```text
Exact
    outranks
Starts With

Within Starts With:
longer matching prefix
    outranks
shorter matching prefix
```

**Primary consumers:**

* `TS_Data_Workbook`
* `modState`
* `frmPicker`
* `modUITheme`

**Responsibility boundary:**

* `modTheme` owns semantic data colors and worksheet formatting.
* `modUITheme` owns application/picker UI appearance.

---

## `Modules/modUITheme`

**Purpose:**
Central UI palette, picker geometry constants, typography, and picker-specific color behavior.

**Contains:**

* Application caption.
* UI fonts.
* Font sizes.
* Hover/lighten constants.
* Accent/border contrast constants.
* Picker item geometry.
* Dynamic-size limits.
* Window margins.
* Default UI palette.
* Default picker-card palette.
* Picker semantic-theme application.
* UI color lighten/darken/blend utilities.

**Important behavior:**

* `ApplyUIPickerTheme` reads semantic colors through `modTheme.TryGetTheme`.
* `tbl_Theme.Bold` is deliberately ignored for picker typography.
* Worksheet-cell Bold remains controlled by `modTheme`.

**Primary consumers:**

* `frmPicker`
* `clsPickerItem`

---

# Runtime Flow Index

## Controlled Entry

```text
TS Data double-click
    ↓
TS_Data_Workbook.Worksheet_BeforeDoubleClick
    ↓
modPicker.OpenPickerForCell
    ↓
modCatalog or modTestTree
    ↓
frmPicker.LoadPicker
    ↓
clsPickerItem instances
    ↓
frmPicker.CommitValue
    ↓
TS_Data_Workbook.Worksheet_Change
    ↓
modState
    ↓
modTheme
```

---

## Test Hierarchy

```text
tbl_TestOutline
    ↓
modTestHeirarchy.RebuildTestHierarchy
    ↓
tbl_TestHierarchy
    ↓
modTestTree
    ↓
clsTestNode
    ↓
modPicker / modState / frmPicker
```

---

## Legacy Test Routing

```text
parent field/value
    ↓
modRouting
    ↓
tbl_Routes
    ↓
catalog table + column
    ↓
modCatalog
    ↓
picker Collection
```

Current use is transitional for Test protocols not yet migrated to the tree hierarchy.

---

## Record Search

```text
customUI14(TS_DB).xml
    ↓
modRibbon
    ↓
modRecordSearch
    ↓
Temporary Sheet View
    ↓
in-memory row matching
    ↓
private row visibility
```

Preset toggle execution is deferred through `Application.OnTime`.

---

## Semantic Theme

```text
tbl_Theme
    ↓
modTheme.TryGetTheme
    ├─ worksheet → ApplyThemeToCell
    └─ picker → modUITheme.ApplyUIPickerTheme
```

---

# Workbook Data Structures Referenced by Code

| Object              | Purpose                              |
| ------------------- | ------------------------------------ |
| `lst_TS_Data`       | Primary troubleshooting record table |
| `lst_UUT`           | Controlled UUT catalog               |
| `lst_Test`          | Controlled Test catalog              |
| `tbl_Theme`         | Semantic color/style rules           |
| `tbl_Routes`        | Legacy dependent-list routing        |
| `tbl_TestOutline`   | Source Test hierarchy outline        |
| `tbl_TestHierarchy` | Generated normalized hierarchy       |

Legacy route-specific list tables may also exist and are consumed through `modCatalog`.

---

# Architectural Boundaries

```text
Workbook events
    → TS_Data_Workbook

Picker routing
    → modPicker

Picker presentation
    → frmPicker / clsPickerItem / modUITheme

Semantic data colors
    → modTheme

Hierarchy generation
    → modTestHeirarchy

Hierarchy traversal
    → modTestTree / clsTestNode

Dependent state
    → modState

Legacy list routing
    → modRouting / modCatalog

Search state and filtering
    → modRecordSearch

Ribbon callback orchestration
    → modRibbon

Ribbon declaration
    → customUI14(TS_DB).xml

Window placement
    → modPositioning

Sheet View diagnostics
    → modSheetView
```

---

# Known Transitional / Non-Obvious Constraints

* Functional uses `tbl_TestHierarchy`.
* Thermal and ATP retain legacy `tbl_Routes` fallback until migrated.
* Hidden Node IDs must remain synchronized with visible hierarchy values.
* Node ID is written before visible picker value for tree-backed fields.
* Search operates inside a Temporary Sheet View.
* Search row hiding cannot rely on Sheet View exit to restore `.Hidden`.
* Sheet View discard uses queued `Alt+N` before `ExitSheetView`.
* Ribbon preset filters must defer substantive work until `onAction` returns.
* Ribbon editBox displayed text may remain stale even when logical query state is blank.
* Avoid persistent Windows mouse hooks for UserForm behavior.
* Debug `Reset State` exists to restore transient workbook/UI state during development.

---
