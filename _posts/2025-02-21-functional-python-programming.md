---
title: Functional Programming in Python and the Multiprocessing Module
description: short overview of functional programming techniques in python with a case study on how to use it with the multiprocessing tool in python 
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
•   Easier to read and maintain as code is more modular
•   Prevention of unwanted side effects as we use functions that don’t modify global state – making them easier to understand 
•   Easier to test
•   Easier to parallelize  tasks as tasks don’t share state between computations
•   More concise 

In this blog, we’ll explore:

•   How functional programming enables concurrency

•   Multiprocessing and threading in Python

•   A real-world case study: Log file analysis with multiprocessing

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

an I/O bound tasks is that which is limited by the speed of inputs and outputs to the processor rather than the speed of processing itself. 

# 5. Case Study: Log File Analysis with Multiprocessing

Many real-world applications involve processing large datasets. One example is analysing Apache log files in Common Log Format (CLF).  Apache log files are text files that record events and activities on an Apache web server, providing insights for monitoring server health, troubleshooting, and ensuring optimal performance and security. These logs are categorized into access logs (recording client requests) and error logs (recording server errors)

Why do we need to analyse CLF logs?

CLF is a widely used format for storing web server access logs. Analysing these logs helps track user behaviour, detect anomalies, and optimize server performance.

## Generating the log files:

Before we start, we need some data, the following repo has code which generates sample log file in our desired format.

https://github.com/UzerM0502/Fake-Apache-Log-Generator

here is a line of what the generated CLF looks like:

```bash
96.41.164.218 - - [25/Mar/2025:13:23:50 +0000] "DELETE /app/main/posts HTTP/1.0" 200 5012 "http://carroll.com/postsfaq.php" "Opera/8.98.(X11; Linux i686; brx-IN) Presto/2.9.173 Version/12.00"

```

## Log Analysis Pipeline

To process CLF logs efficiently, we break the task into smaller steps:

### Step 1: Reading Log Files
Use efficient file reading techniques like gzip.open() to handle compressed logs. In our case we can use a simple “with open …” statement in python which we will enclose in a function and it will yield the log files line by line. 

```python
def read_log_file(log_path: Path) -> Iterator[str]:
    """Read a .log file and yield lines."""
    with open(log_path, "r", encoding="utf-8") as log_file:
        for line in log_file:
            yield line.rstrip()

```

### Step 2: Converting Logs into Named Tuples

Named tuples provide an immutable data structure, ensuring safe concurrent processing. Named Tuples allow us to extract data from complex structures like the Apache CLF. The idea behind Named tuples is to create a class that is an immutable tuple with named attributes. Named Tuples are useful in functional programming as it can improve readability of the code by accessing values using descriptive fields rather than indexing elements in the tuple which usually do not provide context about the data we are extracting. Here is a simple example of named tuples so we can understand how it is used in the Apache CLF analyser. 

```python
from typing import NamedTuple 
 
class Point(NamedTuple): 
    latitude: float 
    longitude: float 
 
class Trip(NamedTuple): 
    start: Point
    end: Point
    distance: float

```
We can access the latitude using the descriptive naming method 

```python
trip = Trip(Point(1.0, 2.0), Point(3.0, 4.0), 5.0)
print('latitude',trip.start.latitude)
```
Which outputs: latitude 1.0

This is better than the original tuple which looks something like this:

```python
point = (1.0, 2.0)
point2 = (3.0, 4.0)
trip = (point, point, 5.0)
print('latitude', trip[0][0])
```

Although this has a correct output there is no way from face value to see that this is the latitude of the start point of the trip.

Also we should note that since it is a class, we can also define methods to compute derived values like so:
```python
Import math
class TripNew(NamedTuple):
    start: Point
    end: Point

    def distance(self) -> float:
        return math.sqrt(
            (self.end.latitude - self.start.latitude) ** 2 +
            (self.end.longitude - self.start.longitude) ** 2
        )
    
leg = TripNew(Point(1.0, 2.0), Point(3.0, 4.0))
print('latitude:', leg.start.latitude)
print('distance:', leg.distance())
```
So using this named tuple data structure we can extract our log files line by line into a class, lets call it Access and it will look like this:

```python
class Access(NamedTuple): 
    host: str 
    identity: str 
    user: str 
    time: str 
    request: str 
    status: str 
    bytes: str 
    referer: str 
    user_agent: str 
 
    @classmethod 
    def create(cls: type, line: str) -> Optional["Access"]: 
        format_pat = re.compile( 
            r"(?P<host>[\d\.]+)\s+" 
            r"(?P<identity>\S+)\s+" 
            r"(?P<user>\S+)\s+" 
            r"\[(?P<time>.+?)\]\s+" 
            r'"(?P<request>.+?)"\s+'
            r"(?P<status>\d+)\s+" 
            r"(?P<bytes>\S+)\s+" 
            r'"(?P<referer>.*?)"\s+'
            r'"(?P<user_agent>.+?)"\s*' 
            ) 
        if match := format_pat.match(line): 
            return cast(Access, cls(**match.groupdict())) 
        return None

```

The Access class creates various fileds for the log entry as well as the create method which creates the access class by parsing the raw string of the log. 

### Step 3: Extracting Additional Fields
Parsing and filtering specific fields for deeper analysis.

### Step 4: Filtering Unnecessary Paths
Excluding irrelevant files and directories to reduce processing time.

### Step 5: Identifying Books in Paths
Filtering logs to extract specific patterns based on file paths.

Our target code will look something like this 

```python
data = path_filter( 
    access_detail_iter( 
        access_iter( 
            local_gzip(filename))))

```

We can use a functional approach to 

