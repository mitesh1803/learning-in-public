
![Progress](https://img.shields.io/badge/Progress-7%25-orange?style=for-the-badge&logo=progress)

# 🤖 Transformers Part 2 — Self-Attention, KV Cache & Multi-Head Attention

## 📝 Topic: The Engine of Context — How Transformers Actually Understand Language
**Source:** From Text to Attention — How Transformers See Language (Part 2)
**Scope:** Self-Attention → Q/K/V → Attention Formula → KV Cache → Multi-Head Attention
**Date:** June 11, 2026

---

## 🎯 Learning Objectives
* Understand what Self-Attention is and why it is the core of every Transformer.
* Explain the three roles every token plays — Query, Key, and Value.
* Break down the Attention formula step by step: `Softmax(QKᵀ / √dk) V`.
* Understand the Attention Matrix and how it maps word-to-word relationships.
* Trace how attention resolves ambiguous words like "bank" or "it".
* Understand the KV Cache — why the first token is slow and subsequent tokens are fast.
* Explain Multi-Head Attention — why one perspective is never enough.
* Know the trade-off between number of heads and head dimension.

---

## 🔄 Part 1 — Recap: The Pipeline So Far

Before Self-Attention runs, every token has gone through three steps:

```
Raw Text
   ↓
Step 1: TOKENIZATION      → text split into subword token IDs
   ↓
Step 2: EMBEDDING         → each ID → 768-dimensional dense vector (meaning)
   ↓
Step 3: POSITIONAL ENCODING → position vector added (meaning + location)
   ↓
Context-Ready Vectors  →  enter the Self-Attention mechanism
```

Each token now knows **WHAT** it means and **WHERE** it sits. Self-Attention is what lets every token understand its **relationship to every other token**.

---

## ⚡ Part 2 — Self-Attention: The Engine of Context

### 💡 The Core Idea

> *"Every token in a sequence attends to every other token simultaneously, creating a rich, multi-layered map of relationships."*

The key difference from everything that came before:

| Old approach (RNN) | Self-Attention |
|---|---|
| Read words one by one | Process all words simultaneously |
| Memory of earlier words fades | Every word directly connects to every other |
| Sequential — can't parallelize | Parallel — massive GPU efficiency |
| Forgets long-range dependencies | Long-range dependencies are trivial |

### 🎯 Why Context Changes Meaning

Words don't have a single fixed meaning. Their meaning is **refined by the words around them**.

```
"I went to the bank to deposit cash"    → bank = financial institution
"I sat on the bank of the river"        → bank = riverbank
```

Static embeddings give "bank" one fixed vector — a blended average of all its meanings. Self-Attention **updates** that vector based on the surrounding sentence. The same word gets a completely different representation depending on its context.

---

## 🔑 Part 3 — Q, K, V: The Three Roles of Every Word

Every token simultaneously plays three roles in Self-Attention:

| Role | Question it answers | Purpose |
|---|---|---|
| **Query (Q)** | "What am I looking for?" | The search criteria — what this token needs to understand itself |
| **Key (K)** | "What do I contain?" | The label — what other tokens use to find this one |
| **Value (V)** | "What do I offer?" | The actual content — what gets passed through when a match is found |

### 🗄️ The Database Analogy

| Type | Behaviour |
|---|---|
| **Hard Lookup (SQL)** | Exact match only — returns one result or nothing |
| **Soft Lookup (Attention)** | Partial matches everywhere — returns a weighted blend of all values |

Attention is a **soft, probabilistic database query**. When "it" queries the sequence, it doesn't get one result — it gets a weighted mixture of all tokens, with "animal" contributing most and "street" contributing a little.

### 🔢 How Q, K, V Are Computed

Each token's embedding is multiplied by **three separate learned weight matrices**:

```
Input embedding X (shape: seq_len × d_model = 6 × 768)

Q = X @ W_Q     (shape: 6 × 64)   ← the searcher
K = X @ W_K     (shape: 6 × 64)   ← the label
V = X @ W_V     (shape: 6 × 64)   ← the content
```

These matrices (`W_Q`, `W_K`, `W_V`) are **learned during training**. The model learns the best way to project embeddings to make attention maximally useful.

The three perspectives of a single word:
* **Q (Searcher)** — the version of the word used to find relevant context
* **K (Label)** — the version other words use to identify this one
* **V (Content)** — the version that provides information once a match is found

---

## 📐 Part 4 — The Attention Formula

```
Attention(Q, K, V) = Softmax( QKᵀ / √dk ) · V
```

This single equation is the core of modern AI. Breaking it down piece by piece:

### Step 1: Dot Product — `QKᵀ`

```
Q shape: (seq_len × dk) = (6 × 64)
Kᵀ shape: (dk × seq_len) = (64 × 6)
Result: (6 × 6) ← the raw similarity score for every token pair
```

Every Query is compared against every Key. The result is a **square matrix** of raw similarity scores — one number for every (token_i, token_j) pair.

High score = "these two tokens are highly related."

### Step 2: Scale — `/ √dk`

```python
scores = Q @ K.T / math.sqrt(d_k)   # d_k = 64, so √dk ≈ 8
```

**Why divide?** With large dimensions, dot products can become very large numbers. Large values cause Softmax to produce near-zero gradients — the model stops learning. Dividing by `√dk` keeps the scores in a stable range.

### Step 3: Softmax

```python
weights = softmax(scores)   # converts raw scores → probabilities summing to 1
```

Turns the similarity matrix into **attention weights**:
* High-similarity pairs get large weights (e.g. 0.72)
* Low-similarity pairs get near-zero weights (e.g. 0.01)
* All weights for a given query sum to exactly 1.0

### Step 4: Weighted Sum — `· V`

```python
output = weights @ V   # shape: (seq_len × dv)
```

Each token's new representation is a **weighted mixture of all Value vectors**, where the weights come from the attention scores.

### 🧩 The Full Picture

```python
import numpy as np

def attention(Q, K, V):
    d_k = Q.shape[-1]
    scores  = Q @ K.T / np.sqrt(d_k)   # Step 1 + 2: similarity + scale
    weights = softmax(scores)           # Step 3: normalize to probabilities
    output  = weights @ V               # Step 4: weighted blend of values
    return output, weights
```

---

## 🗺️ Part 5 — The Attention Matrix

The output of `QKᵀ` after Softmax is a **square grid** — the Attention Matrix.

```
           "The"  "cat"  "sat"  "on"   "the"  "mat"
"The"  [  0.72   0.10   0.05   0.02   0.08   0.03  ]
"cat"  [  0.05   0.68   0.12   0.03   0.05   0.07  ]
"sat"  [  0.03   0.18   0.55   0.08   0.04   0.12  ]
"on"   [  0.02   0.05   0.09   0.62   0.03   0.19  ]
"the"  [  0.65   0.04   0.03   0.08   0.12   0.08  ]
"mat"  [  0.04   0.08   0.11   0.17   0.06   0.54  ]
```

**Reading the matrix:**
* Row = the token asking ("What should I attend to?")
* Column = the tokens being attended to
* Cell value = attention weight (0.0 to 1.0)
* Each row sums to 1.0

**What to notice:**
* **Diagonal is often bright** — words attend strongly to themselves (self-focus)
* **Pronouns attend to referents** — "it" row lights up at "animal"
* **Verbs attend to subjects** — "sat" row lights up at "cat"
* **Articles attend to nouns** — "the" row lights up at the following noun

---

## 🔍 Part 6 — Concrete Walkthrough: Resolving "it"

**Sentence:** *"The animal didn't cross the street because it was too tired."*

```
Step 1 — THE QUERY
  Token "it" generates its Q vector
  Q says: "What noun am I referring to?"

Step 2 — THE MATCH
  Q_it · K_animal = HIGH similarity  (0.71)
  Q_it · K_street = LOW similarity   (0.12)
  Q_it · K_tired  = MEDIUM           (0.31)

Step 3 — SOFTMAX
  weights: animal=0.71, street=0.12, tired=0.31, others≈0
  (normalized to sum to 1.0)

Step 4 — THE UPDATE
  new_it = 0.71 × V_animal + 0.12 × V_street + 0.31 × V_tired + ...

Step 5 — THE RESULT
  The vector for "it" now contains semantic features from "animal"
  The model now "knows" it refers to the animal, not the street
```

The same mechanism resolves every ambiguity in every sentence — pronouns, metaphors, sarcasm, long-range dependencies — all via weighted mixing of Value vectors.

---

## 💾 Part 7 — KV Cache

### ❓ The Generation Problem

When ChatGPT generates a response token by token, each step runs the full attention computation:

```
Step 1: "The"           → attend → generate "cat"
Step 2: "The cat"       → attend → generate "sat"
Step 3: "The cat sat"   → attend → generate "on"
Step 4: "The cat sat on" → attend → generate "the"
```

### 🗑️ The Waste

At each step, the previous tokens' K and V are **recomputed from scratch** — but they haven't changed:

```
Generating "sat" (step 3):
  K_the, V_the  ← SAME as step 1. Already computed. Wasted.
  K_cat, V_cat  ← SAME as step 2. Already computed. Wasted.
  K_sat, V_sat  ← NEW. Actually needs computing.
```

Only the **new token's Q** changes each step. The K and V of all previous tokens are identical to what was computed before.

### ✅ The Solution: Cache K and V

```
Step 1: compute K_the, V_the      → STORE in cache
Step 2: compute K_cat, V_cat      → STORE in cache
        use cached K_the, V_the
Step 3: compute K_sat, V_sat      → STORE in cache
        use cached K_the, K_cat, V_the, V_cat
Step 4: compute K_on, V_on        → STORE in cache
        use all previously cached K, V
```

**Only compute K, V for the new token. Read everything else from cache.**

### ⏱️ Why the First Token is Slow

```
You send a 500-token prompt to ChatGPT:

FIRST token (slow):
  → Process all 500 prompt tokens
  → Compute K, V for all 500 tokens
  → Fill the KV cache from scratch
  → Generate first response token

SUBSEQUENT tokens (fast):
  → K, V for all 500 prompt tokens already cached
  → Only compute K, V for the 1 new token
  → Attend, generate, cache, repeat
```

This is the difference between **"time to first token"** (TTFT) and **"tokens per second"** (TPS) — two completely different performance metrics.

### 📈 KV Cache Grows With Context

```
Context length  →  Cache size   →  Impact
100 tokens      →  Small        →  Fast, low memory
10,000 tokens   →  Medium       →  Noticeably slower
100,000 tokens  →  Large        →  Memory pressure
128K tokens     →  Huge         →  Why long context is expensive
```

**Why it compounds:** Each Transformer layer stores its own K and V. GPT-4 has ~120 layers.

```
Cache memory = seq_len × num_layers × 2 (K+V) × d_model × bytes_per_param
```

A 128K context window with 120 layers and 768-dim KV = gigabytes of GPU memory just for the cache. This is why long-context inference costs more and runs slower.

---

## 🎭 Part 8 — Multi-Head Attention

### ❓ The Problem with One Head

A single attention head uses one set of W_Q, W_K, W_V matrices — one "perspective" on relationships.

But language has **many simultaneous relationship types**:

| Relationship Type | Example |
|---|---|
| **Syntactic** | Subject → verb agreement |
| **Coreference** | Pronoun → referent ("it" → "animal") |
| **Positional** | Adjacent words influencing each other |
| **Semantic** | Topically related concepts |
| **Long-range** | Clause dependencies across many tokens |

One head cannot capture all of these at once.

### ✅ The Solution: Multiple Independent Heads

Run attention multiple times in parallel, each with its own learned W_Q, W_K, W_V:

```
Head 1: W_Q1, W_K1, W_V1  → may learn syntax (subject-verb)
Head 2: W_Q2, W_K2, W_V2  → may learn coreference (pronoun → noun)
Head 3: W_Q3, W_K3, W_V3  → may learn local context (nearby words)
Head 4: W_Q4, W_K4, W_V4  → may learn topic relationships
...
Head h: its own weights    → its own pattern
```

Each head **independently** attends to the sequence and extracts a different type of relational pattern.

### 🔢 The Math

Instead of one 768-dim attention, split across 8 heads of 96 dimensions each:

```
d_model = 768
num_heads = 8
d_head = 768 / 8 = 96

For each head i:
  Q_i = X @ W_Q_i     (shape: seq_len × 96)
  K_i = X @ W_K_i     (shape: seq_len × 96)
  V_i = X @ W_V_i     (shape: seq_len × 96)
  head_i = Attention(Q_i, K_i, V_i)   (shape: seq_len × 96)

Concatenate all heads:
  [head_1 | head_2 | ... | head_8]    (shape: seq_len × 768)

Project back:
  output = concat @ W_O               (shape: seq_len × 768)
```

Total parameters = same as one big attention head. But now the model has **8 independent perspectives**.

### ⚖️ The Head Trade-off

| Configuration | Head size | Perspectives | Problem |
|---|---|---|---|
| 1 head × 768 dims | 768 | 1 | Limited perspective |
| 8 heads × 96 dims | 96 | 8 | ✅ Good balance |
| 16 heads × 48 dims | 48 | 16 | Each head very narrow |
| 64 heads × 12 dims | 12 | 64 | Too small to capture anything |

> **The Goldilocks rule:** More heads = more perspectives, but each head becomes smaller. Too many heads and each head is too narrow to learn meaningful patterns. Typical practice: **8–32 heads** in production models.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Self-Attention** | Every token attending to every other token in the same sequence simultaneously |
| **Query (Q)** | A token's search vector — what it's looking for in the sequence |
| **Key (K)** | A token's label vector — what it advertises to other tokens' queries |
| **Value (V)** | A token's content vector — what gets mixed into outputs when a match is found |
| **Attention Weight** | The probability (0–1) assigned to a token pair — how much one token attends to another |
| **Attention Matrix** | The seq_len × seq_len grid of attention weights after Softmax |
| **Softmax** | Converts raw scores to probabilities summing to 1 |
| **Scaled Dot-Product** | `QKᵀ / √dk` — similarity score stabilized for training |
| **`dk`** | The dimension of the Key/Query vectors — used for scaling |
| **Weighted Sum** | The final output: each token's new representation = weighted blend of all Value vectors |
| **Context-Aware Representation** | A token vector updated to reflect the meaning of the surrounding sentence |
| **Disambiguation** | Resolving an ambiguous word's meaning using surrounding context |
| **KV Cache** | Storing computed K and V vectors to avoid recomputing them for previous tokens |
| **Time to First Token (TTFT)** | How long before ChatGPT starts responding — slow due to prompt prefill |
| **Tokens Per Second (TPS)** | Generation speed after the first token — fast due to KV cache |
| **Prefill** | The phase where the model processes the full input prompt and fills the KV cache |
| **Multi-Head Attention** | Running multiple independent attention computations in parallel, each with own Q/K/V weights |
| **Head Dimension (`d_head`)** | The dimension per attention head = `d_model / num_heads` |
| **Projection Matrix (W_O)** | The learned matrix that combines all head outputs back to `d_model` dimensions |
| **Permutation Invariance** | Without positional encoding, attention treats any word order the same |

---

## 📂 Summary of Tasks
- ✅ Understood: Self-Attention as global parallel processing vs sequential RNN processing.
- ✅ Understood: Q, K, V — the three simultaneous roles of every token.
- ✅ Understood: The database analogy — hard lookup vs soft weighted blend.
- ✅ Understood: How W_Q, W_K, W_V project embeddings into Q, K, V vectors.
- ✅ Broke down: The Attention formula — dot product, scaling, softmax, weighted sum.
- ✅ Understood: The Attention Matrix — what rows, columns, and cell values mean.
- ✅ Traced: The concrete "it" walkthrough — query → match → update → result.
- ✅ Understood: KV Cache — why K/V are cached but Q is always fresh.
- ✅ Understood: Why the first token is slow (prefill) and subsequent tokens are fast (cache).
- ✅ Understood: Why long context is expensive — cache grows linearly with sequence length.
- ✅ Understood: Multi-Head Attention — parallel perspectives on relationships.
- ✅ Understood: The head trade-off — more heads = smaller per-head dimension.

---

## 💡 My Takeaway

Three things clicked today that didn't before:

**On Q, K, V:** The three-role model stopped being abstract when I mapped it to the database analogy. Q is the search query. K is the index. V is the actual data you retrieve. The difference from SQL is that attention never returns exactly one result — it returns a blend of everything, weighted by relevance. That's the "soft" lookup.

**On the Attention Formula:** The scaling factor `/ √dk` seems minor but it's load-bearing. Without it, large dot products push Softmax into a saturation regime where gradients vanish and the model stops learning. One division by a square root is what keeps billion-parameter models trainable.

**On KV Cache:** The TTFT vs TPS distinction now makes complete sense. ChatGPT "thinking" on a long prompt isn't mysterious — it's computing K and V for every token in your prompt from scratch. Once that's done, each new token is cheap. Long context windows are expensive not because attention is slow per se, but because the cache itself takes gigabytes of GPU memory.

---

## 🔗 Resources
* [Harkirat Singh 100x bootcamp]
* [The Illustrated Transformer — Jay Alammar](https://jalammar.github.io/illustrated-transformer/)
* [Attention Is All You Need — Original Paper](https://arxiv.org/abs/1706.03762)
* [Visualizing Attention — BertViz](https://github.com/jessevig/bertviz)
* [KV Cache Explained — Hugging Face](https://huggingface.co/blog/llm-perf-spotlight)
* [3Blue1Brown — Attention in Transformers](https://www.youtube.com/watch?v=eMlx5fFNoYc)
* [The Annotated Transformer — Harvard NLP](https://nlp.seas.harvard.edu/2018/04/03/attention.html)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*

