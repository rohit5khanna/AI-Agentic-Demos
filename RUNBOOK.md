# Runbook — producing authentic model output with OpenRouter

These notebooks ship with **outputs cleared**. They run out of the box in a scripted
MOCK mode (no key), but to publish *authentic* model output you run them once against a
real model and let Jupyter embed the results. This is the exact sequence.

## 1. Install

```bash
pip install openai jupyter nbconvert requests
```

## 2. Point the code at OpenRouter

The notebooks are model-agnostic: they call one `chat()` seam that reads three env vars.
OpenRouter is OpenAI-compatible, so it drops straight in.

```bash
export OPENAI_BASE_URL=https://openrouter.ai/api/v1
export OPENAI_API_KEY=sk-or-...        # your OpenRouter key
export MODEL=openai/gpt-4o-mini        # any model id OpenRouter lists
```

Setting `OPENAI_API_KEY` is what flips the backend from MOCK to REAL — the first cell of
each notebook prints `Backend: REAL model = ...` so you can confirm before trusting output.

### Key hygiene (important)

- **Never commit the key.** `.gitignore` already excludes `.env`. Keep the key in your
  shell env or an untracked `.env`, never in a notebook cell.
- If you'd rather use a file: put the three `export` lines in `.env` and `source .env`
  before launching — do **not** `git add` it.
- Before pushing, sanity-check nothing captured the key:
  `grep -rIn "sk-or-" . && echo "STOP: key found" || echo "clean"`

## 3. Execute and embed outputs

One command runs every notebook and writes the outputs back in place:

```bash
GATE_AUTO=approve jupyter nbconvert --to notebook --execute --inplace *.ipynb
```

- `GATE_AUTO=approve` lets **NB3** and **NB9** run their human-approval gate
  non-interactively (it records the approved-and-logged path). Use `GATE_AUTO=deny` if you
  want to capture the blocked path instead.
- To run interactively instead, open each in Jupyter and Run All — same result.

## 4. What each notebook needs

| Notebook | Needs the model? | Note |
|---|---|---|
| 01 llm_vs_tool | yes | model gets the number via a tool |
| 02 minimal_agent_loop | yes | model plans; data comes from tools |
| 03 human_in_the_loop_gate | yes | run with `GATE_AUTO=approve` |
| 04 memory_and_stale_facts | yes | facts supplied in-context |
| 05 tool_retrieval_and_routing | **no** | pure routing/cost logic — same output real or mock |
| 06 uncertainty_and_abstention | yes | samples at `temperature=1`; questions grounded in a record |
| 07 compounding_reliability | **no** | pure simulation about measurement |
| 08 orchestrator_and_specialists | yes | sub-agents reason over a given patient record |
| 09 fhir_sandbox_agent | yes | mock FHIR bundle by default; `FHIR_BASE_URL` for real |
| 10 safety_prompt_injection | yes | model is *meant* to try the hijack; the gate blocks it |

05 and 07 carry no model call by design (they demonstrate plumbing/measurement), so their
output is identical either way — that's expected, not a bug.

## 5. A note on "authentic"

With a real model the numbers stop being scripted:

- **NB6** confidence now comes from the model's own sampling spread — watch *which*
  questions it turns out to be quietly unsure about. Agreement measures consistency, not
  truth, so a confidently-wrong answer is possible; that's the honest limit the notebook
  states.
- **NB2 / NB3 / NB08 / NB10** reason over data you give them in-context, so they stay
  grounded rather than inventing clinical facts.
- If your chosen model ever emits slightly malformed JSON in the ReAct loops
  (NB2/3/9/10), lower `temperature` or pick a stronger model; the loops expect a JSON
  action object.

## 6. Commit

```bash
git add *.ipynb README.md RUNBOOK.md requirements.txt .gitignore
git commit -m "Add agentic demo notebooks with executed model outputs"
```

Confirm the diff shows embedded outputs and **no key** before you push.
