# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TradingAgents-CN is a Chinese-enhanced multi-agent AI stock analysis platform. It has a **dual architecture**:

1. **Legacy Architecture** (Original Streamlit-based): `tradingagents/` directory with CLI entry point `main.py`
2. **Modern Architecture** (v1.0.0-preview): FastAPI backend (`app/`) + Vue 3 frontend (`frontend/`)

The modern architecture is the current focus for development.

## Development Commands

### Backend (FastAPI)
```bash
cd app
python -m uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### Frontend (Vue 3)
```bash
cd frontend
npm install
npm run dev          # Development server (http://localhost:5173)
npm run build        # Production build
npm run lint         # ESLint with auto-fix
npm run type-check   # TypeScript type checking
```

### Testing
```bash
# Run all tests (excludes integration tests by default)
pytest

# Run specific test file
pytest tests/test_analysis.py

# Run integration tests
pytest -m integration

# Run with verbose output
pytest -v

# Run tests from a specific directory
pytest tests/config/
```

### Docker Deployment
```bash
# Full stack (backend + frontend + MongoDB + Redis)
docker-compose up -d

# With management UIs (Mongo Express + Redis Commander)
docker-compose --profile management up -d

# View logs
docker-compose logs -f backend
docker-compose logs -f frontend
```

### CLI Tools
```bash
# Legacy CLI (Streamlit-based)
python main.py

# Data synchronization scripts (check cli/ directory)
# Tushare, AkShare, BaoStock initialization scripts available
```

## Architecture

### Modern Stack (v1.0.0-preview)

**Backend (`app/`)**
- FastAPI + Uvicorn (RESTful API + WebSocket)
- MongoDB (primary data storage via Motor/pymongo)
- Redis (caching and session management)
- JWT + bcrypt (authentication)
- APScheduler (task scheduling)
- SSE + WebSocket (real-time notifications)

**Frontend (`frontend/`)**
- Vue 3 + Vite
- Element Plus (UI components)
- Pinia (state management)
- Vue Router 4
- ECharts + vue-echarts (charts)
- TypeScript
- Axios (HTTP client)

**Data Sources**
- Chinese markets: AkShare, BaoStock, Tushare
- International: Yahoo Finance, Finnhub, EODHD
- News: Reddit (praw), feedparser

**LLM Providers**
- OpenAI (native support)
- Google AI (Gemini)
- Alibaba Dashscope
- DeepSeek
- Baidu Qianfan
- Anthropic Claude
- Framework: LangChain + LangGraph
- Memory: ChromaDB

### Legacy Architecture (`tradingagents/`)

**Core Agents** (in `tradingagents/agents/`):
- **Analysts**: Market Analyst, Fundamentals Analyst, China Market Analyst, News Analyst, Social Media Analyst
- **Managers**: Research Manager, Risk Manager
- **Researchers**: Bull Researcher, Bear Researcher
- **Risk Management**: Aggressive/Conservative/Neutral Debaters
- **Trader**: Main trading agent

**Data Flow**: Data Sources → Dataflows → Cache → Agents → Analysis → Reports

### Key Directories

- `app/` - FastAPI backend (routers, services, models, schemas)
- `frontend/` - Vue 3 SPA
- `tradingagents/` - Legacy multi-agent framework
- `cli/` - CLI tools and data initialization scripts
- `config/` - Logging and system configuration
- `tests/` - Comprehensive test suite (200+ files)
- `docs/` - Documentation

## Environment Configuration

**Required environment variables** (see `.env.example`):
- Database: `TRADINGAGENTS_MONGODB_URL`, `TRADINGAGENTS_REDIS_URL`
- API: `API_HOST`, `API_PORT`, `CORS_ORIGINS`
- LLM providers: `OPENAI_API_KEY`, `GOOGLE_API_KEY`, etc.
- Data sources: `TUSHARE_TOKEN`, `AKSHARE_*`, etc.
- Docker: Set `DOCKER_CONTAINER=true` in containerized environments

## Important Notes

1. **Data Synchronization**: Before analyzing stocks, run data sync scripts from `cli/` to populate local cache with stock data. Without this, analysis results will have data errors.

2. **Dual Codebase**: When making changes, clarify whether you're working on the legacy `tradingagents/` codebase or the modern `app/` backend.

3. **Frontend-Backend Communication**: The frontend (`frontend/`) communicates with the FastAPI backend at `http://localhost:8000` by default. CORS is configured for ports 3000, 8080, and 5173.

4. **Logging Configuration**: Use `config/logging.toml` (default) or `config/logging_docker.toml` (Docker). The logging system uses `concurrent-log-handler` for Windows compatibility.

5. **License Model**: Mixed licensing - Apache 2.0 for core code, commercial license for `app/` and `frontend/` directories. Personal learning/research use is free for all components.

6. **Test Organization**: Tests are organized by category in `tests/`:
   - `tests/config/` - Configuration tests
   - `tests/integration/` - Integration tests (marked with `@pytest.mark.integration`)
   - `tests/services/` - Service layer tests
   - `tests/middleware/` - Middleware tests
   - Integration tests are excluded by default; use `-m integration` to run them

## Common Issues

- **Chinese Data**: For Chinese stock analysis, ensure Tushare/AkShare/BaoStock tokens are configured and data is synced
- **Model Selection**: The system supports smart model selection - check configuration in both `tradingagents/config/` and `app/` settings
- **Windows Development**: The project uses `concurrent-log-handler` for Windows-friendly log rotation
- **MongoDB/Redis**: In Docker, these services are pre-configured with healthchecks. For local development, ensure they're running before starting the backend
