# NASH: Numerically Aware Scoring Heuristic for Robust Semantic Similarity

This repository is being refactored into **NASH DEMO**, a reusable toolkit for
numeracy-aware semantic similarity. The original research implementation is
preserved in the top-level `nash/`, `scripts/`, and `data/` folders. The new
package API lives under `src/nash/`.

## Abstract

Numerical precision is critical in financial NLP, yet embedding-based semantic similarity metrics exhibit numerical blindness—failing to distinguish contradictory values within similar contexts. We introduce **NASH (Numerically Aware Scoring Heuristic)**, a model-agnostic metric that decouples numerical verification from textual semantic evaluation through a three-stage pipeline: (1) modal separation via numeric masking, (2) dual-channel similarity estimation through masked-text similarity and context-aware numeric alignment, and (3) IDF-weighted aggregation. NASH functions as a drop-in enhancement to existing embedding-based metrics. Validated on our proposed **NumFinE** financial numerical evaluation benchmark and established semantic similarity datasets (STS-B, Financial-STS), NASH achieves substantial improvements in numerical sensitivity (up to +159.6% on listwise ranking) while preserving general semantic performance, establishing a reliable standard for numeracy-aware evaluation.

---

## Overview

This repository provides a **paper-aligned public implementation** of NASH along with the evaluation datasets used in our experiments.

---

## Project Surfaces

NASH DEMO is prepared for local development and public web/API deployment.

- **Frontend**: the React/Vite web demo lives in `app/`. It runs locally with
  Vite and is configured for GitHub Pages at
  `https://cnclabs.github.io/toolkit.fin.nsah/`.
- **Backend**: the FastAPI app lives in `src/nash/server/main.py`. It runs
  locally with `uvicorn` and can be deployed to Hugging Face Spaces Docker at
  `https://raelee1026717-nash-api.hf.space`, with the prediction endpoint
  `https://raelee1026717-nash-api.hf.space/predict`.

---

## Repository Structure

```text
nash_public_release/
├── nash/
├── scripts/
├── data/
├── run_triplet.sh
├── run_crosspair.sh
├── run_listwise.sh
├── requirements.txt
└── README.md
```

---

## Installation

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pip install -e ".[dev,model]"
```

For a package-only install:

```bash
pip install -e .
```

Install model dependencies when using the live SentenceTransformer backend:

```bash
pip install -e ".[model]"
```

---

## Quickstart

Use NASH as a Python metric:

```python
from nash import NASH

metric = NASH(model_name="sentence-transformers/all-MiniLM-L6-v2")

small = metric.score_pair(
    "Revenue increased by 3.56%.",
    "Revenue increased by 4%."
)

large = metric.score_pair(
    "Revenue increased by 3.56%.",
    "Revenue increased by 40%."
)

print(small.final_score, large.final_score)
assert small.final_score > large.final_score
```

Demo-oriented scoring API with JSON trace export:

```python
from nash import NASHMetric, save_trace, score_pair, score_pairs

metric = NASHMetric(
    baseline="bert-base-uncased",
    backend_type="bertscore",
)

trace = metric.score(
    "Revenue increased by 3.5% in Q2.",
    "Revenue increased by 35% in Q2.",
)

print(trace["baseline_comparison"]["baseline_score"])
print(trace["baseline_comparison"]["nash_score"])
trace.to_json("trace.json")
save_trace(trace, "trace-copy.json")

row = score_pair(
    "Revenue increased by 3.5% in Q2.",
    "Revenue increased by 35% in Q2.",
    metric="nash",
    model="bert-base-uncased",
)
print(row["baseline_score"], row["nash_score"])

rows = score_pairs(
    [{"sentence1": "EPS rose from $2.10 to $2.40.", "sentence2": "EPS rose from $2.10 to $4.20."}],
    metric="nash",
    models=["bert-base-uncased", "ProsusAI/finbert"],
)
```

The exported JSON can be uploaded directly in the visualization website.

Load local evaluation data and run simple protocols:

```python
from nash import NASHMetric, evaluate, load_dataset

metric = NASHMetric(
    baseline="bert-base-uncased",
    backend_type="bertscore",
)

dataset = load_dataset("numfine-triplet")
summary = evaluate(dataset, metric, protocol="triplet")
print(summary)
```

Supported dataset names:

```text
numfine-triplet
numfine-crosspair
numfine-listwise
finsts
stsb
```

Score one sentence pair from the command line:

```bash
nash-score \
  --sentence-a "Revenue increased by 3.56%." \
  --sentence-b "Revenue increased by 40%." \
  --metric nash \
  --models bert-base-uncased ProsusAI/finbert sentence-transformers/all-MiniLM-L6-v2
