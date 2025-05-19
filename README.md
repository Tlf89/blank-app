import streamlit as st
import yfinance as yf
import pandas as pd
import requests
from bs4 import BeautifulSoup
from datetime import datetime

st.set_page_config(page_title="📈 Daily Stock & News Tracker", layout="wide")

st.title("📈 Daily Stock & News Tracker")
st.markdown("Get real-time stock data and news updates for top companies.")

@st.cache_data(ttl=3600)
def get_stock_data(tickers):
    data = []
    for ticker in tickers:
        stock = yf.Ticker(ticker)
        info = stock.info
        data.append({
            "Ticker": ticker,
            "Name": info.get("shortName", "N/A"),
            "Price": info.get("currentPrice", "N/A"),
            "Change": info.get("regularMarketChange", "N/A"),
            "Change %": info.get("regularMarketChangePercent", "N/A"),
            "Market Cap": info.get("marketCap", "N/A")
        })
    return pd.DataFrame(data)

@st.cache_data(ttl=3600)
def get_news(ticker):
    search_url = f"https://www.google.com/search?q={ticker}+stock+news&tbm=nws"
    headers = {"User-Agent": "Mozilla/5.0"}
    response = requests.get(search_url, headers=headers)
    soup = BeautifulSoup(response.text, "html.parser")

    articles = []
    for item in soup.select("div.dbsr")[:5]:
        title = item.select_one("div.JheGif.nDgy9d").text
        link = item.a["href"]
        source = item.select_one("div.CEMjEf.NUnG9d").text if item.select_one("div.CEMjEf.NUnG9d") else "Unknown Source"
        articles.append({"title": title, "link": link, "source": source})
    return articles

tickers = ["AAPL", "MSFT", "GOOGL", "AMZN", "TSLA", "META", "NVDA"]
df = get_stock_data(tickers)

st.subheader("📊 Top Stocks")
st.dataframe(df, use_container_width=True)

selected_ticker = st.selectbox("🔍 Select a stock to view news:", tickers)

st.subheader(f"📰 News for {selected_ticker}")
news = get_news(selected_ticker)

for article in news:
    st.markdown(f"**[{article['title']}]({article['link']})**  \n*{article['source']}*")

st.caption(f"Last updated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
