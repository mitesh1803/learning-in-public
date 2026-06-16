# 🤖 Deep Learning Class 4 — Tensors, Matrices & PyTorch
**📝 Topic: The Math & Tools — The Engine Under Every AI Model**

**Scope:** Scalars → Vectors → Matrices → Tensors → Matrix Transformations → PyTorch → Autograd → nn.Module
**Date:** June 16, 2026

---

## 🎯 Learning Objectives
- Understand the dimensional hierarchy: Scalar → Vector → Matrix → Tensor.
- Explain why tensors are the natural data structure for deep learning (batches, heads, positions).
- Understand how a matrix is not just storage — it is a **transformation machine**.
- Break down matrix multiplication as the single operation behind every forward pass, backprop step, and attention score.
- Understand why GPUs exist and what "embarrassingly parallel" means.
- Explain the dot product as a **similarity measure** and connect it to attention scores.
- Use PyTorch tensors: create, move to GPU, reshape, broadcast, and multiply.
- Understand Autograd — what `requires_grad`, `.backward()`, and `zero_grad()` actually do internally.
- Build a model using `nn.Module` and know what `nn.Linear` contains under the hood.
- Recite and explain the **5-line training loop** from memory.

---

## 🔁 Part 1 — Recap: What's Been Hiding in Plain Sight

Every operation from the previous classes, unwrapped:

```
// Neural Networks
weights * inputs + bias
backprop(gradients)

// Transformers
Q = X * Wq
attention = softmax(Q * K.T) * V
```

**What were ALL of these, really?**

> Matrix Multiplication. That's the secret.

Every weight update, every attention score, every embedding lookup — it was always just matmul.

---

## 🧱 Part 2 — The Dimensional Hierarchy

### Scalar — 0D Tensor
A single number. No direction, no structure.

```python
temperature = 72      # °F
user_age    = 25
price       = 4.99
```

📐 Dimensions: **0** — a single point in space.

---

### Vector — 1D Tensor
A list of numbers. Has both magnitude and direction.

```python
location  = [28.6139, 77.2090]     # lat, long
rgb_color = [255, 87, 51]          # red, green, blue
movement  = [3, 4]                 # 3 right, 4 up
```

📐 Dimensions: **1**

You've already seen vectors without realising it:

```python
king_vector  = [0.2, -0.5, 0.8, 0.1, ...]   # 768-dim word embedding
features     = [x₁, x₂, x₃]                 # input to a neuron
```

---

### Matrix — 2D Tensor
A grid of numbers. Indexed by `(row, col)`.

```
[ 1  4  7 ]
[ 2  5  8 ]
[ 3  6  9 ]
```

📐 Dimensions: **2**

Matrices you've already used:

```python
# Class 2 — Neural Networks
W = WeightMatrix()
output = W × input       # transforms one vector into another

# Class 3 — Transformers
Q = input × Wq
K = input × Wk
scores = Q × K.T         # pure matrix math
```

---

### Tensor — ND Generalization

```
Scalar  →  just a number       →  0D Tensor
Vector  →  list of numbers     →  1D Tensor
Matrix  →  grid of numbers     →  2D Tensor
Tensor  →  cube / hypercube    →  3D, 4D, 5D...
```

> It's not a scary new concept — it's just "What if we keep adding dimensions?"

### Why Tensors in Deep Learning?

```python
# A single sentence: "The cat sat" (3 tokens, 768 dims)
sentence = Matrix(3, 768)       # Shape: [3, 768]  → 2D Tensor

# A batch of 8 sentences
batch = Stack(8, 3, 768)        # Shape: [8, 3, 768] → 3D Tensor

# After splitting into 12 attention heads
attention = Split(8, 12, 3, 64) # Shape: [8, 12, 3, 64] → 4D Tensor
```

We need tensors because our data has **structure** — batch, position, head, dimension — all at once.

### Shape — The Most Important Property

```python
# Reading [8, 12, 3, 64]:
#   8  → Batch Size
#  12  → Attention Heads
#   3  → Number of Tokens
#  64  → Head Dimension

# Pro-tip: If you understand shape, you can read any model.
```

---

## 🔄 Part 3 — Matrices as Transformation Machines

> A matrix isn't just storage. A matrix is a **TRANSFORMATION**.

It takes a vector in → and gives a **different vector** out. It's a machine that processes data.

**Capabilities:** Rotate · Scale · Stretch · Reflect · Project · Transform meaning

### Rotation Example

