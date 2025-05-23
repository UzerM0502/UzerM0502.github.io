---
title: Test-Driven Refactoring Migrating a Pandas Application to PySpark
description: How to safely refactor pandas to pyspark for big data application 
author: Uzer
date: 2025-04-25 11:33:00 +0800
categories: [Coding, Python, Big Data]
tags: []
pin: false 
math: true
mermaid: true
---
# Introduction

## Why may you need to go from pandas to Pyspark in the first place?

You may have been tasked by your organisation to make the leap due to a massive increase in data volume, or maybe this is your personal interest and you would like to do this for fun. Whatever the reason is, this blog will show an effective way to transition from pandas to Pyspark.


## What is TDD?

Test-Driven Development (TDD) is a software development approach where developers write automated tests before writing the actual code for a feature. This iterative process involves writing a test that initially fails, then writing just enough code to make the test pass, and finally refactoring both the test and the production code. 

## What about Test-Driven Refactoring??

In this blog we will be looking at Test-Driven REFACTORING (TDR) which is slightly different as we have an existing code base to refactor rather than new features to develop. We have a pandas application which will read in raw data (bronze tier) from an external location (like some cloud location or a database) and will conduct joins and transformations and eventually output a final dataset (silver tier) which can be used for some potential analytics/predictive modelling. 

# Overview of the strategy I used:

1.	Write Capture tests for legacy code 
2.	Make incremental changes to methods to make them static 
3.	Begin to rewrite code emphasising more functional approach 
4.	Change API from pandas to Pyspark 
5.	Run tests on Pyspark methods and ensure they pass
6.	Integrate refactored  classes together and methods together using appropriate interfaces 



# Creating a Safety Net: Testing Legacy Code

I captured the output of the methods by running .to_pickle at each point of interest. This would be in my case after the method was run, therefore capturing the output of the method. Ensuring granular test coverage of the legacy code is vital as it will allow me a safety net to make changes to the code in the future. To create tests we would have to set the internal state of the object manually, to the that which is of before the method is run, then run the method and extract the result. Finally we will assert the DataFrame with that of the captured pickle. Here is a short example:

```python
processor = DataProcessor(customers, orders, products)

# After first merge
processor.merge_customer_orders()
processor .df1.to_pickle("tests/data/after_merge_customer_orders.pkl")

# After second merge
processor.merge_with_products()
processor .df2.to_pickle("tests/data/after_merge_with_products.pkl")
```

```python
def test_merge_with_products_matches_snapshot(processor):
    # Set state to just after merge_customer_orders
    processor.merged_df = pd.read_pickle("tests/data/after_merge_customer_orders.pkl")
    
    processor.merge_with_products()
result = processor.df2
    expected = pd.read_pickle("tests/data/after_merge_with_products.pkl")
    
    pd.testing.assert_frame_equal(result, expected)

```
Now that we have the tests for these methods – we can safely change the code and we would know if we have broken the exisiting system!

# Making Code easier to test 
•	Static methods: why they helped (less reliance on state).
•	The role of the PyCharm debugger in understanding execution flows and method dependencies.
•	Highlight the importance of identifying side effects and external dependencies early.

As mentioned in the previous section we need to manually set the objects state so we can perform the method and ensure it gives us the correct output, but this can be difficult as the attributes inside the objects change over time due to method calls. 
This ends up meaning that state is not always visible, and that the same method will behave differently depending on the state of the application before the method call. Here the PyCharm debugger can aid us as it will allow us to pause execution and allow us to inspect the variables/object attributes and move through the code step by step to see how the variables/state is changing. Knowing this we can “see” inside the object before a method call and therefore set the state of our object in our test correctly!

In our example in the previous section we can see that the methods do not take in any inputs or return any outputs, therefore they are just changing the state of the application which can make it hard to refactor. Therefore we need to rewrite some of our code to make these methods into static methods so that they have no reliance on the state of the application. This will be helpful as pure functions become much easier to test. In the snippet below we can see how code was changed from stateful to a pure static method 

