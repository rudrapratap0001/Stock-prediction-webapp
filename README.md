import streamlit as st
import yfinance as yf
import pandas as pd
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split
import matplotlib.pyplot as plt

st.set_page_config(page_title="Stock Price Predictor", layout="centered")
st.title("📈 Stock Price Prediction App using Linear Regression")

# --- User Inputs ---
stock_symbol = st.text_input("Enter Stock Symbol (e.g., TATAMOTORS.NS, AAPL)", "TATAMOTORS.NS")
start_date = st.date_input("Start Date", pd.to_datetime("2022-01-01"))
end_date = st.date_input("End Date", pd.to_datetime("2024-12-31"))
future_days = st.slider("Days to Predict", min_value=7, max_value=60, value=30)

if st.button("Predict"):
    try:
        # Load data
        stock = yf.download(stock_symbol, start=start_date, end=end_date)
        stock = stock[['Close']].dropna()
        stock['Prediction'] = stock[['Close']].shift(-future_days)

        # Prepare data
        X = stock[['Close']][:-future_days]
        y = stock['Prediction'][:-future_days]
        x_future = stock[['Close']][-future_days:]

        # Split and train
        X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
        model = LinearRegression()
        model.fit(X_train, y_train)

        # Make predictions
        predictions = model.predict(x_future)

        # Prepare dates
        future_dates = pd.date_range(start=stock.index[-1], periods=future_days + 1, freq='B')[1:]
        predicted_df = pd.DataFrame({'Date': future_dates, 'Predicted Price': predictions})

        # Plot
        fig, ax = plt.subplots(figsize=(10, 5))
        ax.plot(stock.index[-60:], stock['Close'][-60:], label="Actual Price")
        ax.plot(predicted_df['Date'], predicted_df['Predicted Price'], label="Predicted Price", linestyle='--')
        ax.set_xlabel("Date")
        ax.set_ylabel("Stock Price")
        ax.set_title(f"Predicted Stock Prices for {stock_symbol}")
        ax.legend()
        st.pyplot(fig)

        # Show data
        st.subheader("📊 Predicted Prices")
        st.dataframe(predicted_df.set_index('Date'))

    except Exception as e:
        st.error(f"Error: {str(e)}")
