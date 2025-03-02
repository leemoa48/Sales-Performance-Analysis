# ETL Process: From Python (Pandas) to MySQL to Power BI

![Screenshot 2025-03-02 151814](https://github.com/user-attachments/assets/0885667c-3a88-4732-add4-2805bceda68a)


## Overview
This project demonstrates an **ETL (Extract, Transform, Load) pipeline** that:
- **Extracts** sales data using Python and Pandas.
- **Transforms** the data by cleaning, normalizing, and structuring it into a star schema.
- **Loads** the processed data into a **MySQL database**.
- **Visualizes** insights using **Power BI**.

## Technologies Used
- **Python (Pandas, SQLAlchemy, Seaborn, Matplotlib)** for data processing.
- **MySQL** for structured data storage.
- **Power BI** for data visualization.

## ETL Process
### 1. Extract Data
We start by importing a CSV file containing sales transactions:
```python
import pandas as pd

df = pd.read_csv("Sales Trans.csv")
df.head()
```
### 2. Transform Data
#### Data Cleaning and Renaming
```python
df = df.rename(columns={
    'TransactionNo': 'transaction_id',
    'CustomerNo': 'customer_id',
    'ProductNo': 'product_id',
    'Quantity': 'quantity',
    'Price': 'price',
    'Date': 'date',
    'Country': 'country',
    'ProductName': 'product'
})
df.drop_duplicates(inplace=True)
df.dropna(subset=['customer_id'], inplace=True)
```
#### Date Processing
```python
df['date'] = pd.to_datetime(df['date'], format='%m/%d/%Y')
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month
df['quarter'] = df['date'].dt.quarter
df['day'] = df['date'].dt.day
```
#### Creating Dimension Tables
```python
# Creating unique key mappings for dimensions
dim_customer = df[['customer_id']].drop_duplicates().reset_index(drop=True)
dim_customer['customer_key'] = dim_customer.index

# Similar transformations for product, date, transaction, and country dimensions
```
#### Creating Fact Table
```python
fact_sales = df[['transaction_id', 'date', 'product_id', 'customer_id', 'country', 'quantity']]
fact_sales = fact_sales.merge(dim_customer, on='customer_id')
```
### 3. Load Data into MySQL
Using **SQLAlchemy** to push data into a MySQL database:
```python
from sqlalchemy import create_engine, text
engine = create_engine('mysql+mysqldb://root:password@localhost:3306/sales_db')

# Load each dimension and fact table
dim_customer.to_sql('dim_customer', con=engine, if_exists='replace', index=False)
engine.execute(text("ALTER TABLE dim_customer ADD PRIMARY KEY (customer_key);"))
```

![Screenshot 2025-03-02 135522](https://github.com/user-attachments/assets/7f9b327b-9ca1-4902-ab3c-c78f7bdcaf1a)


## Power BI Visualization
Once data is stored in **MySQL**, Power BI connects to it for interactive analysis.

### **Key Insights from Power BI Dashboard**
1. **Total Sales:** **$604.47K** (144.94% increase from previous year)
2. **Total Transactions:** **1617** (-42.54% from previous year)
3. **Average Quantity per Transaction:** **7.98** (+16.05%)
4. **Sales Trend:**
   - Highest sales in **November 2019**.
   - Steady growth with some dips mid-year.
5. **Top Selling Products:**
   - **Popcorn Holder**: **$0.58M**.
   - **World War 2 Glasses**: **$0.56M**.
6. **Best Sales Days:**
   - **Sunday and Friday** had peak sales (~13M and 12M respectively).
7. **Sales by Region:**
   - Strongest in **Europe** and **North America**.
8. **Best Performing Month:**
   - **December 2019** had the highest sales at **$6.21M**.

## Future Enhancements
- Automate the ETL pipeline with **Apache Airflow** or **Prefect**.
- Add **machine learning** to predict future sales trends.
- Enhance **Power BI** with real-time updates from MySQL.

## Conclusion
This project provides an end-to-end **ETL pipeline** that successfully extracts, transforms, and loads sales data into MySQL, allowing businesses to analyze trends using Power BI.

