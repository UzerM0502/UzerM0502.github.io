---
title: Functional Programming in Python and the Multiprocessing Module
description: 
author: Uzer
date: 2025-03-25 11:33:00 +0800
categories: [Coding, Python, Parallelism]
tags: [typography]
pin: false 
math: true
mermaid: true
---
# Introduction
Functional Programming is a different way of programming that emphasises the mapping of data from an input to output domain. It avoids mutable data structures and changing state. In functional programming the instructions are not commands they are expressions. In traditional OOP instructions are a concatenation of commands whereas in FP it can be seen as a composition of expressions. 

OOP example 

x = Object(y)
x.method_1()
x.method_2()

FP example 
x = func_2(func_1(y))

the main benefits of using a functional paradigm over an object-oriented approach are:
•	Easier to read and maintain as code is more modular
•	Prevention of unwanted side effects as we use functions that don’t modify global state – making them easier to understand 
•	Easier to test
•	Easier to parallelize  tasks as tasks don’t share state between computations
•	More concise 

In this blog, we’ll explore:

•	How functional programming enables concurrency

•	Multiprocessing and threading in Python

•	A real-world case study: Log file analysis with multiprocessing

# Functional Programming and Concurrency

Functional programming encourages the use of immutable data and stateless functions. By avoiding shared state, we reduce dependencies between different parts of a program, making it easier to execute tasks concurrently without the risk of unexpected side effects.

## Concurrency vs. Parallelism

Concurrency and parallelism are often confused, but they address different problems:

1. Concurrency allows a program to manage multiple tasks at once, improving responsiveness.

2. Parallelism executes tasks simultaneously, often across multiple CPU cores.

# Managing Threads and Shared State 

In Python, multiple threads run within the same process and share memory. While this improves efficiency, it also introduces the risk of race conditions, where multiple threads modify shared data unpredictably.

Python’s Global Interpreter Lock (GIL) prevents multiple threads from executing Python bytecode simultaneously, ensuring data integrity but limiting true parallelism in CPU-bound tasks. One way to prevent conflicts in multithreading is locking, which restricts access to shared resources when necessary.

Concurrency is especially useful for I/O-bound problems, such as:

1. Reading/writing files
2. Handling network requests
3. Web scraping

an I/O bound tasks is that which is limited by the speed of inputs and outputs to the processor rather than the speed of processing it self. 

# 5. Case Study: Log File Analysis with Multiprocessing

Many real-world applications involve processing large datasets. One example is analyzing Apache log files in Common Log Format (CLF).

Why do we need ot analyse CLF logs?

CLF is a widely used format for storing web server access logs. Analysing these logs helps track user behavior, detect anomalies, and optimize server performance.

## Generating the log files:

Before we start, we need some data, the following repo has code which generates sample log file in our desired format.

https://github.com/UzerM0502/Fake-Apache-Log-Generator


## Log Analysis Pipeline

To process CLF logs efficiently, we break the task into smaller steps:

Step 1: Reading Log Files
Use efficient file reading techniques like gzip.open() to handle compressed logs.

Step 2: Converting Logs into Named Tuples
Named tuples provide an immutable data structure, ensuring safe concurrent processing.

Step 3: Extracting Additional Fields
Parsing and filtering specific fields for deeper analysis.

Step 4: Filtering Unnecessary Paths
Excluding irrelevant files and directories to reduce processing time.

Step 5: Identifying Books in Paths
Filtering logs to extract specific patterns based on file paths.

Our target code will look something like this 

```python
data = path_filter( 
    access_detail_iter( 
        access_iter( 
            local_gzip(filename))))

```

We can use a functional approach to 