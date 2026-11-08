# Dana's Bakery

A deterministic LangGraph order-processing example for Google Colab. Python nodes validate bakery orders, check stock and up to two substitutes, select delivery or pickup, calculate discounts, and produce customer messages with an audit diary.

## Required Version

- [danas_bakery.ipynb](danas_bakery.ipynb): complete notebook, graph visualization, and tests.
- [prompt.md](prompt.md): original prompt for the required notebook.

[Open the required version in Colab](https://colab.research.google.com/github/amitbe6120/langgraph-basic-examples/blob/main/examples/danas_bakery/danas_bakery.ipynb)

Open `danas_bakery.ipynb` in Google Colab using **File → Upload notebook**, select a standard CPU runtime with Python 3.13, and choose **Runtime → Run all**. No manual edits, secrets, API keys, or GPU are required. The first cell installs LangGraph using `%pip install -q -U langgraph`. The graph uses the default `draw_mermaid_png()` renderer, which requires internet access. No browser installation is needed.

The notebook contains nine LangGraph nodes, the actual graph diagram, twelve required order scenarios, and additional assertions for validation, normalization, the discount boundary, circular substitution, and independent invocations. Inventory is fixed for every invocation. Messages use deterministic templates. Order processing requires no network calls.

Validation: all cells, default Mermaid PNG rendering, and assertions passed in a fresh local Python 3.13.18 Jupyter kernel. Hosted Google Colab validation has not been performed directly. Execution outputs are omitted from the delivered notebook.

## Bonus Version - Dynamic Inventory Management

The bonus version extends the required LangGraph order system with
inventory that changes across sequential orders during the same Colab
runtime.

It includes:

- A separate `inventory_graph`.
- Live stock checks using the current inventory.
- Inventory reduction only for accepted orders.
- Reduction of the product actually sold after substitution.
- Protection against negative inventory and duplicate deductions.
- No inventory change for invalid or rejected orders.
- Inventory reset and restock utilities.
- A readable inventory-status table.
- Sequential tests demonstrating stock depletion and changing
  substitution results.

The implementation is deterministic and uses no LLM, API key, external
database, or persistent external storage. Restarting the Colab runtime
resets the inventory.

- [danas_bakery_bonus_inventory.ipynb](danas_bakery_bonus_inventory.ipynb): bonus notebook with the live inventory workflow and tests.
- [bonus_prompt.md](bonus_prompt.md): exact prompt used to create the bonus notebook.

[Open the bonus version in Colab](https://colab.research.google.com/github/amitbe6120/langgraph-basic-examples/blob/main/examples/danas_bakery/danas_bakery_bonus_inventory.ipynb)
