# Install required libraries
# pip install streamlit pandas numpy ta yfinance

import streamlit as st
import pandas as pd
import numpy as np
import ta
import yfinance as yf

# Streamlit App Configuration
st.title("Trading Signal Dashboard")
st.sidebar.header("Settings")

# User Inputs
ticker = st.sidebar.text_input("Enter Stock/Asset Symbol", value="EURUSD=X")
timeframe = st.sidebar.selectbox("Select Timeframe", options=["5m", "15m", "1h", "1d"], index=0)
indicator = st.sidebar.selectbox("Select Indicator", options=["RSI", "MACD", "Bollinger Bands"], index=0)

# Fetch Live Data
@st.cache
def fetch_data(ticker):
    interval_map = {"5m": "5m", "15m": "15m", "1h": "1h", "1d": "1d"}
    data = yf.download(tickers=ticker, interval=interval_map[timeframe], period="7d")
    data.reset_index(inplace=True)
    return data

data = fetch_data(ticker)

# Apply Indicators
if indicator == "RSI":
    data["RSI"] = ta.momentum.rsi(data["Close"], window=14)
    st.line_chart(data[["Close", "RSI"]])
    st.write("Signals: Buy if RSI < 30, Sell if RSI > 70")
elif indicator == "MACD":
    macd = ta.trend.MACD(data["Close"])
    data["MACD"] = macd.macd()
    data["Signal Line"] = macd.macd_signal()
    st.line_chart(data[["Close", "MACD", "Signal Line"]])
    st.write("Signals: Buy if MACD crosses above Signal Line, Sell if MACD crosses below")
elif indicator == "Bollinger Bands":
    bb = ta.volatility.BollingerBands(data["Close"], window=20)
    data["Upper Band"] = bb.bollinger_hband()
    data["Lower Band"] = bb.bollinger_lband()
    st.line_chart(data[["Close", "Upper Band", "Lower Band"]])
    st.write("Signals: Buy if Close is near Lower Band, Sell if Close is near Upper Band")

# Signal Output
st.subheader("Signal Recommendations")
if indicator == "RSI":
    data["Signal"] = np.where(data["RSI"] < 30, "Buy", np.where(data["RSI"] > 70, "Sell", "Hold"))
elif indicator == "MACD":
    data["Signal"] = np.where(data["MACD"] > data["Signal Line"], "Buy", "Sell")
elif indicator == "Bollinger Bands":
    data["Signal"] = np.where(data["Close"] < data["Lower Band"], "Buy", np.where(data["Close"] > data["Upper Band"], "Sell", "Hold"))
st.write(data[["Datetime", "Close", "Signal"]].tail(10))
