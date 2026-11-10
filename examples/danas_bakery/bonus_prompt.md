You are working inside the existing Git repository `langgraph-basic-examples`.

The verified required homework notebook is:

`examples/danas_bakery/danas_bakery.ipynb`

Create a separate bonus notebook at:

`examples/danas_bakery/danas_bakery_bonus_inventory.ipynb`

Do not modify, overwrite, rename, move, delete, stage, commit, or push the existing required notebook or any other repository file.

## Objective

Create a bonus extension that adds dynamic inventory management across sequential orders during the same Google Colab runtime.

The required notebook currently treats every `graph.invoke()` call independently and checks the same fixed inventory. The bonus version must preserve the complete required solution and add a separate inventory-aware workflow in which accepted orders reduce the available inventory for later orders.

The bonus notebook must remain deterministic and must not use an LLM, API key, external database, checkpointer, or external storage.

## Preserve the required implementation

Start from the verified required notebook and retain:

- The original `graph`.
- The original state, nodes, routing and business rules.
- The original circular substitution logic.
- The original delivery and pickup rules.
- The original discount calculation.
- The original deterministic messages.
- The original graph diagram.
- The original twelve required scenarios.
- The original validation, boundary and isolation checks.

The original `graph` must continue to behave exactly as it does in the verified required notebook.

Do not change the expected results of the required tests.

## Add a clearly separated bonus section

Append a new notebook section titled:

`Bonus Extension: Dynamic Inventory Management`

Explain that the bonus inventory persists only across sequential invocations within the current notebook runtime. Restarting the Colab runtime resets it. This is an educational single-session implementation and is not intended to provide database persistence or production concurrency.

## Inventory model

Preserve the original fixed inventory as the initial inventory:

```python
INVENTORY = {
    "challah": 10,
    "babka": 0,
    "rugelach": 3,
    "sourdough": 4,
}
```

Create a separate mutable inventory for the bonus workflow:

```python
CURRENT_INVENTORY = INVENTORY.copy()
```

Never modify the original `INVENTORY` dictionary.

The bonus stock-checking node must read from `CURRENT_INVENTORY`.

## Required inventory utilities

Implement these functions:

### `reset_inventory()`

Reset `CURRENT_INVENTORY` to the exact values in `INVENTORY`.

It must mutate the existing dictionary safely:

```python
CURRENT_INVENTORY.clear()
CURRENT_INVENTORY.update(INVENTORY)
```

### `show_inventory()`

Return a readable pandas DataFrame with at least:

- Product
- Current quantity
- Initial quantity
- Units sold or difference from the initial quantity
- Stock status

Use clear stock labels such as:

- `Out of stock`
- `Low stock`
- `In stock`

Document the chosen low-stock threshold.

### `restock_inventory(item, quantity)`

Add a positive integer quantity to a valid current-menu product.

Validate its inputs:

- Reject unsupported products.
- Reject zero or negative quantities.
- Reject booleans, floats, numeric strings and other non-integer values.
- Do not change inventory when validation fails.
- Return or display a clear result describing the restock operation.

## Bonus state

Create a bonus state type that extends or mirrors the original state and includes fields equivalent to:

- `inventory_before`
- `inventory_after`
- `inventory_snapshot`
- `stock_updated`

The final result of an accepted bonus order should make the inventory change visible.

## Bonus graph

Create a separate compiled LangGraph named:

```python
inventory_graph
```

Do not replace or rename the original `graph`.

Reuse the original nodes when their behavior does not need to change.

Create a live-stock node, for example:

```python
check_live_stock
```

It must:

- Check `CURRENT_INVENTORY`.
- Require enough stock for the complete quantity.
- Use the same circular substitution order.
- Check no more than two substitutes.
- Never allow partial fulfilment.
- Never allow inventory to become negative.

Create a separate inventory-update node, for example:

```python
reserve_inventory
```

It must:

- Run only for accepted orders.
- Reduce the inventory of the product actually sold.
- Use `current_item` or the final sold `item`, not always `requested_item`.
- Reduce the stock exactly once.
- Record the quantity before and after the sale.
- Store a copy of the resulting inventory snapshot in the state.
- Append a clear entry to `steps`.

