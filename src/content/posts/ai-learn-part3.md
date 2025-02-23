---
title: "AI learns - part 3"
description: "meta description"
date: 2025-02-23T05:00:00Z
image: "/images/posts/01.jpg"
categories: ["ai"]
authors: ["Balaji Balasundaram"]
tags: ["ai", "ai learns"]
draft: false
---
Let’s go through a simple **AI modeling example** using **supervised learning** to predict house prices.

---

### **Problem: Predicting House Prices**

Imagine you have a dataset of houses with the following information:

* **Size (sq. ft.)**  
* **Number of bedrooms**  
* **Location**  
* **Price**

We want to build an AI model that predicts the **price of a house** based on its features.

---

### **Step 1: Collect & Prepare Data**

We gather historical data, such as:

| Size (sq. ft.) | Bedrooms | Location Score | Price ($) |
| ----- | ----- | ----- | ----- |
| 1000 | 2 | 8 | 150,000 |
| 2000 | 3 | 9 | 250,000 |
| 1500 | 3 | 7 | 180,000 |
| 1800 | 4 | 8 | 220,000 |
| 2500 | 4 | 10 | 300,000 |

This is our **training data**.

---

### **Step 2: Choose a Model**

We use a **Linear Regression** model, which tries to find a mathematical equation:

*Price=(W1×Size)+(W2×Bedrooms)+(W3×Location)+b*


where **W1, W2, W3** are weights the AI learns, and **b** is a bias term.

---

### **Step 3: Train the Model**

The AI **adjusts the weights** by comparing predictions with actual prices and minimizing the error (using a method like Gradient Descent).

Example:

* AI predicts **$200,000**, but the actual price is **$220,000**.  
* It adjusts weights to reduce the difference (error).

After many iterations, the model finds the best weights.

---

### **Step 4: Test the Model**

We give the model **new data** it hasn’t seen before:

| Size (sq. ft.) | Bedrooms | Location Score | Predicted Price ($) |
| ----- | ----- | ----- | ----- |
| 1600 | 3 | 8 | **190,000** |
| 2100 | 4 | 9 | **260,000** |

If predictions are close to actual prices, the model is **well-trained**.

---

### **Step 5: Deploy & Improve**

* The model can now predict house prices for new listings.  
* More data and better training improve accuracy.

---

### **Conclusion**

This is a basic example of AI modeling using **supervised learning**. More advanced models, like **neural networks**, can handle complex patterns.


Here's a simple Python example using **Linear Regression** with `scikit-learn` to predict house prices.

---

### **1️⃣ Install Dependencies**

Make sure you have `scikit-learn` and `numpy` installed:

```bash
pip install scikit-learn numpy pandas matplotlib
```

---

### **2️⃣ Code for Training & Predicting House Prices**

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_absolute_error, mean_squared_error
```

#### 1️⃣ Sample Dataset

```python
data = {

    "Size": [1000, 2000, 1500, 1800, 2500, 3000, 1200, 2200, 2700],
    "Bedrooms": [2, 3, 3, 4, 4, 5, 2, 4, 5],
    "Location": [8, 9, 7, 8, 10, 10, 6, 9, 10],  # Assume higher is better
    "Price": [150000, 250000, 180000, 220000, 300000, 350000, 140000, 280000, 320000]
}

df = pd.DataFrame(data)
```

#### 2️⃣ Split Data (80% Training, 20% Testing)`

```python
X = df[["Size", "Bedrooms", "Location"]]  # Features`
y = df["Price"]  # Target variable`
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)`
```
#### 3️⃣ Train the Model`

```python
model = LinearRegression()
model.fit(X_train, y_train)
```

#### 4️⃣ Make Predictions`

```python
y_pred = model.predict(X_test)
```

#### 5️⃣ Evaluate the Model`
```python
print("Mean Absolute Error:", mean_absolute_error(y_test, y_pred))
print("Mean Squared Error:", mean_squared_error(y_test, y_pred))
```

#### 6️⃣ Display Results`

```python
plt.scatter(X_test["Size"], y_test, color="blue", label="Actual Prices")
plt.scatter(X_test["Size"], y_pred, color="red", label="Predicted Prices")
plt.xlabel("Size (sq. ft.)")
plt.ylabel("Price ($)")
plt.legend()`
plt.show()
```
---

### **🔍 How it Works**

1. We create a **dataset** with house sizes, bedrooms, location scores, and prices.  
2. We split the data into **training** (80%) and **testing** (20%).  
3. We train a **Linear Regression** model to learn from the training data.  
4. We **predict house prices** using test data.  
5. We **evaluate performance** using **Mean Absolute Error (MAE)** and **Mean Squared Error (MSE)**.  
6. We **visualize** actual vs. predicted prices.

---

### **📌 Sample Output**

```mathematica
Mean Absolute Error: 12000.50
Mean Squared Error: 250000000
```

A scatter plot will show how well predictions match actual prices.