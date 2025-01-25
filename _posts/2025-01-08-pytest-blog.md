---
title: Mastering Pytest for Django Applications
description: Creating Fixtures, Mocking Dependencies , and More 
author: Uzer
date: 2025-01-10 11:33:00 +0800
categories: [Coding, Python, Testing]
tags: [typography]
pin: false 
math: true
mermaid: true
---
## Introduction

terminal output without specifying scope = session the fixture runs twice
```bash
(myenv39) uzermughal@Uzers-MacBook-Air Pytest Blog % pytest -rP
============================================= test session starts =============================================
platform darwin -- Python 3.9.21, pytest-8.3.4, pluggy-1.5.0
rootdir: /Users/uzermughal/Documents/Pytest Blog
plugins: django-4.2.0
collected 2 items                                                                                             

tests/test_ex1.py ..                                                                                    [100%]

=================================================== PASSES ====================================================
________________________________________________ test_example1 ________________________________________________
-------------------------------------------- Captured stdout setup --------------------------------------------
run-fixture-1
-------------------------------------------- Captured stdout call ---------------------------------------------
run-test-example1
________________________________________________ test_example2 ________________________________________________
-------------------------------------------- Captured stdout setup --------------------------------------------
run-fixture-1
-------------------------------------------- Captured stdout call ---------------------------------------------
run-test-example2
============================================== 2 passed in 0.02s ==============================================
(myenv39) uzermughal@Uzers-MacBook-Air Pytest Blog % 
```

terminal output with specifying scope = session the fixture runs once
```bash
(myenv39) uzermughal@Uzers-MacBook-Air Pytest Blog % pytest -rP
============================================= test session starts =============================================
platform darwin -- Python 3.9.21, pytest-8.3.4, pluggy-1.5.0
rootdir: /Users/uzermughal/Documents/Pytest Blog
plugins: django-4.2.0
collected 2 items                                                                                             

tests/test_ex1.py ..                                                                                    [100%]

=================================================== PASSES ====================================================
________________________________________________ test_example1 ________________________________________________
-------------------------------------------- Captured stdout setup --------------------------------------------
run-fixture-1
-------------------------------------------- Captured stdout call ---------------------------------------------
run-test-example1
________________________________________________ test_example2 ________________________________________________
-------------------------------------------- Captured stdout call ---------------------------------------------
run-test-example2
============================================== 2 passed in 0.02s ==============================================
(myenv39) uzermughal@Uzers-MacBook-Air Pytest Blog % 
```

here is the code to test the amount of users added to database

```python
import pytest

from django.contrib.auth.models import User
from django.test import Client

@pytest.mark.django_db
def test_user_create_1():
    User.objects.create_user('test_user', '', 'test_password')
    count = User.objects.all().count()
    print('count of recordd in user table for test_user_create_1: ',count)
    assert count == 1

@pytest.mark.django_db
def test_user_create2():
    count = User.objects.all().count()
    print('count of recordd in user table for test_user_create_2: ',count)
    assert count == 0
```

run in terminal: 

```bash 
(myenv39) uzermughal@Uzers-MacBook-Air Pytest Blog % pytest -rP
============================================= test session starts =============================================
platform darwin -- Python 3.9.21, pytest-8.3.4, pluggy-1.5.0
django: settings: core.settings (from env)
rootdir: /Users/uzermughal/Documents/Pytest Blog
plugins: django-4.2.0
collected 4 items                                                                                             

tests/test_ex4.py ..                                                                                    [ 50%]
tests/test_ex1.py ..                                                                                    [100%]

=================================================== PASSES ====================================================
_____________________________________________ test_user_create_1 ______________________________________________
-------------------------------------------- Captured stdout call ---------------------------------------------
count of recordd in user table for test_user_create_1:  1
______________________________________________ test_user_create2 ______________________________________________
-------------------------------------------- Captured stdout call ---------------------------------------------
count of recordd in user table for test_user_create_2:  0
________________________________________________ test_example1 ________________________________________________
-------------------------------------------- Captured stdout setup --------------------------------------------
run-fixture-1
```