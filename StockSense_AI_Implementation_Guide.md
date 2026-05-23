# StockSense AI — MVP Implementation Guide

> **Stack:** Python · uv · Ollama (Llama 3.2 3B) · yfinance · LangChain · Streamlit · Seaborn  
> **OS:** Windows 11 · VS Code  
> **Scope:** Full MVP — Fundamentals + Technical Indicators + Seaborn Charts + News Sentiment + LLM Report

---

## Table of Contents

1. [Project Structure](#1-project-structure)
2. [Phase 0 — One-Time Machine Setup](#2-phase-0--one-time-machine-setup)
3. [Phase 1 — Project Scaffold](#3-phase-1--project-scaffold)
4. [Phase 2 — Data Layer](#4-phase-2--data-layer)
5. [Phase 3 — Analysis Modules](#5-phase-3--analysis-modules)
6. [Phase 4 — LLM Integration](#6-phase-4--llm-integration)
7. [Phase 5 — Streamlit UI](#7-phase-5--streamlit-ui)
8. [Phase 6 — Polish & Run](#8-phase-6--polish--run)
9. [Architecture Diagram (Text)](#9-architecture-diagram-text)
10. [SOLID Principles Applied](#10-solid-principles-applied)
11. [Rubric Checklist](#11-rubric-checklist)

---

## 1. Project Structure

This is the target folder layout you will build toward, step by step.

```
stocksense/
├── pyproject.toml          # uv project config & dependencies
├── .python-version         # pins Python version for uv
├── README.md
├── .env                    # API keys (never commit this)
├── .gitignore
│
├── src/
│   └── stocksense/
│       ├── __init__.py
│       ├── main.py                 # Streamlit entry point
│       │
│       ├── data/
│       │   ├── __init__.py
│       │   └── fetcher.py          # yfinance + news data fetching
│       │
│       ├── analysis/
│       │   ├── __init__.py
│       │   ├── fundamentals.py     # P/E, EPS, ROE, debt ratios
│       │   ├── technicals.py       # RSI, MACD, Bollinger Bands
│       │   └── sentiment.py        # News headline scoring
│       │
│       ├── llm/
│       │   ├── __init__.py
│       │   └── analyst.py          # LangChain + Ollama synthesis
│       │
│       └── charts/
│           ├── __init__.py
│           └── visualizer.py       # Seaborn/Matplotlib figures
│
└── tests/
    ├── __init__.py
    ├── test_fetcher.py
    ├── test_fundamentals.py
    └── test_technicals.py
```

---

## 2. Phase 0 — One-Time Machine Setup

### 2.1 Install uv

Open **PowerShell as Administrator** and run:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Close and reopen PowerShell. Verify:

```powershell
uv --version
```

You should see something like `uv 0.5.x`.

---

### 2.2 Install Ollama and Pull the Model

1. Download Ollama from **https://ollama.com/download** → choose Windows installer.
2. Run the installer. Ollama runs as a background service automatically.
3. Open a new PowerShell window and pull the model:

```powershell
ollama pull llama3.2:3b
```

This downloads ~2 GB. When it finishes, verify it works:

```powershell
ollama run llama3.2:3b "Say hello in one sentence."
```

You should get a short response. Type `/bye` to exit.

> **Important:** Ollama must be running in the background whenever you use StockSense AI. It starts automatically after install, but if you ever restart it manually run: `ollama serve`

---

### 2.3 VS Code Extensions to Install

Open VS Code, press `Ctrl+Shift+X`, and install:

- **Python** (Microsoft)
- **Pylance** (Microsoft)
- **Ruff** (linting & formatting)
- **Even Better TOML** (for pyproject.toml editing)
- **GitLens** (optional but helpful)

---

## 3. Phase 1 — Project Scaffold

### 3.1 Create the Project

Open PowerShell in whatever parent folder you want, e.g. `C:\Projects`:

```powershell
cd C:\Projects
uv init stocksense
cd stocksense
```

Set the Python version (3.11 is stable and well-supported):

```powershell
uv python pin 3.11
```

This creates a `.python-version` file automatically.

---

### 3.2 Create the Full Folder Structure

Run each command below in order:

```powershell
# Source package
mkdir src\stocksense
mkdir src\stocksense\data
mkdir src\stocksense\analysis
mkdir src\stocksense\llm
mkdir src\stocksense\charts

# Tests
mkdir tests

# Create all __init__.py files (makes directories into Python packages)
New-Item src\stocksense\__init__.py -ItemType File
New-Item src\stocksense\data\__init__.py -ItemType File
New-Item src\stocksense\analysis\__init__.py -ItemType File
New-Item src\stocksense\llm\__init__.py -ItemType File
New-Item src\stocksense\charts\__init__.py -ItemType File
New-Item tests\__init__.py -ItemType File

# Create placeholder source files
New-Item src\stocksense\main.py -ItemType File
New-Item src\stocksense\data\fetcher.py -ItemType File
New-Item src\stocksense\analysis\fundamentals.py -ItemType File
New-Item src\stocksense\analysis\technicals.py -ItemType File
New-Item src\stocksense\analysis\sentiment.py -ItemType File
New-Item src\stocksense\llm\analyst.py -ItemType File
New-Item src\stocksense\charts\visualizer.py -ItemType File

# Test files
New-Item tests\test_fetcher.py -ItemType File
New-Item tests\test_fundamentals.py -ItemType File
New-Item tests\test_technicals.py -ItemType File

# Env and git files
New-Item .env -ItemType File
New-Item .gitignore -ItemType File
New-Item README.md -ItemType File
```

Open the project in VS Code:

```powershell
code .
```

---

### 3.3 Configure pyproject.toml

Replace the entire contents of `pyproject.toml` with:

```toml
[project]
name = "stocksense"
version = "0.1.0"
description = "AI-powered stock analysis tool for retail investors"
requires-python = ">=3.11"

dependencies = [
    "yfinance>=0.2.40",
    "pandas>=2.2.0",
    "numpy>=1.26.0",
    "langchain>=0.3.0",
    "langchain-ollama>=0.2.0",
    "streamlit>=1.40.0",
    "seaborn>=0.13.0",
    "matplotlib>=3.8.0",
    "textblob>=0.18.0",
    "python-dotenv>=1.0.0",
    "requests>=2.31.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "pytest-cov>=5.0.0",
    "ruff>=0.6.0",
]

[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I"]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["src/stocksense"]
```

Now install all dependencies:

```powershell
uv sync
uv sync --extra dev
```

This creates a `.venv` folder automatically. Wait for it to finish — it downloads all packages.

---

### 3.4 Configure .gitignore

Paste this into `.gitignore`:

```
.venv/
__pycache__/
*.pyc
.env
*.egg-info/
dist/
build/
.pytest_cache/
.ruff_cache/
*.png
*.jpg
```

---

### 3.5 Configure .env

Paste this into `.env`. Leave the value blank for now — you will only need this if you later add a paid news API. yfinance and Ollama are both free and need no keys.

```
# Optional: Alpha Vantage key for premium news (free tier available)
# ALPHA_VANTAGE_API_KEY=your_key_here

# Ollama host — default is fine for local use
OLLAMA_HOST=http://localhost:11434
```

---

### 3.6 Tell VS Code to Use the uv Virtual Environment

Press `Ctrl+Shift+P` → type **Python: Select Interpreter** → choose the one that shows `.venv` in its path (e.g. `.\.venv\Scripts\python.exe`).

---

## 4. Phase 2 — Data Layer

### 4.1 Write `src/stocksense/data/fetcher.py`

This module is responsible for **one thing only**: fetching raw data. It does not analyse or display anything (Single Responsibility Principle).

```python
"""
data/fetcher.py
Responsible for fetching stock price history, fundamentals,
and news headlines from yfinance. All other modules consume
the data returned here — no fetching happens elsewhere.
"""

from __future__ import annotations

import logging
from dataclasses import dataclass, field
from typing import Optional

import pandas as pd
import yfinance as yf

logger = logging.getLogger(__name__)


@dataclass
class StockData:
    """
    Value object that holds all raw data for one ticker.
    Immutable after construction — analysis modules read it, never write it.
    """
    ticker: str
    info: dict
    history: pd.DataFrame          # OHLCV — daily, last 6 months
    news: list[dict] = field(default_factory=list)


class StockFetcher:
    """
    Fetches data from Yahoo Finance via yfinance.

    Usage:
        fetcher = StockFetcher()
        data = fetcher.fetch("AAPL")
    """

    def fetch(self, ticker: str) -> Optional[StockData]:
        """
        Fetch info, 6-month price history, and recent news for a ticker.

        Returns StockData on success, None if the ticker is invalid.
        """
        ticker = ticker.strip().upper()
        logger.info("Fetching data for %s", ticker)

        try:
            yf_ticker = yf.Ticker(ticker)

            info = yf_ticker.info
            # yfinance returns a minimal dict for bad tickers
            if not info or info.get("regularMarketPrice") is None:
                logger.warning("Ticker %s returned no market price — may be invalid.", ticker)
                return None

            history = yf_ticker.history(period="6mo", interval="1d")
            if history.empty:
                logger.warning("No price history for %s.", ticker)
                return None

            news = yf_ticker.news or []

            return StockData(ticker=ticker, info=info, history=history, news=news)

        except Exception as exc:
            logger.error("Failed to fetch data for %s: %s", ticker, exc)
            return None
```

---

### 4.2 Write the test for it: `tests/test_fetcher.py`

```python
"""
Tests for the StockFetcher class.
Run with:  uv run pytest tests/test_fetcher.py -v
"""

from stocksense.data.fetcher import StockFetcher


def test_fetch_valid_ticker_returns_stock_data():
    fetcher = StockFetcher()
    data = fetcher.fetch("AAPL")
    assert data is not None
    assert data.ticker == "AAPL"
    assert not data.history.empty
    assert isinstance(data.info, dict)


def test_fetch_invalid_ticker_returns_none():
    fetcher = StockFetcher()
    data = fetcher.fetch("XXXXNOTREAL9999")
    assert data is None


def test_fetch_normalises_lowercase_ticker():
    fetcher = StockFetcher()
    data = fetcher.fetch("aapl")
    assert data is not None
    assert data.ticker == "AAPL"
```

Run your first test now to confirm the data layer works:

```powershell
uv run pytest tests/test_fetcher.py -v
```

All three tests should pass. If `XXXXNOTREAL9999` returns data (unlikely but possible), just swap in a different nonsense string.

---

## 5. Phase 3 — Analysis Modules

### 5.1 Write `src/stocksense/analysis/fundamentals.py`

```python
"""
analysis/fundamentals.py
Extracts and scores fundamental financial metrics from a StockData object.
Does not fetch data — receives a StockData value object as input.
"""

from __future__ import annotations

from dataclasses import dataclass
from typing import Optional

from stocksense.data.fetcher import StockData


@dataclass
class FundamentalsReport:
    """Structured result from the fundamentals analyser."""
    ticker: str
    company_name: str
    sector: str
    current_price: Optional[float]
    market_cap: Optional[float]
    pe_ratio: Optional[float]
    eps: Optional[float]
    roe: Optional[float]           # Return on Equity (%)
    debt_to_equity: Optional[float]
    dividend_yield: Optional[float]
    fifty_two_week_high: Optional[float]
    fifty_two_week_low: Optional[float]
    score: int                     # 0–100 composite score
    verdict: str                   # "Strong" | "Moderate" | "Weak"


class FundamentalsAnalyser:
    """
    Analyses fundamental financial ratios.

    The scoring method awards points for healthy ratios and
    deducts for poor ones. All thresholds are rough industry norms
    for general equities — not financial advice.
    """

    def analyse(self, stock_data: StockData) -> FundamentalsReport:
        """Return a FundamentalsReport for the given StockData."""
        info = stock_data.info

        # --- Extract raw values (None if not available) ---
        current_price = info.get("currentPrice") or info.get("regularMarketPrice")
        pe_ratio = info.get("trailingPE")
        eps = info.get("trailingEps")
        roe = self._to_percent(info.get("returnOnEquity"))
        debt_to_equity = info.get("debtToEquity")
        dividend_yield = self._to_percent(info.get("dividendYield"))
        market_cap = info.get("marketCap")
        fifty_two_week_high = info.get("fiftyTwoWeekHigh")
        fifty_two_week_low = info.get("fiftyTwoWeekLow")

        score = self._compute_score(pe_ratio, eps, roe, debt_to_equity)
        verdict = self._score_to_verdict(score)

        return FundamentalsReport(
            ticker=stock_data.ticker,
            company_name=info.get("longName", stock_data.ticker),
            sector=info.get("sector", "Unknown"),
            current_price=current_price,
            market_cap=market_cap,
            pe_ratio=pe_ratio,
            eps=eps,
            roe=roe,
            debt_to_equity=debt_to_equity,
            dividend_yield=dividend_yield,
            fifty_two_week_high=fifty_two_week_high,
            fifty_two_week_low=fifty_two_week_low,
            score=score,
            verdict=verdict,
        )

    # ------------------------------------------------------------------
    # Private helpers
    # ------------------------------------------------------------------

    def _to_percent(self, value: Optional[float]) -> Optional[float]:
        """Convert a decimal ratio (0.15) to a percentage (15.0)."""
        if value is None:
            return None
        return round(value * 100, 2)

    def _compute_score(
        self,
        pe: Optional[float],
        eps: Optional[float],
        roe: Optional[float],
        de: Optional[float],
    ) -> int:
        """
        Simple 0–100 composite score.
        Each criterion contributes up to 25 points.
        """
        score = 50  # neutral baseline

        # P/E ratio: lower is better for value investors (industry avg ~25)
        if pe is not None:
            if pe < 15:
                score += 15
            elif pe < 25:
                score += 5
            elif pe > 40:
                score -= 10

        # EPS: positive is good
        if eps is not None:
            if eps > 5:
                score += 15
            elif eps > 0:
                score += 5
            else:
                score -= 15

        # ROE: >15% is healthy
        if roe is not None:
            if roe > 20:
                score += 10
            elif roe > 10:
                score += 5
            elif roe < 0:
                score -= 10

        # Debt-to-equity: <1.0 is conservative
        if de is not None:
            if de < 50:        # yfinance returns D/E as 50 = 0.50
                score += 10
            elif de > 200:
                score -= 10

        return max(0, min(100, score))

    def _score_to_verdict(self, score: int) -> str:
        if score >= 65:
            return "Strong"
        elif score >= 45:
            return "Moderate"
        return "Weak"
```

---

### 5.2 Write `src/stocksense/analysis/technicals.py`

```python
"""
analysis/technicals.py
Computes RSI, MACD, and Bollinger Bands from price history.
All calculations use pandas — no external TA library needed.
"""

from __future__ import annotations

from dataclasses import dataclass
from typing import Optional

import numpy as np
import pandas as pd

from stocksense.data.fetcher import StockData


@dataclass
class TechnicalsReport:
    """Structured result from the technical analyser."""
    ticker: str
    current_price: float
    rsi: Optional[float]            # 0–100; >70 overbought, <30 oversold
    macd: Optional[float]           # MACD line value
    macd_signal: Optional[float]    # Signal line value
    macd_histogram: Optional[float] # MACD − Signal
    bb_upper: Optional[float]       # Bollinger upper band
    bb_middle: Optional[float]      # 20-day SMA
    bb_lower: Optional[float]       # Bollinger lower band
    rsi_signal: str                 # "Overbought" | "Neutral" | "Oversold"
    macd_signal_label: str          # "Bullish" | "Bearish" | "Neutral"
    price_history: pd.DataFrame     # Full OHLCV + indicators (for charts)


class TechnicalsAnalyser:
    """
    Computes technical indicators from raw price history.

    All methods are pure functions of the input DataFrame —
    no side effects, no network calls.
    """

    RSI_PERIOD = 14
    MACD_FAST = 12
    MACD_SLOW = 26
    MACD_SIGNAL_PERIOD = 9
    BB_PERIOD = 20
    BB_STD = 2

    def analyse(self, stock_data: StockData) -> Optional[TechnicalsReport]:
        """
        Compute RSI, MACD, and Bollinger Bands.
        Returns None if there is insufficient price history (<30 days).
        """
        df = stock_data.history.copy()

        if len(df) < 30:
            return None

        df = self._add_rsi(df)
        df = self._add_macd(df)
        df = self._add_bollinger_bands(df)

        latest = df.iloc[-1]
        current_price = float(latest["Close"])

        rsi = self._safe_float(latest.get("RSI"))
        macd = self._safe_float(latest.get("MACD"))
        macd_signal = self._safe_float(latest.get("MACD_Signal"))
        macd_hist = self._safe_float(latest.get("MACD_Hist"))

        return TechnicalsReport(
            ticker=stock_data.ticker,
            current_price=current_price,
            rsi=rsi,
            macd=macd,
            macd_signal=macd_signal,
            macd_histogram=macd_hist,
            bb_upper=self._safe_float(latest.get("BB_Upper")),
            bb_middle=self._safe_float(latest.get("BB_Middle")),
            bb_lower=self._safe_float(latest.get("BB_Lower")),
            rsi_signal=self._classify_rsi(rsi),
            macd_signal_label=self._classify_macd(macd, macd_signal),
            price_history=df,
        )

    # ------------------------------------------------------------------
    # Indicator calculations
    # ------------------------------------------------------------------

    def _add_rsi(self, df: pd.DataFrame) -> pd.DataFrame:
        """Wilder's RSI using exponential moving average of gains/losses."""
        delta = df["Close"].diff()
        gain = delta.clip(lower=0)
        loss = -delta.clip(upper=0)

        avg_gain = gain.ewm(alpha=1 / self.RSI_PERIOD, min_periods=self.RSI_PERIOD).mean()
        avg_loss = loss.ewm(alpha=1 / self.RSI_PERIOD, min_periods=self.RSI_PERIOD).mean()

        rs = avg_gain / avg_loss.replace(0, np.nan)
        df["RSI"] = 100 - (100 / (1 + rs))
        return df

    def _add_macd(self, df: pd.DataFrame) -> pd.DataFrame:
        """Standard MACD (12, 26, 9)."""
        ema_fast = df["Close"].ewm(span=self.MACD_FAST, adjust=False).mean()
        ema_slow = df["Close"].ewm(span=self.MACD_SLOW, adjust=False).mean()
        df["MACD"] = ema_fast - ema_slow
        df["MACD_Signal"] = df["MACD"].ewm(span=self.MACD_SIGNAL_PERIOD, adjust=False).mean()
        df["MACD_Hist"] = df["MACD"] - df["MACD_Signal"]
        return df

    def _add_bollinger_bands(self, df: pd.DataFrame) -> pd.DataFrame:
        """20-day Bollinger Bands with 2 standard deviations."""
        sma = df["Close"].rolling(window=self.BB_PERIOD).mean()
        std = df["Close"].rolling(window=self.BB_PERIOD).std()
        df["BB_Middle"] = sma
        df["BB_Upper"] = sma + self.BB_STD * std
        df["BB_Lower"] = sma - self.BB_STD * std
        return df

    # ------------------------------------------------------------------
    # Signal classifiers
    # ------------------------------------------------------------------

    def _classify_rsi(self, rsi: Optional[float]) -> str:
        if rsi is None:
            return "Neutral"
        if rsi > 70:
            return "Overbought"
        if rsi < 30:
            return "Oversold"
        return "Neutral"

    def _classify_macd(self, macd: Optional[float], signal: Optional[float]) -> str:
        if macd is None or signal is None:
            return "Neutral"
        if macd > signal:
            return "Bullish"
        if macd < signal:
            return "Bearish"
        return "Neutral"

    def _safe_float(self, value) -> Optional[float]:
        try:
            v = float(value)
            return round(v, 4) if not np.isnan(v) else None
        except (TypeError, ValueError):
            return None
```

---

### 5.3 Write `src/stocksense/analysis/sentiment.py`

```python
"""
analysis/sentiment.py
Scores news headlines using TextBlob polarity analysis.
TextBlob is free, runs locally, and requires no API key.
"""

from __future__ import annotations

from dataclasses import dataclass
from typing import Optional

from textblob import TextBlob

from stocksense.data.fetcher import StockData


@dataclass
class SentimentReport:
    """Structured result from the sentiment analyser."""
    ticker: str
    headline_count: int
    average_polarity: float        # -1.0 (negative) to +1.0 (positive)
    average_subjectivity: float    # 0.0 (objective) to 1.0 (subjective)
    verdict: str                   # "Positive" | "Neutral" | "Negative"
    top_headlines: list[str]       # Up to 5 most recent headlines


class SentimentAnalyser:
    """
    Analyses the sentiment of news headlines fetched from yfinance.
    Uses TextBlob for offline NLP — no API key required.
    """

    def analyse(self, stock_data: StockData) -> SentimentReport:
        """Score each headline and return an aggregated SentimentReport."""
        news = stock_data.news

        if not news:
            return SentimentReport(
                ticker=stock_data.ticker,
                headline_count=0,
                average_polarity=0.0,
                average_subjectivity=0.0,
                verdict="Neutral",
                top_headlines=[],
            )

        polarities = []
        subjectivities = []
        headlines = []

        for article in news[:10]:   # analyse up to 10 most recent
            title = article.get("title", "")
            if not title:
                continue

            blob = TextBlob(title)
            polarities.append(blob.sentiment.polarity)
            subjectivities.append(blob.sentiment.subjectivity)
            headlines.append(title)

        if not polarities:
            avg_polarity = 0.0
            avg_subjectivity = 0.0
        else:
            avg_polarity = sum(polarities) / len(polarities)
            avg_subjectivity = sum(subjectivities) / len(subjectivities)

        return SentimentReport(
            ticker=stock_data.ticker,
            headline_count=len(headlines),
            average_polarity=round(avg_polarity, 4),
            average_subjectivity=round(avg_subjectivity, 4),
            verdict=self._classify(avg_polarity),
            top_headlines=headlines[:5],
        )

    def _classify(self, polarity: float) -> str:
        if polarity > 0.05:
            return "Positive"
        if polarity < -0.05:
            return "Negative"
        return "Neutral"
```

TextBlob requires a one-time download of its language data. Run this once:

```powershell
uv run python -c "import textblob; textblob.download_corpora()"
```

---

### 5.4 Write Tests

#### `tests/test_fundamentals.py`

```python
from stocksense.data.fetcher import StockFetcher
from stocksense.analysis.fundamentals import FundamentalsAnalyser


def test_fundamentals_returns_report_for_valid_ticker():
    data = StockFetcher().fetch("MSFT")
    report = FundamentalsAnalyser().analyse(data)
    assert report.ticker == "MSFT"
    assert report.score >= 0
    assert report.verdict in ("Strong", "Moderate", "Weak")


def test_score_is_bounded():
    data = StockFetcher().fetch("AAPL")
    report = FundamentalsAnalyser().analyse(data)
    assert 0 <= report.score <= 100
```

#### `tests/test_technicals.py`

```python
from stocksense.data.fetcher import StockFetcher
from stocksense.analysis.technicals import TechnicalsAnalyser


def test_technicals_returns_report_for_valid_ticker():
    data = StockFetcher().fetch("AAPL")
    report = TechnicalsAnalyser().analyse(data)
    assert report is not None
    assert report.ticker == "AAPL"


def test_rsi_is_in_valid_range():
    data = StockFetcher().fetch("AAPL")
    report = TechnicalsAnalyser().analyse(data)
    if report.rsi is not None:
        assert 0 <= report.rsi <= 100


def test_rsi_signal_is_valid_label():
    data = StockFetcher().fetch("AAPL")
    report = TechnicalsAnalyser().analyse(data)
    assert report.rsi_signal in ("Overbought", "Neutral", "Oversold")
```

Run all tests:

```powershell
uv run pytest tests/ -v
```

---

## 6. Phase 4 — LLM Integration

### 6.1 Write `src/stocksense/llm/analyst.py`

```python
"""
llm/analyst.py
Uses LangChain + Ollama to synthesise analysis reports into a
plain-language investment thesis. Depends on the three report
dataclasses — does not fetch data or compute indicators itself.
"""

from __future__ import annotations

import logging
from typing import Optional

from langchain_ollama import OllamaLLM
from langchain.prompts import PromptTemplate

from stocksense.analysis.fundamentals import FundamentalsReport
from stocksense.analysis.technicals import TechnicalsReport
from stocksense.analysis.sentiment import SentimentReport

logger = logging.getLogger(__name__)

# ------------------------------------------------------------------
# Prompt template — edit this to tune the LLM's output style
# ------------------------------------------------------------------

_ANALYSIS_TEMPLATE = """\
You are StockSense AI, a clear and objective stock analyst assistant.
Your job is to synthesise financial data into a concise plain-language report.
Always end with a disclaimer that this is not financial advice.

--- STOCK: {ticker} ({company_name}) | Sector: {sector} ---

FUNDAMENTALS
- Current Price: ${current_price}
- P/E Ratio: {pe_ratio}
- EPS: {eps}
- Return on Equity: {roe}%
- Debt-to-Equity: {debt_to_equity}
- Dividend Yield: {dividend_yield}%
- Fundamental Score: {fundamental_score}/100 ({fundamental_verdict})

TECHNICAL INDICATORS (last 6 months)
- RSI (14): {rsi} — {rsi_signal}
- MACD Signal: {macd_signal}
- Price vs Bollinger Bands: Upper={bb_upper}, Middle={bb_middle}, Lower={bb_lower}

NEWS SENTIMENT (last 10 headlines)
- Average Polarity: {sentiment_polarity} ({sentiment_verdict})
- Sample Headlines: {headlines}

---

Write a 3-paragraph investment thesis:
1. Paragraph 1 — Fundamental health summary (2-3 sentences).
2. Paragraph 2 — Technical momentum and trend summary (2-3 sentences).
3. Paragraph 3 — Sentiment context and overall outlook with a Bull/Neutral/Bear label.

Keep language simple and clear. Avoid jargon. Be honest about uncertainty.
End with: "⚠️ Disclaimer: This analysis is for educational purposes only and is not financial advice."
"""

_PROMPT = PromptTemplate(
    input_variables=[
        "ticker", "company_name", "sector", "current_price",
        "pe_ratio", "eps", "roe", "debt_to_equity", "dividend_yield",
        "fundamental_score", "fundamental_verdict",
        "rsi", "rsi_signal", "macd_signal", "bb_upper", "bb_middle", "bb_lower",
        "sentiment_polarity", "sentiment_verdict", "headlines",
    ],
    template=_ANALYSIS_TEMPLATE,
)


class StockAnalystLLM:
    """
    Synthesises three analysis reports into a human-readable thesis
    using a locally-running Llama 3.2 3B model via Ollama.

    Usage:
        analyst = StockAnalystLLM()
        thesis = analyst.synthesise(fundamentals, technicals, sentiment)
    """

    def __init__(self, model: str = "llama3.2:3b", temperature: float = 0.3):
        """
        Args:
            model: Ollama model name. Defaults to llama3.2:3b.
            temperature: Lower = more factual, higher = more creative.
        """
        self._llm = OllamaLLM(model=model, temperature=temperature)
        self._chain = _PROMPT | self._llm

    def synthesise(
        self,
        fund: FundamentalsReport,
        tech: Optional[TechnicalsReport],
        sent: SentimentReport,
    ) -> str:
        """
        Run the LLM chain and return the generated thesis as a string.
        Returns an error message string if the LLM call fails.
        """
        logger.info("Running LLM synthesis for %s", fund.ticker)

        # Format values — replace None with "N/A"
        def fmt(value, suffix="") -> str:
            return f"{value}{suffix}" if value is not None else "N/A"

        headlines_text = (
            "; ".join(sent.top_headlines[:3]) if sent.top_headlines else "No recent headlines"
        )

        inputs = {
            "ticker": fund.ticker,
            "company_name": fund.company_name,
            "sector": fund.sector,
            "current_price": fmt(fund.current_price),
            "pe_ratio": fmt(fund.pe_ratio),
            "eps": fmt(fund.eps),
            "roe": fmt(fund.roe),
            "debt_to_equity": fmt(fund.debt_to_equity),
            "dividend_yield": fmt(fund.dividend_yield),
            "fundamental_score": fund.score,
            "fundamental_verdict": fund.verdict,
            "rsi": fmt(tech.rsi if tech else None),
            "rsi_signal": tech.rsi_signal if tech else "N/A",
            "macd_signal": tech.macd_signal_label if tech else "N/A",
            "bb_upper": fmt(tech.bb_upper if tech else None),
            "bb_middle": fmt(tech.bb_middle if tech else None),
            "bb_lower": fmt(tech.bb_lower if tech else None),
            "sentiment_polarity": sent.average_polarity,
            "sentiment_verdict": sent.verdict,
            "headlines": headlines_text,
        }

        try:
            result = self._chain.invoke(inputs)
            return str(result).strip()
        except Exception as exc:
            logger.error("LLM synthesis failed: %s", exc)
            return (
                f"⚠️ Could not generate AI thesis: {exc}\n"
                "Make sure Ollama is running: `ollama serve`"
            )
```

---

## 7. Phase 5 — Streamlit UI

### 7.1 Write `src/stocksense/charts/visualizer.py`

```python
"""
charts/visualizer.py
Produces Seaborn/Matplotlib figures from TechnicalsReport data.
Each method returns a matplotlib Figure — Streamlit displays it
with st.pyplot(fig). No plt.show() calls here.
"""

from __future__ import annotations

import matplotlib
matplotlib.use("Agg")   # non-interactive backend for Streamlit

import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
import seaborn as sns
import pandas as pd

from stocksense.analysis.technicals import TechnicalsReport

# Apply a clean Seaborn theme globally
sns.set_theme(style="darkgrid", palette="muted")


class StockVisualizer:
    """
    Creates publication-quality charts from a TechnicalsReport.
    All methods are stateless — pass the report, receive a Figure.
    """

    def price_chart(self, report: TechnicalsReport) -> plt.Figure:
        """
        Combined chart: price + Bollinger Bands (top) + RSI (bottom).
        """
        df = report.price_history.dropna(subset=["BB_Upper"])
        fig = plt.figure(figsize=(12, 7))
        gs = gridspec.GridSpec(2, 1, height_ratios=[3, 1], hspace=0.05)

        # --- Top: Price + Bollinger Bands ---
        ax1 = fig.add_subplot(gs[0])
        ax1.plot(df.index, df["Close"], label="Close Price", color="#2196F3", linewidth=1.5)
        ax1.plot(df.index, df["BB_Upper"], label="BB Upper", color="#FF7043",
                 linestyle="--", linewidth=1, alpha=0.7)
        ax1.plot(df.index, df["BB_Middle"], label="BB Middle (SMA 20)", color="#9E9E9E",
                 linestyle="--", linewidth=1, alpha=0.7)
        ax1.plot(df.index, df["BB_Lower"], label="BB Lower", color="#66BB6A",
                 linestyle="--", linewidth=1, alpha=0.7)
        ax1.fill_between(df.index, df["BB_Lower"], df["BB_Upper"],
                         alpha=0.06, color="#2196F3")
        ax1.set_ylabel("Price (USD)", fontsize=11)
        ax1.set_title(f"{report.ticker} — Price & Bollinger Bands", fontsize=13, fontweight="bold")
        ax1.legend(loc="upper left", fontsize=9)
        ax1.set_xticklabels([])

        # --- Bottom: RSI ---
        ax2 = fig.add_subplot(gs[1], sharex=ax1)
        rsi_color = (
            "#EF5350" if report.rsi and report.rsi > 70
            else "#66BB6A" if report.rsi and report.rsi < 30
            else "#42A5F5"
        )
        ax2.plot(df.index, df["RSI"], color=rsi_color, linewidth=1.5, label="RSI (14)")
        ax2.axhline(70, color="#EF5350", linestyle=":", linewidth=1, alpha=0.8)
        ax2.axhline(30, color="#66BB6A", linestyle=":", linewidth=1, alpha=0.8)
        ax2.fill_between(df.index, df["RSI"], 70,
                         where=(df["RSI"] > 70), alpha=0.15, color="#EF5350")
        ax2.fill_between(df.index, df["RSI"], 30,
                         where=(df["RSI"] < 30), alpha=0.15, color="#66BB6A")
        ax2.set_ylim(0, 100)
        ax2.set_ylabel("RSI", fontsize=10)
        ax2.set_xlabel("Date", fontsize=10)
        ax2.legend(loc="upper left", fontsize=9)

        fig.autofmt_xdate(rotation=30)
        fig.tight_layout()
        return fig

    def macd_chart(self, report: TechnicalsReport) -> plt.Figure:
        """MACD line, signal line, and histogram."""
        df = report.price_history.dropna(subset=["MACD"])
        fig, ax = plt.subplots(figsize=(12, 4))

        ax.plot(df.index, df["MACD"], label="MACD", color="#2196F3", linewidth=1.5)
        ax.plot(df.index, df["MACD_Signal"], label="Signal", color="#FF7043",
                linestyle="--", linewidth=1.5)

        # Histogram — green above zero, red below
        colors = ["#66BB6A" if v >= 0 else "#EF5350" for v in df["MACD_Hist"]]
        ax.bar(df.index, df["MACD_Hist"], color=colors, alpha=0.5, width=0.8, label="Histogram")
        ax.axhline(0, color="#9E9E9E", linewidth=0.8)

        ax.set_title(f"{report.ticker} — MACD (12, 26, 9)", fontsize=13, fontweight="bold")
        ax.set_ylabel("MACD Value", fontsize=10)
        ax.set_xlabel("Date", fontsize=10)
        ax.legend(loc="upper left", fontsize=9)
        fig.autofmt_xdate(rotation=30)
        fig.tight_layout()
        return fig

    def fundamentals_bar(self, metrics: dict[str, float]) -> plt.Figure:
        """
        Horizontal bar chart of key fundamental metrics.
        Pass a dict like {"P/E Ratio": 28.5, "ROE (%)": 15.2, ...}
        """
        keys = list(metrics.keys())
        values = list(metrics.values())
        colors = ["#66BB6A" if v > 0 else "#EF5350" for v in values]

        fig, ax = plt.subplots(figsize=(8, max(3, len(keys) * 0.6)))
        bars = ax.barh(keys, values, color=colors, edgecolor="none", height=0.5)
        ax.bar_label(bars, fmt="%.2f", padding=4, fontsize=10)
        ax.set_xlabel("Value", fontsize=10)
        ax.set_title("Key Fundamental Metrics", fontsize=12, fontweight="bold")
        ax.axvline(0, color="#9E9E9E", linewidth=0.8)
        fig.tight_layout()
        return fig
```

---

### 7.2 Write the Streamlit App: `src/stocksense/main.py`

This is the biggest file — the full UI. Read through every comment as you type it.

```python
"""
main.py — StockSense AI Streamlit Application

Run with:
    uv run streamlit run src/stocksense/main.py

Architecture:
    User Input (ticker) → StockFetcher → StockData
    StockData → FundamentalsAnalyser → FundamentalsReport
    StockData → TechnicalsAnalyser   → TechnicalsReport
    StockData → SentimentAnalyser    → SentimentReport
    All reports → StockAnalystLLM    → Thesis text
    TechnicalsReport → StockVisualizer → Seaborn charts
"""

import logging

import streamlit as st

from stocksense.data.fetcher import StockFetcher
from stocksense.analysis.fundamentals import FundamentalsAnalyser
from stocksense.analysis.technicals import TechnicalsAnalyser
from stocksense.analysis.sentiment import SentimentAnalyser
from stocksense.llm.analyst import StockAnalystLLM
from stocksense.charts.visualizer import StockVisualizer

# ------------------------------------------------------------------
# Logging — will appear in your terminal, not in the browser
# ------------------------------------------------------------------
logging.basicConfig(level=logging.INFO, format="%(levelname)s  %(name)s  %(message)s")

# ------------------------------------------------------------------
# Page configuration — must be the first Streamlit call
# ------------------------------------------------------------------
st.set_page_config(
    page_title="StockSense AI",
    page_icon="📈",
    layout="wide",
    initial_sidebar_state="expanded",
)

# ------------------------------------------------------------------
# Module instances — created once, reused across reruns via cache
# ------------------------------------------------------------------
@st.cache_resource
def get_services():
    """
    Instantiate all service objects once per session.
    @st.cache_resource prevents re-initialising the LLM on every interaction.
    """
    return {
        "fetcher": StockFetcher(),
        "fundamentals": FundamentalsAnalyser(),
        "technicals": TechnicalsAnalyser(),
        "sentiment": SentimentAnalyser(),
        "llm": StockAnalystLLM(),
        "charts": StockVisualizer(),
    }


# ------------------------------------------------------------------
# Sidebar
# ------------------------------------------------------------------
def render_sidebar() -> str:
    """Render the sidebar and return the user's ticker input."""
    st.sidebar.title("📈 StockSense AI")
    st.sidebar.markdown("AI-powered stock analysis for retail investors.")
    st.sidebar.markdown("---")

    ticker = st.sidebar.text_input(
        "Enter a stock ticker",
        placeholder="e.g. AAPL, MSFT, TSLA",
        max_chars=10,
    ).strip().upper()

    st.sidebar.markdown("---")
    st.sidebar.markdown("**How it works:**")
    st.sidebar.markdown(
        "1. Enter a ticker\n"
        "2. Fundamentals + technical data is fetched\n"
        "3. News sentiment is analysed\n"
        "4. Llama 3.2 writes a synthesis report\n"
    )
    st.sidebar.markdown("---")
    st.sidebar.caption("⚠️ Not financial advice. For educational use only.")
    return ticker


# ------------------------------------------------------------------
# Section renderers
# ------------------------------------------------------------------
def render_fundamentals(svc: dict, fund) -> None:
    """Display fundamentals scorecard."""
    st.subheader("🏦 Fundamentals")

    verdict_color = {"Strong": "🟢", "Moderate": "🟡", "Weak": "🔴"}
    icon = verdict_color.get(fund.verdict, "⚪")

    col1, col2, col3, col4 = st.columns(4)
    col1.metric("Current Price", f"${fund.current_price:,.2f}" if fund.current_price else "N/A")
    col2.metric("P/E Ratio", f"{fund.pe_ratio:.2f}" if fund.pe_ratio else "N/A")
    col3.metric("EPS", f"{fund.eps:.2f}" if fund.eps else "N/A")
    col4.metric("Score", f"{fund.score}/100  {icon}")

    col5, col6, col7, col8 = st.columns(4)
    col5.metric("ROE", f"{fund.roe:.1f}%" if fund.roe else "N/A")
    col6.metric("Debt/Equity", f"{fund.debt_to_equity:.1f}" if fund.debt_to_equity else "N/A")
    col7.metric("Div. Yield", f"{fund.dividend_yield:.2f}%" if fund.dividend_yield else "N/A")
    col8.metric("52W High", f"${fund.fifty_two_week_high:,.2f}" if fund.fifty_two_week_high else "N/A")

    # Fundamentals bar chart
    chart_data = {k: v for k, v in {
        "P/E Ratio": fund.pe_ratio,
        "EPS ($)": fund.eps,
        "ROE (%)": fund.roe,
        "Debt/Equity": fund.debt_to_equity,
        "Div. Yield (%)": fund.dividend_yield,
    }.items() if v is not None}

    if chart_data:
        fig = svc["charts"].fundamentals_bar(chart_data)
        st.pyplot(fig)


def render_technicals(svc: dict, tech) -> None:
    """Display technical indicators and charts."""
    st.subheader("📊 Technical Analysis")

    if tech is None:
        st.warning("Not enough price history to compute technical indicators.")
        return

    col1, col2, col3 = st.columns(3)
    col1.metric("RSI (14)", f"{tech.rsi:.1f}" if tech.rsi else "N/A",
                delta=tech.rsi_signal)
    col2.metric("MACD Signal", tech.macd_signal_label)
    col3.metric("Current Price", f"${tech.current_price:,.2f}")

    tab1, tab2 = st.tabs(["Price & Bollinger Bands", "MACD"])
    with tab1:
        fig = svc["charts"].price_chart(tech)
        st.pyplot(fig)
    with tab2:
        fig = svc["charts"].macd_chart(tech)
        st.pyplot(fig)


def render_sentiment(sent) -> None:
    """Display news sentiment analysis."""
    st.subheader("📰 News Sentiment")

    verdict_color = {"Positive": "🟢", "Neutral": "🟡", "Negative": "🔴"}
    icon = verdict_color.get(sent.verdict, "⚪")

    col1, col2, col3 = st.columns(3)
    col1.metric("Sentiment", f"{icon} {sent.verdict}")
    col2.metric("Avg Polarity", f"{sent.average_polarity:+.3f}")
    col3.metric("Headlines Analysed", sent.headline_count)

    if sent.top_headlines:
        st.markdown("**Recent Headlines:**")
        for i, headline in enumerate(sent.top_headlines, 1):
            st.markdown(f"{i}. {headline}")


def render_llm_thesis(svc: dict, fund, tech, sent) -> None:
    """Run the LLM and stream the thesis into the UI."""
    st.subheader("🤖 AI Investment Thesis")
    st.caption("Generated by Llama 3.2 3B via Ollama — running 100% locally on your machine.")

    with st.spinner("Llama is thinking... (this takes 10–30 seconds)"):
        thesis = svc["llm"].synthesise(fund, tech, sent)

    st.markdown(thesis)


# ------------------------------------------------------------------
# Main app
# ------------------------------------------------------------------
def main() -> None:
    svc = get_services()
    ticker = render_sidebar()

    st.title("📈 StockSense AI")
    st.markdown("Enter a stock ticker in the sidebar to generate a full AI-powered analysis report.")

    if not ticker:
        st.info("👈 Enter a ticker symbol in the sidebar to get started.")
        return

    # Validate ticker length (basic guard)
    if len(ticker) > 6:
        st.error("That doesn't look like a valid ticker. Please enter 1–6 characters.")
        return

    with st.spinner(f"Fetching data for **{ticker}**..."):
        data = svc["fetcher"].fetch(ticker)

    if data is None:
        st.error(
            f"Could not find data for **{ticker}**. "
            "Check the ticker symbol and try again."
        )
        return

    # Company header
    company = data.info.get("longName", ticker)
    sector = data.info.get("sector", "")
    st.header(f"{company}  ({ticker})")
    if sector:
        st.caption(f"Sector: {sector}")

    st.markdown("---")

    # Run all analysis
    with st.spinner("Running analysis..."):
        fund = svc["fundamentals"].analyse(data)
        tech = svc["technicals"].analyse(data)
        sent = svc["sentiment"].analyse(data)

    # Render sections
    render_fundamentals(svc, fund)
    st.markdown("---")
    render_technicals(svc, tech)
    st.markdown("---")
    render_sentiment(sent)
    st.markdown("---")

    # LLM — behind a button so users choose when to wait
    if st.button("🤖 Generate AI Thesis", type="primary"):
        render_llm_thesis(svc, fund, tech, sent)


if __name__ == "__main__":
    main()
```

---

## 8. Phase 6 — Polish & Run

### 8.1 Run the App

Make sure Ollama is running (check by opening `http://localhost:11434` in your browser — you should see "Ollama is running").

Then in your project root:

```powershell
uv run streamlit run src/stocksense/main.py
```

Streamlit will print a local URL like `http://localhost:8501`. Open it in your browser.

Try a few tickers: `AAPL`, `MSFT`, `TSLA`, `NVDA`.

---

### 8.2 Run the Full Test Suite with Coverage

```powershell
uv run pytest tests/ -v --cov=src/stocksense --cov-report=term-missing
```

---

### 8.3 Lint and Format Your Code

Check for issues:

```powershell
uv run ruff check src/
```

Auto-fix safe issues:

```powershell
uv run ruff check src/ --fix
```

Format code:

```powershell
uv run ruff format src/
```

---

### 8.4 Write Your README.md

Replace the contents of `README.md` with the following (fill in your name):

```markdown
# StockSense AI

AI-powered stock analysis for retail investors.

## What it does
Enter any stock ticker and StockSense AI will:
- Fetch 6 months of price history and fundamentals from Yahoo Finance
- Compute RSI, MACD, and Bollinger Band technical indicators
- Score news headlines with NLP sentiment analysis
- Generate a plain-language investment thesis using Llama 3.2 (running 100% locally)
- Display interactive Seaborn/Matplotlib charts in a Streamlit web UI

## Setup

### Requirements
- Windows 11, Python 3.11
- [uv](https://astral.sh/uv) (package manager)
- [Ollama](https://ollama.com) with `llama3.2:3b` pulled

### Install
```powershell
uv sync
uv sync --extra dev
ollama pull llama3.2:3b
uv run python -c "import textblob; textblob.download_corpora()"
```

### Run
```powershell
uv run streamlit run src/stocksense/main.py
```

Open http://localhost:8501 in your browser.

## Tech Stack
| Layer | Tool |
|-------|------|
| Language | Python 3.11 |
| Package manager | uv |
| Data | yfinance (Yahoo Finance) |
| NLP Sentiment | TextBlob |
| LLM | Llama 3.2 3B via Ollama |
| Orchestration | LangChain |
| Charts | Seaborn + Matplotlib |
| UI | Streamlit |
| Testing | pytest |

## Disclaimer
For educational purposes only. Not financial advice.
```

---

### 8.5 Initialise Git and Push to GitHub

```powershell
git init
git add .
git commit -m "Initial commit: StockSense AI MVP"
```

Create a new **empty** repo on GitHub (no README, no .gitignore), then:

```powershell
git remote add origin https://github.com/YOUR_USERNAME/stocksense.git
git branch -M main
git push -u origin main
```

---

## 9. Architecture Diagram (Text)

Use this to draw your workflow diagram for the presentation:

```
┌─────────────────────────────────────────────────────────────────┐
│                      STREAMLIT WEB UI                           │
│              (User enters ticker, clicks buttons)               │
└──────────────────────────┬──────────────────────────────────────┘
                           │  ticker string
                           ▼
              ┌────────────────────────┐
              │      StockFetcher      │  ← yfinance (Yahoo Finance)
              │   data/fetcher.py      │     • info dict
              └────────────┬───────────┘     • 6-month OHLCV
                           │  StockData      • news list
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
 ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
 │ Fundamentals │  │  Technicals  │  │  Sentiment   │
 │  Analyser    │  │  Analyser    │  │  Analyser    │
 │              │  │ RSI/MACD/BB  │  │  TextBlob    │
 └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
        │                 │                  │
        ▼                 ▼                  ▼
  FundReport        TechReport          SentReport
        │                 │                  │
        └─────────────────┴─────────────────┘
                          │  Three reports
                          ▼
              ┌────────────────────────┐
              │    StockAnalystLLM     │  ← Ollama (local)
              │    LangChain chain     │     Llama 3.2 3B
              │    llm/analyst.py      │
              └────────────┬───────────┘
                           │  Thesis text
                           ▼
              ┌────────────────────────┐
              │    StockVisualizer     │  ← Seaborn + Matplotlib
              │   Seaborn/Matplotlib   │     Price chart
              │  charts/visualizer.py  │     MACD chart
              └────────────┬───────────┘     Fundamentals bar
                           │  Figures
                           ▼
              ┌────────────────────────┐
              │   Streamlit Display    │
              │   st.metric / st.pyplot│
              │   st.markdown (thesis) │
              └────────────────────────┘
```

---

## 10. SOLID Principles Applied

| Principle | Where it shows up in StockSense AI |
|---|---|
| **S** Single Responsibility | `fetcher.py` only fetches. `fundamentals.py` only scores. `visualizer.py` only draws. No module does two jobs. |
| **O** Open/Closed | Add a new indicator by adding a new `_add_*` method in `TechnicalsAnalyser` — existing code never changes. |
| **L** Liskov Substitution | All analysers receive a `StockData` object and return a dataclass. You could swap `TechnicalsAnalyser` for a subclass without changing `main.py`. |
| **I** Interface Segregation | `StockAnalystLLM.synthesise()` receives three small focused dataclasses, not one giant blob object. |
| **D** Dependency Inversion | `main.py` depends on abstract service objects instantiated in `get_services()`. Swap Ollama for a different LLM by changing one line inside `analyst.py`. |

---

## 11. Rubric Checklist

| Rubric Item | Status | Where |
|---|---|---|
| Proposal problem statement | ✅ | Your proposal doc |
| Tools & model chosen | ✅ | `pyproject.toml`, `analyst.py` |
| Application runs end-to-end | ✅ | `main.py` |
| LLM functionality implemented | ✅ | `llm/analyst.py` (Llama 3.2) |
| User input → LLM → output | ✅ | Ticker → thesis via Streamlit |
| Code readability & docs | ✅ | Docstrings on every class & method |
| Workflow diagram | ✅ | Section 9 above |
| Fundamentals analysis | ✅ | `analysis/fundamentals.py` |
| Technical indicators | ✅ | `analysis/technicals.py` |
| News sentiment | ✅ | `analysis/sentiment.py` |
| Charts / visualisation | ✅ | `charts/visualizer.py` (Seaborn) |
| Tests | ✅ | `tests/` folder |
| GitHub deployment | ✅ | Section 8.5 |
| No paid APIs required | ✅ | yfinance + TextBlob + Ollama = all free |
| README with instructions | ✅ | `README.md` |
