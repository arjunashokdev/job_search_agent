# Job Search Agent

A small LangGraph agent that watches a list of companies' careers pages and tells you when a role you care about opens up — so you don't have to manually re-check job boards.

## What it solves

Job hunting usually means repeatedly opening the same handful of careers pages to check for new postings. This notebook automates that: give it a list of companies and a target job title, and it checks each one and reports back `FOUND` / `NOT FOUND` with the matching listing.

## How it works

- **Orchestration**: LangGraph's `create_react_agent` — a minimal ReAct loop where the LLM decides which company to check next.
- **Tool calling**: the agent calls two tools — `list_companies` and `search_careers_page` — to do the actual work.
- **RAG**: each careers page is fetched, chunked, embedded with a local HuggingFace sentence-transformer, and searched with FAISS so only the most relevant snippets reach the LLM.
- **LLM**: an open-source HuggingFace chat model (default: `Qwen/Qwen2.5-7B-Instruct`), loaded directly into the runtime and run 4-bit quantized on GPU — no HuggingFace API key or paid inference endpoint required.
- **Tracing (optional)**: [Arize Phoenix](https://github.com/Arize-ai/phoenix), an open-source LLM observability tool, runs locally and gives you a full trace of every tool call and LLM step — no account or API key needed.

## Requirements

- A Python environment with **GPU access** (e.g. a Google Colab GPU runtime) — the default model is loaded in full and run 4-bit quantized, which needs a CUDA GPU.
- No API keys are required for the LLM or for tracing.

## How to run

1. Open `job_search_agent.ipynb` in Colab (or a local Jupyter environment with a GPU).
2. In the config cell, set:
   - `COMPANIES` — a dict of company name → careers page URL.
   - `TARGET_JOB` — the job title/keywords to watch for.
   - `MODEL_ID` — any HuggingFace chat model that supports tool calling (defaults to `Qwen/Qwen2.5-7B-Instruct`; drop to a smaller model like `Qwen/Qwen2.5-1.5B-Instruct` if you hit GPU memory limits).
3. Run all cells top to bottom. The first run installs dependencies, loads the model into the GPU runtime, and (optionally) starts a local Phoenix tracing UI at `http://localhost:6006`.
4. Re-run the last cell any time you want to re-check — that's your "monitor" step.

## Known limitation

The page fetcher uses a plain HTTP `requests` call, which only works on **server-rendered** careers pages (e.g. Greenhouse, Lever). Heavily JS-rendered boards (e.g. Ashby) return an empty page shell and won't produce results — use a company's Greenhouse/Lever board where available instead.
