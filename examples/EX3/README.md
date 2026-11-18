# Curious Minds Club Chatbot

A Google Colab example using a one-node LangGraph graph, Supabase Postgres checkpointing, and a Gradio interface. Conversations retain their messages and selected club across runtime restarts.

[Open the notebook in Google Colab](https://colab.research.google.com/github/amitbe6120/langgraph-basic-examples/blob/main/examples/EX3/club_chatbot.ipynb)

## Setup and execution

1. Create a Supabase project and save its database password securely.
2. Select **Connect → Connection String → Session pooler** (port 5432). Replace the password placeholder in the URI with your password; URL-encode any special password characters.
3. In Colab Secrets, add `SUPABASE_DB_URL` containing the complete URI and `OPENAI_API_KEY` containing your API key. Enable Notebook access for both. Never put these values into code or outputs.
4. Use a fresh standard CPU runtime and run the notebook cells from top to bottom. The first code cell installs all required dependencies.
5. The model name is configured once in `MODEL_NAME`. The notebook uses the assignment's `openai:gpt-6-luna`; if unavailable to your account, replace that value with an available low-cost chat model before running.
6. After checkpoint setup, inspect the checkpoint tables in the Supabase Table Editor. Use the Gradio interface to select a club, create conversations, and reopen saved conversations.

The notebook uses `prepare_threshold=None` to disable prepared statements; the assignment's description of `prepare_threshold=0` is incorrect.

## Validation

Python cell syntax and isolated function checks with a mocked model and message stand-ins passed during preparation. Full execution in Google Colab with the actual libraries, OpenAI, and Supabase has not been performed by the authoring agent.

The notebook includes the required four-message demo, state assertions, mocked node checks, and manual acceptance tests. Inspect refusal quality yourself. Restart the Colab runtime, rebuild the graph and interface, and reopen a saved conversation to verify that both its messages and club survive.

The standalone demo makes four paid model calls each time it runs. Its persisted `test-1` thread gains eight messages per run. The Gradio link permits visitors to use your API key through the app, and the demo shares its saved conversation list with all visitors. Keep the link private, use fictional data, and stop the app when finished.

## Submission

`prompt.md` contains the implementation specification. Submit the public repository and an accessible Colab link, then add your own 4–6 sentence reflection based on actual tests. Review and clear notebook outputs before committing; generate the required graph drawing and test outputs in your submission copy after checking them for sensitive information.
