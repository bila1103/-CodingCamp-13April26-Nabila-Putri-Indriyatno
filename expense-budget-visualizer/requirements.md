# Requirements Document

## Introduction

The Expense & Budget Visualizer is a mobile-friendly web application that helps users track their daily spending. It provides a clear view of total balance, a transaction history, and a visual chart of spending broken down by category. The app runs entirely in the browser with no backend server, storing all data in the browser's Local Storage. It can be used as a standalone web page or browser extension and must work across all modern browsers.

## Glossary

- **App**: The Expense & Budget Visualizer web application
- **Transaction**: A single financial entry representing either income or an expense, with an amount, category, description, and date
- **Balance**: The running total calculated as the sum of all income transactions minus the sum of all expense transactions
- **Category**: A user-defined or preset label used to group transactions (e.g., Food, Transport, Entertainment)
- **Chart**: A visual representation of spending distribution across categories
- **Local_Storage**: The browser's built-in Web Storage API used to persist transaction data client-side
- **Transaction_List**: The scrollable history view displaying all recorded transactions
- **Category_Filter**: A UI control that limits the Transaction_List to a selected category

## Technical Constraints

### TC-1: Technology Stack
- HTML for structure
- CSS for styling
- Vanilla JavaScript (no frameworks like React, Vue, etc.)

### TC-2: Data Storage
- Use browser Local Storage API
- All data stored client-side only

### TC-3: Browser Compatibility
- Must work in modern browsers (Chrome, Firefox, Edge, Safari)
- Can be used as standalone web app or browser extension

---

## Requirements

### Requirement 1: Display Financial Summary

**User Story:** As a user, I want to see my current balance at a glance, so that I always know my overall financial position.

#### Acceptance Criteria

1. THE App SHALL display the current Balance prominently at the top of the main view.
2. WHEN a Transaction is added or deleted, THE App SHALL recalculate and update the displayed Balance immediately.
3. THE App SHALL display the total income and total expenses as separate summary values alongside the Balance.
4. WHEN no Transactions exist, THE App SHALL display a Balance of zero.

---

### Requirement 2: Add a Transaction

**User Story:** As a user, I want to add income and expense transactions, so that I can record my financial activity.

#### Acceptance Criteria

1. THE App SHALL provide a form with fields for description, amount, category, type (income or expense), and date.
2. WHEN the user submits the form with all required fields filled, THE App SHALL save the Transaction to Local_Storage and update the Transaction_List and Balance.
3. IF the user submits the form with any required field empty, THEN THE App SHALL display an inline validation error identifying the missing field and SHALL NOT save the Transaction.
4. IF the user enters a non-numeric or zero value in the amount field, THEN THE App SHALL display a validation error and SHALL NOT save the Transaction.
5. WHEN a Transaction is saved, THE App SHALL clear the form fields and return focus to the description field.

---

### Requirement 3: View Transaction History

**User Story:** As a user, I want to see a list of all my transactions, so that I can review my spending history.

#### Acceptance Criteria

1. THE App SHALL display all Transactions in the Transaction_List ordered by date, with the most recent first.
2. THE App SHALL display each Transaction entry with its description, amount, category, type, and date.
3. WHILE the Transaction_List contains more entries than fit in the visible area, THE App SHALL make the list scrollable without affecting the rest of the page layout.
4. WHEN no Transactions exist, THE App SHALL display a message indicating that no transactions have been recorded.

---

### Requirement 4: Delete a Transaction

**User Story:** As a user, I want to delete a transaction, so that I can correct mistakes in my records.

#### Acceptance Criteria

1. THE App SHALL display a delete control for each Transaction entry in the Transaction_List.
2. WHEN the user activates the delete control for a Transaction, THE App SHALL remove that Transaction from Local_Storage, update the Transaction_List, and recalculate the Balance.
3. WHEN a Transaction is deleted, THE App SHALL update the spending Chart to reflect the removal.

---

### Requirement 5: Visualize Spending by Category

**User Story:** As a user, I want to see a chart of my spending by category, so that I can understand where my money is going.

#### Acceptance Criteria

1. THE App SHALL display a Chart showing the proportion of total expenses attributed to each Category.
2. WHEN a Transaction is added or deleted, THE App SHALL update the Chart immediately to reflect the current expense data.
3. THE App SHALL label each segment of the Chart with the Category name and its percentage of total expenses.
4. WHEN no expense Transactions exist, THE App SHALL display a placeholder message in the Chart area instead of an empty chart.
5. THE App SHALL render the Chart using only Vanilla JavaScript and HTML Canvas or SVG, without external charting libraries.

---

### Requirement 6: Filter Transactions by Category

**User Story:** As a user, I want to filter my transaction history by category, so that I can focus on a specific area of spending.

#### Acceptance Criteria

1. THE App SHALL provide a Category_Filter control populated with all categories present in the current Transaction data, plus an "All" option.
2. WHEN the user selects a category in the Category_Filter, THE App SHALL update the Transaction_List to show only Transactions matching that category.
3. WHEN the user selects "All" in the Category_Filter, THE App SHALL display all Transactions in the Transaction_List.
4. WHEN a new Transaction with a previously unseen Category is saved, THE App SHALL add that Category to the Category_Filter options.

---

### Requirement 7: Persist Data Across Sessions

**User Story:** As a user, I want my transaction data to be saved between browser sessions, so that I don't lose my records when I close the tab.

#### Acceptance Criteria

1. THE App SHALL write all Transaction data to Local_Storage whenever a Transaction is added or deleted.
2. WHEN the App is loaded, THE App SHALL read all previously stored Transactions from Local_Storage and populate the Transaction_List, Balance, and Chart.
3. IF Local_Storage is unavailable or returns a parse error, THEN THE App SHALL display a warning message and operate with an empty Transaction_List for the current session.

---

### Requirement 8: Mobile-Friendly Layout

**User Story:** As a user, I want the app to work well on my phone, so that I can track spending on the go.

#### Acceptance Criteria

1. THE App SHALL use a responsive layout that adapts to viewport widths from 320px to 2560px without horizontal scrolling.
2. THE App SHALL size all interactive controls (buttons, inputs, selects) to a minimum touch target of 44x44 CSS pixels.
3. THE App SHALL render correctly in portrait and landscape orientations on mobile devices.
4. THE App SHALL load and be interactive within 3 seconds on a standard mobile connection (simulated 4G, ~20 Mbps).