```

The command reports the baseline score, NASH score, text score, and numeric
score for each selected model.

Score a batch file:

```bash
nash-score \
  --input-file examples.csv \
  --task pair \
  --metric nash \
  --models bert-base-uncased ProsusAI/finbert
```

Score a NumFinE-style triplet dataset file:

```bash
nash-score \
  --input-file data/triplet_test.json \
  --task triplet \
  --metric nash \
  --models bert-base-uncased ProsusAI/finbert sentence-transformers/all-MiniLM-L6-v2
```

Other supported dataset file formats:

```bash
nash-score --input-file data/crosspair_test.json --task crosspair --metric nash
nash-score --input-file data/listwise_test.json --task listwise --metric nash
```

Use `--metric baseline` when the original semantic score should be the primary
score column.

By default `nash-score` does not open the local web demo. Add `--open-demo` only
after an `--input-file` run when you want to launch Batch Upload with the
completed batch results.

```bash
nash-score \
  --input-file data/triplet_test.json \
  --task triplet \
  --metric nash \
  --models bert-base-uncased ProsusAI/finbert \
  --open-demo
```

Interactive Example remains a web tab. It is not opened automatically from the
CLI sentence-pair command.

To create an uploadable evaluation result from a small test file and open Batch
Upload, run the backend in live mode first because the script scores the pairs:

```bash
python examples/open_upload_evaluation_result.py --task triplet
```

The script supports all three evaluation formats:

```bash
python examples/open_upload_evaluation_result.py --task triplet
python examples/open_upload_evaluation_result.py --task crosspair
python examples/open_upload_evaluation_result.py --task listwise
```

It uses `data/triplet_test.json`, `data/crosspair_test.json`, or
`data/listwise_test.json`, scores a small expanded subset through the running
local backend, writes `app/public/nash_{task}_test_results.json`, and opens the
Batch Upload tab with that result file.

Use single quotes when a sentence contains currency symbols. In most shells,
`"$10"` inside double quotes is treated as a positional parameter expansion,
which silently removes the dollar amount before Python receives the text:

```bash
nash-score \
  --sentence-a 'EPS was $10 in 2024, a 25% increase.' \
  --sentence-b 'EPS rose 10% to $11.'
```

Run the Milestone 1 tests:

```bash
pytest
```

---

## Visualization Demo

The demo has a FastAPI backend and a React/TypeScript frontend. The public
deployment targets are:

```text
Frontend: https://cnclabs.github.io/toolkit.fin.nsah/
Backend:  https://raelee1026717-nash-api.hf.space
Predict:  https://raelee1026717-nash-api.hf.space/predict
```

Run locally with two terminals.

Terminal 1: backend

```bash
cd codes.fin.bertscore
source .venv/bin/activate
python -m pip install -r requirements.txt -e .
python -m uvicorn nash.server.main:app --reload --host 127.0.0.1 --port 8000
```

Terminal 2: frontend

```bash
cd codes.fin.bertscore/app
npm install
npm run dev -- --host localhost
```

Open the URL printed by Vite, usually:

```text
http://localhost:5173
```

If port `8000` is busy, run the backend on another port and point the frontend
at it:

```bash
python -m uvicorn nash.server.main:app --reload --host 127.0.0.1 --port 8001
```

```bash
cd app
VITE_API_URL=http://127.0.0.1:8001 npm run dev -- --host localhost
```

Frontend production build:

```bash
cd app
npm run build
```

The Vite production base path is `/toolkit.fin.nsah/`, matching GitHub Pages.
Production frontend builds read `app/.env.production`, which points
`VITE_API_URL` at `https://raelee1026717-nash-api.hf.space`.

Deploy the frontend by publishing `app/dist/` from the fixed lab repository:

```bash
git remote add origin https://github.com/cnclabs/toolkit.fin.nsah.git
cd app
npm install
npm run build
```

API endpoints:

```text
GET  /api/health
GET  /api/models
POST /api/explain
POST /predict
```

Hugging Face Spaces Docker deployment uses the repository `Dockerfile`.
Create/update the Space at:

```text
https://huggingface.co/spaces/raelee1026717/nash-api
```

The Docker image exposes port `7860` and starts:

```bash
python -m uvicorn nash.server.main:app --host 0.0.0.0 --port 7860
```

