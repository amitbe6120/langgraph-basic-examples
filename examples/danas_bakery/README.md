# Dana's Bakery

A deterministic LangGraph order-processing example for Google Colab. Python nodes validate bakery orders, check stock and up to two substitutes, select delivery or pickup, calculate discounts, and produce customer messages with an audit diary.

## Files

- [danas_bakery.ipynb](danas_bakery.ipynb): complete notebook, graph visualization, and tests.
- [danas_bakery_prompt.md](../../danas_bakery_prompt.md): the requested Python 3.13 setup and rendering correction.

Open `danas_bakery.ipynb` in Google Colab using **File → Upload notebook**, select a standard CPU runtime with Python 3.13, and choose **Runtime → Run all**. No manual edits, secrets, API keys, or GPU are required. The first cell installs LangGraph using `%pip install -q -U langgraph`. The graph uses the default `draw_mermaid_png()` renderer, which requires internet access. No browser installation is needed.

The notebook contains nine LangGraph nodes, the actual graph diagram, twelve required order scenarios, and additional assertions for validation, normalization, the discount boundary, circular substitution, and independent invocations. Inventory is fixed for every invocation. Messages use deterministic templates. Order processing requires no network calls.

Validation: all cells, default Mermaid PNG rendering, and assertions passed in a fresh local Python 3.13.18 Jupyter kernel. Hosted Google Colab validation has not been performed directly. Execution outputs are omitted from the delivered notebook.
