---
title: Text and Typography
description: Examples of text, typography, math equations, diagrams, flowcharts, pictures, videos, and more.
author: cotes
date: 2024-12-21 11:33:00 +0800
categories: [Blogging, Demo]
tags: [typography]
pin: true 
math: true
mermaid: true
---
# Code Quality Blog

## Code Quality Tools in Python

Our team is currently working on a legacy app which delivers fully automated front-to-back data quality, monitoring, and assurance of trade-level data with the help of machine learning algorithms. The current architecture of the app is unscalable, and the business requires us to widen the scope of the app by onboarding new risk management systems and expanding geographical coverage. We are therefore refactoring the code to meet business requirements and thus use code quality tools. This short post explains some popular Python code quality tools, what benefits they provide, and examples of use.

### What Do We Mean by Good Code Quality?

In short, it means meeting the project specification and requirements, but more generally, having readability, maintainability, and adherence to standards such as PEP8 for Python. PEP8 is a document that provides guidance on how to best write Python code.

The tools we are covering are:

1. **Pylint** - Simple code analysis  
2. **Black** - Code style formatting  
3. **Flake8** - Standard for Python  

---

## Pylint

Pylint is a static code analyzer that doesn’t require you to actually run your code. It helps with bug detection and adherence to standards like PEP8. The main problem Pylint solves is bad “code smells,” meaning aspects of code that indicate potential errors, such as unused imports which are not ideal as they can cause naming conflicts and clutter your code.

### Example: Before Using Pylint

```python
import math
import random  # Unused import
def CircleArea(Radius):
    pi = math.pi  # Unused variable
    diameter = 2 * Radius  # Unused variable    
    if Radius <= 0:
        print("Invalid radius!")  # Bad practice to print errors directly
        return None
    return 3.14159 * Radius * Radius
```

## Black

Black is a Python code formatter that allows teams of developers to have a uniform and consistent coding style across the codebase. The main benefit is to eliminate inconsistent code formatting provided by other tools, which have endless customization. This emphasis on uniformity can aid in readability and end disputes over formatting. 

To format this code using Black, we can simply run the following command in the terminal:

```bash
python -m black \file_before.py
```
## Example of Black in Action
### Before:
```python 
def greet(name):
    print("Hello, " + name + "!")
```
### After:
```python 
def greet(name):
    print(f"Hello, {name}!")
```
## Flake8

The last tool we will discuss is Flake8, which combines a few tools to provide comprehensive linting (automated checking of code). Flake8 performs tasks such as PEP8 checks, logical error checks, and complexity analysis. This can help bring uniform styling, detect errors, and identify complex functions that may be hard to read or maintain.

Function complexity can be an issue, especially when testing, as functions with many decision points (e.g., loops, if statements) require more test cases. Such functions are harder to isolate, making unit testing more time-consuming and challenging.

Flake8 does not automatically modify the code, as it is a static code analyser. It flags bad code smells and potential errors. To run Flake8's complexity checker on code, use the following command:

```bash
python -m flakes \file_before.py --max-complexity=1
```
## Example of the Complexity Checker in Action
### Before:

```python
def post_comment(self):
    if self.success:
        comment = "Build succeeded"
    elif self.warning:
        comment = "Build had issues"
    elif self.failed:
        comment = "Build failed"
```
### After:

```python
def get_comment(self):
    comments = {
        "success": "Build succeeded",
        "warning": "Build had issues",
        "failed": "Build failed",
    }
```


