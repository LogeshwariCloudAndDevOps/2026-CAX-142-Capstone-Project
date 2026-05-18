**StockSense AI**

### What's in the Proposal

**Problem Statement (Section 02)**
The core argument: retail investors lack the tools, expertise, and data pipelines to perform rigorous stock analysis, leading to emotionally-driven, uninformed trades. Three sharp pain points are called out — information overload, the domain expertise gap, and emotional bias — backed by illustrative statistics.

**The AI Solution (Section 03)**
A 6-step AI Agent pipeline is the heart of the proposal:
1. User inputs a ticker → system resolves metadata
2. **Fundamental Analysis Module** — financial ratios, DCF, peer comparison
3. **Technical Analysis Module** — indicators (RSI, MACD, Bollinger Bands) + ML chart pattern classification
4. **Sentiment & News Intelligence** — NLP scoring on real-time news
5. **LLM Synthesis** (Claude API) — resolves conflicting signals, generates investment thesis
6. **Structured Report Output** — Bull/Neutral/Bear score with plain-language explanation

**Key Features (Section 04)**
Six capabilities: Fundamental Scorecard, Technical Chart Analysis, Conversational AI Analyst, News Sentiment Engine, Portfolio Fit Analyser, and Investment Thesis Generator.

**Tech Stack (Section 05)**
React frontend · FastAPI backend · Claude API + LangChain for AI orchestration · Alpha Vantage / Yahoo Finance / SEC EDGAR for data · HuggingFace for NLP sentiment.

**Ethics & Risk (Section 07)**
Addresses hallucination risk, financial advice disclaimers, data privacy, model bias, and API resilience — important for any finance AI project.

**Success Metrics (Section 08)**
>80% user comprehension improvement, <8s report generation time, ≥70% backtesting hit rate on "Buy" signals.