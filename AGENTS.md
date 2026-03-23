# TradingAgents: AI Coding Agent Guide

## Project Overview

**TradingAgents** is a multi-agent LLM financial trading framework that simulates real-world trading firm dynamics. It deploys specialized AI agents (fundamental analysts, sentiment experts, technical analysts, traders, and risk management teams) that collaboratively evaluate market conditions and make trading decisions through dynamic discussions.

This is a Python-based research framework built with LangGraph for orchestrating agent workflows. It supports multiple LLM providers (OpenAI, Google, Anthropic, xAI, OpenRouter, Ollama) and multiple data vendors (yfinance, Alpha Vantage).

- **Version**: 0.2.2
- **Repository**: https://github.com/TauricResearch/TradingAgents
- **Paper**: arXiv:2412.20138

## Technology Stack

### Core Dependencies
- **LangGraph** (>=0.4.8): Agent workflow orchestration
- **LangChain**: LLM abstractions and tool integrations
- **LangChain-OpenAI/Anthropic/Google**: Provider-specific integrations

### Data Sources
- **yfinance**: Free Yahoo Finance data (default, no API key required)
- **Alpha Vantage**: Premium financial data (requires API key)

### CLI & UI
- **Typer**: CLI framework
- **Rich**: Terminal UI and formatted output
- **Questionary**: Interactive prompts

### Data Processing
- **pandas** (>=2.3.0): Data manipulation
- **stockstats** (>=0.6.5): Technical indicators
- **backtrader** (>=1.9.78.123): Backtesting framework

### Infrastructure
- **Redis** (>=6.2.0): Caching and state management
- **uv.lock**: Reproducible dependency lock file

## Project Structure

```
TradingAgents/
├── tradingagents/              # Main package
│   ├── agents/                 # Agent implementations
│   │   ├── analysts/           # Analyst agents (market, news, fundamentals, social)
│   │   ├── researchers/        # Bull/Bear researchers
│   │   ├── risk_mgmt/          # Risk management debators
│   │   ├── managers/           # Portfolio & research managers
│   │   ├── trader/             # Trader agent
│   │   └── utils/              # Agent utilities, tools, memory, states
│   ├── dataflows/              # Data source integrations
│   │   ├── y_finance.py        # Yahoo Finance implementation
│   │   ├── alpha_vantage*.py   # Alpha Vantage implementations
│   │   └── interface.py        # Unified data interface
│   ├── graph/                  # LangGraph workflow
│   │   ├── trading_graph.py    # Main graph orchestrator
│   │   ├── setup.py            # Graph node and edge setup
│   │   ├── conditional_logic.py # Routing logic
│   │   ├── propagation.py      # State propagation
│   │   ├── reflection.py       # Self-reflection capabilities
│   │   └── signal_processing.py # Signal extraction
│   ├── llm_clients/            # LLM provider abstractions
│   │   ├── factory.py          # Client factory
│   │   ├── base_client.py      # Abstract base
│   │   ├── openai_client.py    # OpenAI/OpenRouter/Ollama
│   │   ├── anthropic_client.py # Claude
│   │   └── google_client.py    # Gemini
│   ├── default_config.py       # Default configuration
│   └── __init__.py
├── cli/                        # Command-line interface
│   ├── main.py                 # CLI entry point
│   ├── utils.py                # CLI utilities
│   ├── models.py               # CLI data models
│   ├── config.py               # CLI configuration
│   ├── stats_handler.py        # LLM/tool usage tracking
│   ├── announcements.py        # Remote announcements
│   └── static/                 # Static assets (welcome.txt)
├── tests/                      # Test suite
│   └── test_ticker_symbol_handling.py
├── main.py                     # Example usage script
├── pyproject.toml              # Package configuration
├── uv.lock                     # Dependency lock file
└── .env.example                # Environment template
```

## Agent Architecture

The framework implements a 5-stage hierarchical workflow:

### Stage I: Analyst Team
Specialized analysts gather data and generate reports:
- **Market Analyst**: Technical analysis using indicators (MACD, RSI, etc.)
- **Social Media Analyst**: Sentiment analysis from social data
- **News Analyst**: Global news and macroeconomic impact assessment
- **Fundamentals Analyst**: Company financials and intrinsic value evaluation

### Stage II: Research Team
Debate-based research evaluation:
- **Bull Researcher**: Argues for investment opportunity
- **Bear Researcher**: Argues against investment, identifies risks
- **Research Manager**: Synthesizes debate and produces investment plan

### Stage III: Trader Agent
- Composes analyst and researcher reports
- Determines timing and magnitude of trades
- Produces trading plan

