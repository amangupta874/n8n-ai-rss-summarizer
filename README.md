# n8n AI RSS Summarizer 🚀

[![n8n](https://img.shields.io/badge/n8n-2.44.6-EA4B71?logo=n8n&logoColor=white)](https://n8n.io)
[![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?logo=docker&logoColor=white)](https://www.docker.com)
[![OpenRouter](https://img.shields.io/badge/AI-OpenRouter-6851FF)](https://openrouter.ai)
[![Google Sheets](https://img.shields.io/badge/Integration-Google%20Sheets-34A853?logo=googlesheets&logoColor=white)](https://sheets.google.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An automated, production-ready AI workflow powered by **n8n**, **Docker**, **OpenRouter (Free LLMs)**, and **Google Sheets**.

This workflow monitors an RSS feed (e.g., BBC Technology), selects recent articles, summarizes each article into 2 plain-English sentences using an LLM, and appends the structured results into a Google Sheet in real-time.

---

## 🏗️ Workflow Architecture

```
┌─────────────────────────────────┐
│ When clicking 'Execute workflow'│ (Manual Trigger)
└───────────────┬─────────────────┘
                │
┌───────────────▼─────────────────┐
│            RSS Read             │ Fetches BBC Technology RSS feed
└───────────────┬─────────────────┘
                │
┌───────────────▼─────────────────┐
│             Limit               │ Filters top 5 articles (rate limit friendly)
└───────────────┬─────────────────┘
                │
┌───────────────▼─────────────────┐     ┌────────────────────────┐
│        Basic LLM Chain          │◄────┤ OpenRouter Chat Model  │
│ (3 retries, exponential backoff)│     │ (openrouter/free)      │
└───────────────┬─────────────────┘     └────────────────────────┘
                │
┌───────────────▼─────────────────┐
│          Google Sheets          │ Appends: Title | Link | Published | Summary
└─────────────────────────────────┘
```

---

## 📁 Repository Structure

```
.
├── docker-compose.yml             # Docker configuration for n8n
├── .env.example                   # Environment variable template
├── workflows/
│   ├── RSS_Summarizer.json        # Main RSS -> LLM -> Google Sheets workflow
│   └── My_First_Workflow.json     # Handout introductory test workflow (GitHub Zen)
├── scripts/
│   ├── simulate_pipeline.js       # Standalone test script for RSS & LLM parsing
│   └── start.sh                   # Helper startup script
├── data/
│   ├── rss_summaries_preview.csv  # Sample export of summarized output
│   └── rss_summaries_preview.json # Sample JSON payload
└── README.md
```

---

## ⚡ Quickstart (with Docker)

### Prerequisites
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running.
- A free [OpenRouter](https://openrouter.ai/) account and API key.
- A Google account (for Google Sheets).

### 1. Start n8n with Docker
Clone the repository and run:

```bash
# Start n8n in detached mode
docker compose up -d

# Check container status
docker compose ps
```

Open your browser to: **[http://localhost:5678](http://localhost:5678)**

### 2. Import the Workflows
In the n8n Web UI:
1. Click the **...** menu in the top right.
2. Select **Import from file**.
3. Choose `workflows/RSS_Summarizer.json` (or `workflows/My_First_Workflow.json`).

---

## 🔧 Everyday Docker Commands

Run these inside the project directory:

| Action | Command |
| :--- | :--- |
| **Start n8n** | `docker compose up -d` |
| **Stop n8n** *(preserves workflows and data)* | `docker compose stop` |
| **Restart n8n** | `docker compose start` |
| **View live logs** | `docker compose logs -f` *(Ctrl + C to exit)* |
| **Shut down container** | `docker compose down` |

> ⚠️ **Warning**: Never run `docker compose down -v`. The `-v` flag deletes your persistent Docker volume where workflows and credentials are saved.

---

## 📊 Node Details & Column Expressions

The **Google Sheets** node uses manual mapping to extract data across nodes:

| Column Header | n8n Expression | Description |
| :--- | :--- | :--- |
| **Title** | `={{ $('RSS Read').item.json.title }}` | Original article title |
| **Link** | `={{ $('RSS Read').item.json.link }}` | Canonical link to article |
| **Published** | `={{ $('RSS Read').item.json.pubDate }}` | Article publication timestamp |
| **Summary** | `={{ $json.text }}` | 2-sentence summary produced by the LLM Chain |

---

## 🤖 AI Model Configuration

- **Provider**: OpenRouter
- **Model**: `openrouter/free` (or any free tier like `meta-llama/llama-3.2-3b-instruct:free`)
- **Prompt**:
  ```text
  =Summarize this article in 2 plain-English sentences.
  Title: {{ $json.title }}
  Content: {{ $json.contentSnippet }}
  ```

---

## 📄 License

This project is licensed under the [MIT License](LICENSE).
