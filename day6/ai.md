![Progress](https://img.shields.io/badge/Progress-6%25-orange?style=for-the-badge&logo=progress)

# 🤖 Transformers Part 1 — From Text to Attention

## 📝 Topic: How Transformers See Language — Tokenization, Embeddings & Positional Encoding
**Source:** From Text to Attention — How Transformers See Language (Part 1)
**Scope:** Tokenization → Embeddings → Positional Encoding → The Input Pipeline
**Date:** June 10, 2026

---

## 🎯 Learning Objectives
* Understand the fundamental challenge of converting text into numbers.
* Compare character-level, word-level, and subword tokenization — and know why subword wins.
* Understand Byte Pair Encoding (BPE) — how modern tokenizers are built.
* Know the practical implications of tokenization: context limits, API cost, and model failures.
* Understand word embeddings — how arbitrary token IDs become meaningful dense vectors.
* Explain semantic vector arithmetic: `king − man + woman ≈ queen`.
* Understand the problem of sequence (permutation invariance) and how positional encoding solves it.
* Trace the full 3-step pipeline: Text → Tokens → Embeddings → Positional Encoding.

---

## 🧠 Part 1 — The Fundamental Challenge

> *"Neural networks only understand NUMBERS. Human language is TEXT."*

The entire field of NLP bottoms out in one problem: we need a **translation layer** that converts text into numbers while **preserving semantic relationships**.

| What we already know | What this deck answers |
|---|---|
| Neural networks process numbers | How does text transform into numbers? |
| Models learn by adjusting weights | What does the model actually "see"? |
| Attention focuses on relevant words | How does attention work mechanically? |

---

## 🔤 Part 2 — Three Approaches to Tokenization

### ❌ Approach 1: Character-Level

Assign a unique integer to every individual character.

```
H → 1
e → 2
l → 3
o → 4

"Hello" → [1, 2, 3, 3, 4]
```

**Why it fails:**

| Problem | Detail |
|---|---|
| **The Step Problem** | "understanding" needs 13 separate processing steps for one concept |
| **Manual Pattern Learning** | Model must learn prefixes/roots from raw letters instead of receiving them directly |
| **High Cognitive Load** | Like teaching reading by showing only letters — grammar is almost impossible to build |
| **Extreme Inefficiency** | Massive data consumed learning basic vocabulary instead of higher-level reasoning |

---

### ❌ Approach 2: Word-Level

Assign one unique integer to every whole word.

```
"The cat sat" → [1, 2, 3]
```

**Why it partially works:**
* Each word = one discrete unit of meaning
* Far more intuitive than character-level

**Why it ultimately fails:**

| Problem | Detail |
|---|---|
| **The Form Problem** | Are "cat", "cats", and "cat's" three different concepts? |
| **The Unknown** | Typos like "teh" or rare technical terms break the system |
| **Out of Vocabulary** | Any word not in training = a "hole" in understanding |
| **Parameter Bloat** | 170,000+ English words → 3 billion parameters just for the lookup table |
| **Multilingual Scaling** | Adding each new language multiplies vocabulary size and cost |

---

### ✅ Approach 3: Subword Tokenization (The Solution)

The **Goldilocks zone** — bigger than characters, smaller than words.

```
"understanding" → ["under", "stand", "ing"]
"don't"         → ["don", "'t"]
"Hello world"   → ["Hello", " world"]   ← space attached to following word
"artificial intelligence" → ["art", "ificial", " intelligence"]
```

**Why it works:**
* Common words stay whole → efficient
* Rare words break into recognizable meaningful pieces → handles unknowns
* A vocabulary of **30k–100k tokens** can represent almost any text in any language
* Completely eliminates the "out of vocabulary" problem

---

## ⚙️ Part 3 — Byte Pair Encoding (BPE)

BPE is the algorithm that builds modern tokenizers. Used by GPT, Claude, and most LLMs.

### 🔢 The 4 Steps

```
1. Initialization    → Start with each character as its own token
2. Iterative Merging → Find the most frequent adjacent token pair → merge into one
3. Optimization      → Repeat until reaching the target vocabulary size (e.g. 50k tokens)
4. Self-Organizing   → Frequent patterns become single tokens; rare ones stay fragmented
```

### 🧪 Example: An Unknown Word

```
"Akwirw_ier"  (an unknown word)
        ↓
Tokens:    ["Ak", "w", "ir", "w", " ", "ier"]
Token IDs: [33901, 86, 343, 86, 220, 959]
```

The tokenizer never crashes on unknown words — it falls back to learned subword fragments.

---

## 💡 Part 4 — Why Tokenization Actually Matters

Tokenization isn't just an implementation detail. It directly affects cost, behavior, and fairness.

### 💰 Operational Impact

| Factor | Detail |
|---|---|
| **Context Limits** | GPT-4's 128K context window = 128K **tokens**, not words. ~1.3 tokens per English word, but code can be much higher |
| **Financial Cost** | API billing is per token, not per character. Understanding token density → cheaper prompts |

### 🐛 Model Behavior Failures

| Failure | Why it happens |
|---|---|
| **Counting letters fails** | "How many 'r's in strawberry?" — the model sees subword units, not individual characters |
| **Word reversal fails** | "Reverse the word 'apple'" — the model may not "see" individual letters |
| **Language inequality** | Different languages need different token counts for the same concept — affecting both cost and model performance globally |

> **Key insight:** When an LLM makes a strange error on a seemingly simple task, tokenization is often the root cause. The model is not seeing what you think it's seeing.

---

## 🔢 Part 5 — From Token IDs to Meaning: Embeddings

### ❓ The Problem with Raw IDs

```
"Hello world" → [15496, 995]
```

These IDs are **arbitrary**. The number 15496 doesn't know it represents a greeting. 995 doesn't encode anything about the Earth. There is zero semantic information in the integers themselves.

### 💡 The Solution: Dense Vectors

Instead of one integer, give each token a **vector** — a list of hundreds of numbers where each dimension encodes a different aspect of meaning.

```
Token ID 15496 ("Hello")
     ↓
[0.12, -0.45, 0.88, 0.03, -0.71, ...]   ← 768 numbers
```

This is the **embedding lookup** — the token ID acts as an index into a large matrix. Row 15496 → the 768-dimensional vector for "Hello".

### 📊 Dimensions of Meaning

Each number in the vector encodes a learnable feature. Dimensions like "royalty", "gender", or "animacy" **emerge automatically during training** — nobody labels them.

```
         Royalty   Gender(M)   Edibility
King      0.98       0.95        0.01
Queen     0.97       0.05        0.02
Apple     0.02       0.00        0.94
```

King and Queen cluster together on royalty. Apple lands far away from both.

### 🗺️ Words as Points in Space

```
         semantic space (2D projection)

  king • queen               Apple (Corp) •
  man  • woman
                   apple • orange
  cat  • dog
              car • bicycle
```

**Proximity = shared meaning.** Similar words cluster. Unrelated words are far apart.

### ✨ Semantic Vector Arithmetic

The geometry of meaning-space is consistent across the entire vocabulary:

```
king  − man  + woman ≈ queen
Paris − France + Italy  ≈ Rome
Walking − Walk + Swim  ≈ Swimming
```

This works because embeddings encode **relations as consistent directional offsets**. The man→woman vector mirrors the king→queen vector exactly.

### 📌 The Embedding Lookup Table

```
Embedding Matrix (vocab_size × 768):

Row 0:     [0.21, -0.14, ...]   ← token 0
Row 1:     [0.88,  0.03, ...]   ← token 1
...
Row 15496: [0.12, -0.45, ...]   ← "Hello"
Row 995:   [0.44,  0.91, ...]   ← "world"
```

Looking up an embedding is just a **row indexing operation** — fast, simple, and learned during training.

### ⚠️ The Limitation of Raw Embeddings

Embeddings are **"bag-of-words" by default**. They represent the isolated meaning of a token, completely ignoring word order.

```
"The dog bit the man"  →  same embeddings as
"The man bit the dog"
```

Both sentences produce the same set of embedding vectors. The model can't tell them apart yet. This is called **permutation invariance** — it sees a set of words, not a sequence.

---

## 📍 Part 6 — Positional Encoding

### ❓ The Problem

The Transformer processes all tokens in **parallel**. Unlike RNNs, there is no inherent sense of order. Without extra information:

```
"The dog bit the man"   ←  identical representation to
"The man bit the dog"
```

Position is the primary driver of meaning in language. We must inject "**where**" a word is into "**what**" a word is.

### 💡 The Solution: Add a Position Vector

A small **positional vector** is added to each token's embedding before it enters the Transformer:

```
Final Input = Token Embedding + Positional Encoding

"dog" at position 2:
  embedding:  [0.44,  0.91, -0.23, ...]   ← what "dog" means
  pos vector: [0.84, -0.54,  0.14, ...]   ← position 2 fingerprint
  combined:   [1.28,  0.37, -0.09, ...]   ← meaning + location
```

Now the model knows both what each token means and where it sits in the sequence.

### 〰️ Sinusoidal Positional Encoding

The original Transformer uses **sine and cosine waves** of different frequencies:

```
PE(pos, 2i)   = sin(pos / 10000^(2i / d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i / d_model))
```

Each position gets a unique combination of sine/cosine values across all dimensions — a **unique fingerprint** for every position.

### ✅ Why Sine Waves?

| Property | Why It Matters |
|---|---|
| **Relative Position** | Position `p+k` can be expressed as a linear function of position `p` — the model learns relative distances naturally |
| **Generalization** | Continuous waves allow the model to extrapolate to sequences longer than training data |
| **No Extra Parameters** | Sinusoidal encoding is fixed and pre-calculated — nothing new to learn |
| **Unique Fingerprint** | Multiple frequencies ensure every position in high-dimensional space is distinct |

---

## 🔄 Part 7 — The Full Input Pipeline

```
Raw Text: "The cat sat on the mat"
        ↓
Step 1: TOKENIZATION
        ["The", " cat", " sat", " on", " the", " mat"]
        → Token IDs: [464, 3797, 3332, 319, 262, 2603]
        ↓
Step 2: EMBEDDING LOOKUP
        Each ID → a 768-dimensional dense vector
        [464] → [0.12, -0.45, 0.88, ...]
        [3797] → [0.44, 0.91, -0.23, ...]
        ...
        ↓
Step 3: POSITIONAL ENCODING
        Add position fingerprint to each embedding vector
        Token 0 "The"  + pos_0_vector
        Token 1 "cat"  + pos_1_vector
        Token 2 "sat"  + pos_2_vector
        ...
        ↓
Context-Ready Vectors → enter the Transformer blocks
```

Each token now encodes **WHAT it means** (embedding) + **WHERE it is** (positional encoding). These vectors are ready for the Self-Attention mechanism.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Token** | The basic unit a model processes — a character, word, or subword fragment |
| **Tokenizer** | The system that converts raw text into a sequence of token IDs |
| **BPE** | Byte Pair Encoding — builds a vocabulary by iteratively merging the most frequent character pairs |
| **Vocabulary** | The fixed set of all tokens the model knows (typically 30k–100k) |
| **Out of Vocabulary (OOV)** | A word not in the training vocabulary — subword tokenization eliminates this |
| **Subword** | A token that is part of a word — the Goldilocks unit for tokenization |
| **Token ID** | An arbitrary integer assigned to a token — has no semantic meaning on its own |
| **Embedding** | A dense vector of floats that encodes the semantic meaning of a token |
| **Embedding Matrix** | A learned lookup table mapping every token ID to its embedding vector |
| **Semantic Space** | The high-dimensional vector space where tokens live — proximity = similarity |
| **Vector Arithmetic** | Mathematical operations on embeddings that produce semantically meaningful results |
| **Bag-of-Words** | A representation that ignores word order — the default limitation of raw embeddings |
| **Permutation Invariance** | The property where a model treats `[A, B, C]` and `[C, A, B]` identically |
| **Positional Encoding** | A vector added to each embedding to inject sequence position information |
| **Sinusoidal Encoding** | Positional encoding using sine/cosine waves at different frequencies |
| **Context Window** | The maximum number of tokens a model can process at once |
| **Token Density** | How many tokens a given piece of text produces — varies by language and content type |
| **Dense Vector** | A vector where most values are non-zero — contrasted with sparse one-hot vectors |
| **t-SNE** | A dimensionality reduction technique used to visualize high-dimensional embeddings in 2D/3D |

---

## 📂 Summary of Tasks
- ✅ Understood: The fundamental challenge — text must become numbers that preserve meaning.
- ✅ Understood: Character-level tokenization and its 4 failure modes.
- ✅ Understood: Word-level tokenization and the vocabulary explosion problem.
- ✅ Understood: Subword tokenization — the Goldilocks solution.
- ✅ Understood: BPE — the 4-step algorithm that builds modern tokenizers.
- ✅ Understood: Why tokenization affects context limits, API cost, and model failures.
- ✅ Understood: How token IDs become dense semantic vectors via the embedding lookup.
- ✅ Understood: Semantic vector arithmetic — `king − man + woman ≈ queen`.
- ✅ Understood: The bag-of-words limitation and permutation invariance problem.
- ✅ Understood: Positional encoding — injecting "where" into "what".
- ✅ Understood: Why sinusoidal encoding was chosen — relative position, generalization, no extra params.
- ✅ Traced: The full 3-step pipeline from raw text to context-ready vectors.

---

## 💡 My Takeaway

Two things fundamentally shifted today:

**On Tokenization:** The letter-counting failure of LLMs always seemed like a bug. Now I understand it's structural. When GPT sees "strawberry", it doesn't see 10 individual characters — it sees 2-3 subword tokens. Asking it to count 'r's is like asking someone to count letters in a word they can only read in syllables. The failure isn't stupidity, it's a direct consequence of how tokenization works.

**On Positional Encoding:** The elegance of adding a sinusoidal vector to each embedding is that it's completely free — no extra parameters, no extra training. The wave patterns are pre-calculated and fixed. The model just learns to use position information that's already baked into every vector it receives. That's the kind of design decision that makes an architecture last.

---

## 📈 Next Up
**Transformers Part 2** — Self-Attention: how every token computes its relationship to every other token using Queries, Keys, and Values, and how that produces the context-aware representations that make LLMs work.

---

## 🔗 Resources
* [Harkirat Singh 100x bootcamp]
* [The Illustrated Transformer — Jay Alammar](https://jalammar.github.io/illustrated-transformer/)
* [OpenAI Tokenizer Playground](https://platform.openai.com/tokenizer) — visualize exactly how GPT tokenizes your text
* [BPE Paper — Sennrich et al.](https://arxiv.org/abs/1508.07909)
* [Attention Is All You Need — Original Paper](https://arxiv.org/abs/1706.03762)
* [Word2Vec Explained](https://www.youtube.com/watch?v=LSS_bos_TPI)
* [3Blue1Brown — Neural Networks](https://www.3blue1brown.com/topics/neural-networks)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*
