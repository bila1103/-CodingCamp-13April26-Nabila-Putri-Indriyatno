# Implementation Plan: Expense & Budget Visualizer

## Overview

Build a single `index.html` file containing all HTML, CSS, and inline JavaScript. The implementation follows the MVC pattern: `Store` (localStorage), `Transactions` (pure functions), `View` (DOM + Canvas), and `Controller` (event wiring). No build step, no frameworks, no external libraries except fast-check for tests.

## Tasks

- [x] 1. Scaffold the HTML structure and CSS layout
  - Create `index.html` with semantic HTML regions: summary bar, add-transaction form, chart area, category filter, and transaction list
  - Add responsive CSS using a single-column layout on narrow viewports and a two-column layout (chart + list side-by-side) at ≥768px
  - Ensure all interactive controls (buttons, inputs, selects) have a minimum 44×44 CSS pixel touch target
  - Add a `<canvas id="chart">` element with a fallback text node for browsers that do not support Canvas
  - Add a hidden warning banner element for localStorage errors
  - _Requirements: 1.1, 2.1, 3.3, 5.4, 7.3, 8.1, 8.2, 8.3_

- [x] 2. Implement the `Store` module
  - [x] 2.1 Write the `Store` object with `load()`, `save(txns)`, and `isAvailable()` methods
    - `load()` reads `"ebv_transactions"` from localStorage, parses JSON, and returns `Transaction[]`; catches parse errors and returns `[]`
    - `save(txns)` serializes the array to JSON and writes it to localStorage
    - `isAvailable()` probes localStorage with a test write/read/delete and returns a boolean
    - _Requirements: 7.1, 7.2, 7.3_

  - [x] 2.2 Write property test for transaction persistence round-trip (Property 2)
    - **Property 2: Transaction persistence round-trip**
    - Generate arbitrary valid transaction arrays; assert `JSON.parse(JSON.stringify(txns))` deep-equals the original array (same ids, amounts, types, categories, dates, descriptions)
    - **Validates: Requirements 7.1, 7.2**

- [x] 3. Implement the `Transactions` pure-function module
  - [x] 3.1 Implement `Transactions.add(txns, tx)` — returns a new array with `tx` appended
    - _Requirements: 2.2, 3.2_

  - [x] 3.2 Write property test for add-then-contains (Property 3)
    - **Property 3: Add then contains**
    - Generate arbitrary list + valid new transaction; assert the result of `add(list, tx)` contains an entry with the same id, description, amount, category, type, and date
    - **Validates: Requirements 2.2, 3.2**

  - [x] 3.3 Implement `Transactions.remove(txns, id)` — returns a new array with the matching id excluded
    - _Requirements: 4.2_

  - [x] 3.4 Write property test for delete-removes-exactly-one (Property 4)
    - **Property 4: Delete removes exactly one**
    - Generate a list with at least one transaction; assert `remove(list, id)` excludes that id and all other transactions remain unchanged
    - **Validates: Requirements 4.2**

  - [x] 3.5 Implement `Transactions.filter(txns, cat)` — returns transactions matching `cat`; returns full list when `cat === "All"`
    - _Requirements: 6.2, 6.3_

  - [x] 3.6 Write property test for category filter subset invariant (Property 5)
    - **Property 5: Category filter subset invariant**
    - Generate list + category string (including "All"); assert `filter(list, cat)` returns only transactions whose category equals `cat`; assert `filter(list, "All")` returns the complete list
    - **Validates: Requirements 6.2, 6.3**

  - [x] 3.7 Implement `Transactions.summary(txns)` — returns `{ balance, income, expenses }` derived from the transaction array
    - _Requirements: 1.2, 1.3, 1.4_

  - [x] 3.8 Write property test for balance and summary invariant (Property 1)
    - **Property 1: Balance and summary invariant**
    - Generate arbitrary arrays of valid transactions; assert `summary(txns).balance === incomeSum - expenseSum`, `summary(txns).income === incomeSum`, `summary(txns).expenses === expenseSum`
    - **Validates: Requirements 1.2, 1.3, 1.4**

  - [x] 3.9 Implement `Transactions.byCategory(txns)` — returns a `Map<string, number>` of expense totals per category
    - _Requirements: 5.1, 5.3_

  - [x] 3.10 Write property test for category map totals correctness (Property 6)
    - **Property 6: Category map totals correctness**
    - Generate arbitrary transaction arrays; assert `sum(byCategory(txns).values()) === summary(txns).expenses` and each category key maps to the correct sum of expense amounts for that category
    - **Validates: Requirements 5.1, 5.3**