The deployed API allows CORS from:

```text
http://localhost:5173
http://localhost:3000
http://127.0.0.1:5173
http://127.0.0.1:3000
https://cnclabs.github.io
```

The React UI is organized into three tabs:

Current supported modes:

- **Batch Upload**: one upload button for JSON evaluation results, including
  triplet/crosspair/listwise completed result formats.
- **Interactive Example**: two sentence inputs, fixed NASH threshold, baseline/NASH
  comparison, and NASH visualizations. This tab calls the backend.
- **Cases**: read-only generated examples.

There is no session history in the demo UI.

The fixed examples live in:

```text
app/src/demo_data/demo_traces.json
app/src/demo_data/triplet_demo.json
```

Build it:

```bash
cd app
npm run build
```

Pairwise trace schema:

```json
{
  "id": "example_001",
  "type": "pairwise",
  "title": "Revenue growth mismatch",
  "sentence1": "Revenue increased by 3.5% in Q2.",
  "sentence2": "Revenue increased by 35% in Q2.",
  "baselines": [
    {
      "name": "baseline / bert-base-uncased",
      "baseline_score": 0.982,
      "nash_score": 0.641,
      "text_score": 0.954,
      "numeric_score": 0.21
    }
  ],
  "numbers1": [
    {
      "id": "s1_n1",
      "text": "3.5%",
      "value": 3.5,
      "start": 21,
      "end": 25
    }
  ],
  "numbers2": [
    {
      "id": "s2_n1",
      "text": "35%",
      "value": 35,
      "start": 21,
      "end": 24
    }
  ],
  "alignments": [
    {
      "source": "s1_n1",
      "target": "s2_n1",
      "source_text": "3.5%",
      "target_text": "35%",
      "alignment_similarity": 0.94,
      "numeric_gap": 31.5,
      "status": "aligned"
    }
  ],
  "heatmap": {
    "rows": ["3.5%"],
    "columns": ["35%"],
    "values": [[0.94]]
  },
  "masked_sentence1": "Revenue increased by [NUM] in [NUM].",
  "masked_sentence2": "Revenue increased by [NUM] in [NUM]."
}
```

At this stage, the local demo uses fixed example traces. The same JSON trace
format can later be generated by the Python package or evaluation scripts.

Uploaded traces can come from `NASHMetric.score(...).to_json(...)` or from a
previous dashboard download.

The dashboard also includes:

```text
Triplet Visualization
Batch Aggregate View
Numerical Sensitivity
Session History
```

Batch Upload is intended for result JSON, not raw test cases. Supported result
shapes are:

```text
nash_evaluation_results / nash_cli_results with rows[]
triplet summary.json with task="triplet" and overall.accuracy
crosspair summary.json with task="crosspair" and overall.accuracy
listwise summary.json with task="listwise" and overall.kendall_tau_b_mean
```

Row-level result payloads should contain scored rows:

```json
{
  "kind": "nash_evaluation_results",
  "task": "triplet",
  "rows": [
    {
      "sentence1": "Revenue increased by 3.5% in Q2.",
      "sentence2": "Revenue increased by 35% in Q2.",
      "model": "sentence-transformers/all-MiniLM-L6-v2",
      "baseline_score": 0.95,
      "nash_score": 0.61,
      "text_score": 0.93,
      "numeric_score": 0.20
    }
  ]
}
```

The frontend has a mode selector:

```text
Static trace
```

uses `examples/traces/*.json` and does not load a model.

```text
Live model
```

calls `/api/explain` with multiple baselines. The backend runs
`metric.explain(sentence1, sentence2)` once per baseline and returns both a
comparison table and the per-baseline traces. Available baselines are populated
from:

```text
GET /api/models
```

The default live comparison includes:

```text
bert-base-uncased
ProsusAI/finbert
sentence-transformers/all-MiniLM-L6-v2
```

---

## Running

The original research release provides executable scripts for all evaluation
settings:

```bash
bash run_triplet.sh
bash run_crosspair.sh
bash run_listwise.sh
```

Please refer to these scripts for detailed configurations.

---

## Notes

* This release follows the pipeline described in the paper.
* The dataset used in our experiments is included in the repository.
* The implementation is intended for reproducibility and evaluation purposes.
* The new `src/nash` package currently implements the Milestone 1 local scoring
  pipeline: numeric extraction, masking, numeric similarity, SentenceTransformer
  backend, `NASH` metric class, explanation traces, `nash-score`, and basic unit
  tests.

---
