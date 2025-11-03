## Quick orientation for AI coding agents

This repository is a tiny interactive data-generation agent built with LangChain / LangGraph.
Keep guidance short and concrete so a dev or agent can be productive immediately.

- Entry point: `main.py` — a single-file CLI agent that:
  - Loads environment with `dotenv` and expects `OPENAI_API_KEY` to be set.
  - Defines three reusable tools: `write_json`, `read_json`, and `generate_sample_users` (see `@tool` decorators).
  - Creates an agent via `create_react_agent(llm, TOOLS, prompt=SYSTEM_MESSAGE)` from `langgraph.prebuilt`.
  - Runs an interactive REPL loop when executed as `python main.py`.

- Key files to inspect:
  - `main.py` — core behavior, tool implementations, system prompt, and the `run_agent` wrapper.
  - `pyproject.toml` — project metadata and dependencies (requires Python >= 3.13; depends on `langchain`, `langchain-openai`, `langgraph`, `python-dotenv`).
  - `users.json` — sample output format produced by `write_json` / `generate_sample_users` (useful for examples/tests).

- What I should assume and enforce in edits:
  - Don’t change the agent brain (SYSTEM_MESSAGE) semantics unless asked; tests / examples rely on it.
  - Tool functions are simple synchronous callables decorated with `@tool`. They should return either plain strings or JSON-serializable objects.
  - The agent is multi-turn but `run_agent` wraps the call as a single invoke with message history + `HumanMessage`.

- Developer workflow (how to run & debug):
  - Ensure a Python 3.13+ venv and install dependencies from `pyproject.toml` (pip install from the project or your preferred tool).
  - Provide `OPENAI_API_KEY` in the environment or a `.env` file (project uses `load_dotenv()` in `main.py`).
  - Run interactively: `python main.py` and type prompts such as:
    - "Generate users named John, Jane, Mike and save to users.json"
    - "Create users with last names Smith, Jones aged 25-35 with company.com emails"

- Patterns and examples to follow when editing or extending:
  - Add tools by defining a function and decorating with `@tool`, then append it to the `TOOLS` list. Example: `def my_tool(...):` + `@tool` above it + `TOOLS.append(my_tool)`.
  - Keep tools pure/limited in scope: they should do one job (read file, write file, generate data) and return short status messages or JSON.
  - Messages use LangChain message classes: `HumanMessage`, `AIMessage`, `BaseMessage`. Use these when building `history` or composing `agent.invoke` payloads.

- Known quirks & gotchas (discovered in repository):
  - `main.py` contains a literal line `echo $OPENAI_API_KEY` which is a shell command and will raise an error if executed as Python. Treat it as a comment or remove it when fixing runtime issues.
  - The `pyproject.toml` lists dependencies but the repo has no `requirements.txt`; prefer installing from `pyproject.toml` or directly with pip (e.g., `pip install langchain langchain-openai langgraph python-dotenv`).
  - README.md is empty; prefer adding small usage notes in PRs when changing behavior.

- Minimal contract for contributions:
  - Inputs: user prompt string + optional `history: List[BaseMessage]`.
  - Outputs: `AIMessage` returned by `run_agent` (string content) or tool return values (string/JSON).
  - Error modes: functions return readable error strings (tools and `run_agent` already convert exceptions to message content).

If anything here is unclear or you want me to include sample unit tests or a CI workflow for this agent, tell me which area to expand and I will update the instructions.
