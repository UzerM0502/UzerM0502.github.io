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
# Introduction

## Why Use Pytest?

pytest is one of the most popular testing frameworks for Python. Offering powerful features and a large community making it an excellent choice for testing Django applications.

It works well with django in particular because of its effecient way of providng database connections, using fixtures for reusable logic and factories to make test more flexible.

## Writing tests with Pytest

A common pattern for structuring tests is:

1. Arrange – Prepare test data or set up necessary conditions.

2. Act – Perform the action to be tested, such as calling a function or retrieving data from the database.

3. Assert – Verify that the output matches expectations.

## Setting up Pytest

You should add this file "pytest.ini" in the root of your project directory

```python
[pytest]
DJNAGO_SETTINGS_MODULE = {your_app_name}.settings
python_files = test_*.py 
python_classes = *Test
python_functions = test_*
```
You may need run the following in the terminal 

```bashe
# setting required environment variables
export DJANGO_SETTINGS_MODULE={your_app_name}.settings
```

```bash
# responsible for creating new migrations based on the changes you have made to your models
python manage.py makemigrations 

# responsible for applying and unapplying migrations.
python manage.py migrate 
```
Also for our case it may be useful to have a visualisation of the tables in the database theerefore I downloaded the SQLite extension on vs code.

### Sidenote on migrations
think of migrations as a version control system for your database schema. ```makemigrations``` is responsible for packaging up your model (tables) changes into individual migration files - analogous to commits - and ```migrate``` is responsible for applying those to your database.

# Introduction to Fixrures and Markers 

## Pytest Fixtures

Fixtures in pytest are functions that run before tests to set up necessary data or state. They:

1. Provide database connections, API calls, or data setup.

2. Allow modular and reusable test setups.

3. Can be shared across multiple tests by defining them in conftest.py.

## Pytest Markers

Markers allow selective test execution, improving efficiency for large test suites. Common markers (in this guide) include:

```@pytest.mark.skip``` – Skips a test unconditionally.

```@pytest.mark.django_db``` – Grants access to the Django database for tests requiring it.

# Examples

## Lets look at some basic pytest examples:

```python
import pytest 


# Scope of the test:
# Session level

@pytest.fixture(scope='session')
def fixture_1():
    print('run-fixture-1')
    return 1

def test_example1(fixture_1):
    print('run-test-example1')
    num = fixture_1
    assert num == 1    

def test_example2(fixture_1):
    print('run-test-example2')
    num = fixture_1
    assert num == 1    
```
After importing pytest we set our setup our first fixture with the decorator ```@pytest.fixture(scope='session')```. This allows us to use the ```fixture_1``` function as an input to our tests - in this case it will inject our case with 1 (as it ends with return 1). 
The ```scope = 'session'``` paramter in our fixture means the fixture is setup once per test, meaning that this fixture is dhared across all test functions. This can be beneficial for testing as we only need to setup and teardown the fixture once oer SESSION making it more effecient. The default scope is function which is fine for most test cases. 

This tests outputs the following:

```bash 
(myenv39) uzermughal@Uzers-Air Pytest Blog % pytest -rP tests/test_ex1.py 
============================================== test session starts ===============================================
platform darwin -- Python 3.9.21, pytest-8.3.4, pluggy-1.5.0
django: settings: core.settings (from env)
rootdir: /Users/uzermughal/Documents/Pytest Blog
plugins: factoryboy-2.7.0, django-4.2.0, Faker-36.1.1
collected 2 items                                                                                                

tests/test_ex1.py ..                                                                                       [100%]

===================================================== PASSES =====================================================
_________________________________________________ test_example1 __________________________________________________
--------------------------------------------- Captured stdout setup ----------------------------------------------
run-fixture-1
---------------------------------------------- Captured stdout call ----------------------------------------------
run-test-example1
_________________________________________________ test_example2 __________________________________________________
---------------------------------------------- Captured stdout call ----------------------------------------------
run-test-example2
=============================================== 2 passed in 0.10s ================================================
```

We can see the fixture is only ran once as "run-fixture-1" was printed only once. Note: I used ```pytest -rP``` to run the test because it includes the print statements in the terminal output.

### Managing Fixtures with conftest.py

The conftest.py file is a centralized place to store reusable fixtures, making them available across the entire test suite without needing explicit imports. Remember the ```fixture_1``` we used in the previous exmaple? We can move this to our conftest.py and have the same results and we will do this for all fixtures in the future. Note the conftest.py is stored in the parent folder of the tests.

## Lets look at some basic pytest examples with django models 

```python
import pytest

# Default user table that django creeates
from django.contrib.auth.models import User


# Import the User model from the models.py file
# Uses the User model as an input to the test 

@pytest.mark.django_db
def test_set_check_password(user_1):
    user = user_1
    user.set_password('new_password')
    assert user.check_password('new_password') == True
```
where the fixture is located in the conftest.py file:
```python
#conftest.py file
@pytest.fixture()
def user_1(db):
    user = User.objects.create_user('test_user')
    return user
```
This code has the test for setting and then testing whether a newly set password is correct. Firstly we import all the neccessary modules:

```python
from django.contrib.auth.models import User
```

this line import the default Users table that django creates and uses it as input to the table  

we then mark the user_1 function with the decorator ``` @python.fixture``` showing that it is a fixture ready to be used as a input to our test. This ``` user_1 ``` function takes in a db which is required and allows us to connect to the database where we will access the ```User```model etc. (in django a model is just a table)

This ```db``` fixture is important as it allows us to test the database in a controlled environment and effeciently handles setup and tear down of the databse - meaning it doesnt effect the other test allowing us to run tests in isolation. 

```@pytest.mark.django_db```: a decorator marks the test function to use the database. This is necessary for tests that interact with the database.

```test_set_check_password```: a function takes the user_1 fixture as an argument, which provides the user instance created by the fixture.

#### Inside the test function:

```user.set_password('new_password')```: sets a new password for the user.
```assert user.check_password('new_password') == True```: Asserts that the new password is correctly set by checking if the check_password method returns True for the new password.

In short, the test sets a new password for the user and verifies that the password has been correctly set and therefore ensures that the password setting capability of the Django User model is working as expected

#### Sidenote on db
This db fixture is provided by the pytest plugin (if you were thinking where it came from) and is configured from the settings.py file underneath DATABASES. The default for testing is SQLite but you can configure to which one you want, heres how my DATABASES settings looked like 

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

