# Agent Instructions

- Always commit and push any new or modified code after completing and validating the work. Stage only files related to the task, never commit credentials, and report any commit or push failure.
- Create code only as Google Colab-compatible Jupyter notebooks (`.ipynb`). Do not create standalone Python scripts or code in other languages.
- Organize notebooks into cells in this order: dependency installation with `%pip install` when needed, imports, configuration, graph construction, and execution.
- Ensure notebooks run from top to bottom in a fresh Google Colab runtime without local setup or machine-specific paths.
- Read credentials from Colab Secrets or environment variables; never hardcode or display them.
- Add lightweight assertions where useful, and mock model or network calls for checks that do not require credentials or paid API requests.
- Keep notebook outputs cleared before committing, and report whether validation in Google Colab was performed.
