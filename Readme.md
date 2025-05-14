## GoQuant Real-Time Trade Simulator

This repository contains a Python-based, Jupyter Notebook implementation of a real-time trade simulator for cryptocurrency spot trading on the OKX exchange. The simulator connects to a live Level-2 order book feed via WebSocket, processes tick-by-tick data, and computes key cost metrics, latency measurements, and trade profile predictions.

---

### 🎯 Project Objectives

1. **Stream Processing**: Ingest and parse full L2 order book data in real time.
2. **Cost Estimation**: Compute slippage, fees, market impact, and net cost for a user-specified trade size.
3. **Trade Profile**: Predict the maker vs. taker proportion of the trade.
4. **Performance Metrics**: Measure internal processing latency per tick.
5. **Interactive UI**: Provide live-updating widgets for inputs and outputs within Jupyter.
6. **Modular Code**: Clean architecture with clear separation between data ingestion, computation, and presentation.

---

## 🚀 Quick Start

### 1. Clone the Repo

```bash
git clone https://github.com/yourusername/goquant-simulator.git
cd goquant-simulator
```

### 2. Create & Activate Environment

```bash
python -m venv venv
source venv/bin/activate   # on Windows: venv\\Scripts\\activate
pip install -r requirements.txt
```

*(Alternatively, install directly in Jupyter:)*

```python
!pip install websockets pandas numpy scikit-learn ipywidgets nest_asyncio
```

### 3. Launch Jupyter Notebook

```bash
jupyter notebook
```

Open `Simulator.ipynb` and follow the instructions.

---

## 📂 Project Structure

```
goquant-simulator/
│
├── Simulator.ipynb        # Main Jupyter notebook with code and UI
├── requirements.txt       # Python dependencies
├── docs                  # Documentation (this README, write-up, etc.)
└── video_demo.mp4         # Sample recorded demo of the simulator
```

---

## 🔧 Detailed Setup & Usage

### 1. Prerequisites

* **Python 3.8+**
* **VPN access**: Required to connect to OKX WebSocket endpoints (no account needed).
* **Jupyter Notebook** environment

### 2. Installing Dependencies

Install core libraries:

```bash
pip install websockets pandas numpy scikit-learn ipywidgets nest_asyncio
```

Enable async support inside Jupyter:

```python
import nest_asyncio
nest_asyncio.apply()
```

### 3. Notebook Walkthrough

1. **Widget Setup**: Defines `ipywidgets` for user inputs (asset, quantity, volatility, fee tier) and output labels (mid-price, spread, slippage, fees, impact, net cost, maker %, latency).
2. **Helper Functions**:

   * `parse_orderbook(data)` → NumPy arrays of bids/asks
   * `compute_mid_and_spread(bids, asks)` → midpoint & spread
   * `expected_slippage(vol, qty)` → linear regression on dummy data
   * `expected_fees(qty, tier_str)` → rule-based fee calc
   * `market_impact(qty, vol)` → Almgren–Chriss stub
   * `maker_taker_pct(vol, qty)` → logistic regression on dummy data
3. **Async Loop** (`run_simulator()`):

   * Connects to `wss://ws.gomarket-cpp.goquant.io/ws/l2-orderbook/okx/BTC-USDT-SWAP`
   * Parses each JSON tick
   * Computes all metrics
   * Updates widget values
   * Measures and displays latency

Run the loop:

```python
await run_simulator()
```

---

## 🧠 Technical Explanation

### 1. Data Ingestion

* **Protocol**: WebSocket (`websockets` library).
* **Endpoint**: Full L2 order book for BTC-USDT-SWAP on OKX.
* **Message Handling**: `async for` loop processes each message as it arrives.

### 2. Core Computations

1. **Mid-Price & Spread**

   * Uses only the top-of-book to minimize processing time.
   * `mid = (best_bid + best_ask)/2` and `spread = best_ask - best_bid`.
2. **Slippage Model**

   * **Type**: Simple linear regression (scikit-learn).
   * **Training Data**: Dummy points mapping volatility → slippage %.
   * **Prediction**: Converts % slippage into USD cost.
3. **Fees**

   * Uses OKX’s maker/taker fee tiers (e.g., 0.02%, 0.05%) to calculate a flat fee cost.
4. **Market Impact**

   * **Model**: Almgren–Chriss impact formula stub: `impact = k * qty * vol`, with `k=1e-4`.
5. **Net Cost**

   * Sum of slippage, fees, and market impact.
6. **Maker/Taker Prediction**

   * **Model**: Logistic regression predicting probability of maker trades based on `[volatility, qty]` features.

### 3. Performance Metrics

* **Latency Measurement**: Uses `time.time()` before/after each tick’s processing to compute end-to-end latency in milliseconds.

---

## 📈 Performance Optimizations

* **NumPy Vectorization**: Avoid Python loops when converting and computing on order-book arrays.
* **Level-1 Only**: Restrict calculations to best bid/ask to reduce data volume.
* **Asynchronous I/O**: Non-blocking WebSocket listener to keep UI responsive.
* **Widget Updates**: Batch UI assignments to minimize redraw overhead.

---

## 🔮 Future Improvements

* **Real Model Training**: Replace dummy training data with historical backtest results.
* **Multi-Asset Support**: Parameterize WebSocket endpoint for arbitrary symbols.
* **Advanced Impact Model**: Full Almgren–Chriss with risk aversion and time slicing.
* **Parallel Processing**: Separate data ingest, computation, and UI threads/processes.
* **Dockerization**: Containerize environment for reproducible demos.

---

## 🎥 Demo Recording

A sample video (`video_demo.mp4`) walks through:

1. Notebook structure and markdown explanations.
2. Live data feed and widget interaction.
3. Commentary on model formulas and choices.
4. Observing latency and performance.

---

## 📄 Confidentiality Notice

This code and its outputs are confidential and intended solely for GoQuant’s recruitment process. Do not share publicly (e.g., GitHub, YouTube, blogs).

---

**Good luck, and happy trading simulation!**