```
[-1  0]   [1]   [0]
[ 0  1] × [0] = [1]

Input: [1, 0] (pointing right)
Output: [0, 1] (pointing up!)
The matrix rotated the vector 90°.
```

### Scaling Examples

```python
# Uniform Scale (2x)
[2  0] × [3] = [6]
[0  2]   [4]   [8]

# Stretch & Squish
[3    0  ] × [3] = [9]    ← stretched 3x horizontally
[0   0.5]   [4]   [2]    ← squished 0.5x vertically
```

### Key Insight

> In a neural network, the model **learns** which matrix to use.
> Training is just finding the right transformation for your data.

---

## ✖️ Part 4 — Matrix Multiplication: The Core Operation

### Why MatMul is EVERYTHING

```python
Forward Pass:      X @ W            # learned transformations
Backpropagation:   dL/dY @ W.T      # computing gradients
Attention:         Q @ K.T          # measuring similarity
Word Embeddings:   one_hot @ W_embed # looking up meaning
```

> Every single computation in a modern AI model is essentially a series of matrix multiplications.

### Why GPUs Exist

| Architecture | Cores | Optimized For |
|---|---|---|
| CPU | 8–16 (complex) | Sequential tasks — logic, branching |
| GPU | 5,000+ (simple) | Parallel tasks — same math, thousands of times |

Matrix multiplication is **"embarrassingly parallel"** — every cell in the output matrix can be calculated simultaneously, independently.

---

## 🔵 Part 5 — The Dot Product: The Atomic Operation

```python
v1 = [1, 2, 3]
v2 = [4, 5, 6]

v1 · v2 = (1*4) + (2*5) + (3*6) = 32
```

**Conceptual meaning:**

```
Dot Product  =  Similarity

High value   →  vectors pointing in the same direction
Low/Zero     →  vectors pointing in different directions
```

### Connection Back to Attention

```python
# "The animal didn't cross the street because it was too tired."

Q_sat · K_the    = 0.1
Q_sat · K_cat    = 0.8   # ← high similarity!
Q_sat · K_sat    = 0.9

score = softmax(Q @ K.T)
# The W matrices rotate vectors into a space where
# "relevant" and "similar direction" mean the same thing.
```

Attention scores **are** dot products. The model learns the right rotation.

---

## 🔥 Part 6 — PyTorch: The Tool

### What is PyTorch?

> If deep learning is the car, PyTorch is the engine and dashboard.

Used by: OpenAI · Anthropic · Google DeepMind · Tesla

**Core Capabilities:**

| # | Feature | What it gives you |
|---|---|---|
| 01 | Tensors on GPU | ~100x speedup over CPU math |
| 02 | Autograd | Automatic differentiation — no manual backprop |
| 03 | nn.Module | High-level building blocks for any model |

---

### Tensors in PyTorch

```python
import torch

x = torch.tensor(5.0)                  # Scalar — 0D
v = torch.tensor([1.0, 2.0, 3.0])      # Vector — 1D
m = torch.tensor([[1, 2], [3, 4]])     # Matrix — 2D
t = torch.zeros(2, 3, 4)              # 3D Tensor

print(t.shape)   # torch.Size([2, 3, 4])
# Pro-tip: You will check .shape more than anything else.
```

---

### GPU Acceleration — The Magic Line

```python
x = torch.randn(1000, 1000)             # CPU tensor

device = "cuda" if torch.cuda.is_available() else "cpu"
x = x.to(device)                        # ← THE MAGIC LINE

# Operations now happen across 5,000+ GPU cores
# Result: up to 100x speedup for large matrix math
```

---

### Matrix Multiply — The @ Operator

```python
A = torch.randn(3, 4)   # Shape: [3, 4]
B = torch.randn(4, 5)   # Shape: [4, 5]

C = A @ B               # Shape: [3, 5]
# Or: C = torch.matmul(A, B)

# The Rule: Inner dimensions MUST match!
# (3, 4) @ (4, 5) → (3, 5)
```

---

### Reshaping Tensors

```python
# .view() — Reinterpreting data
x = torch.randn(8, 3, 768)
y = x.view(24, 768)         # [8, 3, 768] → [24, 768]
# Flattened batch × tokens (8 * 3 = 24)

# .permute() — Swapping axes
x = torch.randn(8, 12, 3, 64)
y = x.permute(0, 2, 1, 3)  # [8, 12, 3, 64] → [8, 3, 12, 64]
# Swapping Heads and Tokens for attention calculation

# The Rule: Total number of elements must remain exactly the same.
```

