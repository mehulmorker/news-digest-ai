# News Digest AI

A multi-agent news intelligence pipeline built with [CrewAI](https://docs.crewai.com/). Given a topic, it searches the web, scrapes credible articles, generates multi-tier summaries, and produces a publication-ready briefing — all saved as markdown files.

## How it works

Three agents run sequentially:

| Agent | Role | Tools |
|-------|------|-------|
| `news_hunter_agent` | Searches for recent articles, filters by credibility and relevance, scores each one | Serper (search), Playwright scraper |
| `summarizer_agent` | Produces headline (≤280 chars), executive (150–200 words), and comprehensive (500–700 words) summaries per article | Playwright scraper |
| `curator_agent` | Assembles everything into a final, publication-ready briefing with editorial analysis | — |

Each task writes its own output file:

```
output/
  content_harvest.md   # articles with metadata & scores
  summary.md           # multi-tier summaries per article
  final_report.md      # final briefing (the main deliverable)
```

## Requirements

- Python 3.13
- [uv](https://docs.astral.sh/uv/) (package manager)
- A Chromium installation for Playwright

## Setup

```bash
# Install dependencies
uv sync

# Install Playwright's Chromium browser
uv run playwright install chromium

# Copy the env template and fill in your keys
cp .env.example .env
```

Edit `.env`:

```env
OPENROUTER_API_KEY=your_openrouter_key_here
SERPER_API_KEY=your_serper_key_here
```

- **OpenRouter API key** — get one at [openrouter.ai](https://openrouter.ai/). The agents use `tencent/hy3-preview:free` by default (change in `config/agents.yaml`).
- **Serper API key** — get one at [serper.dev](https://serper.dev/). Used for Google search results.

## Usage

Edit the topic in `main.py`:

```python
result = NewsDigestAI().crew().kickoff(inputs={"topic": "your topic here"})
```

Then run:

```bash
uv run main.py
```

The final briefing is written to `output/final_report.md`.

## Project structure

```
.
├── main.py              # Entry point — wires agents/tasks, kicks off the crew
├── tools.py             # SerperDevTool (search) + custom Playwright scraper
├── config/
│   ├── agents.yaml      # Agent roles, goals, backstories, LLM config
│   └── tasks.yaml       # Task descriptions, expected outputs, output file paths
├── output/              # Generated markdown reports (created at runtime)
└── pyproject.toml
```

## Changing the LLM

All three agents use `openrouter/tencent/hy3-preview:free` by default. To swap models, edit the `llm` field in `config/agents.yaml`. Any OpenRouter-compatible model string works.