For example:

```text
reserve_inventory: reduced rugelach stock from 3 to 1 after selling 2 boxes.
```

Rejected or invalid orders must never run the inventory-update node and must never change `CURRENT_INVENTORY`.

Use a clear flow equivalent to:

```text
START
  -> initialize order
  -> validate order
      -> invalid: reject -> END
      -> valid: check live stock
          -> available: determine delivery
          -> unavailable and fewer than two alternatives checked:
             try substitute -> check live stock
          -> unavailable after two alternatives:
             cancel -> END
  -> calculate total
  -> reserve inventory
  -> create message
  -> END
```

The second substitute must always be checked before cancellation.

## Bonus graph visualization

Display the actual `inventory_graph` using:

```python
display(Image(inventory_graph.get_graph().draw_mermaid_png()))
```

The diagram must visibly include the new inventory-update node and the live-stock loop.

## Customer message and diary

For accepted inventory-aware orders:

- Preserve the original warm deterministic message.
- Optionally include the remaining quantity of the sold product.
- Record the stock change in `steps`.

For rejected orders:

- Preserve the original rejection behavior.
- Do not claim that inventory changed.
- Return `total == 0.0`.
- Leave the complete inventory unchanged.

## Required sequential bonus demonstration

Add a helper named something like:

```python
run_inventory_order()
```

It should print:

- The order input.
- The inventory before the order.
- The result.
- The item actually sold.
- The inventory after the order.
- The chronological `steps`.

Run the following scenarios sequentially without resetting between them.

### Initial state

Call `reset_inventory()` and assert:

```python
CURRENT_INVENTORY == INVENTORY
```

### Order 1

```python
{
    "item": "challah",
    "quantity": 4,
    "day": "Monday",
    "city": ""
}
```

Expected:

- Accepted.
- Sold item: challah.
- Challah inventory: 10 -> 6.

### Order 2

Run the same order again.

Expected:

- Accepted.
- Sold item: challah.
- Challah inventory: 6 -> 2.

### Order 3

Run the same order a third time.

Expected:

- Requested challah has only 2 units.
- First circular substitute babka has 0.
- Second circular substitute rugelach has 3, which is insufficient for 4.
- Rejected.
- No inventory value changes.

### Order 4: substitution reduces the actual sold product

```python
{
    "item": "babka",
    "quantity": 2,
    "day": "Tuesday",
    "city": "Jerusalem"
}
```

Expected:

- Babka unavailable.
- First substitute rugelach succeeds.
- Accepted.
- `item == "rugelach"`.
- Rugelach inventory: 3 -> 1.
- Babka inventory remains 0.
- Total is calculated using the rugelach price.

### Order 5: later substitution uses updated inventory

Run another order for two babkas.

Expected:

- Babka has 0.
- Rugelach now has only 1 and fails.
- Sourdough succeeds as the second substitute.
- Sourdough inventory: 4 -> 2.
- Rugelach remains 1.

### Invalid-order protection

Save a copy of `CURRENT_INVENTORY`, invoke an invalid order with quantity 0, and assert that the complete inventory remains unchanged.

### Rejected-order protection

Save a copy of `CURRENT_INVENTORY`, invoke an order that cannot be fulfilled after two substitutes, and assert that the complete inventory remains unchanged.

### Restock demonstration

Restock rugelach by 5 units.

Assert:

- The inventory increases by exactly 5.
- Other products remain unchanged.
- An invalid restock operation does not change inventory.

### Reset demonstration

Call `reset_inventory()` at the end and assert:

```python
CURRENT_INVENTORY == INVENTORY
```

## Assertions and success output

Add assertions that prove:

- Accepted orders reduce stock exactly once.
- Rejected orders do not reduce stock.
- Invalid orders do not reduce stock.
- The actual substituted product is reduced.
- The original requested product is not reduced when a substitute is sold.
- Stock never becomes negative.
- Sequential calls see the updated inventory.
- Restocking works.
- Reset restores all initial values.
- The original fixed `INVENTORY` remains unchanged.
- The original required `graph` still passes its required tests.

