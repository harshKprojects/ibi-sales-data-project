# ibi-sales-data-project
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
import random

np.random.seed(42)

## --- 1. Generate Product Catalog ---
products = {
    "P001": {"name": "Classic White Shirt", "category": "Shirts", "price": 24.99},
    "P002": {"name": "Slim Fit Jeans", "category": "Pants", "price": 49.99},
    "P003": {"name": "Leather Belt", "category": "Accessories", "price": 19.99},
    "P004": {"name": "Cotton Socks 3-Pack", "category": "Underwear", "price": 12.99},
    "P005": {"name": "Wool Winter Coat", "category": "Outerwear", "price": 129.99},
    "P006": {"name": "Running Sneakers", "category": "Footwear", "price": 79.99},
    "P007": {"name": "Baseball Cap", "category": "Accessories", "price": 14.99},
    "P008": {"name": "Denim Jacket", "category": "Outerwear", "price": 59.99}
}

## --- 2. 50 Customers with Demographics ---
customers = []
for i in range(1, 51):
    gender = random.choice(["M", "F"])
    if gender == "M":
        name = random.choice(["James", "John", "Robert", "Michael", "William"])
    else:
        name = random.choice(["Mary", "Patricia", "Jennifer", "Linda", "Elizabeth"])
    
    customers.append({
        "customer_id": f"C{1000 + i}",
        "name": name,
        "gender": gender,
        "age": random.randint(18, 70),
        "join_date": (datetime.now() - timedelta(days=random.randint(30, 1000))).strftime("%Y-%m-%d")
    })

## --- 3. 500 Random Transactions ---
transactions = []
start_date = datetime(2023, 1, 1)
end_date = datetime(2024, 1, 1)

for _ in range(500):
    customer = random.choice(customers)
    product_id, product = random.choice(list(products.items()))
    
    # Simulate seasonal buying patterns (more coats in winter, etc.)
    transaction_date = start_date + timedelta(days=random.randint(0, 365))
    month = transaction_date.month
    
    # Adjust quantities randomly
    if product["category"] == "Outerwear" and month in [11, 12, 1]:
        quantity = random.choices([1, 2, 3], weights=[0.6, 0.3, 0.1])[0]
    else:
        quantity = random.choices([1, 2], weights=[0.8, 0.2])[0]
    
    transactions.append({
        "transaction_id": f"T{random.randint(10000, 99999)}",
        "customer_id": customer["customer_id"],
        "product_id": product_id,
        "product_name": product["name"],
        "category": product["category"],
        "price": product["price"],
        "quantity": quantity,
        "t_dat": transaction_date.strftime("%Y-%m-%d"),
        "sales_channel": random.choice(["Online", "Store"])
    })

## --- 4. DataFrames ---
df_customers = pd.DataFrame(customers)
df_transactions = pd.DataFrame(transactions)

for _ in range(20):
    idx = random.randint(0, len(df_transactions)-1)
    df_transactions.at[idx, 'quantity'] = -1  # Negative quantity = return

# missing data
df_transactions.loc[random.sample(range(len(df_transactions)), 10), 'sales_channel'] = np.nan

## --- 6. Save Sample Data ---

print("=== Sample Transaction Data ===")
print(df_transactions.head())
print("\n=== Sample Customer Data ===")
print(df_customers.head())

# Calculate total sales per product
print("\n=== TOP SELLING PRODUCTS ===")
product_sales = df_transactions.groupby('product_name')['quantity'].sum().sort_values(ascending=False)
print(product_sales.head())

# Monthly sales trend
print("\n=== MONTHLY SALES ===")
df_transactions['month'] = pd.to_datetime(df_transactions['t_dat']).dt.month_name()
monthly_sales = df_transactions.groupby('month')['price'].sum()
print(monthly_sales)

# Customer segmentation
print("\n=== CUSTOMER SPENDING ===")
customer_value = df_transactions.groupby('customer_id').agg(
    total_spent=('price', 'sum'),
    purchase_count=('transaction_id', 'count')
).sort_values('total_spent', ascending=False)
print(customer_value.head())
