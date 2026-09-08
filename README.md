# AI-agent demo notebooks

Small, self-contained notebooks that build shared intuition, each lighting up **one piece of the agentic workflow** — from "an LLM" up through memory, coordination, uncertainty, evaluation, a real FHIR environment, and safety. They run **immediately with no API key** in a MOCK mode (scripted responses) so you can see the structure, then swap in a real model unchanged.

### Foundations — what an agent is, and how we keep it safe
| Notebook | Piece | Shows |
|---|---|---|
| `01_llm_vs_tool.ipynb` | Tool use | Why a **tool** makes results reliable, where a cleverer prompt doesn't |
| `02_minimal_agent_loop.ipynb` | The control loop | **Perceive → plan → act → observe** in ~30 transparent lines, on an anticoagulant follow-up |
| `03_human_in_the_loop_gate.ipynb` | The human gate | A **consequential action gated to human approval**, enforced by code and logged |

### The properties that decide whether an agent can be trusted in a workflow
| Notebook | Piece | Shows |
|---|---|---|
| `04_memory_and_stale_facts.ipynb` | Memory / state | Durable memory **and its failure mode** — a stale fact resurfacing and driving a wrong action |
| `05_tool_retrieval_and_routing.ipynb` | Selection & cost | Narrow a big tool set to the few relevant ones, and **route easy vs. hard tasks to cheap vs. strong models** |
| `06_uncertainty_and_abstention.ipynb` | Uncertainty | **Knowing when it doesn't know**: self-consistency, abstain/escalate, and the alert-fatigue dial |
| `07_compounding_reliability.ipynb` | Evaluation | *Measure it yourself*: ~0.95/step compounds to ~0.77 end-to-end (pure simulation, no model needed) |
| `08_orchestrator_and_specialists.ipynb` | Coordination | An **orchestrator + specialist sub-agents** with a verify step — and when one agent is the better call |

### Toward the real virtual patient environment
| Notebook | Piece | Shows |
|---|---|---|
| `09_fhir_sandbox_agent.ipynb` | The environment | The same loop + gate over a **FHIR** interface (mock by default; point at Synthea + HAPI FHIR for real) |
| `10_safety_prompt_injection.ipynb` | Safety-as-architecture | A retrieved note hijacks the agent; a **gate + dose-sanity policy** block it — you can't prompt your way to safety |

Suggested walkthrough order: **01 → 02 → 03**, then **04 → 08** for the trust properties, then **09 → 10** to bridge toward the environment.

## Running them

They run out of the box with **no key** in a scripted MOCK mode. To produce authentic
model output, point the one `chat()` seam at a real model with three env vars and re-run.
**For the full run-and-commit sequence (with key hygiene), see [RUNBOOK.md](RUNBOOK.md).**

```bash
pip install openai   # (09 also uses `requests` only if you point it at a real FHIR server)

# Option A — OpenRouter (OpenAI-compatible; one key, many models)
export OPENAI_BASE_URL=https://openrouter.ai/api/v1
export OPENAI_API_KEY=sk-or-...          # never commit this — .env is gitignored
export MODEL=openai/gpt-4o-mini          # any model id OpenRouter lists

# Option B — OpenAI directly
export OPENAI_API_KEY=sk-...
export MODEL=gpt-4o-mini

# Option C — local open-weight model (vLLM / LM Studio / Ollama, OpenAI-compatible)
export OPENAI_BASE_URL=http://localhost:8000/v1
export OPENAI_API_KEY=dummy
export MODEL=meta-llama/Llama-3.1-8B-Instruct
```

Setting `OPENAI_API_KEY` flips the backend from MOCK to REAL; the first cell of each
notebook prints which backend is active. Embed real outputs in one pass with:

```bash
GATE_AUTO=approve jupyter nbconvert --to notebook --execute --inplace *.ipynb
```

The code is model-agnostic: swap the model, keep the loop. Notes:
- **NB3 / NB9** honour `GATE_AUTO=approve|deny` for non-interactive runs.
- **NB6** estimates confidence by sampling the model at `temperature=1`.
- **NB7** needs no API key at all — it is pure simulation about *measurement*.
- **NB9** runs on an in-notebook mock FHIR bundle by default; generate patients with Synthea, load them into a HAPI FHIR server, and `export FHIR_BASE_URL=http://localhost:8080/fhir` to run against real (synthetic) data — the agent code doesn't change.

These are teaching scaffolds. For production experiments, run inside a proper framework (e.g. LangGraph) over Synthea + HAPI FHIR — the agent logic stays the same. The one piece none of these build, and the paper flags as the real gap, is a **dynamics layer** (an action changing future patient state); that's where a research program adds the most.
