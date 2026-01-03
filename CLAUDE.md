# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Smart Parser is a single-page, pure front-end table parsing/filtering/joining tool. It requires zero dependencies and runs completely offline in the browser. The entire application is contained in a single `index.html` file with embedded CSS and vanilla JavaScript.

## Development

### Running the Application
```bash
# No build process required - simply open in browser
open index.html

# Or use a simple HTTP server for testing
python -m http.server 8000
# Then visit http://localhost:8000
```

### File Structure
- `index.html` - Single-file application containing HTML, CSS, and all JavaScript logic (~2,500 lines)
- `README.md` - Chinese language documentation

## Architecture

### Core Modules (all in index.html)

1. **Store** (lines 776-810)
   - Central state management persisted to `localStorage`
   - Key: `v16_4_store` in browser localStorage
   - Manages multiple tabs/docs, each with independent raw data, UI rules, and configuration
   - Handles theme switching and global views

2. **Parser** (lines 812-860)
   - Parses text input into structured table data
   - Supports multiple formats: tab-separated (TAB), comma-separated (CSV), fixed-width (FIXED), whitespace-delimited (WS)
   - Auto-detects format based on `validflag` header line
   - Tables are delimited by `table-data <tablename>` markers

3. **Joiner** (lines 862-955)
   - Executes JOIN operations (inner/left) between tables or views
   - Supports recursive view references with cycle detection
   - Parses select tokens with aliases (e.g., `left.col AS Alias`)
   - Provides join statistics (matched rows, left/right-only rows)

4. **JoinEditor** (lines 957-1800+)
   - Modal UI for managing join view configurations
   - Drag-and-drop interface for column ordering
   - Field mapping and alias configuration
   - Views stored globally in `Store.state.globalViews`

5. **Exporter** (lines ~600-774)
   - Generates Excel (.xlsx) files as pure JavaScript ZIP implementation
   - Creates proper OOXML structure (Content_Types, workbook, worksheets)
   - Exports JSON for backup/restore
   - Supports TSV/HTML clipboard copy for selected cells

6. **App** (lines ~1800+)
   - Main application controller
   - `proc()` method applies filters, highlights, and column focus rules
   - Renders preview tables with inline editing support
   - Manages cell selection popover for column filtering

### Data Flow

1. User pastes raw text into "数据源" (Data Source) textarea
2. Click "解析" (Parse) → Parser extracts tables
3. Tables stored in current tab's `raw` field in Store
4. `App.proc()` applies UI rules:
   - Global filter (all tables)
   - Table-level filter (current target table)
   - Highlight rules
   - Column filters (per-column search)
   - Focus columns (show only selected columns)
5. Results rendered via `App.renderPreview()`
6. Export buttons call `Exporter.toExcel()`

### Filter Rule Syntax

Rules support:
- **Exact match**: `key=value`, `key!=value`
- **Contains**: `Msg:Error` (colon is contains operator)
- **Numeric comparison**: `CPU>90`, `Score<=60`
- **Regex**: `/timeout|error/`
- **Logic**: Space = AND, Pipe (`|`) = OR

### localStorage Schema

```javascript
{
  docs: [
    {
      id: string,
      title: string,
      raw: string,  // original input text
      ui: {
        displayTables: string[] | null,  // null = show all
        enabledViews: string[],
        targetTable: string,
        rules: { [tableName]: { filter: string, hl: string, focus: string[] } },
        columnFilters: { [tableName]: { [columnName]: string } },
        collapsedTables: { [tableName]: boolean },
        // ... other UI state
      }
    }
  ],
  activeId: string,
  theme: 'light' | 'dark',
  globalViews: [
    {
      view: string,  // view name
      left: string,  // left table/view name
      right: string, // right table/view name
      type: 'inner' | 'left',
      on: string,    // e.g., "id=userId,name=userName"
      select: string // comma-separated output columns
    }
  ]
}
```

### Key Implementation Details

- **No external dependencies** - All code including ZIP/Excel generation is handwritten vanilla JS
- **Chinese language UI** - All labels and messages are in Chinese
- **Version-specific localStorage key** - `v16_4_store` allows for future migrations
- **Table name prefix** - JOIN views are prefixed with `JOIN:` to distinguish from raw tables
- **Smart whitespace parsing** - Parser tries whitespace split first, falls back to detected mode if column count mismatch

### Making Changes

When modifying `index.html`:
1. Keep all code in the single file structure
2. Maintain localStorage backward compatibility or update the storage key version
3. Preserve Chinese language UI text
4. Test both light and dark themes
5. Verify offline functionality (no external CDNs)
