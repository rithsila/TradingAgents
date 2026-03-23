# XAUUSD/Gold Forex Support Implementation Plan

## Executive Summary

This document outlines a comprehensive plan to add **XAUUSD (Gold/USD)** and broader **Forex/Commodities** support to the TradingAgents framework. The implementation will enable users to analyze gold spot prices and other currency pairs alongside traditional equities.

---

## Table of Contents

1. [Current State Analysis](#current-state-analysis)
2. [Target State Vision](#target-state-vision)
3. [Implementation Phases](#implementation-phases)
4. [Technical Architecture](#technical-architecture)
5. [Data Source Strategy](#data-source-strategy)
6. [Agent Adaptations](#agent-adaptations)
7. [Configuration Changes](#configuration-changes)
8. [Testing Strategy](#testing-strategy)
9. [Risks & Mitigation](#risks--mitigation)

---

## Current State Analysis

### What's Supported Now

| Aspect | Current Support |
|--------|----------------|
| **Instruments** | Stocks, ETFs (equity-only) |
| **Data Sources** | yfinance, Alpha Vantage (equity endpoints) |
| **Analyst Types** | Market, Social, News, Fundamentals |
| **Fundamental Data** | P/E ratio, EPS, Balance Sheets, Income Statements |
| **Technical Indicators** | SMA, EMA, MACD, RSI, Bollinger Bands, ATR |
| **Symbol Format** | Ticker + exchange suffix (e.g., `SPY`, `AAPL`, `CNC.TO`) |

### Limitations for XAUUSD

| Issue | Impact |
|-------|--------|
| Equity-only API endpoints | Cannot fetch forex/commodity OHLCV data |
| Fundamental analysts expect company data | No P/E, EPS, balance sheets for gold |
| Symbol format assumptions | XAUUSD uses different conventions (forex pairs) |
| News filtering | May not properly filter for commodity-specific news |
| `insider_transactions` tool | Irrelevant for forex/commodities |

---

## Target State Vision

### Supported Instruments

| Instrument Type | Example Symbols | Use Case |
|-----------------|-----------------|----------|
| **Forex Spot** | `XAUUSD`, `EURUSD`, `GBPUSD` | Currency pair trading |
| **Gold ETF** | `GLD`, `IAU`, `GLDM` | Equity-based gold exposure |
| **Gold Futures** | `GC=F` (via yfinance) | Futures trading |
| **Crypto** | `BTCUSD` (future extensibility) | Cryptocurrency trading |

### Supported Analysts per Instrument Type

```
+-----------------------------------------------------------------------------+
|                    INSTRUMENT TYPE → ANALYST MAPPING                        |
+------------------+----------------------------------------------------------+
| Instrument Type  | Supported Analysts                                       |
+------------------+----------------------------------------------------------+
| Stock/ETF        | market, social, news, fundamentals (all 4)               |
| Forex (XAUUSD)   | market, social, news (fundamentals disabled)             |
| Gold ETF (GLD)   | market, social, news, fundamentals (all 4)               |
| Gold Futures     | market, social, news (fundamentals disabled)             |
+------------------+----------------------------------------------------------+
```

---

## Implementation Phases

### Phase 1: Foundation (Week 1-2)
**Goal**: Enable basic XAUUSD price data fetching

#### Tasks

1. **Create Instrument Type Detection System**
   ```python
   # tradingagents/dataflows/instrument_utils.py
   
   class InstrumentType(Enum):
       STOCK = "stock"
       ETF = "etf"
       FOREX = "forex"
       COMMODITY = "commodity"
       FUTURES = "futures"
   
   def detect_instrument_type(symbol: str) -> InstrumentType:
       """Detect instrument type from symbol format."""
       # XAUUSD, EURUSD, GBPUSD -> FOREX
       # GC=F, SI=F -> FUTURES
       # GLD, IAU -> ETF (detect via API)
       # Default -> STOCK
   ```

2. **Add Forex Data Fetcher (Alpha Vantage)**
   - Create `tradingagents/dataflows/alpha_vantage_forex.py`
   - Implement `get_forex_daily()` using `FX_DAILY` endpoint
   - Implement `get_forex_intraday()` using `FX_INTRADAY` endpoint
   - Support XAUUSD specifically (from_symbol=XAU, to_symbol=USD)

3. **Add Gold Futures Support (yfinance)**
   - Extend `y_finance.py` to handle `GC=F` (Gold Futures)
   - Support `SI=F` (Silver Futures) for future extensibility

4. **Update Data Interface**
   - Add `forex_data` category to `TOOLS_CATEGORIES`
   - Add vendor mappings for forex data

#### Deliverables
- [ ] `instrument_utils.py` with detection logic
- [ ] `alpha_vantage_forex.py` with FX endpoints
- [ ] Updated `interface.py` with forex routing
- [ ] Unit tests for instrument detection

---

### Phase 2: Technical Analysis Support (Week 2-3)
**Goal**: Enable technical indicators for XAUUSD

#### Tasks

1. **Extend Technical Indicators for Forex**
   - Alpha Vantage supports forex technical indicators (use `symbol=XAUUSD` format)
   - yfinance uses `stockstats` which works with any OHLCV data

2. **Update Market Analyst**
   - Ensure market analyst can work with forex data
   - No changes needed if data interface is unified

3. **Add Forex-Aware Symbol Formatting**
   ```python
   # For Alpha Vantage forex indicators
   def format_symbol_for_indicator(symbol: str, instrument_type: InstrumentType) -> str:
       if instrument_type == InstrumentType.FOREX:
           return symbol  # XAUUSD format already correct
       return symbol
   ```

#### Deliverables
- [ ] Technical indicators working with XAUUSD
- [ ] Test cases for forex indicator fetching

---

### Phase 3: News & Sentiment (Week 3-4)
**Goal**: Enable gold/forex-specific news and sentiment

#### Tasks

1. **Extend News Data Tools**
   ```python
   # tradingagents/agents/utils/news_data_tools.py
   
   @tool
   def get_forex_news(
       symbol: Annotated[str, "forex pair symbol (e.g., XAUUSD)"],
       start_date: Annotated[str, "start date YYYY-MM-DD"],
       end_date: Annotated[str, "end date YYYY-MM-DD"],
   ) -> str:
       """Fetch news specifically for forex pairs using Alpha Vantage."""
       # Use NEWS_SENTIMENT endpoint with tickers=FOREX:XAU
   ```

2. **Create Forex-Aware News Analyst**
   - Clone of `news_analyst.py` but uses `get_forex_news` tool
   - Or extend existing news analyst to detect instrument type

3. **Update Social Media Analyst**
   - Use generic gold/forex search terms when analyzing XAUUSD
   - Search for: "gold price", "XAUUSD", "gold trading"

#### Deliverables
- [ ] `get_forex_news()` tool
- [ ] Forex-aware news analyst
- [ ] Updated social media analyst

---

### Phase 4: Smart Agent Selection (Week 4-5)
**Goal**: Disable fundamentals analyst for non-equity instruments

#### Tasks

1. **Create Instrument-Aware Analyst Router**
   ```python
   # tradingagents/graph/instrument_router.py
   
   def get_compatible_analysts(symbol: str, requested_analysts: List[str]) -> List[str]:
       """Return analysts compatible with the instrument type."""
       instrument_type = detect_instrument_type(symbol)
       
       incompatible = {
           InstrumentType.FOREX: ["fundamentals"],
           InstrumentType.COMMODITY: ["fundamentals", "insider_transactions"],
           InstrumentType.FUTURES: ["fundamentals", "insider_transactions"],
       }
       
       return [a for a in requested_analysts if a not in incompatible.get(instrument_type, [])]
   ```

2. **Update Graph Setup**
   - Filter analyst nodes based on instrument type
   - Show warning if fundamentals selected for XAUUSD

3. **Update CLI**
   - After ticker input, detect instrument type
   - Show available analysts
   - Pre-filter incompatible options

#### Deliverables
- [ ] `instrument_router.py` with filtering logic
- [ ] Updated `GraphSetup` to filter analysts
- [ ] CLI updates for analyst filtering

---

### Phase 5: Configuration & Documentation (Week 5-6)
**Goal**: Make XAUUSD easy to use via configuration

#### Tasks

1. **Add Forex Vendor Configuration**
   ```python
   # default_config.py
   DEFAULT_CONFIG = {
       # ... existing config ...
       "data_vendors": {
           # ... existing vendors ...
           "forex_data": "alpha_vantage",  # Options: alpha_vantage
       },
   }
   ```

2. **Add XAUUSD Examples to CLI Help**
   ```python
   TICKER_INPUT_EXAMPLES = (
       "Examples: SPY, CNC.TO, 7203.T, "
       "XAUUSD (Gold/USD forex), GC=F (Gold Futures), GLD (Gold ETF)"
   )
   ```

3. **Documentation Updates**
   - Add XAUUSD section to README.md
   - Document supported symbols and limitations
   - Add forex-specific configuration examples

4. **Example Scripts**
   - `examples/xauusd_analysis.py` - Basic XAUUSD analysis
   - `examples/gold_etf_comparison.py` - Compare XAUUSD vs GLD

#### Deliverables
- [ ] Updated `default_config.py`
- [ ] Updated CLI help text
- [ ] README updates
- [ ] Example scripts

---

## Technical Architecture

### New File Structure

```
tradingagents/
├── dataflows/
│   ├── instrument_utils.py        # NEW: Instrument type detection
│   ├── alpha_vantage_forex.py     # NEW: Forex data fetcher
│   └── interface.py               # MODIFIED: Add forex routing
├── agents/
│   └── utils/
│       ├── instrument_router.py   # NEW: Analyst filtering logic
│       └── forex_data_tools.py    # NEW: Forex-specific tools
└── graph/
    └── setup.py                   # MODIFIED: Filter analysts
```

### Data Flow for XAUUSD

```
+--------------------------------------------------------------------------+
|                         XAUUSD DATA FLOW                                 |
+--------------------------------------------------------------------------+

User Input: XAUUSD
       |
       v
+-----------------+
| Instrument Type |---> FOREX detected
|   Detection     |
+--------+--------+
         |
         v
+------------------------------------------------------------------+
|                    ANALYST SELECTION                            |
|  Compatible: [market, social, news]                            |
|  Incompatible: [fundamentals] ---> Disabled with warning       |
+------------------------------------------------------------------+
         |
         v
+------------------------------------------------------------------+
|                     DATA FETCHING                               |
+------------------------------------------------------------------+
| Market Analyst:                                                 |
|   |- get_stock_data ---> alpha_vantage_forex.get_fx_daily()    |
|   |- get_indicators ---> alpha_vantage_indicator (XAUUSD)      |
|                                                                 |
| News Analyst:                                                   |
|   |- get_news ---> alpha_vantage_news (tickers=FOREX:XAU)      |
|   |- get_global_news ---> yfinance_news (gold/economy search)  |
|                                                                 |
| Social Analyst:                                                 |
|   |- get_news ---> (gold sentiment search)                     |
+------------------------------------------------------------------+
         |
         v
+------------------------------------------------------------------+
|                  RESEARCH & DECISION                            |
|  (Same workflow as equities)                                    |
+------------------------------------------------------------------+
```

---

## Data Source Strategy

### Alpha Vantage Forex API Endpoints

| Endpoint | Function | Purpose |
|----------|----------|---------|
| `FX_DAILY` | Daily OHLCV | Primary price data for XAUUSD |
| `FX_INTRADAY` | Intraday OHLCV | High-frequency analysis |
| `FX_WEEKLY` | Weekly OHLCV | Long-term trend analysis |
| `NEWS_SENTIMENT` | News with forex filter | Gold/forex specific news |
| `SMA`, `EMA`, `RSI`, etc. | Technical indicators | Works with XAUUSD format |

### Symbol Format Mapping

| User Input | Alpha Vantage Format | yfinance Format | Purpose |
|------------|----------------------|-----------------|---------|
| `XAUUSD` | `from_symbol=XAU, to_symbol=USD` | `XAUUSD=X` | Forex spot |
| `GC=F` | N/A (futures not supported) | `GC=F` | Gold Futures |
| `GLD` | `symbol=GLD` | `GLD` | Gold ETF (equity) |

---

## Agent Adaptations

### Market Analyst (No Changes Needed)

The market analyst uses:
- `get_stock_data` → Will route to forex data fetcher
- `get_indicators` → Already supports forex symbols

### News Analyst (Minor Changes)

**Current**: Uses `get_news` which expects equity symbols
**New**: Extended to support forex symbols via `FOREX:XAU` notation

```python
# In alpha_vantage_news.py
def get_forex_news(symbol: str, start_date: str, end_date: str) -> str:
    # Parse XAUUSD -> from=XAU, to=USD
    from_curr = symbol[:3]
    to_curr = symbol[3:]
    
    params = {
        "tickers": f"FOREX:{from_curr}",  # Alpha Vantage format
        # ... other params
    }
```

### Fundamentals Analyst (Disabled for Forex)

**Reason**: Gold has no:
- P/E ratio, EPS
- Balance sheets
- Income statements
- Insider transactions

**Behavior**:
```python
if instrument_type == InstrumentType.FOREX:
    console.print(
        "[yellow]Warning: Fundamentals analyst disabled for XAUUSD "
        "(not applicable to forex pairs)[/yellow]"
    )
```

### Social Media Analyst (Enhanced)

**Current**: Generic sentiment analysis
**New**: Gold-specific keywords when analyzing XAUUSD

```python
def get_sentiment_keywords(symbol: str) -> List[str]:
    if symbol == "XAUUSD":
        return ["gold price", "XAUUSD", "gold trading", "precious metals"]
    return [symbol]
```

---

## Configuration Changes

### New Configuration Options

```python
# tradingagents/default_config.py

DEFAULT_CONFIG = {
    # ... existing config ...
    
    # Forex-specific settings
    "data_vendors": {
        # ... existing vendors ...
        "forex_data": "alpha_vantage",  # Options: alpha_vantage
    },
    
    # Instrument-specific behavior
    "instrument_settings": {
        "forex": {
            "enable_fundamentals": False,
            "enable_insider_transactions": False,
            "default_analysts": ["market", "news"],
        },
        "commodity": {
            "enable_fundamentals": False,
            "enable_insider_transactions": False,
        },
    },
}
```

---

## Testing Strategy

### Unit Tests

```python
# tests/test_instrument_detection.py

def test_detect_forex_xauusd():
    assert detect_instrument_type("XAUUSD") == InstrumentType.FOREX

def test_detect_forex_eurusd():
    assert detect_instrument_type("EURUSD") == InstrumentType.FOREX

def test_detect_futures_gc():
    assert detect_instrument_type("GC=F") == InstrumentType.FUTURES

def test_detect_stock_spy():
    assert detect_instrument_type("SPY") == InstrumentType.STOCK
```

### Integration Tests

```python
# tests/test_xauusd_workflow.py

@pytest.mark.skipif(not ALPHA_VANTAGE_KEY, reason="No API key")
def test_xauusd_data_fetching():
    data = get_fx_daily("XAU", "USD", "2024-01-01", "2024-01-31")
    assert "open" in data.lower()
    assert "close" in data.lower()

def test_xauusd_analyst_filtering():
    analysts = get_compatible_analysts("XAUUSD", ["market", "news", "fundamentals"])
    assert "fundamentals" not in analysts
    assert "market" in analysts
```

### Manual Test Cases

| Test Case | Input | Expected Behavior |
|-----------|-------|-------------------|
| Basic XAUUSD | `XAUUSD` with `market` + `news` | Successful analysis, no fundamentals |
| Gold Futures | `GC=F` with `market` + `news` | Uses yfinance, successful analysis |
| Gold ETF | `GLD` with all analysts | Full analysis including fundamentals |
| Invalid Combination | `XAUUSD` with `fundamentals` | Warning, fundamentals disabled |

---

## Risks & Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Alpha Vantage rate limits | Medium | High | Implement fallback to yfinance `GC=F` |
| XAUUSD data format differences | Medium | Medium | Normalize OHLCV format in interface |
| News API doesn't support forex | Low | Medium | Use generic global news for forex |
| User confusion about limitations | Medium | Low | Clear CLI warnings and documentation |
| Breaking existing equity flow | Low | High | Comprehensive regression testing |

---

## Future Extensibility

This architecture enables future support for:

1. **More Forex Pairs**: EURUSD, GBPUSD, USDJPY (same logic)
2. **Cryptocurrencies**: BTCUSD, ETHUSD (similar pattern)
3. **More Commodities**: Silver (XAGUSD), Oil (WTI, Brent)
4. **Custom Data Sources**: Integrate additional forex providers

---

## Success Criteria

- [ ] User can run `tradingagents` and analyze `XAUUSD` successfully
- [ ] Technical indicators work for XAUUSD
- [ ] News analysis includes gold-specific content
- [ ] Fundamentals analyst is disabled with clear warning
- [ ] Existing equity analysis remains unaffected
- [ ] Documentation clearly explains XAUUSD support

---

## Timeline Summary

| Phase | Duration | Key Deliverable |
|-------|----------|-----------------|
| Phase 1 | 2 weeks | Foundation: data fetching |
| Phase 2 | 1 week | Technical indicators |
| Phase 3 | 1 week | News & sentiment |
| Phase 4 | 1 week | Smart agent selection |
| Phase 5 | 1 week | Configuration & docs |
| **Total** | **6 weeks** | Full XAUUSD support |

---

### Virtual Environment Setup

To work on this implementation:

```bash
# Navigate to project
cd /Users/rithsila/Projects/TradingAgents

# Create and activate virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -e .
```

---

*Document Version: 1.0*
*Last Updated: 2026-03-23*
