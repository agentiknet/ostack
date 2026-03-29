# AI / Model Training — Architecture Topics

Read this file when the user selected project type: **AI / model training**.

Walk through each topic below **one at a time** via AskUserQuestion.

**Important context:** The user works with Rust, C, and Python for AI projects, adapting to context. Language choice is a real discussion here, not a default.

---

## Common Topics

### 1. Language & Framework

Ask:
> What's the primary work for this AI project?
>
> - **A)** Training a model (data processing + training loop) → **Python** (PyTorch/JAX ecosystem, no alternative)
> - **B)** High-performance inference server → **Rust** (with Python bindings for model loading) or **Python** (FastAPI/vLLM)
> - **C)** Data pipeline / preprocessing → **Python** (pandas/polars) or **Rust** (for performance-critical ETL)
> - **D)** Research / experimentation → **Python** (Jupyter + PyTorch)
> - **E)** Mix — training in Python, serving in Rust/C
>
> **Recommendation:** Python is unavoidable for training (the ML ecosystem is Python). For serving, Rust is worth considering if latency matters. For most projects, pure Python is fine.

If Python:
> Package manager?
> - **A)** uv (recommended — fast, modern, replaces pip+venv+poetry)
> - **B)** Poetry
> - **C)** pip + venv (simple, no extra tools)

### 2. Database

Ask:
> Does this project need a database? (Beyond file storage for datasets/models)
>
> - **A)** No — just file storage (models, datasets on GCS)
> - **B)** Metadata store — experiment results, run configs → **Firestore** or **SQLite**
> - **C)** Full application database — if this AI project includes a web frontend → choose as needed

### 3. Authentication

Usually minimal for AI projects:
> Who uses this?
>
> - **A)** Just me / my team (no auth needed, or basic API key)
> - **B)** External users via API (need API key management)
> - **C)** Web UI for users (need full auth — redirect to SaaS route topics)

### 4. Hosting & Scaling

**Cloud Run may NOT be the default here.** Discuss:
> What does deployment look like?
>
> - **A)** API server for inference (no GPU) → **Cloud Run** (fine, standard containers)
> - **B)** GPU inference → **Compute Engine with GPU** or **Vertex AI endpoints**
> - **C)** Training jobs → **Compute Engine with GPU** or **Vertex AI Training** or external (Lambda Labs, RunPod)
> - **D)** Batch processing → **Cloud Run Jobs** or **Compute Engine**
>
> **Flag:** If GPU is needed, discuss cost. Cloud GPUs are expensive. External providers (Lambda Labs, RunPod) can be 2-5x cheaper for training. GCP GPUs are better for inference (close to your other infra).

### 5. CI/CD

For AI projects, CI/CD is different:
> What needs to happen on push?
>
> - **A)** Standard: lint + test + deploy inference server → GitHub Actions
> - **B)** ML pipeline: lint + test + trigger training run → GitHub Actions + Vertex AI Pipelines
> - **C)** Minimal: just lint + test (manual deploys for now)

---

## AI-Specific Topics

### 6. GPU Strategy

Ask:
> Do you need GPUs? If yes, for what?
>
> - **A)** No GPUs — CPU-only inference or data processing
> - **B)** Training — need GPUs for model training
> - **C)** Inference — need GPUs for real-time inference
> - **D)** Both training and inference
>
> If B or D:
> - **Cloud GPUs (Vertex AI, Compute Engine):** Easy GCP integration, pay-per-minute, more expensive
> - **Lambda Labs:** Cheapest cloud GPUs, great for training, less integrated
> - **RunPod:** Serverless GPUs, good for burst training
> - **Local:** If you have a GPU machine, cheapest long-term
>
> **Recommendation:** Lambda Labs or RunPod for training (cost), GCP for inference (latency, integration).

### 7. Dataset Storage & Versioning

Ask:
> How large are your datasets and how do you want to manage them?
>
> - **A)** Small (< 10GB) → **GCS bucket** (simple, cheap)
> - **B)** Large (10GB-1TB) → **GCS bucket + DVC** (Data Version Control — git for data)
> - **C)** Very large (> 1TB) → **GCS bucket + manifest files** (track what's where without downloading everything)
> - **D)** Using HuggingFace datasets → **HuggingFace Hub** (built-in versioning)

### 8. Experiment Tracking

Ask:
> How do you want to track experiments (hyperparameters, metrics, model versions)?
>
> - **A)** Weights & Biases (W&B) — industry standard, free tier, great visualizations
> - **B)** MLflow — open source, self-hosted, more control
> - **C)** TensorBoard — basic, built into PyTorch/TF, no server needed
> - **D)** None — manual tracking (fine for small projects)
>
> **Recommendation:** W&B for serious projects (it's free for personal use). TensorBoard for quick experiments.

### 9. Model Serving & Inference

Ask:
> How will the trained model be used?
>
> - **A)** REST API endpoint → Cloud Run (CPU) or Vertex AI Endpoints (GPU)
> - **B)** Batch processing → Cloud Run Jobs or Vertex AI Batch Prediction
> - **C)** Embedded in another app → export model, no separate server
> - **D)** Not sure yet — just training for now
>
> If A: Discuss inference framework (vLLM for LLMs, TorchServe, ONNX Runtime, custom FastAPI).

### 10. Inference Cost Estimation

If serving is needed, discuss:
> Expected request volume?
>
> - **A)** Low (< 1000 req/day) → CPU inference on Cloud Run (cheapest, ~$5-20/month)
> - **B)** Medium (1K-100K req/day) → depends on model size; may need GPU
> - **C)** High (> 100K req/day) → dedicated GPU instances, need cost modeling
>
> Provide a rough cost estimate based on the choices made.

### 11. Open-Source Licensing

Ask:
> Are you using pre-trained models? If so, which?
>
> Check the license:
> - **MIT / Apache 2.0** — use commercially, no restrictions
> - **Llama license** — free for most uses, revenue cap
> - **GPL** — must open-source derivative works
> - **Research-only** — cannot use commercially
>
> **Flag licensing risks** if the user plans commercial use with a restrictive model.

---

## Scaffold Spec

```
{project-slug}/
├── src/                        # Main source code
│   ├── train.py                # Training entry point
│   ├── model.py                # Model definition
│   ├── data.py                 # Data loading / preprocessing
│   ├── config.py               # Hyperparameters / config
│   └── serve.py                # Inference server (if serving)
├── tests/
│   └── test_model.py           # Basic model tests
├── data/                       # Local data (gitignored)
├── models/                     # Saved models (gitignored)
├── Dockerfile                  # For Cloud Run deployment (if serving)
├── .dockerignore
├── pyproject.toml              # Python project config (uv/poetry)
├── .python-version             # Python version pin
└── ruff.toml                   # Linter config
```