### Stage IV: Risk Management Team
Multi-perspective risk assessment:
- **Aggressive Analyst**: Growth-focused risk assessment
- **Conservative Analyst**: Safety-focused risk assessment
- **Neutral Analyst**: Balanced risk assessment

### Stage V: Portfolio Manager
- Final decision authority
- Approves/rejects transaction proposals
- Executes orders through simulated exchange

## Configuration System

### Default Configuration (`tradingagents/default_config.py`)

```python
DEFAULT_CONFIG = {
    "project_dir": "...",
    "results_dir": "./results",
    "data_cache_dir": ".../dataflows/data_cache",
    "llm_provider": "openai",
    "deep_think_llm": "gpt-5.2",
    "quick_think_llm": "gpt-5-mini",
    "backend_url": "https://api.openai.com/v1",
    "google_thinking_level": None,      # "high", "minimal"
    "openai_reasoning_effort": None,    # "medium", "high", "low"
    "anthropic_effort": None,           # "high", "medium", "low"
    "max_debate_rounds": 1,
    "max_risk_discuss_rounds": 1,
    "max_recur_limit": 100,
    "data_vendors": {
        "core_stock_apis": "yfinance",       # Options: alpha_vantage, yfinance
        "technical_indicators": "yfinance",
        "fundamental_data": "yfinance",
        "news_data": "yfinance",
    },
    "tool_vendors": {},  # Tool-level overrides
}
```

### Environment Variables

Copy `.env.example` to `.env` and configure:

```bash
# LLM Providers (set at least one)
OPENAI_API_KEY=...
GOOGLE_API_KEY=...
ANTHROPIC_API_KEY=...
XAI_API_KEY=...
OPENROUTER_API_KEY=...

# Data Provider
ALPHA_VANTAGE_API_KEY=...

# Optional
TRADINGAGENTS_RESULTS_DIR=./results
```

## Build and Installation

### Using pip
```bash
# Clone repository
git clone https://github.com/TauricResearch/TradingAgents.git
cd TradingAgents

# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install package
pip install .
```

### Using uv (recommended for development)
```bash
# Install dependencies from lock file
uv sync

# Run in virtual environment
uv run tradingagents
```

## Running the Application

### CLI Mode (Interactive)
```bash
# Installed command
tradingagents

# Or run directly
python -m cli.main
```

The CLI will prompt for:
1. Ticker symbol (e.g., SPY, CNC.TO, 7203.T)
2. Analysis date (YYYY-MM-DD)
3. Analyst team selection (market, social, news, fundamentals)
4. Research depth (1, 3, or 5 rounds)
5. LLM provider and models
6. Provider-specific thinking configuration

### Programmatic Usage
```python
from tradingagents.graph.trading_graph import TradingAgentsGraph
from tradingagents.default_config import DEFAULT_CONFIG

config = DEFAULT_CONFIG.copy()
config["llm_provider"] = "openai"
config["deep_think_llm"] = "gpt-5.2"
config["quick_think_llm"] = "gpt-5-mini"
config["max_debate_rounds"] = 2

# Configure data vendors
config["data_vendors"] = {
    "core_stock_apis": "yfinance",
    "technical_indicators": "yfinance",
    "fundamental_data": "yfinance",
    "news_data": "yfinance",
}

ta = TradingAgentsGraph(debug=True, config=config)
state, decision = ta.propagate("NVDA", "2026-01-15")
print(decision)
```

## Testing

### Running Tests
```bash
# Run all tests
python -m unittest discover tests/

# Run specific test file
python -m unittest tests.test_ticker_symbol_handling

# Run with pytest (if installed)
pytest tests/
```

### Test Coverage
Current test suite is minimal and focuses on:
- Ticker symbol normalization (exchange suffix preservation)
- Instrument context building

**TODO**: Expand test coverage for:
- Agent state transitions
- Graph workflow execution
- LLM client integrations
- Data flow interfaces

## Code Style Guidelines

### Python Style
- **Formatter**: Use consistent 4-space indentation
- **Line Length**: 100 characters (inferred from codebase)
- **Imports**: Grouped by standard library, third-party, local
- **Type Hints**: Used for function signatures and key data structures
- **Docstrings**: Google-style docstrings for classes and methods

### Naming Conventions
- **Classes**: PascalCase (`TradingAgentsGraph`, `AgentState`)
- **Functions/Methods**: snake_case (`create_market_analyst`, `propagate`)
- **Constants**: UPPER_CASE (`DEFAULT_CONFIG`, `ANALYST_ORDER`)
- **Private**: Leading underscore for internal methods (`_log_state`)

