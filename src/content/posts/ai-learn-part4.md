---
title: "AI learns - part 4"
description: "meta description"
date: 2025-02-23T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["ai"]
authors: ["Balaji Balasundaram"]
tags: ["ai", "ai learns"]
draft: false
---
Great! Let's retrain the model with the expanded dataset to improve accuracy. 🚀

🔄 Steps to Retrain the Model
📌 1️⃣ Load & Prepare Data
First, load the updated dataset, clean it, and split it into training/testing sets.

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

# Load dataset (assuming you've expanded it)
df = pd.read_csv("house_prices.csv")  # Replace with actual dataset path

# Select features and target variable
X = df[["Size", "Bedrooms", "Location"]]  # Features
y = df["Price"]  # Target (house price)

# Train-test split (80% training, 20% testing)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Normalize features (StandardScaler ensures all features contribute equally)
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

📌 2️⃣ Train a Better Model
Since linear regression may not be enough, let's use a more powerful Random Forest Regressor.

```python
from sklearn.ensemble import RandomForestRegressor

# Train the model
model = RandomForestRegressor(n_estimators=200, max_depth=10, random_state=42)
model.fit(X_train_scaled, y_train)

# Make predictions
y_pred = model.predict(X_test_scaled)
```
📌 3️⃣ Evaluate Model Accuracy
To see if the model improved, check error metrics like MAE (Mean Absolute Error).

```python

from sklearn.metrics import mean_absolute_error, mean_squared_error

mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)

print(f"Mean Absolute Error (MAE): ${mae:.2f}")
print(f"Root Mean Squared Error (RMSE): ${rmse:.2f}")
```
📌 4️⃣ Visualize Predictions
Let's compare actual vs. predicted house prices.

```python

import matplotlib.pyplot as plt

plt.figure(figsize=(8, 5))
plt.scatter(y_test, y_pred, color="blue", alpha=0.6)
plt.plot([min(y_test), max(y_test)], [min(y_test), max(y_test)], color="red", linestyle="dashed")
plt.xlabel("Actual Prices ($)")
plt.ylabel("Predicted Prices ($)")
plt.title("Actual vs. Predicted House Prices")
plt.grid(True)
plt.show()
```
🚀 Next Steps
✔ Check if error is reduced (MAE, RMSE should be lower).
✔ If still high, tune hyperparameters (max_depth, n_estimators).
✔ If data is too small, consider more synthetic data.