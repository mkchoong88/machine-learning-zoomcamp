# ML Zoomcamp 2026 deployment release

This homework uses a frozen lead-scoring release for the 2026 cohort. The
artifact, dataset checksum, training script, API, and container definition are
in this directory. No prior-cohort model or generated answer set is used.

The model was trained from
[`course_lead_scoring_2026.csv`](../../data/course_lead_scoring_2026.csv) with
the following observable features:

| Feature | Type | Missing-value rule |
| --- | --- | --- |
| `lead_source` | categorical | `NA` |
| `industry` | categorical | `NA` |
| `employment_status` | categorical | `NA` |
| `location` | categorical | `NA` |
| `number_of_courses_viewed` | integer | training-set median |
| `annual_income` | number | training-set median |
| `interaction_count` | integer | training-set median |
| `lead_score` | number | training-set median |

We use a `DictVectorizer` followed by
`LogisticRegression(solver="liblinear", C=1.0)`. The training script records
the data checksum and imputation values in `model_metadata.json`.
`lead_score` is available before conversion and `converted` is the target.

The API accepts the eight observable fields in the release. Missing numeric
values are replaced with medians calculated from the training CSV and recorded
in `feature_defaults.json`; missing categories use `NA`.

## Local run

```bash
uv sync --locked
uv run python smoke_test.py
uv run uvicorn predict:app --host 0.0.0.0 --port 9696
```

In another terminal:

```bash
python q6_test.py
```

## Container run

The checked-in `Dockerfile` is the canonical 2026 container configuration. It uses
Python `3.11.15-slim-bookworm`, `uv 0.10.11`, and the checked-in `uv.lock`.
It copies the frozen artifact and starts the API on port `9696`.

```bash
docker build -t zoomcamp-model:2026-hw10 .
docker run --rm -p 9696:9696 zoomcamp-model:2026-hw10
```

The API exposes `GET /health` and `POST /predict`. The model artifact is
addressed by its checksum in `model_metadata.json`; the API health response
also reports the feature-default checksum. The Dockerfile pins the Python and
uv image digests and installs from the checked-in lockfile.
