# Design Document: Expense & Budget Visualizer

## Overview

The Expense & Budget Visualizer is a single-file, client-side web application built with HTML, CSS, and Vanilla JavaScript. It runs entirely in the browser with no server dependency, persisting all data in `localStorage`. The app provides a financial summary (balance, income, expenses), a transaction form, a scrollable transaction history with category filtering, and a pie/donut chart rendered via HTML Canvas.

The entire app ships as a single `.html` file so it works over the `file://` protocol in Chrome, Firefox, Edge, and Safari without any build step or server.

---

## Architecture

The app follows a simple **Model → View → Controller** pattern implemented without any framework:

```
┌─────────────────────────────────────────────────────┐
│                     index.html                      │
│                                                     │
│  ┌──────────┐   ┌──────────────┐   ┌─────────────┐ │
│  │  Store   │◄──│  Controller  │──►│    View     │ │
│  │(localStorage)│  (app.js     │   │ (DOM + Canvas│ │
│  │          │   │  inline)     │   │  rendering) │ │
│  └──────────┘   └──────────────┘   └─────────────┘ │
└─────────────────────────────────────────────────────┘
```

- **Store**: A thin wrapper around `localStorage` that serializes/deserializes the transaction array as JSON.
- **Controller**: Event handlers that call Store methods and trigger View re-renders.
- **View**: Pure render functions that rebuild DOM sections and redraw the Canvas chart from the current state.

State is a single array of transaction objects held in memory and mirrored to `localStorage` on every mutation.

---

## Components and Interfaces

### 1. Layout Regions

```
┌──────────────────────────────────┐
│         Summary Bar              │  Balance, Income, Expenses
├──────────────────────────────────┤
│         Add Transaction Form     │  Description, Amount, Category,
│                                  │  Type (income/expense), Date
├──────────────────────────────────┤
│         Chart Area               │  Canvas pie/donut chart
├──────────────────────────────────┤
│  Category Filter  │  (select)    │
├──────────────────────────────────┤
│         Transaction List         │  Scrollable, most-recent first
└──────────────────────────────────┘
```

On wider viewports (≥768px) the Chart and Transaction List sit side-by-side.

### 2. JavaScript Modules (inline `<script>`)

#### `Store`
```js
Store.load()          // → Transaction[]  (reads localStorage)
Store.save(txns)      // → void           (writes localStorage)
Store.isAvailable()   // → boolean
```

#### `Transactions`
```js
Transactions.add(txns, tx)     // → Transaction[]  (pure, returns new array)
Transactions.remove(txns, id)  // → Transaction[]  (pure, returns new array)
Transactions.filter(txns, cat) // → Transaction[]  (pure)
Transactions.summary(txns)     // → { balance, income, expenses }
Transactions.byCategory(txns)  // → Map<string, number>  (expense totals per category)
```

#### `Chart`
```js
Chart.draw(canvas, categoryMap) // → void  (renders pie/donut on Canvas 2D context)
```

#### `View`
```js
View.renderSummary(summary)
View.renderList(txns)
View.renderFilter(categories, selected)
View.renderChart(categoryMap)
View.showFormError(field, message)
View.clearForm()
```

#### `Controller`
Wires DOM events to Store + Transactions + View. Entry point is `init()` called on `DOMContentLoaded`.

### 3. Transaction Object Shape

```js
{
  id:          string,   // crypto.randomUUID() or Date.now().toString()
  description: string,   // non-empty
  amount:      number,   // positive float, stored as-is
  category:    string,   // non-empty
  type:        "income" | "expense",
  date:        string    // ISO 8601 date string "YYYY-MM-DD"
}
```

---

## Data Models

### Transaction (runtime + storage)

| Field       | Type                    | Constraints                        |
|-------------|-------------------------|------------------------------------|
| id          | string                  | unique, non-empty                  |
| description | string                  | non-empty, trimmed                 |
| amount      | number                  | > 0, finite                        |
| category    | string                  | non-empty, trimmed                 |
| type        | `"income"` \| `"expense"` | one of two values                |
| date        | string                  | ISO 8601 `YYYY-MM-DD`              |

### Derived State (computed, never stored)

| Value        | Derivation                                      |
|--------------|-------------------------------------------------|
| balance      | `sum(income amounts) - sum(expense amounts)`    |
| totalIncome  | `sum(tx.amount where tx.type === "income")`     |
| totalExpenses| `sum(tx.amount where tx.type === "expense")`    |
| categoryMap  | `Map<category, sum(expense amounts)>` for chart |

### localStorage Schema

Key: `"ebv_transactions"`  
Value: JSON-serialized `Transaction[]`

```json
[
  {
    "id": "1720000000000",
    "description": "Groceries",
    "amount": 45.50,
    "category": "Food",
    "type": "expense",
    "date": "2024-07-03"
  }
]
```

---

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Balance and summary invariant

*For any* array of transactions, the computed summary SHALL satisfy: `balance === totalIncome - totalExpenses`, `totalIncome === sum of all income amounts`, and `totalExpenses === sum of all expense amounts`.

