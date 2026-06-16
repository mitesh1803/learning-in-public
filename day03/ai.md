![Progress](https://img.shields.io/badge/Progress-10%25-orange?style=for-the-badge&logo=progress)

# 🐍 Python Basics + Vectors & Matrices — AI Engineering

## 📝 Topic: Python Fundamentals (Ch. 1–4) + Vectors, Matrices & Operations
**Sources:** Python for Everybody — Dr. Charles R. Severance | AI Engineering from Scratch  
**Scope:** Why Python → Variables → Conditionals → Functions → ML Python Patterns → Vectors → Matrices → NumPy  
**Date:** June 7, 2026

---

## 🎯 Learning Objectives
* Understand what a program is and how Python interprets vs compiles code.
* Work with values, types, variables, operators, expressions, and user input.
* Control program flow using `if`, `elif`, `else`, `try`, and `except`.
* Call built-in functions, use the `math` module, and define your own functions.
* Set up a reproducible ML Python environment using `uv` and `pyproject.toml`.
* Build a `Vector` and `Matrix` class from scratch — no NumPy.
* Implement a complete dense neural network forward pass using only pure Python.
* Translate everything to NumPy and understand what the framework does under the hood.

---

## 🏗️ Part 1 — Why Should You Learn to Write Programs?

### 💡 The Core Idea

> *"Computers are good at things that humans are not — repetitive, precise, high-volume tasks."*

A computer is a machine constantly asking **"What next?"** — up to 3 billion times per second. Programming is how we answer that question in advance.

### 🖥️ Computer Hardware Architecture

| Component | Role |
|---|---|
| **CPU** | The brain — executes instructions |
| **Main Memory** | Fast, temporary storage — lost on shutdown |
| **Secondary Memory** | Slow, persistent storage — disk, USB |
| **Input/Output** | Keyboard, screen, mouse, speaker |
| **Network** | Slow, unreliable secondary memory over distance |

### 🧱 The 5 Building Blocks of Every Program

Every program ever written — in any language — is built from these:

1. **Input** — Get data from the outside world (keyboard, file, sensor)
2. **Output** — Display results on screen or write to a file
3. **Sequential execution** — Run statements one after another, top to bottom
4. **Conditional execution** — Check a condition, then branch
5. **Repeated execution** — Loop through a set of statements
6. **Reuse** — Define once, call many times (functions)

### ⚠️ The 3 Types of Errors

| Error Type | What it means | Example |
|---|---|---|
| **Syntax** | Violated Python's grammar rules | `primt('hello')` — typo in `print` |
| **Logic** | Code runs but does the wrong thing | Calculating the wrong formula |
| **Semantic** | Code is correct but intent is wrong | Turning left instead of right |

### 🐍 Interpreter vs Compiler

```
Interpreter → reads source code line by line, executes immediately  ← Python
Compiler    → translates entire program to machine code first, then runs (.exe)
```

The `>>>` prompt means Python is ready and waiting for your next instruction.

---

## 📦 Part 2 — Variables, Expressions & Statements

### 🔢 Values and Types

Every value in Python has a type:

```python
type(17)        # <class 'int'>    — whole number
type(3.14)      # <class 'float'>  — decimal number
type('hello')   # <class 'str'>    — text string
type(True)      # <class 'bool'>   — True or False
```

**Semantic trap:**
```python
print(1,000,000)   # prints: 1 0 0  ← Python reads it as three separate integers!
```

### 📌 Variables

A variable is a name that refers to a value. `=` creates the variable and stores the value.

```python
message = 'And now for something completely different'
n       = 17
pi      = 3.1415926535897931
```

**Naming rules:**
* Can contain letters, numbers, underscores — cannot start with a number
* Cannot be a Python reserved keyword
* Case-sensitive: `Name` ≠ `name`

```python
# ✅ Valid
my_name = "Mitesh"
airspeed_of_unladen_swallow = 42

# ❌ Invalid
76trombones = 'big parade'   # starts with a number
more@       = 1000000        # illegal character
class       = 'Physics'      # reserved keyword
```

### ➕ Operators and Expressions

```python
20 + 32      # addition
hour - 1     # subtraction
hour * 60    # multiplication
minute / 60  # division        → float in Python 3
minute // 60 # floor division  → int
5 ** 2       # exponentiation  → 25
7 % 3        # modulus (remainder) → 1
```

**Order of operations — PEMDAS:**
```
Parentheses → Exponents → Multiplication/Division → Addition/Subtraction
```

**Modulus tricks:**
```python
x % 2    # → 0 means even, 1 means odd
x % 10   # → rightmost digit
x % 100  # → last two digits
```

### 🔤 String Operations

```python
'100' + '150'   # → '100150'      (concatenation — NOT addition)
'Test ' * 3     # → 'Test Test Test '  (repetition)
```

### ⌨️ User Input

```python
name  = input('What is your name? ')   # always returns a string
hours = int(input('Enter hours: '))     # convert to int explicitly
rate  = float(input('Enter rate: '))    # convert to float explicitly
```

### 💬 Comments

```python
# Bad comment (redundant — just restates the code):
v = 5   # assign 5 to v

# Good comment (explains WHY):
v = 5   # velocity in meters/second
```

### 🔑 Mnemonic Variable Names

```python
# Hard to understand
a = 35.0; b = 12.50; c = a * b

# Clear and self-documenting ✅
hours = 35.0
rate  = 12.50
pay   = hours * rate
```

---

## 🔀 Part 3 — Conditional Execution

### ✅ Boolean Expressions & Comparisons

```python
5 == 5   # True      ← == is comparison
5 == 6   # False

x != y   # not equal
x > y    # greater than
x < y    # less than
x >= y   # greater than or equal
x <= y   # less than or equal
```

> ⚠️ `=` is assignment. `==` is comparison. Confusing them is the #1 syntax mistake.

### 🔗 Logical Operators

```python
x > 0 and x < 10           # both must be true
n % 2 == 0 or n % 3 == 0   # at least one must be true
not (x > y)                 # reverses the boolean
```

### 🌿 if / elif / else

```python
# Simple if
if x > 0:
    print('x is positive')

# if-else
if x % 2 == 0:
    print('x is even')
else:
    print('x is odd')

# Chained conditionals
if x < y:
    print('x is less than y')
elif x > y:
    print('x is greater than y')
else:
    print('x and y are equal')
```

Conditions are checked **top-down** — first `True` branch executes, rest skipped.

### 🛡️ try / except — Catching Errors Gracefully

```python
# Without — crashes on bad input
fahr = float(input('Enter Fahrenheit: '))

# With — recovers gracefully ✅
try:
    fahr = float(input('Enter Fahrenheit: '))
    cel  = (fahr - 32.0) * 5.0 / 9.0
    print(cel)
except:
    print('Please enter a number')
```

Think of `try/except` as an **insurance policy** on a risky block of code.

### ⚡ Short-Circuit Evaluation & Guardian Pattern

Python stops evaluating a logical expression the moment the result is certain:

```python
# If x >= 2 is False → Python stops. Never evaluates (x/y). No crash.
x >= 2 and (x/y) > 2

# Guardian pattern — check y != 0 BEFORE dividing ✅
x >= 2 and y != 0 and (x/y) > 2
```

---

## 🔧 Part 4 — Functions

### 📞 What is a Function?

> *"A function is a named sequence of statements that performs a computation."*

Three key terms:
* **Argument** — value you pass IN when calling
* **Parameter** — variable that receives the argument INSIDE the function
* **Return value** — what the function sends back OUT

### 🏗️ Built-in Functions

```python
max('Hello world')   # → 'w'   (largest character)
min('Hello world')   # → ' '   (smallest character — a space)
len('Hello world')   # → 11    (number of characters)
```

### 🔄 Type Conversion Functions

```python
int('32')      # → 32    (string to int)
int(3.99999)   # → 3     (truncates — does NOT round)
int(-2.3)      # → -2

float(32)      # → 32.0
float('3.14')  # → 3.14

str(32)        # → '32'
str(3.14)      # → '3.14'
```

### 📐 Math & Random Modules

```python
import math

math.sqrt(2)           # → 1.4142...
math.log10(100)        # → 2.0
math.sin(math.pi / 2)  # → 1.0
math.pi                # → 3.14159...

# Degrees → radians
radians = 45 / 360.0 * 2 * math.pi
math.sin(radians)      # → 0.7071...
```

```python
import random

random.random()          # float between 0.0 and 1.0
random.randint(5, 10)    # int between 5 and 10 inclusive
random.choice([1,2,3])   # random element from a list
```

### ✍️ Defining Your Own Functions

```python
def print_lyrics():
    print("I'm a lumberjack, and I'm okay.")
    print('I sleep all night and I work all day.')
```

**Anatomy:**
```
def         →  signals a function definition
print_lyrics →  function name (same rules as variable names)
()          →  parentheses — arguments go here
:           →  colon ends the header
    body    →  indented 4 spaces
```

> ⚠️ **Rule:** Define a function BEFORE you call it. Python reads top to bottom.

### 🌊 Flow of Execution

```
Program starts at line 1 → runs top to bottom
↓
Hits a function CALL → jumps into the function body
↓
Executes the function → returns exactly where it left off
↓
Continues to end of program
```

### 📥 Parameters and Arguments

```python
def print_twice(bruce):    # 'bruce' is the PARAMETER
    print(bruce)
    print(bruce)

print_twice('Spam')        # 'Spam' is the ARGUMENT
```

The variable name outside and the parameter name inside are completely independent.

### 🍎 Fruitful vs Void Functions

```python
# Fruitful — returns a value with return ✅
def addtwo(a, b):
    return a + b

x = addtwo(3, 5)
print(x)    # → 8

# Void — performs an action, returns None implicitly
def print_twice(bruce):
    print(bruce)
    print(bruce)

result = print_twice('Bing')
print(result)   # → None
```

| | Fruitful | Void |
|---|---|---|
| Has `return` | ✅ | ❌ |
| Produces a value | ✅ | ❌ |
| Returns `None` | ❌ | ✅ implicitly |
| Examples | `math.sqrt()`, `len()`, `addtwo()` | `print()`, `print_twice()` |

### 🤔 Why Write Functions?

1. **Readability** — give a group of statements a meaningful name
2. **No repetition** — write once, call many times
3. **Easier debugging** — test each piece independently
4. **Reusability** — well-designed functions work across many programs

---

## 🐍 Part 5 — Python Patterns That Matter for ML

### 🏗️ The Four-Layer AI Stack

Before any model code, the environment must be solid:

```
4. AI/ML Libraries   →  PyTorch, transformers, NumPy, etc.
3. Language Runtimes →  Python 3.12, Node.js, Rust
2. Package Managers  →  uv, pnpm, cargo
1. System Foundation →  OS, shell, git, editor, GPU drivers
```

> **Rule:** Always install bottom-up. Skip a layer and everything above it breaks silently.

### 📦 Virtual Environments — Non-Negotiable in ML

PyTorch, JAX, and TensorFlow each ship their own CUDA bindings. A global `pip install` overwrites whatever was there before.

```
Without venvs → CONFLICT: only one torch version can exist

With venvs:
├── Project A (.venv/) → torch 2.4.0 (CUDA 12.4)
└── Project B (.venv/) → torch 2.1.0 (CUDA 11.8)
```

**Setup with `uv` (10–100× faster than pip):**

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
uv python install 3.12
uv venv
source .venv/bin/activate        # Linux/macOS

which python   # must show .venv/bin/python — NOT /usr/bin/python
```

**`pyproject.toml` — replaces `requirements.txt` and `setup.py`:**

```toml
[project]
name = "ai-engineering-from-scratch"
requires-python = ">=3.11"
dependencies = ["numpy>=1.26", "matplotlib>=3.8", "jupyter>=1.0"]

[project.optional-dependencies]
torch = ["torch>=2.3", "torchvision>=0.18"]
llm   = ["anthropic>=0.39", "openai>=1.50"]
```

```bash
uv pip install -e ".[torch]"      # base + PyTorch
uv pip install -e ".[torch,llm]"  # everything
```

### ⚠️ The 5 Mistakes That Wreck ML Environments

| Mistake | What Breaks | Fix |
|---|---|---|
| `pip install` without activating | Installs to system Python | Always activate first |
| Mixing `pip` inside `conda` | Dependency tracking breaks | Let conda manage everything |
| Forgetting to activate | Wrong Python, missing packages | Check prompt shows `(.venv)` |
| Committing `.venv/` to git | 200MB–2GB bloat, not portable | Add `.venv/` to `.gitignore` |
| CUDA version mismatch | GPU silently not used | PyTorch CUDA ≤ driver CUDA |

### 🔑 ML-Specific Python Patterns

**List comprehensions** — used throughout numerical code:

```python
result = [a * b for a, b in zip(row, col)]

# Memory-efficient generator for large datasets
def batches(data, size):
    for i in range(0, len(data), size):
        yield data[i:i + size]
```

**Dunder methods** — needed to build Vector and Matrix from scratch:

```python
class Vector:
    def __add__(self, other): ...   # enables:  v1 + v2
    def __mul__(self, scalar): ...  # enables:  v * 3
    def __repr__(self): ...         # clean printing in notebooks
```

**Type hints** — tensor shape bugs are silent and catastrophic:

```python
def matmul(a: list[list[float]], b: list[list[float]]) -> list[list[float]]:
    # shape: (m × n) @ (n × p) → (m × p)
    ...
```

**`*args` and `**kwargs`** — every ML framework's API uses these:

```python
def train(model, *args, lr=1e-3, epochs=10, **kwargs):
    ...
```

---

## 📐 Part 6 — Vectors, Matrices & Operations

> *Every neural network is just matrix multiplication with extra steps.*

### 🔢 What is a Vector?

A vector is not just a list of numbers — it is a **point (or direction) in n-dimensional space**.

```python
word_embedding   = [0.2, -0.4, 0.8, ...]  # 768 numbers → location in meaning-space
image_pixels     = [0.9, 0.1, 0.5, ...]   # pixel values → location in pixel-space
user_preferences = [0.3, 0.7, 0.1, ...]   # features    → location in taste-space
```

Search, RAG, and recommendations are all just finding vectors that are close to each other.

### 🔢 What is a Matrix?

A matrix is a **transformation machine** — it takes every point in space and moves it: rotates, stretches, projects.

A layer with 784 inputs and 128 outputs uses a **128×784** weight matrix.

```
A = | 1  2  3 |     ← 2×3 matrix (2 rows, 3 columns)
    | 4  5  6 |
```

### ⚙️ The Operations Map

| Operation | What it does | Neural network use |
|---|---|---|
| **Addition** | Element-wise combine | Adding bias to layer output |
| **Scalar multiply** | Scale every element | Learning rate × gradients |
| **Matrix multiply** | Transform vectors via dot products | Layer forward pass |
| **Transpose** | Flip rows and columns | Backpropagation |
| **Determinant** | Single-number volume measure | Checking invertibility |
| **Inverse** | Undo a transformation | Solving linear systems |
| **Identity** | Do-nothing matrix | Residual connections (ResNets) |

### ⚠️ Element-wise vs Matrix Multiplication

The #1 source of shape bugs in ML:

**Element-wise (Hadamard)** — matching positions, same shape required:
```
| 1  2 |   | 5  6 |   |  5  12 |
| 3  4 | * | 7  8 | = | 21  32 |
```

**Matrix multiply** — dot products of rows × columns, inner dims must match:
```
| 1  2 |   | 5  6 |   | 19  22 |
| 3  4 | @ | 7  8 | = | 43  50 |
```

**The shape rule — the most important rule in all of ML:**
```
(m × n) @ (n × p) = (m × p)
          ↑↑
    inner dims must match

(128 × 784) @ (784 × 1) = (128 × 1)
   weights       input       output
```

Every `RuntimeError: mat1 and mat2 shapes cannot be multiplied` in PyTorch traces back to this rule.

### 📡 Broadcasting — How Bias Addition Works

When shapes don't match, NumPy/PyTorch stretches the smaller array automatically:

```python
matrix = [[1, 2, 3],
          [4, 5, 6]]    # shape (2, 3)
bias   = [10, 20, 30]   # shape (3,)

# Broadcasting stretches bias across both rows:
result = [[11, 22, 33],
          [14, 25, 36]]
```

---

## 🔨 Part 7 — Build It From Scratch

### Vector Class

```python
class Vector:
    def __init__(self, data):
        self.data = list(data)
        self.size = len(self.data)

    def __repr__(self):
        return f"Vector({self.data})"

    def __add__(self, other):
        return Vector([a + b for a, b in zip(self.data, other.data)])

    def __sub__(self, other):
        return Vector([a - b for a, b in zip(self.data, other.data)])

    def __mul__(self, scalar):
        return Vector([x * scalar for x in self.data])

    def dot(self, other):
        return sum(a * b for a, b in zip(self.data, other.data))

    def magnitude(self):
        return sum(x ** 2 for x in self.data) ** 0.5
```

### Matrix Class

```python
class Matrix:
    def __init__(self, data):
        self.data  = [list(row) for row in data]
        self.rows  = len(self.data)
        self.cols  = len(self.data[0])
        self.shape = (self.rows, self.cols)

    def __add__(self, other):
        return Matrix([
            [self.data[i][j] + other.data[i][j] for j in range(self.cols)]
            for i in range(self.rows)
        ])

    def scalar_multiply(self, scalar):
        return Matrix([
            [self.data[i][j] * scalar for j in range(self.cols)]
            for i in range(self.rows)
        ])

    def element_wise_multiply(self, other):
        return Matrix([
            [self.data[i][j] * other.data[i][j] for j in range(self.cols)]
            for i in range(self.rows)
        ])

    def matmul(self, other):
        return Matrix([
            [
                sum(self.data[i][k] * other.data[k][j] for k in range(self.cols))
                for j in range(other.cols)
            ]
            for i in range(self.rows)
        ])

    def transpose(self):
        return Matrix([
            [self.data[j][i] for j in range(self.rows)]
            for i in range(self.cols)
        ])

    def determinant(self):
        if self.shape == (2, 2):
            return self.data[0][0] * self.data[1][1] - self.data[0][1] * self.data[1][0]
        det = 0
        for j in range(self.cols):
            minor = Matrix([
                [self.data[i][k] for k in range(self.cols) if k != j]
                for i in range(1, self.rows)
            ])
            det += ((-1) ** j) * self.data[0][j] * minor.determinant()
        return det

    def inverse_2x2(self):
        det = self.determinant()
        if det == 0:
            raise ValueError("Singular matrix — no inverse exists")
        return Matrix([
            [ self.data[1][1] / det, -self.data[0][1] / det],
            [-self.data[1][0] / det,  self.data[0][0] / det]
        ])

    @staticmethod
    def identity(n):
        return Matrix([[1 if i == j else 0 for j in range(n)] for i in range(n)])
```

### 🧠 A Complete Neural Network Forward Pass

```python
import random

inputs  = Matrix([[0.5], [0.8], [0.2]])                                      # shape (3, 1)
weights = Matrix([[random.uniform(-1, 1) for _ in range(3)] for _ in range(2)])  # (2, 3)
bias    = Matrix([[0.1], [0.1]])                                              # shape (2, 1)

def relu_matrix(m):
    return Matrix([[max(0, val) for val in row] for row in m.data])

pre_activation = weights.matmul(inputs) + bias   # (2,3) @ (3,1) = (2,1)
output         = relu_matrix(pre_activation)      # shape (2, 1)

print(f"Input shape:   {inputs.shape}")
print(f"Weights shape: {weights.shape}")
print(f"Output shape:  {output.shape}")
```

> `output = relu(W @ x + b)` — three operations. That's the **entire forward pass of a dense layer**. Every dense layer in every neural network does exactly this.

---

## ⚡ Part 8 — The Same Thing in NumPy

```python
import numpy as np

A = np.array([[1, 2], [3, 4]])
B = np.array([[5, 6], [7, 8]])

print(A + B)               # element-wise addition
print(A * B)               # element-wise multiply (Hadamard)
print(A @ B)               # matrix multiply
print(A.T)                 # transpose
print(np.linalg.det(A))    # determinant
print(np.linalg.inv(A))    # inverse
print(np.eye(2))            # 2×2 identity matrix

# Neural network layer — one line
inputs  = np.random.randn(3, 1)
weights = np.random.randn(2, 3)
bias    = np.array([[0.1], [0.1]])
output  = np.maximum(0, weights @ inputs + bias)   # relu(Wx + b)
print(f"Layer: {weights.shape} @ {inputs.shape} = {output.shape}")
```

The `@` operator calls `__matmul__` — the dunder method you just implemented. NumPy runs it with BLAS routines written in C and Fortran. Same math, ~100× faster.

---

## 📖 Key Terms

| Term | What it means |
|---|---|
| **Variable** | A name that refers to a value stored in memory |
| **Assignment** | A statement that binds a name to a value (`x = 5`) |
| **Expression** | A combination of values, variables, and operators that produces a result |
| **Modulus** | `%` — yields the remainder of integer division |
| **Boolean** | A value that is either `True` or `False` |
| **Short-circuit** | Python stops evaluating a logical expression once the result is certain |
| **Guardian pattern** | A safety check placed before a risky operation in a logical expression |
| **Fruitful function** | A function that returns a value |
| **Void function** | A function that performs an action but returns `None` |
| **Parameter** | A variable inside a function that receives the argument |
| **Flow of execution** | The order in which statements are executed during a program run |
| **Virtual environment** | An isolated directory with its own Python interpreter and packages |
| **Lockfile** | Pins every package to an exact version — guarantees identical installs |
| **Vector** | An ordered list of numbers representing a point in n-dimensional space |
| **Matrix** | A linear transformation — maps vectors from one space to another |
| **Dot product** | Multiply matching elements, sum the results — measures similarity |
| **Matrix multiply** | Dot products between every row and every column. Order matters. |
| **Transpose** | Swap rows and columns. Turns m×n into n×m. Critical in backpropagation. |
| **Determinant** | Measures how much a matrix scales area or volume. Zero = space crushed. |
| **Broadcasting** | Stretching a smaller array to match a larger one by repeating dimensions |
| **Element-wise** | Operate on matching positions. Both arrays must be the same shape. |
| **Shape rule** | `(m×n) @ (n×p) = (m×p)` — inner dimensions must match |

---

## 📂 Summary of Tasks
-  Why Python, hardware, building blocks, error types.
-  Values, types, variables, operators, input, comments.
-  Booleans, `if/elif/else`, `try/except`, guardian pattern.
-  Built-in functions, type conversion, `math` module, defining functions.
-  Understood: Fruitful vs void functions, parameters vs arguments, flow of execution.
-  Set up: Python environment with `uv` and `pyproject.toml`.
-  Understood: Why every ML project needs its own isolated virtual environment. 
- Built: `Vector` class from scratch — add, sub, dot, magnitude.
-  Built: `Matrix` class from scratch — add, matmul, transpose, determinant, inverse.
-  Implemented: Complete dense layer forward pass — `relu(W @ x + b)`.
-  Translated: Everything above to NumPy and understood what the framework does.

---

## 💡 My Takeaway

Two things clicked today that weren't obvious before:

**On Python:** The mental model of **flow of execution** is everything. When Python hits a function call it doesn't continue to the next line — it detours into the function body, executes everything, then comes back exactly where it left off. Once that lands, function composition stops being magic. Also `try/except` isn't just error handling — it's a design decision about which code paths are uncertain by nature.

**On Vectors and Matrices:** The biggest shift isn't syntax — it's thinking in **transformations over entire arrays** instead of iterating element by element. Once you've written `matmul` by hand with nested loops, `A @ B` in NumPy is never a black box. And after you've implemented the forward pass yourself — `relu(W @ x + b)` — every "complex" neural network layer is just that, repeated many times with learned weights.

---

## 📈 Next Up
**Matrix Transformations & Eigenvalues** — how matrices rotate and project space, and why eigenvalues power PCA, RNN stability, and LoRA fine-tuning.

---

## 🔗 Resources
* [py4e.com](https://www.py4e.com) — Official course site with exercises and screencasts
* [Python 3 docs — Built-in functions](https://docs.python.org/3/library/functions.html)
* [Python 3 docs — math module](https://docs.python.org/3/library/math.html)
* [uv docs](https://docs.astral.sh/uv/) — the Python package manager used in this course
* [3Blue1Brown — Essence of Linear Algebra](https://www.3blue1brown.com/topics/linear-algebra)
* [NumPy broadcasting rules](https://numpy.org/doc/stable/user/basics.broadcasting.html)
* [Course repo — AI Engineering from Scratch](https://github.com/rohitg00/ai-engineering-from-scratch)

---
*Follow my journey! Feel free to ⭐ this repository to stay updated.*