# ChefCoach — A Duolingo-Style Cooking Coach Agent (RAG + Skill Progression)

ChefCoach is an AI agent that teaches cooking as a curriculum:
**skills → practice → feedback → level-up**, not just one-off recipe recommendation.

This repo is designed to match the project proposal:
- **Structured learner profile** (skill vector + preferences + constraints) stored in **SQLite/JSON**
- **Skill graph + curriculum planner** to pick the next lesson
- **RAG retrieval** with **ChromaDB** (fine-grained chunks → reassembled recipe packets)
- **RLHF-inspired feedback loop** (rule-based weight updates; no PPO training)
- **Streamlit UI** for a live demo

---

## 1) Project Structure

```
chefcoach/
  app.py                 # Streamlit UI (Onboarding + Guided Session + Dashboard)
  setup_rag.py            # Build Chroma vector DB (one-time)
  requirements.txt
  .env.example

  data/
    skills.json           # 8 core skills + prerequisites
    lessons.json          # lesson templates + quizzes
    sample_recipes.json   # small fallback recipe set (always works)
    technique_qna.json    # Tier-2 technique snippets for "why/how" help

  modules/
    profile_manager.py
    skill_assessor.py
    curriculum_planner.py
    rag_retriever.py      # Chroma RAG + packet reassembly
    coach_generator.py    # LLM prompting + "I'm stuck" helper
    feedback_updater.py
    llm_client.py         # Gemini/OpenAI/mock LLM interface
    technique_qna.py      # optional local retrieval for technique explanations

  evaluation/
    run_eval.py
    llm_judge.py
    curriculum_test.py
    grounding_check.py
    user_study_template.py
```

---

## 2) Quick Start (VSCode)

### Step 0 — Create and activate a virtual environment

```bash
python -m venv .venv
# macOS/Linux
source .venv/bin/activate
# Windows
# .venv\Scripts\activate
```

### Step 1 — Install dependencies

```bash
pip install -r requirements.txt
```

### Step 2 — Configure environment variables

```bash
cp .env.example .env
```

Recommended: **Gemini online LLM**
- Set `LLM_PROVIDER=gemini`
- Put your `GEMINI_API_KEY=...`
- Optionally set `GEMINI_MODEL=gemini-2.5-flash-lite`

### Step 3 — Build the RAG index (one-time)

```bash
python setup_rag.py
```

- If you have Food.com CSV files:
  - Put `RAW_recipes.csv` into `chefcoach/data/RAW_recipes.csv` (required for Food.com RAG)
  - Put `RAW_interactions.csv` into `chefcoach/data/RAW_interactions.csv` (optional, enriches packets with average rating + rating_count)
  - See `/data/FOODCOM_SETUP.md` for a full download/placement checklist
- Otherwise it will index the built-in `data/sample_recipes.json` so the demo still works.

### Step 4 — Run the UI

```bash
streamlit run app.py
```

Open the URL shown in your terminal (usually http://localhost:8501).

---

## 3) Demo Script (2 minutes)

1. **Onboarding**
   - Fill your constraints (diet/time/equipment)
   - Complete the skill quiz

2. **Guided Session**
   - Add optional session goal + available ingredients
   - Click **Generate Coaching Session**
   - Observe: target skill, difficulty, recipe packets (RAG) -> coaching
   - Ask an **"I'm stuck"** question (with optional technique snippet support)
   - Complete quiz and submit feedback on the same page

3. **Dashboard**
   - Check session timeline, skill radar, unlock state, and next lesson recommendation

4. **Session 2**
   - Generate again and confirm adaptation from prior feedback

---

## 4) Evaluation

Run all evaluation methods:

```bash
python evaluation/run_eval.py
```

Notes:
- `evaluation/llm_judge.py` supports an independent judge model via `JUDGE_*` env vars.
- Current recommended default is local Ollama judge: `llama3.1:8b`.
- `evaluation/grounding_check.py` now evaluates real generated coaching text against retrieved packets (not simulated filler text).

For fair **offline vs online** comparison on metric A (same judging scale):
- keep generation config as you like (`LLM_PROVIDER` + model),
- but fix judge config across runs via `JUDGE_*` env vars.

Example (same local judge for both runs):
```bash
JUDGE_PROVIDER=openai
JUDGE_OPENAI_API_BASE=http://localhost:11434/v1
JUDGE_OPENAI_API_KEY=ollama
JUDGE_MODEL_NAME=llama3.1:8b
EVAL_A_SCENARIO_LIMIT=24
```

`EVAL_A_SCENARIO_LIMIT` controls metric-A scenario count.
Default is 24 (3 learner levels x 8 feedback variants). Lower it (for example 8)
for faster debugging, then restore 24 for final reporting.

---

## 5) Dataset --- Download Data --- The data file is too large to upload --- Download from website
Use Kaggle website to get a Food.com dataset that includes
`RAW_recipes.csv` and `RAW_interactions.csv`.
