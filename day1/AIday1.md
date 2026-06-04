# Phase 0 — Setup & Tooling

> *Your tools shape your thinking. Set them up once, set them up right.*

**Status:** ✅ Completed  
**Course:** [AI Engineering from Scratch](https://github.com/rohitg00/ai-engineering-from-scratch)

---

## Overview

Phase 0 is the foundation layer. Before writing a single line of model code, the entire engineering environment needs to be production-ready. This phase covers every layer of the AI engineering stack — from shell configuration to Docker orchestration.

Most people skip this phase. Then they spend hours debugging import errors, version conflicts, and missing CUDA drivers instead of learning.

---

## Lessons Completed

### 01 · Dev Environment

Set up the four-layer AI engineering stack from scratch:

```
4. AI/ML Libraries   → PyTorch, transformers, etc.
3. Language Runtimes → Python 3.12, Node 22, Rust
2. Package Managers  → uv, pnpm, cargo
1. System Foundation → OS, shell, git, editor, GPU drivers
```

**Key tools installed:**
- `uv` for Python — 10–100x faster than pip, handles venvs automatically
- `fnm` + `pnpm` for Node.js
- `rustup` for Rust toolchain

**Verification script:** `phases/00-setup-and-tooling/01-dev-environment/code/verify.py`

**What I learned:** Installing bottom-up matters. Skipping a layer (especially GPU drivers before PyTorch) creates silent failures that show up 20 lessons later.

---

### 02 · Git & Collaboration

Configured git identity, daily workflow, and experiment branching.

**The only commands that matter for this course:**

| Command | When |
|---------|------|
| `git clone` | Get the course repo |
| `git add` + `git commit` | Save your work |
| `git push` | Back it up |
| `git checkout -b` | Isolate experiments |
| `git log --oneline` | Review what you built |

**What I learned:** `git checkout -b experiment/new-optimizer` before every experiment. Merging or discarding is easy. Undoing 40 commits on main is not.

---

### 03 · GPU Setup & Cloud

Verified local GPU access, benchmarked CPU vs GPU matrix multiplication, and configured Google Colab as a fallback.

**Key concept — the fp16 memory rule:**
```
Model VRAM footprint ≈ 2 bytes × number of parameters
A 7B parameter model = ~14 GB in fp16
```

**Benchmark result on my machine:**
```python
# CPU: ~3.8s for 5000×5000 matrix multiplication
# GPU: benchmarked on Colab T4
# Speedup: ~45x
```

**What I learned:** `device = torch.device("cuda" if torch.cuda.is_available() else "cpu")` — always write device-agnostic code. The lessons that need GPU say so explicitly.

---

### 04 · APIs & Keys

Secured API key management and made first API calls via SDK and raw HTTP.

**Key rule:** Never put API keys in source code. Always use environment variables or `.env` files (gitignored).

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
```

**Did both approaches:**
- SDK call (Anthropic Python client)
- Raw HTTP via `urllib.request` — understanding what the SDK does under the hood helps when debugging

**What I learned:** The raw HTTP call demystifies everything. It's just a POST with an API key header. The SDK is a thin wrapper. Knowing this saves debugging time.

---

### 05 · Jupyter Notebooks

Installed JupyterLab, learned keyboard shortcuts, magic commands, and the critical rules for not shooting yourself in the foot.

**The three notebook traps:**
1. **Out-of-order execution** — always Kernel > Restart & Run All before sharing
2. **Hidden state** — deleted cells leave variables in memory
3. **Memory leaks** — load large datasets, never free them

**Magic commands I now use constantly:**
```python
%timeit np.random.randn(10000)   # benchmark a line
%%time                            # benchmark a whole cell
%matplotlib inline                # plots in-notebook
```

**The rule:** Explore in notebooks. Ship in scripts.

---

### 06 · Python Environments

Solved dependency hell with isolated virtual environments and `pyproject.toml`.

**Why this matters in ML specifically:** PyTorch, JAX, and TensorFlow each ship their own CUDA bindings. A global `pip install` overwrites whatever was there before.

**Course strategy — per-phase environments:**
```
ai-engineering-from-scratch/
├── .venv/                  ← phases 0–3
├── phases/
│   ├── 04-neural-networks/.venv/  ← PyTorch
│   └── 11-llm-apis/.venv/         ← API SDKs only
```

**`pyproject.toml` with optional dependency groups:**
```toml
[project.optional-dependencies]
torch = ["torch>=2.3", "torchvision>=0.18"]
llm   = ["anthropic>=0.39", "openai>=1.50"]
```

**What I learned:** Always activate before installing. Check `which python` to verify. Commit the lockfile, not the `.venv` folder.

---

### 07 · Docker for AI

Built a GPU-enabled Docker image and a multi-service Compose stack.

**Why Docker matters more in AI than in most fields:**
- CUDA 12.4 code doesn't run on CUDA 11.8
- Model weights are 14 GB — volumes prevent re-downloading on every rebuild
- Real AI apps are multi-service: inference server + vector DB + web layer

**Key Compose stack:**
```yaml
services:
  ai-dev:   # PyTorch + Jupyter, GPU-enabled
  qdrant:   # Vector database for RAG experiments
```

**What I learned:** The NVIDIA Container Toolkit is the bridge between Docker and the host GPU driver. One command — `--gpus all` — exposes the GPU inside the container. Without the toolkit installed, that flag silently does nothing.

---

### 08 · Editor Setup

Configured VS Code with the extensions and settings that matter for AI work.

**Essential extensions:**
- `ms-python.python` + `ms-python.vscode-pylance` — type checking, autocomplete
- `ms-toolsai.jupyter` — notebooks inside the editor
- `ms-vscode-remote.remote-ssh` — edit code on remote GPU boxes as if local
- `charliermarsh.ruff` — fast linting

**Key settings:**
```json
{
  "python.analysis.typeCheckingMode": "basic",
  "editor.formatOnSave": true,
  "notebook.output.scrolling": true,
  "editor.rulers": [88, 120]
}
```

**What I learned:** Remote SSH is the most underrated extension in the list. SSH into a Lambda GPU instance and edit, run, and debug as if you're local. It changes the workflow completely.

---

### 09 · Data Management

Loaded, streamed, split, and versioned datasets using the Hugging Face `datasets` library.

**Format comparison:**

| Format | Size | Speed | Best For |
|--------|------|-------|----------|
| CSV | Large | Slow | Human readability |
| JSON | Large | Slow | APIs, nested data |
| Parquet | Small | Fast | Storage, analytics |
| Arrow | Small | Fastest | In-memory processing |

**Streaming large datasets:**
```python
# Load row-by-row without downloading the full dataset
dataset = load_dataset("wikimedia/wikipedia", streaming=True)
```

**The three-way split with fixed seed:**
```python
split = dataset.train_test_split(test_size=0.2, seed=42)
# Always set seed. Same seed = same split = reproducible experiments.
```

**What I learned:** The HF `datasets` library handles caching automatically at `~/.cache/huggingface/`. Once downloaded, data loads instantly. Parquet is the right storage format for anything you're keeping.

---

### 10 · Terminal & Shell

Learned piping, redirects, background processes, and tmux for managing training runs.

**The setup I now use for every training run:**
```bash
tmux new -s train
# Pane 1: python train.py
# Pane 2: watch -n1 nvidia-smi
# Pane 3: tail -f logs/experiment.log | grep loss
# Ctrl+B, d → detach. Training keeps running.
```

**Redirects I use constantly:**
```bash
python train.py > output.log 2>&1  # both stdout and stderr to file
tail -f train.log | grep --line-buffered "loss"  # live loss monitoring
```

**What I learned:** `nohup` keeps a process alive after terminal close but you can't reattach. tmux lets you reattach. Always use tmux for anything longer than a few minutes.

---

### 11 · Linux for AI

Survival guide for operating on a remote Linux GPU box — because that's where all serious AI work runs.

**The 15 commands that cover 95% of what you'll do:**
```bash
# Move around
pwd, ls -la, cd, find

# Files
cp -r, mv, rm -rf, mkdir -p

# Read
cat, head -20, tail -f, less

# Search
grep -r, find . -name "*.pt"

# Permissions
chmod +x, sudo

# Packages
sudo apt update && sudo apt install -y

# Processes
htop, ps aux | grep python, kill

# Disk
df -h, du -sh *
```

**macOS → Linux gotcha I hit:** `sed -i '' 's/a/b/' file` on macOS. On Linux it's `sed -i 's/a/b/' file`. The empty string breaks it.

**What I learned:** Case-sensitivity. `Model.py` and `model.py` are two different files on Linux. Named my files badly and spent 20 minutes confused.

---

### 12 · Debugging & Profiling

Built a debugging toolkit for catching AI-specific bugs before they waste compute.

**The core insight:**
> The worst AI bugs don't crash. They train silently on garbage and report a beautiful loss curve.

**The four silent failures to always check for:**

1. **Shape mismatch** — use forward hooks to map every tensor shape through the model
2. **NaN loss** — conditional `breakpoint()` when `loss > 100` or `torch.isnan(loss)`
3. **Data leakage** — check train/test ID overlap before training, not after getting 99% accuracy
4. **Wrong device** — all tensors and the model must be on the same device

**My debugging workflow:**
```
Before training  → check_shapes() with a sample batch
First 10 steps   → debug_print() on loss, outputs, gradients
During training  → TensorBoard: loss, lr, gradient norms
When it breaks   → breakpoint() at the failure point
For performance  → time data loading vs forward vs backward
```

**What I learned:** Data loading is usually the bottleneck, not the GPU. `num_workers > 0` in your DataLoader is often a bigger win than a faster GPU card.

---

## Key Takeaways from Phase 0

**1. Install bottom-up, always.** System → package managers → runtimes → AI libraries. Missing a layer creates silent failures.

**2. Every project gets its own virtual environment.** In ML especially — framework version conflicts are real and painful.

**3. tmux is not optional.** Training runs outlive terminal sessions. Always run long jobs inside a named tmux session.

**4. The worst bugs don't crash.** Silent failures — NaN gradients, data leakage, wrong device — are harder to catch than exceptions. Build checks for them from the start.

**5. Notebooks for exploration, scripts for production.** Notebooks are lab benches. Ship what works as `.py` files.

---

## What's Next

**Phase 1: Math Foundations**

Linear algebra, calculus, probability, and statistics — the mathematical backbone that makes everything in phases 2–20 make sense rather than feel like magic.

---

## Resources

- [Course repo: ai-engineering-from-scratch](https://github.com/rohitg00/ai-engineering-from-scratch)
- [uv docs](https://docs.astral.sh/uv/)
- [JupyterLab docs](https://jupyterlab.readthedocs.io/)
- [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
- [Hugging Face datasets](https://huggingface.co/docs/datasets)
- [tmux cheat sheet](https://tmuxcheatsheet.com/)

---

