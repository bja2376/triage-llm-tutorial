# Classifying ED Visits with an LLM: Zero-Shot, Few-Shot and Chain-of-Thought on MIMIC-IV-ED

A small, locally run LLM reads an emergency department triage record and guesses the ICD-10 chapter of the diagnosis the ED later recorded. The tutorial compares three prompting techniques on the same visits and scores each one against the real ED diagnosis.

- **Notebook:** [`triage_llm_tutorial.ipynb`](triage_llm_tutorial.ipynb), the whole tutorial with outputs
- **Read-only version:** [`triage_llm_tutorial.pdf`](triage_llm_tutorial.pdf), if you just want to read it without running anything

## What you'll learn

1. Run an open-weight LLM on your own machine with [Ollama](https://ollama.com), so patient data never leaves it.
2. Join MIMIC-IV-ED tables into one row per ED stay, without leaking the answer into the prompt.
3. Map ICD-10 codes to chapters and use them as classification labels.
4. Turn a table row into a text prompt.
5. Ask for structured JSON output with Pydantic and Ollama's `format=`.
6. Compare **zero-shot**, **few-shot** and **chain-of-thought** prompting, and score them with accuracy, an invented-code rate and a confusion matrix.

## Quick start

You need Python 3.10+ and about 3 GB of free disk space for the model.

```bash
git clone https://github.com/bja2376/triage-llm-tutorial.git
cd triage-llm-tutorial

python -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install jupyter
jupyter notebook triage_llm_tutorial.ipynb
```

1. **Install Ollama** from [ollama.com](https://ollama.com) and keep the app running.
2. **Run All.** The first cells install the Python packages and download the model (`qwen2.5:3b`, about 2 GB).
3. **Expect about 10 minutes** for the full run on a laptop, mostly the chain-of-thought prompt. Lower `N_CASES` in the config cell for a faster run.

The data is already in `data/`, so there's nothing to download and no PhysioNet account is needed.

**Using VS Code?** Open this folder itself (File → Open Folder), not a parent folder. Otherwise VS Code won't find `.venv` when you pick a kernel.

## Repository layout

```
triage_llm_tutorial.ipynb   the tutorial
triage_llm_tutorial.pdf     exported copy with all outputs
data/ed/                    MIMIC-IV-ED Demo: edstays, triage, diagnosis
data/hosp/                  MIMIC-IV Demo: patients, d_icd_diagnoses
```

## Tutorial outline

| Step | What happens |
|---|---|
| 1. Setup | Install packages, pull the model, time one call |
| 2. Data | Load 5 tables and join them into one row per ED stay (221 stays, 63 patients) |
| 3. Labels | Keep ICD-10 stays (122), map each primary diagnosis to its chapter |
| 4. Prompt | `make_prompt` turns a triage row into a short text summary |
| 5. Ask the model | Same stays, three prompts; JSON answers validated with Pydantic |
| 6. Score | Chapter accuracy, invented-code rate, confusion matrix |
| 7. Limitations | What the results can and can't tell you |

## Results at a glance

On 94 scored ED stays, **no prompt clearly beat zero-shot**: zero-shot 31%, few-shot 30%, chain-of-thought 24% chapter accuracy. With a sample this small, a gap needs to be about 10 points before it's clearly real. Few-shot invented fewer ICD codes than zero-shot (43% vs. 49%), while chain-of-thought invented the most (60%) and took about 5× as long. See the notebook for the full discussion.

## Troubleshooting

- **`ConnectionError` from `ollama`:** the Ollama app isn't running. Start it and rerun the cell.
- **`No module named pip` in the install cell:** the environment was made with `uv venv`. Recreate it with `uv venv --seed`, or use `python -m venv`.
- **A run seems stuck:** small models can loop while writing JSON. Every call is capped at 512 output tokens (`num_predict`), so a stuck call eventually ends and counts as a parse failure.

## Data and license

MIMIC-IV-ED Demo and MIMIC-IV Demo, PhysioNet, released under the [Open Data Commons Open Database License v1.0](https://opendatacommons.org/licenses/odbl/1-0/). These are small, openly licensed samples. The full MIMIC databases need credentialed access and must not be sent to hosted LLM services; this tutorial uses a local model so the same code would work on them.
