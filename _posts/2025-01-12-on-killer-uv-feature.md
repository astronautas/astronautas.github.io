---
layout: post
title: On One Killer UV Feature
---

In my view, neither performance nor trying to be Python-aligned is what sets [uv](https://github.com/astral-sh/uv) apart. Don’t get me wrong — try switching from uv to Poetry, and you’ll quickly notice how sluggish it (poetry) feels. uv goes extra miles to adhere to PEPs, and IMHO it's the go-to package manager for Python these days. But these aren’t the features that surprised me most.

There’s one small feature I initially overlooked that truly makes uv intriguing:

Imagine doing some ad-hoc scripting in Python 3.12. You run python, and you're in the REPL, ready to go. But what if you need to pull in a dependency, like Pandas? Here’s where it gets interesting. 

You would either:

1. Run `pip install pandas`, potentially modifying your global environment, or
2. Take the proper route:
    * Create a virtual environment
    * pip install pandas
    * Activate the virtual environment
    * Run python

This gets even more interesting if you need some other Python version than your global one. You would then use something like `pyenv`, install the version and set that version as local.

So, worst case:
1. `pyenv install 3.12`.
2. `pyenv local 3.12`.
3. `python -m venv .venv`.
4. `source .venv/bin/activate`.
5. `pip install pandas`.
6. `python`.

With uv, it's just 1 command:
```
uv run --python 3.12 --with pandas python
```

easy to remember, and no trace left behind. Enjoy scripting!