At the end, print:

```text
All dynamic inventory bonus checks passed.
```

## Notebook quality

The notebook must:

- Run from top to bottom in a fresh Google Colab session.
- Use `%pip install -q -U langgraph`.
- Avoid Playwright, Pyppeteer, Chromium and greenlet.
- Use concise Markdown explanations.
- Clearly distinguish required functionality from bonus functionality.
- Use no API key.
- Use no external database.
- Avoid hidden mutation outside the explicitly documented `CURRENT_INVENTORY`.
- Keep the implementation understandable for a student presentation.

## File and Git restrictions

Create only:

`examples/danas_bakery/danas_bakery_bonus_inventory.ipynb`

Do not modify:

- `examples/danas_bakery/danas_bakery.ipynb`
- `examples/danas_bakery/README.md`
- `examples/danas_bakery/prompt.md`
- any repository-level files

Do not run `git add`, `git commit`, or `git push`.

After creating the notebook, report:

- the created file path;
- a short summary of the added nodes and utilities;
- the bonus scenarios included;
- confirmation that the original notebook was unchanged.

# UI Enhancement Refinement

Work inside the existing `langgraph-basic-examples` repository.

Update the existing bonus notebook in place:

`examples/danas_bakery/danas_bakery_bonus_inventory.ipynb`

Do not create another notebook.

Do not modify:

- `examples/danas_bakery/danas_bakery.ipynb`
- `examples/danas_bakery/prompt.md`
- `examples/danas_bakery/bonus_prompt.md`
- `examples/danas_bakery/README.md`
- any unrelated repository file

Do not run `git add`, `git commit`, or `git push`.

## Objective

Improve the presentation and usability of the existing dynamic-inventory bonus notebook.

The dynamic inventory extension is already part of the bonus. Add a polished interactive user interface to the same bonus notebook so that a user can:

- Enter an order without writing Python code.
- Clearly understand whether it was accepted or rejected.
- See the requested product and the product actually sold.
- See substitutions clearly.
- See pricing and discount details.
- See inventory before and after the order.
- Follow the graph journey in a readable format.
- Restock products.
- Reset inventory.
- View the current inventory in a clear dashboard.

This is a presentation and usability enhancement. Do not change the verified business logic.

## Preserve all verified functionality

Keep unchanged in behavior:

- The original required `graph`.
- The dynamic `inventory_graph`.
- The menu, prices and initial inventory.
- Input validation.
- The circular substitution sequence.
- The two-substitute limit.
- Delivery and pickup rules.
- Discount rules.
- Deterministic customer messages.
- `CURRENT_INVENTORY`.
- `reset_inventory()`.
- `show_inventory()`.
- `restock_inventory()`.
- `check_live_stock`.
- `reserve_inventory`.
- No inventory change for invalid or rejected orders.
- Reduction of the product actually sold.
- Protection against negative inventory and duplicate deductions.
- All existing assertions and expected results.

The following messages must still appear when the notebook runs:

```text
All required scenarios and additional checks passed.
```

```text
All dynamic inventory bonus checks passed.
```

## Keep one bonus notebook

The complete solution must remain in:

`examples/danas_bakery/danas_bakery_bonus_inventory.ipynb`

Do not create:

- `danas_bakery_bonus_inventory_ui.ipynb`
- another copy of the bonus notebook
- any notebook at the repository root

## Colab-compatible interface

Use `ipywidgets` and standard notebook display utilities:

```python
import ipywidgets as widgets
from IPython.display import display, HTML
```

For Google Colab widget compatibility, safely enable the custom widget manager:

```python
try:
    from google.colab import output
    output.enable_custom_widget_manager()
except ImportError:
    pass
```

Do not use:

- Gradio
- Streamlit
- Flask
- an external web server
- an LLM
- an API key
- an external database
- Playwright
- Pyppeteer
- Chromium

The interface must work after selecting `Runtime -> Run all` in a fresh Google Colab runtime.

