## Homework Week 5

This week focus on deploying machine learning models. The model was trained from
[`course_lead_scoring_2026.csv`](./data/course_lead_scoring_2026.csv).


We use a `DictVectorizer` followed by
`LogisticRegression(solver="liblinear", C=1.0)`. 
The training script records the data checksum and imputation values in `model_metadata.json`.
`lead_score` is available before conversion and `converted` is the target.

## Setup

***Requirement***:
Python 3.11.15 and `uv`

#### Step 1: Install `uv` in workspace thru terminal
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```
or
```bash
pip install uv
```

#### Step 2: Create a new project
```bash
uv init 05-deployment
cd 05-deployment
```
or
```bash
mkdir 05-deployment
cd 05-deployment
uv init
```

#### Step 3: Configure manually the `pyproject.toml` and use `uv lock` to generate the `uv.lock` file
```bash
uv lock
```

#### Step 4: Change the required python version in `.python-version` file and use `uv sync --locked` to install dependencies in the virtual environment
```bash
uv sync --locked
```

#### Step 5: Check the virtual environment is working properly by running the `smoke_test`.py
The smoke test loads the checked-in model and verifies that inference is
deterministic. Check the artifact before using it:

```bash
sha256sum pipeline.bin
```

It must be:

```text
1646bbdcd38d4f044da6b630c5b332c93a314245a8c21929011c42de51f629f1  pipeline.bin
```

Run the script:
```bash
uv run python smoke_test.py
```


## Question 1 — environment check

Run `uv --version` and record the version. 
```text
0.13.6
```

## Question 2 — locked dependency

Open `pyproject.toml` and `uv.lock`. Which Scikit-Learn version is part of the 2026 reference environment?

```text
1.7.2
```

## Question 3 — load the model

Write a Python script that loads `pipeline.bin` with `pickle` and calls
`predict_proba` for this lead:

```json
{
  "lead_source": "paid_ads",
  "industry": "technology",
  "employment_status": "employed",
  "location": "north_america",
  "number_of_courses_viewed": 2,
  "annual_income": 79276.0,
  "interaction_count": 4,
  "lead_score": 0.41
}
```
Run the script `q3_lead.py`
```bash
uv run python q3_lead.py
```

Result in terminal:
```text
{'conversion_probability': 0.532994, 'conversion': True}
```

## Question 4 — serve the model

Start the provided API:

```bash
uv run uvicorn predict:app --host 0.0.0.0 --port 9696
```

The API has `GET /health` and `POST /predict`. Send this second lead:

```json
{
  "lead_source": "organic_search",
  "industry": "technology",
  "employment_status": "employed",
  "location": "europe",
  "number_of_courses_viewed": 4,
  "annual_income": 80304.0,
  "interaction_count": 7,
  "lead_score": 0.74
}
```

After activate uvicorn, open new terminal and run the following:

```bash
curl -s http://localhost:9696/predict \
  -H 'Content-Type: application/json' \
  -d '{
    "lead_source": "organic_search",
    "industry": "technology",
    "employment_status": "employed",
    "location": "europe",
    "number_of_courses_viewed": 4,
    "annual_income": 80304.0,
    "interaction_count": 7,
    "lead_score": 0.74
  }'
```

Result in terminal:
```text
{"conversion_probability":0.769799,"conversion":true}
```

## Question 5 — look at the container configuration

Which Python base-image tag is declared in the Dockerfile?

```text
python:3.11.15-slim-bookworm
```

## Question 6 — run the container

Build and run it:

```bash
docker build -t zoomcamp-model:2026-hw5 .
docker run --rm -p 9696:9696 zoomcamp-model:2026-hw5
```

After activate docker run, open new terminal and run the following to find the `container id` or `name`:

```bash
cd 05-deployment
python docker ps
```

Next run the following (copy paste the `id` or `name` displayed in the terminal):
```bash
docker exec -it <CONTAINER_ID_OR_NAME> python q6_test.py
```

Result in terminal:
```text
{'conversion_probability': 0.769799, 'conversion': True}
```

The expected result is the same reference inference as Question 4. 
Also verify that:

```bash
curl -s http://localhost:9696/health
```

returns `{"status":"ok", ...}` and the model checksum recorded in
`model_metadata.json`.

Result in terminal:
```text
{"status":"ok","model_sha256":"1646bbdcd38d4f044da6b630c5b332c93a314245a8c21929011c42de51f629f1","feature_defaults_sha256":"1694f842b908fac9f53c0cc19be6d91f934befc63af23cc32fefafde69acd0cd"}
```

To see all the existing docker images:
```bash
docker images
```

To remove the docker image used in this homework:
```bash
docker rmi zoomcamp-model:2026-hw5
```