Before 
```python
class DataProcessor:
    def __init__(self, customers_df, orders_df, products_df):
        self.customers = customers_df
        self.orders = orders_df
        self.products = products_df
        self.df1 = None

    def merge_customer_orders(self):
        self.df1 = pd.merge(self.orders, self.customers, on='customer_id', how='left')
        self.df1['order_date'] = pd.to_datetime(self.df1['order_date'])
        self.df1['order_month'] = self.df1['order_date'].dt.to_period('M')

```
Relies on internal state (self.customers, self.orders)
Stores result internally (self.df1)
Harder to test — requires setting up full object state

After 

```python
class DataProcessor:
    @staticmethod
    def merge_customer_orders(customers, orders):
        df = pd.merge(orders, customers, on='customer_id', how='left')
        df['order_date'] = pd.to_datetime(df['order_date'])
        df['order_month'] = df['order_date'].dt.to_period('M')
        return df

```
Takes all input as parameters
Returns result directly — no side effects
Easy to test, reuse, and reason about independently of class state


Since our function signature has changed we have to tweak our tests too:

```python

def test_merge_with_products_matches_snapshot():
    df1 = pd.read_pickle("tests/data/after_merge_customer_orders.pkl")
    expected = pd.read_pickle("tests/data/after_merge_with_products.pkl")

    result = DataProcessor.merge_with_products(df1, products)
    
    pd.testing.assert_frame_equal(result, expected)

```

# Incremental Refactoring into PySpark

Our refactored application was to run in databricks so I decided to my refactoring in the notebook environment. Also notebooks allow for experimentation with inputs and outputs as well as easy DataFrame visualisation. Referring back to our example – to go from pandas to Pyspark we would need to isolate the function (which we did by making it static) then start swapping out the pandas syntax for the approprtiate one in pysaprk:


- Replace pd.merge with join.

- Use PySpark’s withColumn, to_date, and date_format functions.

- Return a DataFrame transformation that works lazily in Spark.

We get:

```python 

from pyspark.sql import functions as F

class DataProcessor:
    @staticmethod
    def merge_customer_orders(customers_df, orders_df):
        df = orders_df.join(customers_df, on='customer_id', how='left')

        # Convert string to date
        df = df.withColumn("order_date", F.to_date("order_date"))

        # Extract year-month as string (like '2024-06')
        df = df.withColumn("order_month", F.date_format("order_date", "yyyy-MM"))

        return df

```

# Maintaining Parity Between Pandas and Pyspark
Since we changed our source code to Pyspark we need to change the implementation of our unit tests too so that we can make sure our code is still working as expected. This can be a simple change as reading the pickle file into pandas then into spark DataFrame and passing it into our new test, then calling our refactored (Pyspark) method then converting output back to pandas DataFrame to assert our result and expected DataFrame. This is best shown in an example:

```python

from pyspark.sql import SparkSession
import pandas as pd
import pytest
from my_module import DataProcessor  # Adjust this to your real module

spark = SparkSession.builder.getOrCreate()

def test_merge_with_products_matches_snapshot():
    # Load the pickle files as Pandas DataFrames
    df1_pandas = pd.read_pickle("tests/data/after_merge_customer_orders.pkl")
    expected_pandas = pd.read_pickle("tests/data/after_merge_with_products.pkl")

    # Convert Pandas to Spark DataFrames
    df1_spark = spark.createDataFrame(df1_pandas)
    products_spark = spark.createDataFrame(products)  # Assuming `products` is already available as a pandas DataFrame

    # Call the PySpark refactored method
    result_spark = DataProcessor.merge_with_products(df1_spark, products_spark)

    # Convert result back to Pandas for comparison
    result_pandas = result_spark.toPandas()

    # Reset index if necessary to avoid mismatches
    result_pandas = result_pandas.reset_index(drop=True)
    expected_pandas = expected_pandas.reset_index(drop=True)

    # Assert
    pd.testing.assert_frame_equal(result_pandas, expected_pandas)

```

You could also build some sort of helper function like ```read_pickle_as_spark(path)``` to help deal with schema errors or empty dataframes etc. 

Now that you have all your refactored tests in place and your legacy code is modularised and dependencies are cut, you can safely refactor into Pyspark API to scale your application to bigger data!

# Some Challenges I faced

When I began refactoring, I was unaware of the interconnected nature of the source code I was dealing and this meant I had to refactor methods in other classes to be compatible with the code changes I made. This deep coupling of the software and particularly in OO design increased the radius and scope of the work I had to do and therefore the amount of time I spent on this was a little longer than originally planned 



