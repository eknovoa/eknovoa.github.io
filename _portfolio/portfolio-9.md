---
title: "Coffee Sales Data Cleaning & Exploration Project"
excerpt: "<img src='https://images.unsplash.com/photo-1495474472287-4d71bcdd2085?q=80&w=1740&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D'>"
collection: portfolio
---

**Project Overview**

In this SQL project, I practiced creating a database and a table using __pgAdmin 4__ and imported a csv file. I then made a backup copy of the table, viewed the data and cleaned several columns. After the data was ready to work with, I began analyzing fictitious cafe sales data to uncover performance metrics.

Data Source: Kaggle Dataset, found [here](https://www.kaggle.com/datasets/ahmedmohamed2003/cafe-sales-dirty-data-for-cleaning-training)

You can view the full project [here](https://github.com/eknovoa/SQL/blob/main/coffee_sales_project.sql).

<br>
<br>
**About the Data**

Number of Rows: 10,000</br>
Number of Columns: 8

The dataset contains synthetic data representing sales transactions in a cafe. This dataset is intentionally "dirty," with missing values, inconsistent data, and errors introduced to provide a realistic scenario for data cleaning and exploratory data analysis (EDA).

| Column Name      | Description                            |
| ---------------- | -------------------------------------- |
| transaction_id   | unique identifier for each transaction |
| item             | name of item purchased                 |
| quantity         | quantity of item purchased             |
| price_per_unit   | price of a single unit of the item     |
| total_spent      | total amount spent on transaction      |
| payment_method   | the method of payment used             |
| location         | location where transaction occurred    |
| transaction_date | the date of the transaction            |


**Learning Objectives**

- Practice DDL (Data Definition Language) SQL commands: CREATE and ALTER
- Practice DML (Data Manipulation Language) SQL commands: COPY and UPDATE
- Practice DQL (Data Query Language) SQL commands: SELECT, FROM, WHERE, GROUP BY, HAVING, DISTINCT, ORDER BY, LIMIT, CTEs, and Window Functions

**Cleaning Tasks**

- Updated the 'item', 'quantity', 'price_per_unit', 'total_spent', 'payment_method', 'location', and 'transaction_date' columns to correctly represent null values in SQL
- Changed the 'quantity' column data type to be SMALLINT
- Changed the 'price_per_unit' column data type to be MONEY
- Changed the 'total_spent' column data type to be MONEY
- Changed the 'transaction_date' column data type to be DATE
- Added a new 'category' column with VARCHAR(255) data type for menu items: Beverages, Sandwiches & Salads, Pastries
- Updated the coffee_sales table to set the category to be equal to 'Beverages' if the item value is either Juice, Coffee, Smoothie, or Tea
- Updated the coffee_sales table to set the category to be equal to 'Sandwiches & Salads' if the item value is either Salad or Sandwich
- Updated the coffee_sales table to set the category to be equal to 'Pastries' if the item value is either Cake or Cookie


**Practice Questions**

- Which transactions with the total amount spent were equivalent to the highest amount spent across all transactions?
- What is the most common payment method used at the cafe?
- Which month had the highest sales?
- Were there more takeaway orders or in-store?
- Did in-store or takeaway orders have the highest sales?
- Which item sold the most?
- Which category had the highest sales?
- What was the average total spent amongst all transactions?
- What was the sales performance by month and the net change from previous month?
- Which months had a positive net change in sales?
- What were the most popular items? Display their ranks.
- What was the total sales by month? Compare to the rolling 3-month average of sales to smooth out data spikes.