---

### Broadcasting — PyTorch's Secret Weapon

```python
A = torch.ones(3, 4)               # Shape: [3, 4]
B = torch.tensor([1, 2, 3, 4])     # Shape: [4]

C = A + B                          # Result: [3, 4]
# PyTorch "stretches" B to match every row of A automatically.

# The Rule: Dimensions must be equal OR one of them must be 1.
```

---

## 🤖 Part 7 — Autograd: The Real Reason PyTorch Exists

### How it Works Internally

When you create a tensor with `requires_grad=True`, PyTorch starts **rolling a tape** — recording every operation applied to it.

```
01. Tracks every operation  →  PyTorch remembers how you got to the result
02. Builds a graph          →  A dynamic computation graph of all tensors
03. Computes gradients      →  One .backward() call finds all derivatives
```

> You define the forward pass. PyTorch handles the backward pass.

### Autograd in Action

```python
x = torch.tensor(2.0, requires_grad=True)

y = x**2 + 5              # function: y = x² + 5

y.backward()              # compute all gradients automatically

print(x.grad)             # tensor(4.0)  ← dy/dx = 2x = 2(2) = 4
```

### The Computation Graph

```
x → **2 → +5 → y
      ← .backward() traverses this in reverse
      ← applies chain rule at every node
```

Backpropagation is just a **reverse traversal** of this graph.

### Manual vs Autograd — Side by Side

```python
# Manual Backprop (Painful — one mistake = model never trains)
d_loss_y  = 2 * (y_pred - y_true)
d_y_relu  = 1 if z > 0 else 0
d_relu_w  = x.T
d_loss_w  = d_relu_w @ (d_loss_y * d_y_relu)
w        -= lr * d_loss_w

# PyTorch Autograd (Magic)
loss = criterion(y_pred, y_true)
loss.backward()       # ← ALL OF THE ABOVE IN ONE LINE
optimizer.step()
```

### Why `zero_grad()` and `no_grad()` Exist

```python
# torch.no_grad()       → stops PyTorch from recording (for inference)
# optimizer.zero_grad() → PyTorch accumulates gradients by default.
#                         If you call .backward() twice without clearing,
#                         gradients ADD UP → model explodes.
```

---

## 🏗️ Part 8 — nn.Module: Building Models

### The Pattern

```python
class SimpleNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.layer1 = nn.Linear(784, 128)
        self.relu   = nn.ReLU()
        self.layer2 = nn.Linear(128, 10)

    def forward(self, x):
        x = self.layer1(x)
        x = self.relu(x)
        x = self.layer2(x)
        return x

# Declare layers in __init__ — each is a matrix (a transformation).
# Define data flow in forward. That's the entire model.
```

### What's Inside `nn.Linear`?

```python
layer = nn.Linear(3, 2)

layer.weight   # Shape: [2, 3]
               # [[0.12, -0.45, 0.89],
               #  [-0.34, 0.22, -0.11]]

layer.bias     # Shape: [2]
               # [0.05, -0.02]

# Under the hood: y = x @ W.T + b
# It's just Matrix Multiplication + Vector Addition.
```

`nn.Linear(784, 128)` creates a **128×784 weight matrix**. Every forward pass does `input @ weight.T + bias`.

---

## 🔁 Part 9 — The 5-Line Training Loop

```python
for epoch in range(10):
    y_pred = model(x_batch)          # 1. Forward Pass
    loss   = criterion(y_pred, y_batch)  # 2. Compute Loss
    optimizer.zero_grad()            # 3. Clear Old Gradients
    loss.backward()                  # 4. Backward Pass (The Calculus)
    optimizer.step()                 # 5. Update Weights
```

### The 5 Lines, Decoded

| Step | Name | What it does |
|---|---|---|
| 1 | Forward | Predict using current weights. "How did we do?" |
| 2 | Loss | Measure the error. "How wrong were we?" |
| 3 | zero_grad | Flush the toilet. Don't let old gradients leak. |
| 4 | Backward | Find how each weight caused the error. |
| 5 | Step | Nudge weights in the right direction. |

> ⚠️ If you miss step 3, gradients accumulate and your model **explodes**.

