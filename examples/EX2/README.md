# TaskNest Support Ticket Router

A minimal LangGraph example: one structured OpenAI classification call, one conditional edge, and three fixed team replies. `prompt.md` is the implementation specification; `support_ticket_router.ipynb` is the deliverable.

## Run in Google Colab

1. Upload `support_ticket_router.ipynb` to https://colab.research.google.com/ (File > Upload notebook), or open it from GitHub after publishing.
2. Use a fresh standard Python CPU runtime; no GPU is required.
3. In the Secrets panel (key icon), add `OPENAI_API_KEY` with your own OpenAI API key and enable **Notebook access**. Never put the value in code or output.
4. Run cells from top to bottom. Installation, live OpenAI requests, and Mermaid PNG rendering require network access. API usage requires an account with access and available quota.
5. Review the graph drawing, six expected-versus-actual classifications, assertions, and mock checks. The example plus six tickets makes seven live requests, with retries disabled.
6. Complete the 3–5 sentence reflection using actual results.

The only model call is marked in `classify`; the edge decision is marked in `route`. Model name appears once in the configuration cell. Errors are surfaced without retrying or silently routing to General.

## Checks without credentials

The notebook includes mock checks for all three branches, input preservation, fixed handoffs, and one classifier invocation per ticket. To run just these in Colab, run installation/imports, the instruction constant cell, the state/schema definitions cell, the nodes and graph construction, the result-check definitions, and the mock cell. Skip secret loading, model construction, graph drawing, and live execution. No API key or network calls are needed by the mock checks themselves; installation still requires network.

## Validation and submission

Fresh Google Colab execution and live model classification have **not** been performed here. Local checks passed using the real LangGraph and langchain-openai packages: notebook schema and cell syntax; exact graph topology; all three mocked routes; four result fields and fixed replies; one classifier invocation per ticket; offline structured-model construction; and restoration of the real classifier after mocking. No live API request was made. Do not treat mock success as evidence of model accuracy.

Keep repository notebook outputs cleared. In your shared Colab copy, retain the graph drawing and live test outputs, save, and grant the lecturer access. Submit the public GitHub repository URL and the accessible Colab URL; neither has been published by this implementation. Never include a secret in files, outputs, or Git history.

## Model retirement fix

The original `gpt-4.1-nano` was shut down on October 23, 2026 and returns `model_not_found`. The notebook now uses `gpt-6-luna`, the homework's example, with `reasoning_effort="none"` and `temperature=0`. See [deprecations](https://developers.openai.com/api/docs/deprecations), [model details](https://developers.openai.com/api/docs/models/gpt-6-luna), and [parameter compatibility](https://developers.openai.com/api/docs/guides/latest-model#update-api-and-model-parameters).

For an existing Colab copy, change `MODEL_NAME` to `"gpt-6-luna"`, add `reasoning_effort="none"` to `ChatOpenAI(...)`, then rerun from configuration through the live example (or restart the session and run all). Rerunning only the example leaves the old classifier object in memory. This fix has not been validated with a live API call.