### Module Organization
- Each agent type in its own module
- Utility functions grouped by purpose (`agent_utils.py`, `memory.py`)
- Clear separation between data layer (`dataflows/`), agent layer (`agents/`), and orchestration layer (`graph/`)

## Key Design Patterns

### 1. Agent State Management
Uses LangGraph's `MessagesState` with typed annotations:
```python
class AgentState(MessagesState):
    company_of_interest: Annotated[str, "Company ticker"]
    trade_date: Annotated[str, "Trading date"]
    market_report: Annotated[str, "Market analysis report"]
    # ... additional state fields
```

### 2. LLM Client Factory Pattern
```python
from tradingagents.llm_clients import create_llm_client

client = create_llm_client(
    provider="openai",  # or "anthropic", "google", etc.
    model="gpt-5.2",
    base_url="https://api.openai.com/v1"
)
llm = client.get_llm()
```

### 3. Data Vendor Abstraction
Tool-level and category-level vendor configuration:
```python
# Category-level (applies to all tools in category)
config["data_vendors"]["core_stock_apis"] = "yfinance"

# Tool-level override (takes precedence)
config["tool_vendors"]["get_stock_data"] = "alpha_vantage"
```

### 4. Graph Node Creation
Each agent is a factory function returning a callable node:
```python
def create_market_analyst(llm):
    def market_analyst(state: AgentState):
        # Agent logic
        return state
    return market_analyst
```

## Security Considerations

### API Key Management
- **NEVER** commit API keys to version control
- Use `.env` file (already in `.gitignore`)
- For production, use secret management services

### Data Handling
- Financial data is cached locally in `dataflows/data_cache/`
- Cache directory is in `.gitignore`
- No sensitive user data is transmitted to third parties beyond API calls

### LLM Provider Security
- All API calls use HTTPS
- API keys are passed via headers, not URL parameters
- Supports custom base URLs for self-hosted/proxy setups (Ollama, OpenRouter)

## Development Workflow

### Adding a New Agent
1. Create agent module in appropriate subdirectory (`analysts/`, `researchers/`, etc.)
2. Implement factory function that returns node callable
3. Add to `tradingagents/agents/__init__.py`
4. Register in `GraphSetup.setup_graph()` in `graph/setup.py`
5. Add conditional edges if needed in `conditional_logic.py`

### Adding a New Data Source
1. Implement data fetcher in `dataflows/`
2. Add vendor identifier to `dataflows/config.py` if needed
3. Register tools in `tradingagents/agents/utils/agent_utils.py`
4. Update `trading_graph.py` `_create_tool_nodes()` if adding new tool categories

### Adding LLM Provider Support
1. Create client class inheriting from `BaseLLMClient`
2. Add to `llm_clients/factory.py`
3. Update CLI model selections in `cli/utils.py`

## Output and Results

### CLI Output Structure
Results are saved to `./results/{ticker}/{date}/`:
```
results/
└── SPY/
    └── 2026-01-15/
        ├── message_tool.log       # Execution trace
        ├── reports/               # Individual report sections
        │   ├── market_report.md
        │   ├── sentiment_report.md
        │   ├── news_report.md
        │   ├── fundamentals_report.md
        │   ├── investment_plan.md
        │   ├── trader_investment_plan.md
        │   └── final_trade_decision.md
        └── complete_report.md     # Consolidated report
```

### Programmatic Access
```python
# Access state after propagation
state, decision = ta.propagate("SPY", "2026-01-15")
market_report = state["market_report"]
investment_plan = state["investment_debate_state"]["judge_decision"]
final_decision = state["final_trade_decision"]

# Reflection for learning from results
ta.reflect_and_remember(returns_losses=1000)
```

## Common Issues and Solutions

### Issue: API Rate Limits
**Solution**: Configure multiple data vendors or increase cache usage:
```python
config["data_vendors"]["core_stock_apis"] = "yfinance"  # Free tier
```

### Issue: Out of Memory with Large Debates
**Solution**: Reduce debate rounds:
```python
config["max_debate_rounds"] = 1
config["max_risk_discuss_rounds"] = 1
```

### Issue: Model Not Available
**Solution**: Check provider-specific model names in `cli/utils.py` and use exact identifier.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes following code style guidelines
4. Add tests for new functionality
5. Submit a pull request

For questions or discussions, join the community:
- Discord: https://discord.com/invite/hk9PGKShPK
- GitHub: https://github.com/TauricResearch
