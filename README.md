# 🌍 AI Trip Planner — Agentic Travel Planning Application

An AI-powered travel planning agent that generates comprehensive, real-time trip itineraries using a LangGraph ReAct agent architecture. Users interact through a Streamlit chat interface backed by a FastAPI server, and the agent autonomously calls tools to fetch live data on attractions, restaurants, weather, transportation, expenses, and currency conversion.

---

## Architecture Overview

```
┌─────────────────┐       HTTP POST /query       ┌──────────────────────┐
│  Streamlit UI   │ ─────────────────────────────▶│   FastAPI Backend    │
│  streamlit_app  │                               │      main.py         │
└─────────────────┘                               └────────┬─────────────┘
                                                           │
                                                           ▼
                                                  ┌──────────────────────┐
                                                  │   LangGraph Agent    │
                                                  │  (ReAct / StateGraph)│
                                                  └────────┬─────────────┘
                                                           │ bind_tools
                              ┌────────────────────────────┼────────────────────────────┐
                              ▼                            ▼                            ▼
                   ┌─────────────────┐        ┌─────────────────┐          ┌─────────────────────┐
                   │  Place Search   │        │  Weather Tool   │          │  Calculator Tools   │
                   │ Google Places + │        │ OpenWeatherMap  │          │  + Currency Conv.   │
                   │ Tavily fallback │        │                 │          │  (ExchangeRate API) │
                   └─────────────────┘        └─────────────────┘          └─────────────────────┘
```

---

## Features

- **Dual itinerary generation** — produces both a mainstream tourist plan and an off-beat alternative for every destination
- **Real-time place search** — Google Places API with automatic Tavily web search fallback
- **Live weather data** — current conditions and multi-day forecast via OpenWeatherMap
- **Expense planning** — hotel cost estimation, total trip cost, and daily budget breakdown
- **Currency conversion** — real-time exchange rates for any currency pair
- **Markdown export** — travel plans saved as formatted `.md` files with timestamps
- **Switchable LLM backend** — supports OpenAI (GPT) and Groq (DeepSeek) via config

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Streamlit |
| Backend API | FastAPI + Uvicorn |
| Agent Framework | LangGraph (ReAct / StateGraph) |
| LLM Providers | OpenAI GPT, Groq (DeepSeek) |
| LLM Integration | LangChain, LangChain-OpenAI, LangChain-Groq |
| Place Search | Google Places API (`langchain-google-community[places]`) |
| Web Search Fallback | Tavily (`langchain-tavily`) |
| Weather | OpenWeatherMap REST API |
| Currency | ExchangeRate API |
| Config | PyYAML |
| Environment | python-dotenv |

---

## Project Structure

```
AI_Trip_Planner/
├── agent/
│   └── agentic_workflow.py      # LangGraph StateGraph builder — wires LLM + tools
├── tools/                       # LangChain @tool wrappers (exposed to the agent)
│   ├── place_search_tool.py     # Attractions, restaurants, activities, transportation
│   ├── weather_info_tool.py     # Current weather + forecast
│   ├── expense_calculator_tool.py  # Hotel cost, total & daily budget
│   └── currency_conversion_tool.py # Currency conversion
├── utils/                       # Core service logic (API calls, calculations)
│   ├── place_info_search.py     # Google Places + Tavily search implementations
│   ├── weather_info.py          # OpenWeatherMap API client
│   ├── currency_converter.py    # ExchangeRate API client
│   ├── expense_calculator.py    # Pure math helpers
│   ├── model_loader.py          # LLM loader (OpenAI / Groq)
│   ├── config_loader.py         # YAML config reader
│   └── save_to_document.py      # Markdown file export
├── prompt_library/
│   └── prompt.py                # System prompt for the travel agent
├── config/
│   └── config.yaml              # LLM provider and model configuration
├── streamlit_app.py             # Streamlit chat UI
├── main.py                      # FastAPI server + /query endpoint
├── setup.py                     # Package setup
└── requirements.txt
```

---

## Getting Started

### Prerequisites

- Python 3.10+
- API keys for the services listed below

### 1. Clone the repository

```bash
git clone https://github.com/your-username/AI_Trip_Planner.git
cd AI_Trip_Planner
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure environment variables

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_api_key
GROQ_API_KEY=your_groq_api_key
GPLACES_API_KEY=your_google_places_api_key
TAVILY_API_KEY=your_tavily_api_key
OPENWEATHERMAP_API_KEY=your_openweathermap_api_key
EXCHANGE_RATE_API_KEY=your_exchangerate_api_key
LANGCHAIN_API_KEY=your_langchain_api_key        # optional, for LangSmith tracing
```

### 4. Configure the LLM provider

Edit `config/config.yaml` to set your preferred model:

```yaml
llm:
  openai:
    provider: "openai"
    model_name: "gpt-4o-mini"
  groq:
    provider: "groq"
    model_name: "deepseek-r1-distill-llama-70b"
```

### 5. Run the application

Start the FastAPI backend:

```bash
uvicorn main:app --reload
```

In a separate terminal, start the Streamlit frontend:

```bash
streamlit run streamlit_app.py
```

Open your browser at `http://localhost:8501` and start planning your trip.

---

## How It Works

1. The user submits a travel query (e.g. *"Plan a 5-day trip to Kyoto in October"*) via the Streamlit UI.
2. The UI sends a `POST /query` request to the FastAPI backend.
3. The backend initialises a **LangGraph ReAct agent** with all tools bound to the LLM.
4. The agent autonomously decides which tools to call — fetching places, weather, calculating costs, converting currencies — iterating until it has all the information it needs.
5. The final response is a fully formatted Markdown travel plan returned to the UI and optionally saved as a `.md` file.

---

## API Keys Required

| Service | Purpose | Get Key |
|---|---|---|
| OpenAI | Primary LLM | [platform.openai.com](https://platform.openai.com) |
| Groq | Alternative LLM (faster inference) | [console.groq.com](https://console.groq.com) |
| Google Places | Attractions, restaurants, activities | [Google Cloud Console](https://console.cloud.google.com) |
| Tavily | Web search fallback for place data | [tavily.com](https://tavily.com) |
| OpenWeatherMap | Weather data | [openweathermap.org](https://openweathermap.org/api) |
| ExchangeRate API | Currency conversion | [exchangerate-api.com](https://exchangerate-api.com) |
