# LangGraph Learning Notebooks (Student Edition)

This repository contains a **hands-on sequence of Jupyter notebooks** that introduce core patterns for building **agentic workflows with LangGraph** (and related tooling): from a simple ReAct agent, to search, persistence/streaming, human-in-the-loop, and a small end-to-end writing workflow.

The project targets **Python 3.13**, uses **Conda** for the runtime (Python + Jupyter + system libs), and **uv** for Python dependency resolution + locking (`pyproject.toml` + `uv.lock`). A **pip-compatible, pinned lockfile** (`requirements.locked.txt`) is committed for installation into the Conda environment.

---

## Two audiences, two workflows

### If you're a user (you just want to run the notebooks)
You will:
1. Create the Conda env from `environment.yaml`
2. Install Python packages into that env using **the exported requirements lock** (`requirements.locked.txt`)
3. Run JupyterLab from that env

### If you're a developer (you will change dependencies)
You will:
1. Use `uv add ...` to change dependencies (updates `pyproject.toml` and `uv.lock`)
2. Export a pip-compatible lock (`requirements.locked.txt`)
3. Install that lock into the Conda runtime env **additively** (no uninstalls)

Why the extra export step? `uv.lock` is a uv project lockfile (TOML). `uv pip install` expects a **requirements.txt-style** lock, not `uv.lock`.

> ⚠️ IMPORTANT (shared Conda env): **Do not use `uv pip sync`** in this workflow.  
> `uv pip sync` removes packages not listed in the requirements file and can uninstall Conda-owned tooling such as JupyterLab.  
> Use `uv pip install -r requirements.locked.txt` instead.

---

## Quick Start (users)

```bash
# 1) Create + activate the conda runtime env
conda env create -f environment.yaml
conda activate langchain

# 2) Install the pinned Python dependencies into THIS conda env
# (requirements.locked.txt is generated from uv.lock and committed)
uv pip install -r requirements.locked.txt

# 3) Register the Jupyter kernel (one-time)
python -m ipykernel install --user --name langchain --display-name "Python (langchain)"

# 4) Launch JupyterLab
python -m jupyter lab
```

Open any notebook and select the **Python (langchain)** kernel.

> If `python -m jupyter lab` fails with “No module named jupyter”, install JupyterLab into the conda env:
> `conda install -c conda-forge jupyterlab ipykernel`

---

## Developer workflow (changing dependencies)

### 0) Prereqs
- Have the conda env active when working on deps (so uv can pin to the same Python):
  ```bash
  conda activate langchain
  ```

### 1) Add / update a dependency
```bash
uv add <package-name>
uv lock
```

### 2) Export a pip-compatible lockfile for the conda runtime
```bash
uv pip compile pyproject.toml -o requirements.locked.txt
```

### 3) Apply the lockfile to the active conda env (additive; safe in shared envs)
```bash
conda activate langchain
uv pip install -r requirements.locked.txt
```

### 4) Commit the right files
Commit these when dependencies change:
- `pyproject.toml`
- `uv.lock`
- `requirements.locked.txt`

Commit/update this when **conda-level** dependencies change (Python version, JupyterLab, CUDA/PyTorch, etc.):
- `environment.yaml` (recommended export: `conda env export --from-history > environment.yaml`)

---

## Repository Structure

```
.
├── environment.yaml
├── pyproject.toml
├── uv.lock
├── requirements.locked.txt
├── README.md
├── Lesson_1_Student.ipynb
├── Lesson_2_Student_UPDATED.ipynb
├── Lesson_3_Student.ipynb
├── Lesson_4_Student.ipynb
├── Lesson_5_Student_UPDATED.ipynb
└── Lesson_6_Student.ipynb
```

---

## Notebook Overview

### Lesson 1 — Simple ReAct Agent from Scratch (`Lesson_1_Student.ipynb`)
Builds a minimal ReAct-style loop: alternating between “think → act → observe” steps, wiring in a tool call, and iterating until a stopping condition is met. The focus is on understanding the mechanics of an agent loop before using higher-level abstractions.

### Lesson 2 — LangGraph Components (`Lesson_2_Student_UPDATED.ipynb`)
Introduces the building blocks of LangGraph: **state**, **nodes**, **edges**, and how messages and control flow move through a graph. The goal is to learn how to express agent behavior as a graph rather than a single monolithic loop.

### Lesson 3 — Agentic Search (`Lesson_3_Student.ipynb`)
Compares “regular” (single-pass) search with **agentic search**, where the system can iteratively refine queries, gather additional context, and retry when results are insufficient—illustrating why iterative, tool-using loops can outperform one-shot retrieval.

### Lesson 4 — Persistence and Streaming (`Lesson_4_Student.ipynb`)
Covers two production-oriented capabilities:
- **Streaming** model output (e.g., token streaming) for responsive UX
- **Persistence / checkpointing** so long-running or multi-step executions can resume, inspect intermediate state, and support robust execution patterns

### Lesson 5 — Human in the Loop (`Lesson_5_Student_UPDATED.ipynb`)
Demonstrates workflows where humans supervise critical steps: approvals, corrections, or interventions. You’ll see how to **pause**, **inspect**, and **resume** graphs safely, enabling controlled automation in real-world settings.

### Lesson 6 — Essay Writer (`Lesson_6_Student.ipynb`)
An end-to-end mini project that composes multiple steps into a coherent writing workflow (planning → drafting → revising), showcasing how graph structure can coordinate tasks and produce higher-quality long-form output.

---

## API Keys

Set your OpenAI API key:

```bash
export OPENAI_API_KEY=your-key-here
```

Or via a `.env` file:

```
OPENAI_API_KEY=your-key-here
```

---

## Troubleshooting

### “jupyter: command not found”
Install Jupyter into the conda env:
```bash
conda install -c conda-forge jupyterlab ipykernel
```
And launch via:
```bash
python -m jupyter lab
```

### Notebook can’t import a package you just installed
You are almost certainly on the wrong kernel. In Jupyter: **Kernel → Change Kernel → Python (langchain)**.

To confirm inside a notebook cell:
```python
import sys
sys.executable
```

### Don’t use this (common mistake)
```bash
uv pip sync uv.lock
```
`uv.lock` is not a requirements file. Use:
- `uv sync` (for uv-managed project environments), or
- `uv pip compile ...` + `uv pip install -r requirements.locked.txt` (for conda runtime envs)

---

## Requirements

- macOS, Linux, or WSL2
- Conda (Miniconda or Anaconda)
- uv installed
- OpenAI API key
