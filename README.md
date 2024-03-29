# MyAbsInterp
Implementation of Rosendahl, Mads. "Introduction to abstract interpretation." Computer Science University of Copenhagen (1995).

## Overview of Abstract Interpretation

The core idea of abstract interpretation is constructing an "abstract interpretation" of a programming language in addition to its usual meaning. The abstract interpretation can be used to reason the behaviour of the program.

In this article, we denote the standard interpretation (mapping from a program to a function) as `I_1` and the abstract interpretation as `I_2`. We also denote the relation between `I_1` and `I_2` as `R`: `I_1[p] R I_2[p]`, where `p` is the input program.

### Example.1 Rule-of-Sign

Rule-of-sigin is a simple abstract interpretation problem, where the goal is estimating whether the output of an expression is positive or negative.

#### Syntax

```
exp ::= n           - number
    | exp + exp     - addition
    | exp * exp     - multiplication
```

#### Standard Interpretation

```
E_{std}[[exp]] : \mathcal{Z} 
E_{std}[[n_i]] = n_i 
E_{std}[[exp_1 + exp_2]] = E_{std}[[exp_1]] + E_{std}[[exp_2]] 
E_{std}[[exp_1 * exp_2]] = E_{std}[[exp_1]] * E_{std}[[exp_2]]
```

#### Abstract Interpretation

We introduce a set of abstract values, `Sign = {zero, pos, neg, num}`. The adition and multiplication on `Sign` can be defined as the below tables.

```
⊕ : Sign × Sign → Sign

⊕    zero pos neg num
zero zero pos neg num
pos   pos pos num num
neg   neg num neg num
num   num num num num
```

```
⊗ : Sign × Sign → Sign

⊗    zero  pos  neg  num
zero zero zero zero zero
pos  zero  pos  neg  num
neg  zero  neg  pos  num
num  zero  num  num  num
```

Then, we define the abstract interpretation with the above operators.


```
E{ros}[[exp]] : Sign 
E{ros}[[ni]] = sign(ni) 
E{ros}[[exp_1 + exp_2]] = E{ros}[[exp_1]] ⊕ E{ros}[[exp_2]] 
E{ros}[[exp_1 ∗ exp_2]] = E{ros}[[exp_1]] ⊗ E{ros}[[exp_2]] 
```

, where `sign(x) = if x > 0 then pos else if x < 0 then neg else zero`.

#### Relation

- an abstract value -> a set of integers (concretisation)

```
γ : Sign -> P(\mathcal{Z}) \setminus {∅} 
γ(zero) = {0} 
γ(pos) = {x | x > 0} 
γ(neg) = {x | x < 0} 
γ(num) = \mathcal{Z}
```

- a set of integers -> an abstract value (abstraction)

```
α : P(\mathcal{Z}) \setminus {∅} -> Sign 
α(X) = zero if  (X = {0}) 
     = pos  if  (∀{x ∈ X}. x > 0) 
     = neg  if  (∀{x ∈ X}. x < 0) 
     = num  otherwise
```

The relationship  between `γ` and `α` is as follows.

```
∀{s} ∈ Sign.  α(γ(s)) = s 
∀{X} ∈ P(\mathcal{Z}) \setminus ∅.  γ(α(X)) \supseteq X
```

Then, the additino and multiplication can be written as follows.

```
s_1 ⊕ s_2 = α({x_1 + x_2 | x_1 ∈ γ(s_1) ∧ x_2 ∈ γ(s_2)}) 
s_1 ⊗ s_2 = α({x_1 ∗ x_2 | x_1 ∈ γ(s_1) ∧ x_2 ∈ γ(s_2)}) 
```

- Excercise

Prove `∀{exp}. {E_{std}[[exp]]} \subseteq γ(E_{ros}[[exp]])`.

### Example.2 Strictness Analysis

We call a function `f` is $strict$ if it maps the bottom element (meaning $undefined$) `⊥` to the bottom element; `f(⊥) = ⊥`. Thus, the results of calling `f` by value and calling `f` by need are the same.

#### A lazy functional language

- Semantics domains

```
D = V_{⊥}         - values
φ = (D^k -> D)^n  - function denotations
```

- Semantics functions

```
E[[exp]]  : φ -> D^k -> D
P[[prog]] : φ
```

- Definition

```
E[[c_i]] φ v                      = const_i                                         
E[[x_i]] φ v                      = v_i                                             
E[[a_i(e_1, ..., e_k)]] φ v       = strict basic_i<E[[e_1]] φ v, ..., E[[e_k]] φ v>
E[[if e_1 then e_2 else e_3]] φ v = cond(E[[e_1]] φ v, E[[e_2]] φ v, E[[e_3]] φ v)
E[[f_i(e_1, ..., e_k)]] φ v       = f_i<E[[e_1]] φ v, ..., E[[e_k]] φ v>            

P[[f_1(x_1, ..., x_k) = e_1
        .
        .
   f_n(x_1, ..., x_k) = e_n]]        = fix \lambda φ. <E[[e_1]] φ, ..., E[[e_2]] φ>
```

, where

```
strict f<v_1, ..., v_k> = if v_1 = ⊥ ∨ .... v_k = ⊥ then ⊥ else f(v_1, ..., v_k)
```

Each definition corresponds to the mapping of `name of the i-th constant -> value of the i-th constant`, `name of the i-th variable -> value of the i-th variable`, `name of the i-th basic function -> strictly evaluated value of that function on v`, `control flow`, and `name of the i-th user-defined function -> evaluated value of that function on v`, respectively.

#### Abstract domain

To examine the strictness of a function, we use a two-point domain named `\mathcal{2} = {0, 1}` ordered by `0 <= 1`.

Here, we define two utility operators.

```
d_1 ∧ d_2 = min(d_1, d_2)
d_2 ∨ d_2 = max(d_1, d_2)
```

Then, we use `\mathcal{2}` to describe if an element in `D` is defined or not, meaning that it is `⊥` or not. Specifically, we define the abstract function as follows:

```
α : D → \mathcal{2}
α(d) = if d = ⊥ then 0 else 1
```



