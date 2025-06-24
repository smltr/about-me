# 2a.md - SheetRocks Deep Dive

## My Role at SheetRocks (2021-2023)

  **Context:**
    - Founding engineer & first hire at seed-stage startup
    - Web-based spreadsheet app to compete with Google Sheets & Excel
    - Worked directly with technical founder
    - Worked on every area of project

  **Tech Stack:**
    - Frontend: React, TypeScript, Jest, SCSS
    - Backend: Go REST API & Calculation engine, MongoDB for metadata, Redis for spreadsheet data
    - Infra: AWS EC2, S3, blue-green deployments
    - GitHub, Docker

  **My Responsibilities:**
    - Own feature dev end-to-end: design → implementation → testing → deployment
    - Big fixes, updates
    - Manage deployment pipeline and production releases
    - Provide on-call support

## Examples of Major Features I Owned

  **1. Formula Range Selection**
    - **Problem:** Users typing formulas need visual way to select cell ranges
    - `=SUM(` -> user clicks on cell A1 -> `=SUM(A1`
    - **Solution:**
      - Click-and-drag to visually select ranges, auto-populating formula text
      - Real-time highlighting of cells as users type range references
      - Complex state management between formula editor and spreadsheet grid
    - **Technical challenges:**
      - Parsing formulas in real-time, had to ensure frontend JS logic matched backend Go logic for references (A1:A5)
      - Ensuring real-time performance using selective rendering

    Picture -> pics/range-selection.png

  pseudocode:

  ```javascript
    // state represents react state
    class Sheet {
      state = {
        selectedRanges: [
          { type: 'normal', range: 'A1:B3' },
          { type: 'formulaReference', range: 'C1:C5' }
        ],
        selectionInProgress: true,
        selectionType: 'formulaReference',
        cellReferenceSelectionRanges: ['A1:A5', 'C1:C3'],
        cells: {
          'A1': { data: '100', styles: { color: '#000' } },
          'B1': { data: '200', styles: { merged: true } }
          // ...
        }
      }

      render() {
        return rows.map(rowIndex =>
          <Row
            rowIndex={rowIndex}
            selectedRanges={this.state.selectedRanges}
            cellReferenceRanges={this.state.cellReferenceSelectionRanges}
            selectionInProgress={this.state.selectionInProgress}
            cells={this.getCellsForRow(rowIndex)}
          />
        )
      }
    }

    class Row {
      shouldComponentUpdate(nextProps) {
        // Only re-render if this row contains newly selected cells
        return this.rowHasSelectionChanges(nextProps.selectedRanges) ||
                this.rowHasFormulaReferences(nextProps.cellReferenceRanges)
      }

      render() {
        return columns.map(colIndex => {
          const cellRef = `${colIndex}${this.props.rowIndex}`

          return <Cell
            cellRef={cellRef}
            data={this.props.cells[cellRef]}
            isSelected={this.isCellInRanges(cellRef, this.props.selectedRanges)}
            isFormulaRef={this.isCellInRanges(cellRef, this.props.cellReferenceRanges)}
            selectionInProgress={this.props.selectionInProgress}
          />
        })
      }
    }

    class Cell {
      shouldComponentUpdate(nextProps) {
        // Only re-render if this specific cell's selection state changed
        // of course there are many other cases such as data being changed, but not representing that here
        // much more complicated selection style in reality as well, depends on if cell is on outer edge of selection, in middle, what type of selection, etc
        return nextProps.isSelected !== this.props.isSelected ||
                nextProps.isFormulaRef !== this.props.isFormulaRef ||
                nextProps.data !== this.props.data
      }

      render() {
        const className = [
          'cell',
          this.props.isSelected ? 'selected' : '',
          this.props.isFormulaRef ? 'formula-reference' : '',
          this.props.selectionInProgress ? 'selecting' : ''
        ].join(' ')

        return <div className={className}>{this.props.data}</div>
      }
    }

    function isCellInRanges(cellRef, ranges) {
      // for each range
      // Convert "A1:B5" and "C3" into row/col comparison
      // Return true if C3 falls within A1:B5 bounds
    }

  ```

  **2. Cross-Sheet References**
    - **Problem:** Users need to reference data from other sheets in workbook (like `=Sheet2!A1:B5`)
    - **Solution:**
      - Extended formula parsing to handle sheet references
      - Backend API changes to resolve cross-sheet dependencies
    - **Technical challenges:**
      - Recalculation chain
      - Front end parsing (don't highlight cells from this sheet)

    Picture -> pics/cross-sheet-reference.png

  **3. Macros & Automations System**
    - **Problem:** Programmable spreadsheet actions and scheduled tasks
    - **Solution:**
      - JavaScript execution via Docker
      - Container images saved for period of time for reuse
      - Macro recording: capture user actions → generate automation scripts
      - Visual code editor using Monaco
      - Scheduling system for automated runs
    - **Technical challenges:**
      - Security isolation with Docker
      - Determining which UI state to capture for macro recording
      - Injecting spreadsheet data API to JavaScript runtime
      - Scheduling and testing automations run as expected

    Picture -> pics/automation-editor.png

  **4. Data Sorting Algorithm**
    - **Problem:** Sort spreadsheet data by columns (my first major feature)
    - **Solution:**
      - Research on Google Sheets sorting behavior
      - Backend sorting algorithm handling mixed data types
      - Frontend UI for sort configuration
    - **Technical challenges:**
      - Handling different data types
      - Sorting by multiple columns

    Picture -> pics/sort.png

  **5. Cell Merging**
    - **Problem:** Users need to merge cells for formatting/layout
    - **Solution:**
      - Data model changes for merged cell representation
      - Frontend rendering logic for merged cells
      - Formula calculation adjustments
    - **Technical challenges:** Representing merged cells in both frontend state and backend storage

    Picture -> pics/merge.png

  **6. Dropdowns/Data validation:**
    - **Problem:** Data validation via dropdowns
    - **Solution:**
      - Formula calculation adjustments with new parameter representing validations on backend
      - Added visual element to select an item from a dropdown
    - **Technical challenges:**
      - rendering dropdown in correct position in relation to parent cell
      - data integrity

    Picture -> pics/dropdowns.png

  **7. Order of operations:**
    - **Problem:** Ensure math ops in formulas were calculated in expected order
    - **Solution:**
      - Crawled node tree for formula and reorganized
      - Performed crawl on formula parse so that later calculations wouldn't be affected
    - **Technical challenges:** It took me a while to grok how the formula parser and resulting node tree worked

    Picture -> pics/order-of-ops.png

    Diagram:

      Formula: = 1 + 2 * 3

      BEFORE (naive left-to-right parsing):
          MULTIPLY
         /        \
        ADD        3
       /   \
      1     2

      This would calculate: (1 + 2) * 3 = 9 which is wrong


      AFTER (order of operations fix):
            ADD
           /   \
          1    MULTIPLY
               /       \
              2         3

      This calculates: 1 + (2 * 3) = 7 ✅


      TRANSFORMATION ALGORITHM:
      1. Crawl tree from root
      2. If inner node has higher precedence than outer node:
         - Swap child ↔ parent relationship

      Step-by-step for our example:

         Original:           After swap:
         MULTIPLY            ADD
         /      \           /   \
        ADD      3    →    1    MULTIPLY
       /   \                   /       \
      1     2                 2         3

         │
         └─ Detected: ADD (precedence 1) inside MULTIPLY (precedence 2)
            └─ Swap: Make ADD the parent, move its right child to MULTIPLY's left

      The key: "bubbling up" higher precedence operations by swapping parent-child relationship.

      Nodes were a generic interface all having a `Calculate()` method defined in Go.

## Other Areas I Contributed To

  Other areas of product I worked on:

  - Multi-threaded calculation engine
  - API docs
  - End-to-end testing for all new features and bug fixes
  - Added to built in formula library
  - Formula parser logic
  - Clipboard compatibility with other spreadsheet apps
  - Cell formatting and styling systems such as freezing row/columns
  - Adding, deleting, copy/cut/pasting data in a sheet
  - Authentication and permissions
    - Sharing workbooks/sheets, Edit vs. Read-Only access

  Other misc.

  - Helping new hires ramp up on codebase
  - Contributed to discussions around company vision and roadmap
  - User interviews/focus group via video chat

## Working style

  - Fully remote position
  - Met with team in person on a few occasions
  - Weekly stand ups, daily communication via slack/video chat

## Summary

  - Seed stage, worked alongside technical founder
  - Full stack: Go/React/TS
  - Owned features
  - Wore many hats
  - 2 years of hands-on experience

Next, deep dive on FindServers.net -> findservers.md
