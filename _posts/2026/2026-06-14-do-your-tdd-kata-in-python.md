---
layout: post
title: "Do your TDD Kata in Python"
date: 2026-01-07
categories: [testing, tdd, python]
tags: [testing, tdd, kata, python, summer, sunday, self-growing]
---

It is a sunday in June, the fresh air is dancing through the house. That particular moment of the day that everything is calm and easy, and you feel in peace, and you could feel just in love with the moment, but that it is too deep for this post. So, I have been doing TDD Katas with Python and here is one of the simplest and one I like the most, because it was the first one and I rushed too much, for sure, the first time you do something you make a lot of mistakes.

I am not going to explain here what TDD is, or the RED-GREEN-REFACTOR is not a song or a new joke from the space.

## The Kata: Add two numbers

Let's start with a minimal setup, we are going to use pytest for tests:

```python
mkdir add_two_numbers_kata
cd add_two_numbers_kata
uv init
uv add --dev pytest
```

And add the following lines:

```pyproject.toml```

```
[tool.pytest.ini_options]
pythonpath = ["src"]
```

The structure for your project:

```
❯ tree
.
├── README.md
├── main.py
├── pyproject.toml
├── src
│   └── calculator.py
├── tests
│   └── test_calculator.py
└── uv.lock

4 directories, 8 files
```

## Step 1: RED, is going to fail (don't panic)

```tests/test_calculator.py```

```
from calculator import add

def test_add_two_positive_numbers():
    assert add(2, 3) == 5
```
To run the test we could go with:

```
pytest -v (with source .venv/bin/activate)

uv run pytest -v (no uv venv running)
```

The test is asking for a module/funcion that does not exist yet, so we are good.

```E   ModuleNotFoundError: No module named 'calculator'```

## Step 2: Green, minimal code to make your test pass

```src/calculator.py```

```
def add(a, b):
    return 5
```

After that your test will be green.

At this moment, we could add some refactor but it is better to do it later.


## Step 3: RED, add a new test that add different numbers

Ok, how to decide which test to write? What I do, each test should show the code something it didn't know yet.

```
def test_add_different_numbers():
    assert add(1,3) == 4
```
This test fails because our code is static and only working for ```2 + 3 = 5```

## Step 4: GREEN, make the add return the sum of two numbers

Our calculator.py looks something like this now to make pass the test:

```
def add(a, b):
    return a + b
```

For this example I am avoiding the REFACTOR part, because it is too simple. But we should refactor to make our code more readable or avoid repetition or other things we are not discussing in this post.

Just for adding some edge cases to work on this example we could write more tests:

```
def test_add_zero():
    assert add(0, 7) == 7

def test_add_floats():
    assert add(1.5, 2.5) == 4.0

def test_add_negative_numbers():
    assert add(-5, -3) == -8
```
## Reflection

I wanted to start adding TDD to my projects, so start practicing these kind of katas sounded like the way to learn and put all the pieces together, like feeling comfortable in a new pair of shoes. What surprised me the most it was the tiny steps I take during the process of the Red-Green-Refactor. You don't write a full implementation and then test it; you write one assertion, then the smallest possible code to make it pass, and only then ask "what should this code learn next?". It's a slower rhythm than I was used to, but you stop carrying the mental load of "did I think about all the cases?".

I have been using TDD on some of my projects like my [mini-langfuse SDK](https://github.com/marialobillo/mini-langfuse)