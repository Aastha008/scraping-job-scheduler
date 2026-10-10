# Scraping Job Scheduler

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/downloads/)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Code Style: Black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Type Checking: Pydantic](https://img.shields.io/badge/schema-pydantic-red.svg)](https://docs.pydantic.dev/)

An enterprise-grade, automated system for orchestrating, executing, and monitoring periodic web scraping workflows. Scraping Job Scheduler combines flexible selector-based DOM extraction, strict payload type enforcement, exponential backoff retries, and real-time observability in a single lightweight framework.

---

## Problem Statement & Target Audience

### The Problem
Engineering and growth teams frequently rely on fragile, ad-hoc scraping scripts. These one-off scripts lack central orchestration, automated retry policies, output schema validation, and structured output routing. As target websites change or experience transient downtime, these scripts fail silently or emit malformed records, causing downstream data corruption and excessive maintenance overhead.

### Target Audience
* **Data Engineering Teams** requiring predictable, resilient, and schema-enforced web data ingestion.
* **Growth & Market Intelligence Analysts** needing automated periodic competitive intelligence gathering without manual script execution.
* **Backend Engineers** seeking a modular, extensible scraping foundation that integrates directly into event-driven or API-first microservice architectures.

---

## Key Features

* **Declarative Pipeline Configuration**: Define job specs, target URLs, dynamic headers, CSS/XPath selector mappings, and cron schedules in clean YAML or JSON models.
* **Robust Execution Engine**: Powered by APScheduler with support for standard cron syntax, concurrent execution limits, dynamic thread management, and exponential backoff retry strategies.
* **Strict Payload Validation**: Integrated Pydantic domain models sanitize, validate, and type-cast DOM extractions, discarding or quarantining invalid records before payload export.
* **Interactive Terminal UI**: Rich-powered terminal dashboard providing real-time visibility into active job states, live execution logs, and live selector preview utilities.
* **Extensible Data Exporters**: Plug-and-play export subsystem for dispatching validated payloads to downstream webhooks, file systems, or database targets.
* **HTTP API & CLI Access**: Expose scheduler controls via a RESTful HTTP interface or operate locally via an intuitive command-line entry point.

---

## Architecture & System Design

The architecture is built around decoupled modules that separate scheduler management, request handling, schema validation, and output delivery.

```
                  +--------------------------+
                  | Pipeline Config (YAML)   |
                  +------------+-------------+
                               |
                               v
                     +-------------------+
                     |    src/core.py    | <--- Pipeline Parsing & Core Orchestration
                     +---------+---------+
                               |
      +------------------------+------------------------+
      |                        |                        |
      v                        v                        v
+-----------+           +--------------+         +--------------+
| src/api.py|           |  APScheduler |         | src/models.py|
| (REST API)|           |  (Cron Exec) |         |  (Pydantic)  |
+-----+-----+           +-------+------+         +------+-------+
      |                         |                       ^
      |                         v                       |
      |                 +---------------+               |
      +---------------->| Scraping Engine|---------------+
                        | (HTTPX / BS4) |  (Validates Payload)
                        +-------+-------+
                                |
                                v
                        +---------------+
                        | Data Exporter |
                        |  (Webhooks)   |
                        +---------------+
```

### Core Architectural Modules

| Module | Purpose |
| :--- | :--- |
| `src/models.py` | Pydantic domain models for pipeline configs, extraction rules, schema payloads, and job execution states. |
| `src/core.py` | Declarative configuration parser, extraction engine (HTTPX/BeautifulSoup), retry executor, and APScheduler lifecycle manager. |
| `src/api.py` | FastAPI/REST API endpoints allowing programmatic pipeline registration, job trigger, and status checks. |
| `src/main.py` | Unified CLI entry point initializing the scheduler process, API server, and Rich interactive TUI session. |

---

## Technology Stack

| Category | Technology | Usage |
| :--- | :--- | :--- |
| **Language** | Python 3.10+ | Core runtime environment |
| **Parsing & Scraping** | BeautifulSoup4, HTTPX | Async HTTP request dispatching and HTML DOM parsing |
| **Scheduling** | APScheduler | Cron-based job scheduling and execution queue management |
| **Validation** | Pydantic v2 | Data contract definitions, type validation, and serialization |
| **CLI & Terminal UI** | Rich, Typer / Argparse | Terminal visualization, live log streams, and interactive menus |
| **Testing** | pytest, pytest-asyncio | Unit testing, validation suite, and mock server assertions |

---

## Installation & Virtual Environment Setup

### Prerequisites
* Python 3.10 or higher
* `pip` and `venv` modules

### Quickstart Setup

1. **Clone the Repository**
   ```bash
   git clone https://github.com/daily-project-factory/scraping-job-scheduler.git
   cd scraping-job-scheduler
   ```

2. **Create and Activate a Virtual Environment**
   ```bash
   # On macOS/Linux:
   python3 -m venv venv
   source venv/bin/activate

   # On Windows:
   python -m venv venv
   .\venv\Scripts\activate
   ```

3. **Install Dependencies**
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

---

## Environment Variables Configuration

Copy the example environment configuration file to create your local `.env`:

```bash
cp .env.example .env
```

### Configurable Variables

| Variable | Default | Description |
| :--- | :--- | :--- |
| `APP_ENV` | `development` | Deployment stage (`development`, `production`, `test`) |
| `LOG_LEVEL` | `INFO` | Logging verbosity (`DEBUG`, `INFO`, `WARNING`, `ERROR`) |
| `MAX_CONCURRENT_JOBS` | `10` | Maximum parallel scraping jobs allowed in APScheduler |
| `DEFAULT_RETRY_LIMIT` | `3` | Default retries for transient HTTP/parsing failures |
| `DEFAULT_BACKOFF_FACTOR` | `2.0` | Multiplier for exponential delay between job retries |
| `EXPORT_WEBHOOK_URL` | `http://localhost:8000/webhook` | Default endpoint target for payload extraction delivery |
| `API_HOST` | `0.0.0.0` | Host interface binding for the API service |
| `API_PORT` | `8080` | Port binding for the API service |

---

## Usage Guide

### Declarative Pipeline Definition

Pipelines are declared in JSON/YAML structure matching the `PipelineConfig` Pydantic model:

```json
{
  "id": "job_tech_news_01",
  "name": "Tech News Headlines Extractor",
  "cron_expression": "*/15 * * * *",
  "target_url": "https://news.ycombinator.com",
  "selectors": [
    {
      "field_name": "title",
      "selector": ".titleline > a",
      "attribute": "text",
      "required": true
    },
    {
      "field_name": "link",
      "selector": ".titleline > a",
      "attribute": "href",
      "required": true
    }
  ],
  "export_target": "https://api.myanalytics.com/v1/ingest"
}
```

### CLI Commands

The CLI allows running background scheduler daemons, executing individual pipelines directly, or rendering the terminal dashboard.

```bash
# Start the scheduler backend daemon with interactive Rich TUI
python -m src.main daemon --tui

# Manually trigger a single scraping execution pipeline dry-run
python -m src.main execute --config examples/news_pipeline.json

# Launch the embedded API REST Server
python -m src.main serve --host 127.0.0.1 --port 8080
```

### API Endpoints

When running the API service via `src/api.py`, the following REST endpoints are accessible:

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/health` | Healthcheck and scheduler status telemetry |
| `GET` | `/jobs` | List all registered job configurations and execution history |
| `POST` | `/jobs` | Register a new pipeline definition dynamically |
| `POST` | `/jobs/{id}/run` | Immediately trigger an ad-hoc run for job `{id}` |
| `DELETE` | `/jobs/{id}` | Deregister and remove a job schedule |

---

## Automated Testing Instructions

The repository includes test suites covering core parsing algorithms, retry mechanisms, model validations, and REST API routes.

### Executing Tests via `pytest`

Run the full automated test suite:
```bash
pytest
```

Run tests with detailed output and code coverage reporting:
```bash
pytest -v --cov=src tests/
```

Run specific test files:
```bash
# Run core parsing and scraping logic tests
pytest tests/test_core.py

# Run API endpoint interface tests
pytest tests/test_api.py
```

---

## Project Structure

```text
scraping-job-scheduler/
├── .env.example
├── .gitignore
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── api.py
│   ├── core.py
│   ├── main.py
│   └── models.py
└── tests/
    ├── __init__.py
    ├── test_api.py
    └── test_core.py
```

---

## Roadmap / Future Enhancements

- [ ] **Headless Browser Integration**: Add optional Playwright/Puppeteer support for JS-heavy dynamic single-page applications.
- [ ] **Distributed Execution Backend**: Introduce a Redis/Celery queue driver for horizontal scaling across worker nodes.
- [ ] **S3 / Blob Storage Exporters**: Native integration to stream large extraction outputs directly to AWS S3, Google Cloud Storage, or Azure Blob Storage.
- [ ] **Auto-healing Selector Heuristics**: Leverage lightweight local LLMs/heuristics to re-detect changed HTML DOM paths when selectors fail.

---

## License

Distributed under the MIT License. See `LICENSE` for details.