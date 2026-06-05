# Python Brushup + Phase 1 — Linear Algebra

> *Every AI model is just matrix math wearing a fancy hat.*

**Status:** ✅ Completed (Linear Algebra block)  
**Course:** [AI Engineering from Scratch](https://github.com/rohitg00/ai-engineering-from-scratch)  
**Phase 1 scope:** 22 lessons — Math Foundations  
**This doc covers:** Python brushup + Lessons 01–03 (Linear Algebra Intuition, Vectors & Matrix Operations, Matrix Transformations)

---

## Python Brushup

Before Phase 1 math could start, I went back and deliberately practiced the Python patterns that ML code actually uses. Web dev Python and ML Python use the same language but different muscles.

### What I focused on

**List comprehensions and generators**

```python
# This pattern is everywhere in numpy-free implementations
result = [a * b for a, b in zip(row, col)]

# Generator for memory-efficient processing
def batches(data, size):
    for i in range(0, len(data), size):
        yield data[i:i + size]
```

**Dunder methods** — building the Vector and Matrix classes from scratch required these

```python
class Vector:
    def __add__(self, other): ...   # v1 + v2
    def __mul__(self, scalar): ...  # v * 3
    def __repr__(self): ...         # clean printing in notebooks
```

**Type hints** — tensor shape bugs are silent and catastrophic

```python
def matmul(a: list[list[float]], b: list[list[float]]) -> list[list[float]]:
    # shape: (m x n) @ (n x p) -> (m x p)
    ...
```

**Context managers** — important for GPU memory and file handles in training scripts

```python
with torch.no_grad():          # don't track gradients during inference
    output = model(input)

with open("log.txt", "a") as f:
    f.write(f"epoch {e}: loss={loss:.4f}\n")
```

**`*args` and `**kwargs`** — every ML framework's API uses these heavily

```python
# PyTorch, NumPy, HuggingFace all use this pattern
def train(model, *args, lr=1e-3, epochs=10, **kwargs):
    ...
```

### Key insight

The biggest shift wasn't syntax. It was thinking in transformations over entire arrays rather than iterating element-by-element. `matmul` from scratch makes this concrete — once you've written the nested loop, you understand why vectorization matters and what `@` is actually doing.

---

## Phase 1, Lesson 01 — Linear Algebra Intuition

**Time:** ~60 min | **Type:** Learn | **Languages:** Python, Julia

### The core reframe

| Before this lesson | After this lesson |
|---|---|
| Vector = list of numbers | Vector = a point (or direction) in n-dimensional space |
| Matrix = table of numbers | Matrix = a transformation that moves every point in space |
| Dot product = multiply and sum | Dot product = similarity score between two directions |

### Vectors represent everything in AI

```python
word_embedding   = [0.2, -0.4, 0.8, ...]   # 768 numbers = a location in meaning-space
image_pixels     = [0.9, 0.1, 0.5, ...]    # flat pixel values = a point in pixel-space
user_preferences = [0.3, 0.7, 0.1, ...]   # behavioral features = a point in taste-space
```

Search, RAG, and recommendation systems are all just finding vectors that are close to each other.

### Dot product = the core of similarity search

```python
def dot(a, b):
    return sum(x * y for x, y in zip(a, b))

# Same direction → positive (similar)
# Perpendicular  → 0       (unrelated)
# Opposite       → negative (dissimilar)

def cosine_similarity(a, b):
    return dot(a, b) / (magnitude(a) * magnitude(b))
```

This is literally how attention scores in transformers are computed. `Q @ K.T` — dot products between every query and every key.

### Linear independence and rank

If `v3 = 2*v1 + v2`, adding v3 gives the model zero new information. Worse — the weight matrix becomes singular, no unique solution for weights exists.

```
# Rank-deficient example
v1 = [1, 0, 0]
v2 = [0, 1, 0]
v3 = [2, 1, 0]   # = 2*v1 + v2 → dependent, contributes no new dimension
```

| Situation | What it means for ML |
|---|---|
| Full rank | Unique least-squares solution, well-conditioned model |
| Rank-deficient | Redundant features, infinitely many weight solutions |
| Near rank-deficient | Ill-conditioned: tiny input changes → wild output swings |

### Projection — the core of linear regression and PCA

```python
def project(a, b):
    # Component of a in the direction of b
    scalar = dot(a, b) / dot(b, b)
    return [scalar * x for x in b]
```

Linear regression finds weights by projecting observations onto the column space of the feature matrix. PCA projects data onto directions of maximum variance. Both are projections.

### Gram-Schmidt → why QR decomposition matters

Takes any set of independent vectors and produces an orthonormal basis. Under the hood this is how QR decomposition works, which is the standard numerical method for solving least-squares regression — more stable than Gaussian elimination.

### Where this appears in real AI

| Concept | Where it shows up in production |
|---|---|
| Dot product | Attention scores, cosine similarity in RAG |
| Matrix multiply | Every neural network layer |
| Linear independence | Feature selection, avoiding multicollinearity |
| Rank | LoRA (low-rank adaptation), solvability of linear systems |
| Projection | Linear regression, PCA |
| Gram-Schmidt / QR | Numerical solvers, eigenvalue computation |

**LoRA specifically:** Instead of updating a 4096×4096 weight matrix (16M params), LoRA updates two matrices of size 4096×16 and 16×4096 (131K params). The rank-16 constraint says the update lives in a 16-dimensional subspace of the full 4096-dimensional space. That is the rank concept applied directly.

---

## Phase 1, Lesson 02 — Vectors, Matrices & Operations

**Time:** ~60 min | **Type:** Build | **Languages:** Python, Julia

### Built from scratch: Matrix class

```python
class Matrix:
    def __init__(self, data): ...
    def __add__(self, other): ...           # element-wise addition
    def scalar_multiply(self, s): ...       # scale every element
    def element_wise_multiply(self, o): ... # Hadamard product
    def matmul(self, other): ...            # dot products of rows × cols
    def transpose(self): ...               # flip rows and columns
    def determinant(self): ...             # recursive cofactor expansion
    def inverse_2x2(self): ...             # adjugate / determinant
    @staticmethod
    def identity(n): ...                   # I_n
```

### The shape rule — the most important thing

```
(m × n) @ (n × p) = (m × p)
         ↑↑
   must match

# Neural network layer:
(128 × 784) @ (784 × 1) = (128 × 1)
  weights        input      output
```

Every shape mismatch error in PyTorch traces back to this rule.

### Element-wise vs matrix multiplication

```python
# Element-wise (Hadamard): matching positions, same shape required
[[1,2],[3,4]] * [[5,6],[7,8]] = [[5,12],[21,32]]

# Matrix multiply: row dot column, inner dims must match
[[1,2],[3,4]] @ [[5,6],[7,8]] = [[19,22],[43,50]]
```

These are different operations with different results. Confusing them is a common source of silent wrong answers.

### Broadcasting — how bias addition works

```python
# This:
matrix + bias_vector   # shapes (2,3) + (3,) 

# Is equivalent to:
matrix + [[b1,b2,b3], [b1,b2,b3]]   # NumPy stretches automatically
```

Every framework does this automatically. Understanding it prevents confusion when shapes seem inconsistent but code runs fine.

### A neural network layer, from scratch

```python
inputs  = Matrix([[0.5], [0.8], [0.2]])    # shape (3,1)
weights = Matrix([[...], [...]])            # shape (2,3)
bias    = Matrix([[0.1], [0.1]])            # shape (2,1)

pre_act = weights.matmul(inputs) + bias    # (2,3)@(3,1) = (2,1)
output  = relu_matrix(pre_act)             # shape (2,1)
```

`output = relu(W @ x + b)` — that IS the entire forward pass of a dense layer.

---

## Phase 1, Lesson 03 — Matrix Transformations & Eigenvalues

**Time:** ~75 min | **Type:** Build | **Languages:** Python, Julia

### Transformation matrices built from scratch

```python
import math

def rotation_2d(theta):
    c, s = math.cos(theta), math.sin(theta)
    return [[c, -s], [s, c]]

def scaling_2d(sx, sy):
    return [[sx, 0], [0, sy]]

def shearing_2d(kx, ky):
    return [[1, kx], [ky, 1]]

def reflection_y():
    return [[-1, 0], [0, 1]]
```

### Determinant as a geometric measure

| Transform | det | What it means |
|---|---|---|
| Rotation | 1.0 | Area preserved |
| Scale (2×, 3×) | 6.0 | Area multiplied by 6 |
| Shear | 1.0 | Area preserved |
| Reflection | -1.0 | Area preserved, orientation flipped |
| Singular matrix | 0.0 | Space crushed to lower dimension |

### Composition order matters

```python
R = rotation_2d(π/2)
S = scaling_2d(2, 0.5)

rotate_then_scale = S @ R   # Apply R first, then S
scale_then_rotate = R @ S   # Apply S first, then R

# These produce different results. Matrix multiplication is NOT commutative.
```

### Eigenvalues and eigenvectors

Most vectors change direction when a matrix hits them. Eigenvectors don't — the matrix only scales them.

```
A @ v = λ * v

v = eigenvector (direction that survives)
λ = eigenvalue  (how much it stretches or shrinks)
```

For a 2×2 matrix, eigenvalues solve the characteristic equation:
```
λ² - trace(A)·λ + det(A) = 0
```

### Why eigenvalues matter in AI

**PCA** — the eigenvectors of the covariance matrix are the principal components. The eigenvalues tell you how much variance each component captures. Sort descending, keep top k → dimensionality reduction.

**RNN stability** — eigenvalues with magnitude > 1: outputs explode. Magnitude < 1: outputs vanish. This is the vanishing/exploding gradient problem stated in one sentence.

**Spectral methods** — graph neural networks use eigenvalues of the adjacency matrix. Spectral clustering uses eigenvalues of the graph Laplacian. The eigenvectors reveal the graph's community structure.

### Eigendecomposition

```python
# A = V @ D @ V^(-1)
# V = columns are eigenvectors
# D = diagonal matrix of eigenvalues

vals, vecs = np.linalg.eig(A)
D = np.diag(vals)
V = vecs

# Reconstruct A:
reconstructed = V @ D @ np.linalg.inv(V)
np.allclose(A, reconstructed)  # True
```

Meaning: "rotate into eigenvector coordinates, scale along each axis, rotate back." Every matrix can be understood as a sequence of three operations.

---

## Key Takeaways from the Linear Algebra Block

**1. "What does this do to space?" is the right question.** Every matrix has a geometric action. Asking that question first makes every derivation more intuitive.

**2. Build it from scratch once.** After implementing `matmul` by hand, `A @ B` in NumPy is never a black box. Same for dot product, transpose, determinant.

**3. The dot product is the fundamental operation of modern AI.** Attention, similarity search, projection — they all bottom out in `sum(a[i] * b[i] for i in ...)`.

**4. Shape errors are geometry errors.** A shape mismatch is a statement about incompatible spaces. Fixing it means understanding what transformation you're trying to apply.

**5. Rank is information density.** Low-rank matrices carry less information than their size suggests. LoRA exploits this. Regularization exploits this. Knowing this changes how you think about model architecture.

---

## What's Next in Phase 1

- **Lesson 04:** Calculus for ML — derivatives, gradients, the chain rule
- **Lesson 05:** Automatic differentiation — how PyTorch's `backward()` actually works
- **Lesson 06:** Probability and distributions
- **Lesson 07:** Bayes' theorem
- **Lesson 08:** Optimization — gradient descent and its variants

---

## Resources

- [3Blue1Brown — Essence of Linear Algebra](https://www.3blue1brown.com/topics/linear-algebra) — the geometric intuition that makes all of this click
- [3Blue1Brown — Eigenvectors and Eigenvalues](https://www.3blue1brown.com/lessons/eigenvalues)
- [Stanford CS229 Linear Algebra Review](http://cs229.stanford.edu/section/cs229-linalg.pdf) — ML-specific reference
- [NumPy broadcasting rules](https://numpy.org/doc/stable/user/basics.broadcasting.html)
- Course: [ai-engineering-from-scratch](https://github.com/rohitg00/ai-engineering-from-scratch)

---

*Journey documented publicly. → [your GitHub] | [your X] | [your LinkedIn]*