**Validates: Requirements 1.2, 1.3, 1.4**

### Property 2: Transaction persistence round-trip

*For any* valid transaction array, serializing it to JSON and deserializing it SHALL produce an array that is structurally equivalent to the original (same ids, amounts, types, categories, dates, descriptions).

**Validates: Requirements 7.1, 7.2**

### Property 3: Add then contains

*For any* transaction list and any valid new transaction, after adding the transaction the list SHALL contain an entry with the same id, description, amount, category, type, and date.

**Validates: Requirements 2.2, 3.2**

### Property 4: Delete removes exactly one

*For any* transaction list containing a transaction with a given id, after removing that id the resulting list SHALL not contain any transaction with that id, and all other transactions SHALL remain unchanged.

**Validates: Requirements 4.2**

### Property 5: Category filter subset invariant

*For any* transaction list and any category string (including "All"), filtering by a specific category SHALL return only transactions whose category field equals the filter value; filtering by "All" SHALL return the complete list unchanged.

**Validates: Requirements 6.2, 6.3**

### Property 6: Category map totals correctness

*For any* transaction list, the sum of all values in the category expense map SHALL equal the total expenses derived from the same list, and each key SHALL map to the correct sum of expense amounts for that category.

**Validates: Requirements 5.1, 5.3**

### Property 7: Validation rejects all invalid inputs

*For any* form submission where at least one required field (description, amount, category, type, date) is empty or whitespace-only, OR where the amount is non-numeric, zero, negative, or non-finite, the validation function SHALL reject the submission and return an error identifying the offending field, leaving the transaction list unchanged.

**Validates: Requirements 2.3, 2.4**

---

## Error Handling

| Scenario | Handling |
|---|---|
| `localStorage` unavailable (private mode, quota exceeded) | `Store.isAvailable()` returns `false`; app displays a banner warning and runs with in-memory state only |
| `localStorage` parse error (corrupted JSON) | `Store.load()` catches `JSON.parse` exception, returns `[]`, displays warning banner |
| `crypto.randomUUID` unavailable (old browser) | Fall back to `Date.now().toString() + Math.random()` for id generation |
| Form submitted with empty fields | Inline error message shown beneath the offending field; form not submitted |
| Form submitted with invalid amount | Inline error shown on amount field; form not submitted |
| Canvas not supported | Hide chart area; show static text fallback |
| No expense transactions | Chart area shows "No expense data yet" placeholder text instead of empty canvas |

---

## Testing Strategy

### Unit / Example Tests

Test the pure functions in `Transactions` and `Store` with concrete examples:

- `summary([])` → `{ balance: 0, income: 0, expenses: 0 }`
- `summary([income 100, expense 40])` → `{ balance: 60, income: 100, expenses: 40 }`
- `add([], tx)` → array of length 1 containing `tx`
- `remove([tx], tx.id)` → `[]`
- `filter([tx1(Food), tx2(Transport)], "Food")` → `[tx1]`
- `byCategory([expense Food 10, expense Food 5, expense Transport 20])` → `Map{Food→15, Transport→20}`
- Validation: empty description → error; amount "abc" → error; amount 0 → error; amount -5 → error

### Property-Based Tests

Using [fast-check](https://github.com/dubzzz/fast-check) (or equivalent), run minimum 100 iterations per property:

- **Property 1** — Generate arbitrary arrays of valid transactions; assert `summary(txns).balance === incomeSum - expenseSum`, `summary(txns).income === incomeSum`, `summary(txns).expenses === expenseSum`
- **Property 2** — Generate arbitrary transaction arrays; assert `JSON.parse(JSON.stringify(txns))` deep-equals original
- **Property 3** — Generate arbitrary list + valid new transaction; assert `add(list, tx)` contains `tx` with all fields intact
- **Property 4** — Generate list with at least one tx; assert `remove(list, id)` excludes that id and preserves all others
- **Property 5** — Generate list + category (including "All"); assert `filter(list, cat)` returns exactly the matching subset; `filter(list, "All")` returns full list
- **Property 6** — Generate arbitrary transaction arrays; assert `sum(byCategory(txns).values()) === summary(txns).expenses` and each category key maps to the correct sum
- **Property 7** — Generate form objects with blank/whitespace required fields OR invalid amounts (0, negative, NaN, Infinity, strings); assert validation rejects and identifies the offending field

Tag format for each test: `// Feature: expense-budget-visualizer, Property N: <property_text>`

### Integration / Smoke Tests

- Load the HTML file in a browser; verify `localStorage` key `ebv_transactions` is written after adding a transaction
- Reload the page; verify previously added transactions are restored
- Verify the app loads and is interactive within 3 seconds (Chrome DevTools throttled 4G)
- Verify layout renders without horizontal scroll at 320px, 768px, 1440px, and 2560px viewport widths
- Verify all interactive controls meet the 44×44 CSS pixel minimum touch target size
