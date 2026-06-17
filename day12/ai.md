![Progress](https://img.shields.io/badge/Progress-12%25-orange?style=for-the-badge&logo=progress)


## 📝 Topic: Building & Training a Modern LLM — What Changed Since 2017
**Source:** Fast-Tracking the Course of AI — Presentation Slides  
**Scope:** Classic Transformer → Four Architectural Upgrades → RMSNorm → SwiGLU → RoPE → GQA → The Modern LLM Block  
**Date:** June 17, 2026

---

## 🎯 Learning Objectives
* Understand the Classic Transformer (GPT-2 era) and its four original components.
* Identify the four key architectural upgrades that define every modern open-weight LLM.
* Explain why RMSNorm replaced LayerNorm and what was dropped — and why.
* Understand Pre-Norm vs Post-Norm and why placement of normalization matters at scale.
* Understand how SwiGLU improves information flow over ReLU FFNs using learned gating.
* Explain why Sinusoidal position encoding hit a wall — and how RoPE solved it.
* Understand the KV Cache memory bottleneck and how GQA addresses it.
* See how all four upgrades combine into the Modern Transformer Block used today.

---

## 🏛️ Part 1 — The Classic Transformer (GPT-2 Era, 2019)

The original 2017 "Attention Is All You Need" architecture — essentially GPT-2 — was built on four components:

| Component | Classic Implementation |
|---|---|
| **Position Encoding** | Token Embedding + Sinusoidal Position |
| **Normalization** | LayerNorm (Post-Norm) |
| **Attention** | Multi-Head Attention (MHA) |
| **Feed-Forward** | FFN with ReLU |

This architecture was groundbreaking — but as models scaled to billions of parameters and longer contexts, each of these four components hit a wall.

---

## ⚡ Part 2 — The Four Upgrades

Every modern open-weight LLM — Llama, Mistral, Gemma, DeepSeek — shares the same four architectural improvements over the 2017 original:

| Component | Classic (2017–2019) | Modern (2023–2025) |
|---|---|---|
| **Normalization** | LayerNorm | **RMSNorm** |
| **Feed-Forward** | ReLU FFN | **SwiGLU** |
| **Position Encoding** | Sinusoidal | **RoPE** |
| **Attention** | Multi-Head (MHA) | **Grouped Query (GQA)** |

> *Used by: Llama, Mistral, Gemma, DeepSeek, and virtually every modern open-weight LLM.*

Each upgrade targets a specific bottleneck. The focus shifted from correctness to **stability & efficiency** at scale.

---

## 🔧 Part 3 — RMSNorm

### What Changed?

| Step | LayerNorm | RMSNorm |
|---|---|---|
| **Formula** | Mean + Variance + Normalize + Scale + Shift | RMS + Normalize + Scale |
| **What's gone** | Mean subtraction + bias term (shift) | — |
| **Why removed** | Centering wasn't earning its keep | — |

**RMSNorm** drops mean subtraction and the bias term entirely, maintaining a non-zero mean. This simplifies the computation without losing training stability.

### The Benefit

> *Simpler, faster, and stabilizes training just as well. Crucial for scaling to billions of parameters.*

### The Code

```python
class RMSNorm(nn.Module):
    def __init__(self, dim, eps=1e-6):
        super().__init__()
        self.eps = eps
        self.weight = nn.Parameter(torch.ones(dim))

    def forward(self, x):  # Square, Mean, Root
        rms = torch.sqrt(x.pow(2).mean(-1, keepdim=True) + self.eps)
        return (x / rms) * self.weight
```

**Why it works in 3 points:**

| Property | What It Means |
|---|---|
| **Efficiency** | Fewer operations than LayerNorm |
| **Parameters** | No mean subtraction or bias term |
| **Stability** | Maintains training stabilization |

---

## 📐 Part 4 — Pre-Norm vs Post-Norm

The *placement* of normalization turns out to matter enormously at scale.

| Approach | When It Normalizes | Used By |
|---|---|---|
| **Post-Norm (Original)** | After the sublayer | Classic Transformer (2017) |
| **Pre-Norm (Modern)** | Before the sublayer | All modern LLMs |

**Why Pre-Norm wins:**

In Post-Norm, the LayerNorm sits after attention and FFN layers. In Pre-Norm, normalization is placed inside the residual connection — before the attention and fully connected layers. This produces better gradients and is **essential for deep models with 30+ layers**.

> *Residual connection carries the raw signal. Normalization stabilizes what flows into each sublayer.*

---

## 🔀 Part 5 — SwiGLU

### What is it?

SwiGLU is a smarter version of the Feed-Forward Network (FFN) that uses **learned gating** to control information flow.

```
SiLU(x) = x · σ(x)
```

SiLU is a smooth, non-monotonic alternative to ReLU's hard cutoff.

### How It Works — Two Paths

```
Input
  ├── Path A → Gate (SiLU activation)
  └── Path B → Raw signal

Output = Gate × Raw Signal
```

One path creates a gate using SiLU; the other carries the raw signal. The gate *learns* which information is worth passing forward — a hard cutoff (ReLU) can't do this.

### ReLU vs SiLU

| Property | ReLU | SiLU (SwiGLU) |
|---|---|---|
| **Cutoff** | Hard (zero below 0) | Smooth, non-monotonic |
| **Gating** | None | Learned gate controls flow |
| **Expressiveness** | Lower | Higher — richer representation |

---

## 📍 Part 6 — RoPE (Rotary Position Embedding)

### The Problem with Sinusoidal Positions

The original 2017 position encoding had two structural weaknesses:

| Problem | What It Means |
|---|---|
| **Absolute Encoding** | Encodes "I am token #47" rather than context-aware relationships |
| **No Relative Distance** | Doesn't naturally capture how far apart two tokens are |

### The RoPE Solution

Instead of adding position information to embeddings, RoPE **rotates** the query and key vectors based on their position in the sequence.

> *"Like hands on a clock: the angle between 3 and 5 is the same as between 7 and 9. The gap is what matters."*

**Why this is powerful:**

The dot product between two rotated vectors depends only on their **relative distance**, not their absolute positions. The model naturally understands "these two tokens are 5 positions apart" regardless of where in the sequence they sit.

### RoPE Mechanics

```
For each pair of dimensions (2i, 2i+1):
  → Apply a 2D rotation by angle θ_m
  → θ_m encodes the token's position m
```

| Property | What It Means |
|---|---|
| **Dimension Pairing** | Dimensions are paired and rotated together based on position |
| **Frequency Scaling** | Low dims rotate fast (nearby relationships); high dims rotate slowly (distant relationships) |
| **Relative Distance** | Dot product depends only on gap between tokens, not absolute index |

---

## 💾 Part 7 — The Memory Bottleneck & GQA

### The KV Cache Problem

During generation, the model stores all past Key and Value vectors to avoid recomputation. This is the **KV Cache**.

The math for a standard 32-layer model at 8K tokens:

```
2 × 32 heads × 8192 tokens
× 128 head_dim × 2 bytes
≈ 128 MB per layer

~4 GB TOTAL CACHE
*Just for 8K tokens. 128K would require 64 GB.*
```

**The Wall:** For long contexts, the cache size exceeds GPU memory before the model even starts "thinking."

Memory usage grows linearly with both sequence length and number of heads — Multi-Head Attention doesn't scale.

### GQA — Grouped Query Attention

Instead of every query head having its own key and value, **Grouped Query Attention (GQA)** shares KV pairs across groups of query heads.

| Approach | KV Heads | Notes |
|---|---|---|
| **Multi-Head (MHA)** | 1 KV head per Q head | Full memory cost |
| **Grouped-Query (GQA)** | 1 KV head per group of Q heads | Sweet spot |
| **Multi-Query (MQA)** | 1 KV head for all Q heads | Max compression, quality drop |

> **The Sweet Spot:** 4× less KV cache memory than standard MHA, with almost no drop in model quality.

---

## 🏗️ Part 8 — The Modern Transformer Block

All four upgrades combine into the block that powers today's LLMs:

```
Input Tokens
     │
     ▼
[Position Embedding — RoPE]
     │
     ▼
┌─────────────────────────┐
│  RMSNorm (Pre-Norm)     │  ← Stability
│  Masked Self-Attention  │  ← GQA (memory efficiency)
│  + Residual Connection  │
├─────────────────────────┤
│  RMSNorm (Pre-Norm)     │  ← Stability
│  FFN — SwiGLU           │  ← Richer representation
│  + Residual Connection  │
└─────────────────────────┘
     │
     ▼
Output Token Vectors
```

| Component | Role in the Modern Block |
|---|---|
| **RoPE** | Rotary embeddings for infinite relative context |
| **RMSNorm** | Pre-normalization for extreme training stability |
| **GQA** | KV-cache sharing for memory-efficient inference |
| **SwiGLU** | Gated feed-forward paths for richer representation |

---

## 📊 Part 9 — The Full Comparison

| Feature | Classic (2017) | Modern (2024) |
|---|---|---|
| **Normalization** | LayerNorm (Post-Norm) | RMSNorm (Pre-Norm) |
| **Position** | Absolute Sinusoidal | Rotary (RoPE) |
| **Attention** | Multi-Head (MHA) | Grouped-Query (GQA) |
| **Activation** | ReLU | SwiGLU |
| **Focus** | Correctness | Stability & Efficiency |

The 2017 Transformer was built to work. The 2024 Transformer is built to *scale*.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Classic Transformer** | The original 2017 architecture (GPT-2 era): LayerNorm, Sinusoidal, MHA, ReLU FFN |
| **RMSNorm** | Root Mean Square Normalization — drops mean subtraction and bias for speed and stability |
| **Pre-Norm** | Placing normalization before the sublayer (inside the residual path) for better gradients |
| **Post-Norm** | Placing normalization after the sublayer — the original 2017 approach |
| **SwiGLU** | A gated FFN variant using SiLU activation; learns to gate information flow |
| **SiLU** | Sigmoid Linear Unit — a smooth, non-monotonic activation function: `x · σ(x)` |
| **Sinusoidal Encoding** | Absolute position encoding from the original Transformer; doesn't capture relative distance |
| **RoPE** | Rotary Position Embedding — rotates Q/K vectors by position angle; encodes relative distance |
| **KV Cache** | Stores past Key and Value vectors during generation to avoid recomputation |
| **MHA** | Multi-Head Attention — each query head has its own K and V heads |
| **GQA** | Grouped Query Attention — shares K/V heads across groups of query heads |
| **MQA** | Multi-Query Attention — a single K/V head shared by all query heads |
| **Residual Connection** | A skip connection that adds the input directly to the sublayer output |

---

## 📂 Summary of Tasks
Studied the Classic Transformer (GPT-2 era) and its four original components.
Mapped the four architectural upgrades from Classic (2017) to Modern (2024).
Learned how RMSNorm simplifies LayerNorm by dropping mean subtraction and the bias term.
Implemented RMSNorm in PyTorch and understood its efficiency and stability benefits.
Understood Pre-Norm vs Post-Norm and why modern LLMs normalize before sublayers.
Studied SwiGLU's two-path gating mechanism and how SiLU improves over ReLU.
Learned why Sinusoidal position encoding fails — absolute encoding, no relative distance.
Understood how RoPE encodes relative position by rotating Q/K vectors using the clock analogy.
Calculated the KV Cache memory cost: ~4 GB for 8K tokens on a 32-layer model.
Learned how GQA achieves 4× memory reduction while preserving model quality.
Combined all four upgrades into the Modern Transformer Block used by Llama, Mistral, Gemma, and DeepSeek.

---

## 💡 My Takeaway

Two things fundamentally changed how I think about LLM architecture after going through this:

**On RMSNorm + Pre-Norm:** The original LayerNorm wasn't wrong — it was just doing more work than necessary. Dropping mean subtraction and moving normalization before the sublayer seems like a small tweak, but at 30+ layers and billions of parameters, it's the difference between stable training and divergence. The insight is that simplicity at scale beats theoretical completeness.

**On RoPE + GQA:** Both solve the same class of problem — the original design didn't account for what happens when you scale context length. Sinusoidal encoding knows *where* a token is but not *how far* it is from another. GQA realizes that not every query head needs its own K/V pair — sharing across groups gives you 4× memory savings nearly for free. The pattern: the 2017 Transformer was built to be correct. The 2024 Transformer was re-engineered to be deployable.

The most important meta-lesson: modern LLMs aren't a new invention. They're the same Transformer — with four precisely targeted fixes that together make billion-parameter, long-context training actually feasible.

---

## 🔗 Resources
* [Attention Is All You Need — Original Paper (2017)](https://arxiv.org/abs/1706.03762)
* [RoFormer: Enhanced Transformer with Rotary Position Embedding](https://arxiv.org/abs/2104.09864)
* [GQA: Training Generalized Multi-Query Transformer Models](https://arxiv.org/abs/2305.13245)
* [GLU Variants Improve Transformer (SwiGLU Paper)](https://arxiv.org/abs/2002.05202)
* [Root Mean Square Layer Normalization (RMSNorm)](https://arxiv.org/abs/1910.07467)
* [The Illustrated Transformer — Jay Alammar](https://jalammar.github.io/illustrated-transformer/)
* [Llama 2 Paper — Meta AI](https://arxiv.org/abs/2307.09288)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*