This loop is the heartbeat of every deep learning model — from MNIST to GPT-4, the pattern is the same.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| Scalar | A single number — 0D tensor |
| Vector | A list of numbers with direction — 1D tensor |
| Matrix | A 2D grid of numbers — a transformation machine |
| Tensor | The generalization: N-dimensional array |
| Shape | How many dimensions exist and how big each is — the most important property |
| Matrix Multiplication | The single core operation behind every deep learning computation |
| Dot Product | Scalar result of two vectors — measures similarity / alignment |
| Embarrassingly Parallel | A computation where every output cell is independent — ideal for GPUs |
| PyTorch | The standard deep learning framework — tensors, autograd, nn.Module |
| `requires_grad` | Flag that tells PyTorch to start recording a computation tape on this tensor |
| Autograd | PyTorch's automatic differentiation engine — computes gradients via graph traversal |
| Computation Graph | The dynamic directed graph PyTorch builds tracking every operation |
| `.backward()` | Traverses the computation graph in reverse, applying the chain rule at every node |
| `zero_grad()` | Clears accumulated gradients — must be called before every `.backward()` |
| `torch.no_grad()` | Pauses gradient tracking — used during inference to save memory |
| `nn.Module` | PyTorch's model container — layers in `__init__`, data flow in `forward` |
| `nn.Linear(in, out)` | Creates a weight matrix of shape `[out, in]` — computes `x @ W.T + b` |
| `.view()` | Reshapes a tensor without copying data — total elements must stay the same |
| `.permute()` | Swaps axes of a tensor — used heavily in attention head rearrangement |
| Broadcasting | PyTorch auto-expands smaller tensors to match larger shapes for element-wise ops |

---

## 📂 Summary of Tasks

- ✅ Understood: The 0D→1D→2D→ND dimensional hierarchy and what each level represents.
- ✅ Understood: Why tensors are necessary — data has structure (batch, position, head, dim) that maps naturally to ND arrays.
- ✅ Understood: Shape as the primary lens for reading any model or tensor operation.
- ✅ Understood: A matrix is a transformation machine, not just storage — it rotates, scales, stretches, and projects vectors.
- ✅ Understood: Why matrix multiplication is the single engine behind forward pass, backprop, attention, and embeddings.
- ✅ Understood: Why GPUs exist — matmul is embarrassingly parallel; every output cell is independent.
- ✅ Understood: The dot product as a similarity measure and how this maps to attention score computation.
- ✅ Built: PyTorch tensors — creation, GPU movement, `@` matmul, `.view()`, `.permute()`, broadcasting.
- ✅ Understood: How Autograd works internally — tape recording, computation graph, reverse traversal, chain rule.
- ✅ Understood: Why `zero_grad()` is mandatory — gradient accumulation will corrupt training if skipped.
- ✅ Built: A model using `nn.Module` — declared layers in `__init__`, connected them in `forward`.
- ✅ Understood: What `nn.Linear` actually is — a weight matrix doing `x @ W.T + b`.
- ✅ Memorized: The 5-line training loop and what each line does mechanically.

---

## 💡 My Takeaway

Three things clicked today that didn't before:

**On Matrices as Transformations:** I had been thinking of weight matrices as lookup tables — containers of numbers. The shift was realising they're **transformation machines**. When you do `output = W × input`, you're not retrieving data — you're rotating and stretching a vector into a new space. Training is just finding the right transformation. That's a completely different mental model.

**On Autograd:** The tape metaphor made it concrete. `requires_grad=True` doesn't just set a flag — it tells PyTorch "start filming everything that happens to this tensor." `.backward()` is just playing the film in reverse and applying the chain rule at each frame. `zero_grad()` clears the tape before the next shot. Once you see it as a recorder, the three function calls stop being arbitrary API and start being obvious.

**On the 5-Line Loop:** Every class introduced more abstraction — embeddings, attention, transformers. The training loop cuts back through all of it. No matter how complex the architecture, learning is always: predict → measure error → clear tape → compute gradients → nudge weights. That loop being identical from a two-layer MNIST net to GPT-4 is genuinely surprising and worth anchoring on.

---

## 🔗 Resources
- [Rohit Ghumare AI Engineering Curriculum]
- [PyTorch Official Docs — torch.Tensor](https://pytorch.org/docs/stable/tensors.html)
- [PyTorch Autograd Tutorial](https://pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html)
- [3Blue1Brown — Essence of Linear Algebra](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab)
- [Andrej Karpathy — micrograd (manual autograd from scratch)](https://github.com/karpathy/micrograd)
- [The Matrix Calculus You Need for Deep Learning — Parr & Howard](https://explained.ai/matrix-calculus/)

---

*Follow my journey! Feel free to ⭐ this repository to stay updated.*
