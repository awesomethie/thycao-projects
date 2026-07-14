# Cobase Security Questionnaire Assistant — Setup Guide

A Claude-powered assistant that answers security & due-diligence questionnaires,
grounded in Cobase's policies, audit reports, and a curated answer bank. Claude is
called through **Microsoft Foundry** (Azure's model platform); the app and knowledge
base are version-controlled here on GitHub.

> **Terminology note.** This is **not** an OpenAI "Custom GPT" (that product runs on
> OpenAI's GPT models inside ChatGPT and can't use Claude). This is a small custom
> application you host, that calls the **Claude API** via Microsoft Foundry. The
> knowledge base is the same either way.

---

## Architecture

```
                 ┌───────────────────────────┐
   questionnaire │  app/answer_questionnaire  │
   question ────▶│  (Python)                  │
                 └─────────────┬──────────────┘
                               │  system prompt = instructions + entire
                               │  knowledge base (cached), question as user turn
                               ▼
                 ┌───────────────────────────┐
                 │  Claude (Opus 4.8)         │
                 │  via Microsoft Foundry     │◀── Azure resource + API key
                 └─────────────┬──────────────┘
                               ▼
                    grounded, cited answer
```

- **Knowledge base** — [`CustomGPT_Knowledge/`](../CustomGPT_Knowledge/): ~19 files
  (topic-sorted answer bank, company facts, whitepapers, audit reports, policies, CSV
  registers). Built from the source documents; see the repo history.
- **Retrieval strategy** — the whole knowledge base (~475K tokens) fits inside Claude's
  1M-token context window, so we load all of it into a **cached system prompt** rather
  than running a vector database. First call writes the cache; every later question is a
  cheap cache-read. (Switch to RAG only if you outgrow the context window or want to cut
  per-call cost at very high volume.)

---

## Repository structure

```
.
├── app/
│   └── answer_questionnaire.py   # reference implementation
├── docs/
│   └── README.md                 # this guide
├── CustomGPT_Knowledge/          # the knowledge base (gitignored — confidential)
├── requirements.txt
├── .env.example                  # copy to .env and fill in
└── .gitignore
```

---

## Prerequisites

- **Python 3.10+**
- An **Azure account** with access to **Microsoft Foundry** (formerly Azure AI Foundry)
- Permission to deploy Anthropic/Claude models in Foundry (may require enabling the
  Anthropic models in your Azure subscription/region)
- `git`

---

## Step 1 — Get Claude on Microsoft Foundry

1. Sign in to the **Azure AI Foundry** portal (`ai.azure.com`).
2. Create (or open) a **Foundry resource / project** in a region where Anthropic models
   are offered.
3. In the **model catalog**, find **Claude** (e.g. *Claude Opus 4.8*) and **deploy** it.
   Note the **deployment / model name** — you'll set it as `CLAUDE_MODEL`.
4. From the resource's **Keys and Endpoint** page, copy:
   - the **resource name / endpoint** → `ANTHROPIC_FOUNDRY_RESOURCE`
   - an **API key** → `ANTHROPIC_FOUNDRY_API_KEY`

> Anthropic-on-Foundry is currently a **beta** integration; exact catalog names, regions,
> and the model-id string can change. Confirm the deployment name in the portal and adjust
> `CLAUDE_MODEL` if the API returns a 404.
>
> Alternatives (same app, different client): the **direct Anthropic API**
> (`api.anthropic.com`) is the simplest path if you're not tied to Azure, and **AWS
> Bedrock** if your infra is on AWS.

---

## Step 2 — Clone and configure

```bash
git clone https://github.com/<your-org>/<your-repo>.git
cd <your-repo>

cp .env.example .env
# edit .env and paste your Foundry resource + key
```

## Step 3 — Install dependencies

```bash
python -m venv .venv
# Windows:  .venv\Scripts\activate
# macOS/Linux:  source .venv/bin/activate
pip install -r requirements.txt
```

## Step 4 — Run it

Load the `.env` (e.g. `set -a; . ./.env; set +a` on bash, or use `python-dotenv`), then:

```bash
# single question
python app/answer_questionnaire.py "Do you encrypt data at rest and in transit?"

# a batch — one question per line
python app/answer_questionnaire.py --file questions.txt
```

Each answer ends with a `Source:` line citing the knowledge file(s) used.

---

## Step 5 — How the code works

[`app/answer_questionnaire.py`](../app/answer_questionnaire.py):

1. **Builds the client** — `AnthropicFoundry(api_key=..., resource=...)`.
2. **Loads the knowledge base** — concatenates every file in `CustomGPT_Knowledge/` with
   `===== FILE: <name> =====` delimiters.
3. **Calls Claude** with:
   - `system` = the assistant **instructions** + the **entire knowledge base**, with
     `cache_control` on the knowledge block (prompt caching → cheap repeat calls).
   - `messages` = the question.
   - `thinking={"type": "adaptive"}` and `effort: "medium"` — adjust per question difficulty.
   - **streaming** (`.stream()` + `get_final_message()`) so long / thinking responses don't
     hit HTTP timeouts.

The behavioral rules (source priority, no-blanks, cite sources, surface the ⚠️ breach-history
flag) live in `SYSTEM_INSTRUCTIONS` at the top of the file — edit there to tune behavior.

---

## Step 6 — Put it on GitHub

```bash
git init
git add app docs requirements.txt .env.example .gitignore
git commit -m "Claude questionnaire assistant (Foundry)"
git branch -M main
git remote add origin https://github.com/<your-org>/<your-repo>.git
git push -u origin main
```

> ⚠️ **Confidentiality — read before pushing.** The knowledge base and source documents
> (`CustomGPT_Knowledge/`, `Policies/`, `Filled Questionnaires/`, `Whitepapers and client
> audit reports/`) contain **internal policies, signed audit reports, and your answer
> playbook**. They are **gitignored** by default so they are *not* pushed. Either:
> - keep the repo **private**, **and/or**
> - store the knowledge base in **Azure Blob Storage / Key Vault** and load it at runtime
>   (don't commit it at all — the safest option).
>
> Never commit `.env` or API keys.

### CI / deployment secrets
Put `ANTHROPIC_FOUNDRY_API_KEY` and `ANTHROPIC_FOUNDRY_RESOURCE` in **GitHub → Settings →
Secrets and variables → Actions** (for CI) or in your Azure host's app settings — never in
the repo.

---

## Step 7 — (Optional) Deploy on Azure

The app is plain Python, so any of these work:
- **Azure Functions** — wrap `answer_question()` in an HTTP-triggered function; load the
  knowledge base from Blob Storage on cold start.
- **Azure Container Apps / App Service** — containerize with `requirements.txt`; set the
  env vars as app settings; mount or download the knowledge base at startup.

Keep the knowledge base out of the image — pull it from Blob Storage / Key Vault so secrets
and confidential docs never live in the container image or the repo.

---

## Cost & model notes

- **Model:** Claude Opus 4.8 (`claude-opus-4-8`). Adaptive thinking on; `effort` controls the
  quality/cost trade-off (`low`→`max`). Use `medium` for routine questions, raise for hard ones.
- **Prompt caching** makes the big knowledge base cheap on repeat calls — the first question
  pays the full read + a cache-write premium; subsequent questions within the cache TTL read
  it at ~0.1× cost. Batch a questionnaire in one run to maximize cache hits.
- The knowledge base (~475K tokens) is large but well under the 1M context window. If it ever
  approaches the limit, drop the CSV registers from the loaded set or move to RAG.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `404` / model not found | `CLAUDE_MODEL` must match the **deployment name** in the Foundry catalog |
| `401` / auth error | Check `ANTHROPIC_FOUNDRY_API_KEY` and `ANTHROPIC_FOUNDRY_RESOURCE`; regenerate the key if needed |
| `No knowledge files found` | Set `KNOWLEDGE_DIR` or run from the repo root so `CustomGPT_Knowledge/` is visible |
| Slow first answer | Expected — the first call writes the prompt cache; later answers are fast |
