Create a complete Google Colab-compatible Python notebook that implements Dana's Bakery order-processing system as a LangGraph StateGraph.
The notebook must be self-contained, deterministic, easy for a student to read, and runnable from a fresh Google Colab session. Do not use an LLM, an API key, external services, a database, or persistent memory. Use ordinary Python functions as graph nodes.
Build the notebook in clearly labeled sections and executable cells:
1. Title and short explanation of the system.
2. Install required packages, including LangGraph and any package needed to display the graph.
3. Imports.
4. Business constants: menu, inventory, substitution sequence, delivery fee, and discount rules.
5. LangGraph state definition.
6. Node and routing functions.
7. Graph construction and compilation.
8. Graph visualization using draw_mermaid_png() and display of the resulting image in the notebook.
9. A reusable helper for running and clearly printing test orders.
10. At least the test cases specified below, with assertions or explicit expected results.
The cells must run in order in a fresh Colab runtime without manual edits and without an API key.
The compiled graph must be stored in a variable named graph and support this exact invocation style:
```python
result = graph.invoke({
    "item": "babka",
    "quantity": 2,
    "day": "Thursday",
    "city": "Jerusalem"
})
```
Every invocation must return a dictionary containing at least these six keys:
- status: exactly "accepted" or "rejected".
- item: the product actually sold. If the order is rejected before a product can be sold, retain the normalized requested item when possible.
- delivery: exactly "pickup" or "delivery". For an invalid or cancelled order, use "pickup" as the deterministic default because no delivery will occur.
- total: the final amount to pay in NIS, including delivery. Use 0.0 for every rejected order.
- message: a warm, polite, deterministic reply to the customer.
- steps: a list of strings recording, in chronological order, the nodes and important decisions through which the order passed.
The result may contain additional internal fields, but the six required keys must always be present.
Each input order contains:
- item: product name.
- quantity: requested quantity.
- day: day of the week.
- city: customer city. An empty or whitespace-only city means pickup from the bakery.
Normalize string inputs by stripping surrounding whitespace. Compare product names, days, and cities case-insensitively, but store canonical product names in lowercase and canonical day/city values where useful.
Use the following menu and unit prices:
```python
MENU = {
    "challah": 18.0,
    "babka": 45.0,
    "rugelach": 30.0,
    "sourdough": 25.0,
}
```
Use the following fixed inventory for each independent graph invocation:
```python
INVENTORY = {
    "challah": 10,
    "babka": 0,
    "rugelach": 3,
    "sourdough": 4,
}
```
rugelach represents a box of rugelach, so its quantity is the number of boxes.
croissant is an obsolete product and must not be accepted, even though an old menu listed it at 12 NIS. Treat it like any unsupported product.
Do not decrement or persist inventory between separate calls to graph.invoke(). Each test order is evaluated independently against the fixed inventory above.
Validate the order before checking stock.
An order is invalid if any of the following is true:
- A required input key is missing.
- item is not a string after basic type checking or is not one of the four current menu products.
- quantity is not a positive integer. Reject zero, negative numbers, booleans, floats, numeric strings, and other types.
- day is not a string or is not one of: Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday. Accept case variations and normalize them.
- city is neither a string nor None. Treat None as an empty city for pickup.
For an invalid order:
- Set status to "rejected".
- Set total to 0.0.
- Set delivery to "pickup".
- Produce a polite deterministic message explaining that the order could not be understood or processed.
- Record the validation failure in steps.
- End the graph without stock, delivery, or price calculation.
An item is available only when its inventory is greater than or equal to the complete requested quantity. Never partially fulfil an order.
Use this circular substitution order:
```text
babka -> rugelach -> sourdough -> challah -> babka
```
Equivalently, define:
```python
NEXT_ITEM = {
    "babka": "rugelach",
    "rugelach": "sourdough",
    "sourdough": "challah",
    "challah": "babka",
}
```
When the requested item does not have sufficient stock:
1. Check the next product in the circular sequence.
2. Keep the original requested quantity.
3. If the substitute has enough stock for the complete quantity, sell it at the substitute's price.
4. Otherwise, check the next product in the circular sequence.
5. Check no more than two substitute products in total.
6. If neither of the two substitutes has sufficient stock, reject/cancel the order.
The two-substitute limit counts checked alternatives, not only successful customer-facing suggestions. Never loop indefinitely and never check a third substitute. Because only two substitutes may be checked, the originally requested item cannot be reached again during the same order.
For a cancelled order:
- Set status to "rejected".
- Set total to 0.0.
- Set delivery to "pickup".
- Return the normalized original requested item in item. Keep the last candidate only in the internal current_item field and in steps.
- Produce a warm apology explaining that the requested quantity is unavailable.
- Record the original stock failure, each substitute checked, and the cancellation in steps.
For an accepted substituted order, item must contain the substitute actually sold, and the message must clearly tell the customer about the substitution.
Apply delivery logic only after an item with sufficient stock has been found.
- If city is Jerusalem (case-insensitive), set delivery to "delivery" and add a 20 NIS delivery fee.
- If city is empty, whitespace-only, or None, set delivery to "pickup" with no delivery fee.
- If the city is any place outside Jerusalem, keep the order accepted, change fulfilment to "pickup", charge no delivery fee, and explain in the message that delivery is available only inside Jerusalem.
Ignore the earlier statement that orders over 200 NIS receive a 10% discount. It was explicitly overridden by the final rule from the person responsible for money.
Use the final rule:
- Calculate the baked-goods subtotal as actual_item_price * quantity.
- If the baked-goods subtotal is strictly greater than 150 NIS and the order day is not Friday, apply a 15% discount to the baked-goods subtotal.
- Do not apply a discount when the subtotal is exactly 150 NIS.
- Never apply a discount on Friday.
- Never discount the delivery fee.
- Add the delivery fee only after calculating the discounted baked-goods amount.
- Round the final monetary values to two decimal places to avoid floating-point display artifacts.
Use this calculation order:
```python
goods_subtotal = unit_price * quantity
discounted_goods = goods_subtotal * 0.85 if goods_subtotal > 150 and day != "Friday" else goods_subtotal
total = round(discounted_goods + delivery_fee, 2)
```
The client originally asked for AI-written warm and personal messages, but the technical requirements prohibit LLM calls. Resolve this conflict by generating messages from deterministic Python templates.
Messages should be friendly and should reflect all relevant outcomes, including:
- successful order;
- substitute product used;
- delivery in Jerusalem;
- free pickup;
- outside-Jerusalem order changed to pickup;
- discount applied;
- invalid input;
- cancellation after two unavailable substitutes.
Do not call OpenAI or any other model or API.
Use a TypedDict state. It should contain the four inputs, all six required output fields, and internal fields needed by the workflow. Include fields equivalent to:
- requested_item
- current_item
- suggestion_count
- unit_price
- goods_subtotal
- discount_amount or an equivalent discount flag/value
- delivery_fee
- outside_jerusalem
- substituted
Initialize mutable and derived fields inside an initialization node. Do not rely on shared mutable default lists.
Implement genuine LangGraph control flow with multiple nodes and conditional edges. Do not place the entire workflow in one node.
Use nodes with responsibilities equivalent to:
1. initialize_order
2. validate_order
3. reject_invalid_order
4. check_stock
5. try_substitute
6. cancel_unavailable_order
7. determine_delivery
8. calculate_total
9. create_message
Use conditional routing at least after validation, stock checking, and substitution checking. The exact function and variable names may differ if the responsibilities remain clear.
The intended flow is:
```text
START
  -> initialize order
  -> validate order
      -> invalid: reject invalid order -> END
      -> valid: check stock
          -> enough stock: determine delivery
          -> insufficient stock and fewer than two substitutes have been checked: try substitute -> check stock again
          -> insufficient stock and two substitutes have already been checked: cancel unavailable order -> END
  -> determine delivery
  -> calculate total
  -> create message
  -> END
```
Ensure that the graph's actual conditional edges match the implemented logic and the displayed diagram.
The substitution counter must work as follows: it starts at 0; try_substitute moves current_item to the next circular product and increments the counter; that newly selected candidate must always pass through check_stock, including when the counter becomes 2. Only if check_stock finds that candidate unavailable while the counter is already 2 may the graph route to cancellation. This prevents both premature cancellation and a forbidden third substitute.
steps is an audit diary for Avi. Every node that runs must append one concise, human-readable entry in chronological order. Important decisions should also be visible.
The diary must make it possible to understand:
- whether validation passed or failed;
- which product was requested;
- whether stock was sufficient;
- every substitute checked, in order;
- why an order was cancelled;
- whether delivery, ordinary pickup, or forced pickup was selected;
- whether the discount was applied or skipped, including Friday cases;
- the final total or rejection.
Do not sort or overwrite the diary. Preserve execution order.
After compiling the graph, draw and display it in the notebook using draw_mermaid_png(), for example:
```python
from IPython.display import Image, display

display(Image(graph.get_graph().draw_mermaid_png()))
```
The rendered graph must show the nodes and conditional branches used by the actual implementation.
Include at least these test orders. Clearly print each input and result, and add assertions for the key expected values. Together, the tests must exercise every possible branch in the graph.
1. Available item and Jerusalem delivery, no discount
```python
{"item": "challah", "quantity": 2, "day": "Thursday", "city": "Jerusalem"}
```
Expected: accepted, challah, delivery, total 56.0.
2. Empty city means pickup
```python
{"item": "sourdough", "quantity": 2, "day": "Monday", "city": ""}
```
Expected: accepted, sourdough, pickup, total 50.0.
3. Outside Jerusalem is converted to pickup
```python
{"item": "rugelach", "quantity": 2, "day": "Tuesday", "city": "Tel Aviv"}
```
Expected: accepted, rugelach, pickup, total 60.0, and a message explaining the delivery limitation.
4. First substitute succeeds
```python
{"item": "babka", "quantity": 2, "day": "Thursday", "city": "Jerusalem"}
```
Expected: babka is unavailable, rugelach succeeds as the first substitute, accepted, delivery, total 80.0.
5. Second substitute succeeds
```python
{"item": "babka", "quantity": 4, "day": "Wednesday", "city": ""}
```
Expected: babka fails, rugelach fails, sourdough succeeds as the second substitute, accepted, pickup, total 100.0.
6. Two substitutes fail and the order is cancelled
```python
{"item": "babka", "quantity": 8, "day": "Thursday", "city": "Jerusalem"}
```
Expected: babka, rugelach, and sourdough do not have sufficient stock; rejected; total 0.0.
7. Discount applies only to baked goods
```python
{"item": "challah", "quantity": 10, "day": "Thursday", "city": "Jerusalem"}
```
Expected: goods subtotal 180.0, discounted goods 153.0, delivery fee 20.0, final total 173.0.
8. No discount on Friday
```python
{"item": "challah", "quantity": 10, "day": "Friday", "city": "Jerusalem"}
```
Expected: no discount, final total 200.0.
9. Circular substitution from challah
```python
{"item": "challah", "quantity": 11, "day": "Monday", "city": ""}
```
Expected: challah fails, circular first substitute babka fails, circular second substitute rugelach fails, rejected, total 0.0. The steps must prove the circular order.
10. Invalid quantity
```python
{"item": "challah", "quantity": 0, "day": "Monday", "city": "Jerusalem"}
```
Expected: rejected during validation, total 0.0.
11. Unsupported product
```python
{"item": "pizza", "quantity": 1, "day": "Monday", "city": "Jerusalem"}
```
Expected: rejected during validation, total 0.0.
12. Obsolete croissant product
```python
{"item": "croissant", "quantity": 1, "day": "Monday", "city": "Jerusalem"}
```
Expected: rejected during validation, total 0.0.
Also include a boundary assertion showing that a baked-goods subtotal of exactly 150 NIS does not receive the discount. You may create an additional controlled test for this boundary if the fixed inventory prevents reaching it through a naturally accepted order; do not change the required menu or inventory to make the main system behave differently.
- Add concise comments and Markdown explanations suitable for a student submission.
- Avoid unnecessary abstraction, hidden global mutation, and overly complex code.
- Use constants rather than scattering prices and rules throughout the nodes.
- Ensure every test result contains all six required output keys.
- Ensure rejected orders always have total == 0.0.
- Ensure substitutions retain the original quantity and use the substitute's price.
- Ensure the discount uses the actual sold product's subtotal.
- Ensure the delivery fee is never discounted.
- Ensure Friday comparisons work after input normalization.
- Ensure steps accurately matches the nodes and decisions executed.
- Run all cells and tests before finishing the notebook.
At the end of the notebook, add a short Markdown summary explaining how the story was translated into graph state, nodes, conditional edges, deterministic message templates, and tests.
Return the finished .ipynb notebook, not merely code snippets or an explanation.