- [-] 4. Implement form validation
  - [x] 4.1 Write a `validate(formData)` function that checks all required fields (description, amount, category, type, date) for emptiness/whitespace and validates amount is a positive finite number; returns `null` on success or `{ field, message }` on failure
    - _Requirements: 2.3, 2.4_

  - [x] 4.2 Write property test for validation rejects all invalid inputs (Property 7)
    - **Property 7: Validation rejects all invalid inputs**
    - Generate form objects with blank/whitespace required fields OR invalid amounts (0, negative, NaN, Infinity, non-numeric strings); assert `validate(formData)` returns an error object identifying the offending field and that the transaction list remains unchanged
    - **Validates: Requirements 2.3, 2.4**

- [x] 5. Checkpoint — Ensure all pure-function tests pass
  - Run the test file and confirm all unit and property tests for `Store`, `Transactions`, and `validate` pass before proceeding to UI work
  - Ensure all tests pass, ask the user if questions arise.

- [x] 6. Implement the `View` module
  - [x] 6.1 Implement `View.renderSummary(summary)` — updates the balance, income, and expenses display in the summary bar
    - _Requirements: 1.1, 1.2, 1.3, 1.4_

  - [x] 6.2 Implement `View.renderList(txns)` — rebuilds the transaction list DOM; shows an empty-state message when `txns` is empty; renders each entry with description, amount, category, type, date, and a delete button
    - _Requirements: 3.1, 3.2, 3.3, 3.4, 4.1_

  - [x] 6.3 Implement `View.renderFilter(categories, selected)` — rebuilds the category filter `<select>` with an "All" option plus one option per unique category; preserves the currently selected value
    - _Requirements: 6.1, 6.4_

  - [x] 6.4 Implement `View.showFormError(field, message)` and `View.clearForm()` — displays inline validation errors beneath the relevant field and resets all form fields
    - _Requirements: 2.3, 2.4, 2.5_

- [x] 7. Implement the `Chart` module
  - [x] 7.1 Implement `Chart.draw(canvas, categoryMap)` — renders a pie/donut chart on the Canvas 2D context; labels each segment with category name and percentage; shows a placeholder message when `categoryMap` is empty; hides the chart area and shows a static fallback when Canvas is not supported
    - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

  - [x] 7.2 Implement `View.renderChart(categoryMap)` — calls `Chart.draw` with the current canvas element and category map
    - _Requirements: 5.1, 5.2_

- [x] 8. Implement the `Controller` and wire everything together
  - [x] 8.1 Write the `Controller.init()` function called on `DOMContentLoaded`
    - Load transactions from `Store.load()`; check `Store.isAvailable()` and show the warning banner if false
    - Call all `View.render*` functions with the initial state
    - _Requirements: 7.2, 7.3_

  - [x] 8.2 Wire the add-transaction form `submit` event
    - Run `validate(formData)`; if invalid call `View.showFormError` and return
    - Build a transaction object with `crypto.randomUUID()` (falling back to `Date.now() + Math.random()`)
    - Call `Transactions.add`, `Store.save`, then re-render summary, list, filter, and chart
    - Call `View.clearForm()` and return focus to the description field
    - _Requirements: 2.2, 2.3, 2.4, 2.5, 6.4_

  - [x] 8.3 Wire the transaction list delete button `click` event (event delegation)
    - Call `Transactions.remove`, `Store.save`, then re-render summary, list, filter, and chart
    - _Requirements: 4.2, 4.3_

  - [x] 8.4 Wire the category filter `change` event
    - Call `Transactions.filter` with the selected value and re-render the list only
    - _Requirements: 6.2, 6.3_

- [x] 9. Final checkpoint — End-to-end smoke check
  - Open `index.html` directly in a browser via `file://` and verify: adding a transaction updates the summary, list, and chart; deleting a transaction removes it and updates all views; reloading the page restores all transactions from localStorage; the layout renders without horizontal scroll at 320px, 768px, 1440px, and 2560px viewport widths
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Property tests use [fast-check](https://github.com/dubzzz/fast-check); tag each test with `// Feature: expense-budget-visualizer, Property N: <property_text>`
- All pure functions (`Transactions.*`, `validate`) are testable in isolation without a DOM
- The entire app ships as a single `index.html` — no build step, no npm, no bundler
- Property tests and unit tests can live in a separate `tests/` directory loaded via `<script type="module">` or run with a test runner that supports ESM
