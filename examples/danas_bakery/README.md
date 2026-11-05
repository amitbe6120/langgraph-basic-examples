# Dana's Bakery

Open `danas_bakery.ipynb` in Google Colab using **File → Upload notebook**, select a standard CPU runtime, and choose **Runtime → Run all**. No manual edits, secrets, API keys, or GPU are required. Setup downloads pinned Python dependencies, Chromium, and a Mermaid library; graph rendering and order processing then run locally in the runtime.

The notebook contains nine LangGraph nodes, the actual graph diagram, twelve required order scenarios, and additional assertions for validation, normalization, the discount boundary, circular substitution, and independent invocations. Inventory is fixed for every invocation. Messages use deterministic templates.

Validation: all notebook cells and assertions were executed in a fresh local Python 3.11 Jupyter kernel, including the setup and PNG rendering cells. A hosted Google Colab runtime was not available for direct validation. Execution outputs are omitted from the delivered notebook.
