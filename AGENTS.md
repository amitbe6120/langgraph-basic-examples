# Repository Guidelines

## Required Output: Google Colab Python

This project generates only Python code intended to run in Google Colab. Apply this requirement to all new examples and code changes. Organize code into notebook-ready cells, with dependency installation first, followed by imports, configuration, graph construction, and execution. Use Colab-supported setup commands such as `%pip install` when needed. Do not generate code in other languages or require local terminal setup, virtual environments, or machine-specific paths.

## Project Structure & Module Organization

This directory contains basic LangGraph notebooks. `examples/danas_bakery/` contains a self-contained Colab notebook and README; its assertions live inside the notebook. `ticket_router.ipynb` is an existing top-level notebook. No shared build or test tooling has been added.

For new examples, use a separate directory for each topic, such as `examples/simple_graph/`, with a Colab-compatible notebook or Python code arranged into copyable notebook cells and a short README. Keep examples self-contained and sample inputs alongside the example that uses them. Do not commit generated outputs.

## Build, Test, and Development Commands

No build or test tooling is configured. Include required dependencies in an initial Colab `%pip install` cell. Document how to open or paste the example into Colab and run all cells from top to bottom in a fresh runtime. State any required runtime settings or credentials.

## Coding Style & Naming Conventions

For Python additions, use four-space indentation, `snake_case` for functions and modules, and `PascalCase` for classes. Use descriptive names for graph nodes and state fields. Keep examples small, explain routing decisions, and separate graph construction from interactive execution. No formatter or linter is currently configured.

## Testing Guidelines

No testing framework or coverage threshold exists. Add lightweight Python assertions where useful to check state updates, conditional routing, and termination. Mock model and network calls so routine checks require neither credentials nor paid API requests. Validate examples by running cells in order in a fresh Colab runtime; report when this validation has not been performed.

## Commit & Pull Request Guidelines

No Git history is available to establish existing conventions. Use short, imperative commit subjects, such as `Add conditional routing example`. Pull requests should describe the example or behavior changed, include reproduction commands and validation results, and link relevant issues when available.

## Security & Configuration

Read API credentials from Colab Secrets or environment variables. Never hardcode or commit secrets, and avoid displaying them in notebook output. Explain required secret names using placeholder values.
