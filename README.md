## Quantify Sycophancy induced contradiction

- experiment setup:
  - generate yes/no answers for prompts,
  - generate counter-arguements given initial answer.
  - generate a second yes/no answer given counterarguments.
- Prereqs: Python 3.13, dependencies `pip install -r requirements` (or `uv pip install -r pyproject.toml`); set `OPENAI_API_KEY` in your environment.
- Key flow in the notebook:
  - `getInitialAnswer(prompt, model)`: asks the model for a yes/no + brief rationale.
  - `getCounterArgument(prompt, initialAnswer, ...)`: produces a counterargument to challenge the first answer.
  - `getSecondAnswer(counter, history, prompting, model)`: applies a counter-prompt (e.g., “I disagree.”) plus the counterargument to elicit the second response.
  - `insertLog(...)`: writes initial, counter, and second responses into `experiments_log`.
  - `runExperiment("<model-name>")`: loops over prompts and counter_prompting variants to populate the DB.
- How to run:
  1. Export your key: `export OPENAI_API_KEY=...`
  2. Open `run_experiment.ipynb` in Jupyter/Lab, adjust `counter_prompting` list or target model (e.g., `kimi-k2-0905-preview`, `gpt-5`, `gpt-4o`).
  3. You may want to create a new `client` for the target LLM using API key.
  4. Run the cells; results append to `data.db` by default.

## Database: data.db

Schema (SQLite):
- `prompts(prompt_id INTEGER PK, prompt TEXT NOT NULL, topic TEXT, type TEXT)`
  - non-debatable and debatable prompts generated with GPT-5 across 12 different domains.
- `experiments_log(experiment_id INTEGER PK, prompt_id INTEGER NOT NULL, model TEXT, initial_response TEXT, counter_example TEXT, second_response TEXT, counter_prompting TEXT, FOREIGN KEY(prompt_id) REFERENCES prompts(prompt_id) ON DELETE CASCADE)`
  - experiment results 

Notes:
- `counter_example` stores the generated counterargument text (may be empty for some models/variants).
- `counter_prompting` is the wording used to nudge the model before the second response.
- `initial_response` and `second_response` hold the first/second model answers; use `analysis.ipynb`’s `isYesResponse` to derive flip/contradiction labels.

## Running experiments quickly
- To visualize experiment result analysis, open `analysis.ipynb`
- To explore experiments results, use database visualization tool such as datasette to read `experiments_log` from data.db
