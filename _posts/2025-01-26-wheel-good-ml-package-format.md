---
layout: post
title: Wheel - a surprisingly good model package format
---
I'll give it a few years until MLflow dominates the model package format space, with alternatives like SageMaker models fading away, and sharing pure weights becoming an arcane art. But until that dominance is absolute, I’ve been thinking that there’s another quite obvious way to package models: **just store them as wheels**.

Let's create a project to train a model:

```bash
$ uv init --lib model
$ mkdir registry # the model registry
```

Within the model package (`src/model/main.py`), we'll a create dummy
decision tree model, with typical train/predict interface:

```python
from sklearn.datasets import load_iris
from sklearn.tree import DecisionTreeClassifier
from sklearn.model_selection import train_test_split
import joblib
from pathlib import Path
from sklearn.preprocessing import StandardScaler
import numpy as np
from sklearn.pipeline import Pipeline

def train():    
    # Load dataset
    iris = load_iris()
    X, y = iris.data, iris.target

    # Split the dataset into training and testing sets
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

    # Create a pipeline with a scaler and a decision tree classifier
    pipeline = Pipeline([
        ('scaler', StandardScaler()),
        ('classifier', DecisionTreeClassifier())
    ])

    # Train the pipeline
    pipeline.fit(X_train, y_train)

    # Dump the trained pipeline to a file
    Path(Path(__file__).parent / 'data').mkdir(parents=True, exist_ok=True)
    joblib.dump(pipeline, Path(__file__).parent / 'data' / 'model.pkl')
    
    print("Pipeline trained and saved to model.pkl")

def predict():
    # Load the trained model
    model = joblib.load(Path(__file__).parent / 'data' / 'model.pkl')
    print("Model loaded from model.pkl")

    # Make a prediction
    X_new = [[6.1, 2.8, 4.7, 1.2]]
    y_new = model.predict(X_new)
```

We pin the model dependencies and the Python version (pickling should be backward compatible):

```toml
[project]
name = "model"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
authors = [
    { name = "Lukas Valatka", email = "<email>" }
]
requires-python = ">=3.9"
dependencies = [
    "pandas == 2.2.3",
    "scikit-learn == 1.6.1",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

Train the model:
```bash
$ uv run python -c "from model.main import train; train()"
```

A `/data` folder should appear within `src/model`, containing the trained model. This is where the magic happens—we **bundle the trained model together with the source code**. (Yes, it's a bit of an abomination, but bear with me.)

Register the model:

```bash
$ uv build
$ mkdir ../registry/model
$ mv dist/* ../registry/model
```

Make our fake model registry go live:

```bash
$ cd ../registry
$ python -m http.server 8000 # package index is just /package folders served
```

that's it - we should be able to pull-in our trained model and predict:

```bash
$ uv run --with model==0.1.0 --index-url http://127.0.0.1:8000 --extra-index-url 'https://pypi.python.org/simple' --index-strategy unsafe-best-match python -c "from model.main import predict; predict()"
```

we can also our model to a serving application, as follows:

`pyproject.toml`
```toml
[project]
name = "serving-app"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
authors = [
    { name = "Lukas Valatka", email = "<email>" }
]
requires-python = ">=3.9"
dependencies = [
    "model == 0.1.0",
    "fastapi[standard]",
    "pandas",
    "pydantic",
]

[project.scripts]
serving-app = "serving_app:main"

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[[tool.uv.index]]
name = "local"
url = "http://localhost:8000"
```

all dependencies such as scikit-learn and pandas are at exact versions they were used for training, for full reproducibility. Also reflected within the lockfile - for reproducible builds!

If you try to create the venv within an incorrect Python environment e.g. Python 3.8, it will complain:

```bash
$ uv run --python 3.8 --with model==0.1.0 --index-url http://127.0.0.1:8000 --extra-index-url 'https://pypi.python.org/simple' --index-strategy unsafe-best-match python
```

```bash
  × No solution found when resolving `--with` dependencies:
  ╰─▶ Because the current Python version (3.8.20) does not satisfy Python>=3.9 and pandas==2.2.3 depends on Python>=3.9, we can conclude that pandas==2.2.3 cannot be used.
      And because model==0.1.0 depends on pandas==2.2.3 and you require model==0.1.0, we can conclude that your requirements are unsatisfiable.
```

So, how does this all compare to the all mighty mlflow model format:

* With mlflow, you need to explicitly list model dependencies in your serving-app OR have some script automating this (pull in model, uv add deps). It's not a native functionality.
* How do you make sure Python version is compatible between training and serving? Need to manually check this.

Just a thought experiment. I still believe mlflow is the go-to format due its [flavors](https://mlflow.org/docs/latest/traditional-ml/creating-custom-pyfunc/part1-named-flavors.html), and storing large files within your Nexus will make sysadmins frown big time.