## Improve the existing output presentation

The bonus section currently prints large raw Python dictionaries. Replace or supplement these verbose displays with readable visual summaries.

Do not remove the assertions.

For the sequential bonus demonstration:

- Keep all existing test logic.
- Avoid printing full raw state dictionaries unless placed inside an optional technical-details section.
- Display a compact summary table containing:
  - Order number
  - Requested product
  - Quantity
  - Product actually sold
  - Status
  - Delivery method
  - Final total
  - Inventory change
- Display inventory changes in a clear format such as:
  - `challah: 10 → 6`
  - `rugelach: 3 → 1`
- Use green status styling for accepted orders.
- Use red status styling for rejected orders.
- Use amber styling for substitutions and low-stock warnings.
- Clearly state `Inventory unchanged` for rejected or invalid orders.
- Keep full `steps` available in a collapsible technical-details or order-journey area.

The automated tests must remain deterministic and must still fail visibly if an assertion fails.

## Add an interactive dashboard section

Add a final section titled:

`Interactive Bakery Dashboard`

Before showing the dashboard:

1. Complete all required and bonus automated tests.
2. Call `reset_inventory()`.
3. Confirm that `CURRENT_INVENTORY == INVENTORY`.
4. Build the dashboard without consuming inventory.

## Visual design

Use a clean professional bakery-inspired style:

- Very light cream background.
- Warm brown headings.
- Soft gold or orange accents.
- Green for accepted orders.
- Red for rejected orders and errors.
- Amber for low stock and substitutions.
- Rounded cards.
- Clear spacing.
- Readable typography.
- Sufficient contrast.

Use HTML and CSS inside the notebook for styling, but keep the implementation understandable and concise.

Do not depend only on color. Always display explicit text such as:

- `Accepted`
- `Rejected`
- `Low stock`
- `Out of stock`
- `Substituted`

## Dashboard header

Display:

- `Dana's Bakery`
- `LangGraph Order & Inventory Dashboard`
- A short explanation that inventory updates across orders during the current notebook session.

Also show this session note:

```text
Inventory is maintained during the current notebook runtime.
Restarting the runtime resets it to the initial values.
```

## Inventory cards

Display one card for each product:

- Challah
- Babka
- Rugelach
- Sourdough

Each card must show:

- Current quantity.
- Initial quantity.
- Stock status.

Use the existing low-stock threshold.

Also provide a readable inventory table based on `show_inventory()`.

Refresh the cards and table after:

- An accepted order.
- A rejected order.
- An invalid order.
- A restock operation.
- An inventory reset.

A rejected order must show the same inventory before and after.

## Order form

Create a clear order form containing:

### Product

Use a dropdown with:

- challah
- babka
- rugelach
- sourdough

### Quantity

Use `BoundedIntText` or another appropriate integer widget.

The UI should prevent or clearly report a non-positive quantity, while the graph's own validation must remain the authoritative validation.

### Day

Use a dropdown with all seven weekday names.

### City

Use a text field with placeholder guidance:

```text
Leave empty for pickup; delivery is available only in Jerusalem
```

### Submit button

Use a prominent button labeled:

`Process Order`

When clicked:

1. Disable the button.
2. Build the four-field order dictionary.
3. Invoke only:

```python
inventory_graph.invoke(order)
```

4. Do not duplicate the graph's business logic in the event handler.
5. Display the result.
6. Refresh inventory.
7. Re-enable the button in a `finally` block.
8. Show a friendly error message instead of a raw traceback if an unexpected error occurs.

Prevent duplicate click handlers if the dashboard cell is rerun.

## Order result card

For an accepted order, show:

- `Order Accepted`
- Requested product
- Product actually sold
- Quantity
- Whether a substitution occurred
- Delivery or pickup
- Goods subtotal
- Discount amount
- Delivery fee
- Final total
- Inventory before
- Inventory after
- Customer message

Use graph-returned values. Do not independently recalculate prices.

If a substitution occurred, display a prominent message such as:

```text
Substitution: babka → rugelach
```

For a rejected order, show:

- `Order Rejected`
- Requested product
- Customer message or rejection reason
- Final total: 0.00 NIS
- `Inventory unchanged`

Format all monetary values to two decimal places.

Escape user-provided values before inserting them into HTML.

## Order journey

Display `steps` in a collapsible accordion titled:

`Order Journey`

Number the steps chronologically.

Do not show the complete raw result dictionary in the main interface.

If useful for debugging, place raw state data in a second collapsed section titled:

`Technical Details`

The default view should remain understandable to a non-programmer.

## Inventory management panel

Add a separate panel titled:

`Inventory Management`

Include:

- Product dropdown.
- Positive integer restock quantity.
- `Restock Product` button.
- `Reset Inventory` button.
- A confirmation checkbox or two-step confirmation for resetting all inventory.

The restock button must call the existing:

```python
restock_inventory()
```

The reset button must call the existing:

```python
reset_inventory()
```

Do not reproduce these functions' logic in the UI.

After each operation:

- Display a clear success or validation message.
- Refresh the inventory dashboard.
- Do not clear unrelated interface content unnecessarily.

## Output architecture

Use separate `widgets.Output()` areas for:

- Current inventory.
- Order result.
- Order journey.
- Inventory-management messages.

When refreshing, clear only the relevant output area.

Do not use `clear_output()` in a way that removes the entire dashboard.

Organize the code into small functions with responsibilities equivalent to:

- `build_dashboard()`
- `refresh_inventory_view()`
- `handle_order_submit()`
- `render_order_result()`
- `render_order_steps()`
- `handle_restock()`
- `handle_inventory_reset()`

Exact names may differ.

## Accessibility and usability

Ensure that:

- Every field has a visible label.
- Buttons use clear action text.
- Status is communicated by both text and color.
- Text wraps within cards.
- The interface fits a normal Colab width.
- The interface remains usable by keyboard.
- Errors are shown near the action that caused them.
- Raw tracebacks are not shown to ordinary users.
- The order form and inventory-management form are visually separated.

## Graph visibility

Retain both existing graph diagrams.

Near the bonus graph, add a concise explanation of:

- `check_live_stock`
- the circular substitution loop
- `reserve_inventory`
- rejected branches bypassing inventory reservation

Do not replace `draw_mermaid_png()` with a manually drawn diagram.

## UI verification

Add automated smoke checks before displaying the dashboard.

Verify that:

- `inventory_graph` exists.
- Building the dashboard does not change inventory.
- Refreshing the inventory view does not change inventory.
- The order handler is configured to use `inventory_graph`.
- The required widgets are created.
- `CURRENT_INVENTORY` equals `INVENTORY` when the user first sees the dashboard.
- No order is submitted automatically.

Print:

```text
Interactive dashboard checks passed.
```

## Final execution order

A fresh `Runtime -> Run all` must:

1. Install dependencies.
2. Run all required tests.
3. Print `All required scenarios and additional checks passed.`
4. Run all dynamic inventory bonus tests.
5. Print `All dynamic inventory bonus checks passed.`
6. Reset inventory.
7. Build and verify the dashboard.
8. Print `Interactive dashboard checks passed.`
9. Display the interactive dashboard with the initial inventory.

## Final verification

After editing the notebook:

- Verify that it remains valid notebook JSON.
- Verify that no existing tests or assertions were removed.
- Verify that the original required notebook is unchanged.
- Verify that no new notebook was created.
- Verify that only the existing bonus notebook was modified.

Do not stage, commit, or push.

Report:

- The modified notebook path.
- The UI components added.
- How the verbose bonus outputs were improved.
- Confirmation that all existing logic and assertions were preserved.
- Confirmation that no other file was modified.

## Refinement summary

The bonus stayed in the same notebook; no separate UI notebook was created. The existing inventory logic was preserved, and ipywidgets was added as the presentation and interaction layer. Compact order summaries and expandable steps replace the verbose raw bonus output. The dashboard supports order processing, live inventory display, restocking, reset with confirmation, and an expandable chronological Order Journey.
