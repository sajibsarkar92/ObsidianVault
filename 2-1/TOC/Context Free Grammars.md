# Context-Free Grammars (CFG)




## Introduction to Context-Free Grammars


![[Chapter-5-2.webp]]

### What is a Context-Free Grammar?

A **Context-Free Grammar (CFG)** is a formal notation system used to describe and generate languages. Think of it as a set of rules that tells us how to construct valid strings in a language.

**Key Properties:**

- CFGs are more powerful than finite automata and regular expressions
- They can describe patterns that regular expressions cannot
- However, they still have limitations and cannot define all possible languages
- Every regular language is also a context-free language (CFL)
- Regular languages form a **proper subset** of context-free languages

**Intuitive Analogy:** Think of a CFG like a recipe book. Each recipe (production rule) tells you how to create or expand certain ingredients (variables) into dishes or simpler ingredients (terminals and other variables). Just as you follow recipes step-by-step to make a meal, you follow production rules to generate valid strings in the language.

---

## Informal Example: Palindromes

![[Chapter-5-3.webp]]

### Understanding Palindromes

A **palindrome** is a string that reads the same forwards and backwards.

**Examples:**

- "otto" is a palindrome
- "madamimadam" is a palindrome
- For binary alphabet Σ = {0, 1}: "0110" is a palindrome, "10101" is a palindrome

### Why Palindromes Aren't Regular

Let's consider L_pal = {w ∈ Σ* : w = $w^R$} (all palindromes)

**Proof that palindromes are not regular:**

1. Assume L_pal is regular
2. By the pumping lemma, there exists some n
3. Consider the string 0^n 1 0^n (which is a palindrome)
4. When reading 0^n, a finite automaton must make a loop
5. If we omit the loop, we get a string that's still supposedly in the language but is no longer a palindrome
6. **Contradiction!** Therefore, palindromes cannot be regular

### Recursive Definition of Palindromes

Instead, we define palindromes **inductively** (recursively):

**Basis (Base cases):**

- ε (empty string) is a palindrome
- "0" is a palindrome
- "1" is a palindrome

**Induction (Recursive cases):**

- If w is a palindrome, then **0w0** is also a palindrome
- If w is a palindrome, then **1w1** is also a palindrome

**Example of building palindromes:**

- Start with ε → add 1's → "1" → add 0's → "010" → add 1's → "10101"
- Start with "0" → add 1's → "101" → add 0's → "01010"

This recursive structure is exactly what CFGs are designed to capture!

---

## Informal Grammar Example

![[Chapter-5-4.webp]]**

### Grammar for Palindromes

Here's how we express the palindrome language using grammar rules:

**Productions (Rules):**

1. P → ε
2. P → 0
3. P → 1
4. P → 0P0
5. P → 1P1

**Terminology:**

- **Terminals:** 0 and 1 (the actual symbols in our strings)
- **Variable (Non-terminal):** P (represents the class of palindromic strings)
- **Start symbol:** P (where we begin derivations)
- **Productions:** Rules 1-5 (tell us how to rewrite P)

**How it works:**

- Rules 1-3 give us base cases (ε, 0, 1)
- Rules 4-5 give us recursive cases (wrap palindromes with matching symbols)

**Example derivation:** To generate "0110":

- P → 0P0 (using rule 4)
- 0P0 → 01P10 (using rule 5 on the middle P)
- 01P10 → 0110 (using rule 1 on the middle P)

---

## Formal Definition of CFG

![[Chapter-5-5.webp]]

### The Four Components

A **Context-Free Grammar G** is formally defined as a quadruple: **G = (V, T, P, S)**

**1. V (Variables/Non-terminals):**

- A finite set of symbols representing language classes
- Usually written in uppercase: A, B, C, S, P, etc.
- These are placeholders that can be expanded using production rules
 
**2. T (Terminals):**

- A finite set of actual symbols that appear in strings
- Usually written in lowercase: a, b, c, 0, 1, etc.
- These are the "atoms" that cannot be broken down further
- **Important:** V ∩ T = ∅ (variables and terminals are disjoint sets)

**3. P (Productions):**

- A finite set of rules of the form: V → (V ∪ T)*
- Each rule has a **head** (single variable) and a **body** (string of variables and/or terminals)
- Format: Variable → string of variables and terminals

**4. S (Start Symbol):**

- A designated variable in V where all derivations begin
- Represents the entire language we're defining

### Example: Formal Palindrome Grammar

**$G_{pal}$ = ({P}, {0, 1}, A, P)**

Where A = {P → ε, P → 0, P → 1, P → 0P0, P → 1P1}

**Shorthand notation:** When multiple productions have the same head, we can combine them:

- A = {P → ε | 0 | 1 | 0P0 | 1P1}
- The vertical bar "|" means "or"

---

## Example: Grammar for {0^n 1^n | n ≥ 1}

![[Chapter-5-6.webp]]

### Understanding the Language

The language **L = {$0^n$ $1^n$ | n ≥ 1}** consists of strings with equal numbers of 0's followed by equal numbers of 1's:

- "01" (n=1)
- "0011" (n=2)
- "000111" (n=3)
- etc.

**This language is also not regular!** (Can be proven with pumping lemma)

### The Grammar

**Productions:**

- S → 01
- S → 0S1

**Recursive definition:**

- **Basis:** "01" is in the language
- **Induction:** If w is in the language, then 0w1 is also in the language

### Derivation Examples

**Generating "0011":**

- S → 0S1 (wrap with 0 and 1)
- 0S1 → 0(01)1 (replace S with 01)
- Result: "0011"

**Generating "000111":**

- S → 0S1
- 0S1 → 0(0S1)1 = 00S11
- 00S11 → 00(01)11
- Result: "000111"

**Key Insight:** Each application of S → 0S1 adds one 0 to the left and one 1 to the right, maintaining the balance!

### Formal Definition

**G_equ = ({S}, {0, 1}, A, S)**

Where A = {S → 01, S → 0S1}

**Conventions:**

- A, B, C, ... represent variables
- a, b, c, ... represent terminals
- w, x, y, z represent strings of terminals

---

## Derivation Using Grammar

![[Chapter-5-7.webp]]

### Two Approaches to String Generation

There are two equivalent ways to determine if a string belongs to a language defined by a CFG:

**1. Recursive Inference (Bottom-Up)**

- Start with base productions (terminals)
- Apply rules from **body to head**
- Build up larger strings until we reach the start symbol

**2. Derivations (Top-Down)**

- Start with the start symbol
- Apply rules from **head to body**
- Break down until we get a string of terminals

**Both methods are equivalent!** They just work in opposite directions.

---

## Recursive Inference

![[Chapter-5-9.webp]]

### Bottom-Up String Construction

**Recursive inference** means we start with known facts (base cases) and use production rules to infer new facts.

**Example Grammar (G1):**

- E → I | E + E | E * E | (E)
- I → a | b | Ia | Ib | I0 | I1

### Inference Table Example

This table shows step-by-step how we build up knowledge about which strings belong to which variables:

|String|Language|Production Used|Based on String(s)|
|---|---|---|---|
|a|I|I → a|- (base case)|
|b|I|I → b|- (base case)|
|b0|I|I → I0|(ii) uses b|
|b00|I|I → I0|(iii) uses b0|
|a|E|E → I|(i) uses a from I|
|b00|E|E → I|(iv) uses b00 from I|
|a + b00|E|E → E + E|(v) and (vi)|
|(a + b00)|E|E → (E)|(vii) uses previous|
|a*(a + b00)|E|E → E * E|(v) and (viii)|

**Key Points:**

- Each row depends on previous rows (shown in "Based on" column)
- We build complex strings from simpler ones
- Eventually, we can generate arbitrary expressions

---

## Derivations

![[Chapter-5-10.webp]]

### The Derivation Relation (⇒)

To work **top-down**, we need a formal way to describe applying production rules.

**Definition of the "derives" relation:**

Let G = (V, T, P, S) be a CFG. If:

- α and β are strings of variables and terminals
- A is a variable (A ∈ V)
- A → γ is a production in P

Then we write: **αAβ ⇒ αγβ**

**In words:** "αAβ derives αγβ"

**What this means:**

- We can replace variable A with γ anywhere it appears in a string
- The parts before (α) and after (β) stay unchanged
- We're applying one production rule

**Example:**

- If we have E + A and production A → b
- Then E + A ⇒ E + b

---

## Zero or More Derivation Steps

![[Chapter-5-11.webp]]

### The Reflexive-Transitive Closure (⇒*)

The symbol **⇒*** means "derives in zero or more steps" (reflexive and transitive closure of ⇒)

**Formal Definition:**

**Basis (Zero steps):**

- For any string α ∈ (V ∪ T)_, we have α ⇒_ α
- Any string derives itself in zero steps

**Induction (One or more steps):**

- If α ⇒* β and β ⇒ γ, then α ⇒* γ
- If we can get from α to β, and from β to γ, we can get from α to γ

**Intuitive Understanding:**

- ⇒ means "one step"
- ⇒* means "any number of steps (including zero)"
- This is like "closure" in graph theory—if you can reach B from A, and C from B, you can reach C from A

**Examples:**

- E ⇒* E (zero steps)
- E ⇒ E + E ⇒* E + E (one step)
- E ⇒ E + E ⇒ I + E ⇒ a + E (three steps, so E ⇒* a + E)

---

## Example of Derivation
![[Chapter-5-12.webp]]

### Deriving a*(a + b000)

**Full derivation:**

```python
E ⇒ E * E         (use E → E * E)
  ⇒ I * E         (use E → I on left E)
  ⇒ a * E         (use I → a)
  ⇒ a * (E)       (use E → (E))
  ⇒ a * (E + E)   (use E → E + E inside parentheses)
  ⇒ a * (I + E)   (use E → I on left E inside)
  ⇒ a * (a + E)   (use I → a)
  ⇒ a * (a + I)   (use E → I on right E)
  ⇒ a * (a + I0)  (use I → I0)
  ⇒ a * (a + I00) (use I → I0 again)
  ⇒ a * (a + b00) (use I → b, noting b00 = b·0·0)
```

### Important Observations

**Note 1: Multiple Choices** At each step, we might have several rules to choose from:

- I * E ⇒ a * E ⇒ a * (E) (replace I first, then add parentheses)
- I * E ⇒ I * (E) ⇒ a * (E) (add parentheses first, then replace I)

**Note 2: Not All Paths Succeed** Some derivation choices lead to dead ends:

- If we start with E ⇒ E + E, we can't derive a * (a + b000)
- We need to choose productions carefully to reach our target string

**Key Principle:** __A string w of terminals is in the language of variable A if and only if A ⇒_ w_*

This establishes the equivalence between recursive inference and derivation!

---

## Leftmost and Rightmost Derivations

![[Chapter-5-13.webp]]

### Restricting Derivation Choices

To reduce ambiguity and make derivations more systematic, we can impose ordering constraints:

**Leftmost Derivation (⇒_lm):**

- At each step, replace the **leftmost variable** with one of its production bodies
- Scan the string from left to right, expand the first variable you encounter

**Rightmost Derivation (⇒_rm):**

- At each step, replace the **rightmost variable** with one of its production bodies
- Scan the string from right to left, expand the last variable you encounter

### Example: Leftmost Derivation

The derivation from page 12 is leftmost:

```cpp
E ⇒_lm E * E ⇒_lm I * E ⇒_lm a * E ⇒_lm ...
```

At each step, we expand the leftmost variable.

### Example: Rightmost Derivation

For the same string a*(a + b00):

```python
E ⇒_rm E * E          (start)
  ⇒_rm E * (E)        (expand rightmost E)
  ⇒_rm E * (E + E)    (expand the E in parentheses)
  ⇒_rm E * (E + I)    (expand rightmost E)
  ⇒_rm E * (E + I0)   (expand rightmost I)
  ⇒_rm E * (E + I00)  (expand rightmost I)
  ⇒_rm E * (E + b00)  (expand rightmost I)
  ⇒_rm E * (I + b00)  (expand remaining E)
  ⇒_rm E * (a + b00)  (expand I)
  ⇒_rm I * (a + b00)  (expand leftmost E)
  ⇒_rm a * (a + b00)  (expand final I)
```

**Key Insight:** Both leftmost and rightmost derivations produce the same final string, but in different orders. They represent different strategies for applying the same grammar rules.

---

## The Language of a Grammar

![[Chapter-5-14.webp]]

### Formal Definition

For a CFG G = (V, T, P, S), the **language of G** is:

__L(G) = {w ∈ T_ | S ⇒*_ w}__

**In words:** The language consists of all strings of terminals that can be derived from the start symbol.

**Key Components:**

- **w ∈ T***: w is a string containing only terminals (no variables)
- __S ⇒_ w_*: There exists a derivation from the start symbol S to w
- This is called a **context-free language (CFL)**

### Examples

**For palindrome grammar $G_{pal}:$

- L($G_{pal}$) = {ε, 0, 1, 00, 11, 010, 101, 0110, 1001, ...}
- All binary palindromes

**For equal 0s and 1s grammar $G_{equ}$ :**

- L($G_{equ}$ )= {01, 0011, 000111, 00001111, ...}
- All strings of form 0^n 1^n where n ≥ 1

**Important:** If L can be generated by some CFG G, we call L a **context-free language**.

---

## Theorem: Correctness of Palindrome Grammar

![[Chapter-5-15.webp]]

### Statement

**Theorem:** A string w ∈ {0,1}* is in L($G_{pal}$) if and only if w = $$w^R$$ (w is a palindrome)

This has two directions to prove:

1. **(⊇) If w is a palindrome, then w ∈ L($G_{pal}$)**
2. **(⊆) If w ∈ L($G_{pal}$), then w is a palindrome**

### Proof of Direction 1: w = $$w^R$$ implies w ∈ L($G_{pal}$l)

**Given:** w is a palindrome (w =$$w^R$$) **Prove:** w ∈ L($G_{pal}$)

**Method:** Induction on the length of w

**Basis (|w| = 0 or |w| = 1):**

- w is either ε, 0, or 1
- We have productions: P → ε, P → 0, P → 1
- Therefore P ⇒ w in one step for all base cases ✓

**Induction (|w| ≥ 2):**

- **Assumption:** For all palindromes x with |x| < |w|, we have P ⇒* x
- Since w is a palindrome with length ≥ 2:
    - Either w = 0x0 where x = x^R, or
    - w = 1x1 where x = x^R
- **Case w = 0x0:**
    - By inductive hypothesis: P ⇒* x (since |x| < |w|)
    - Using production P → 0P0:
        - P ⇒ 0P0 ⇒* 0x0 = w ✓
- **Case w = 1x1:** Similar argument using P → 1P1 ✓

**Conclusion:** Every palindrome can be derived from P.

---

## Proof (continued)

![[Chapter-5-16.webp]]

### Proof of Direction 2: w ∈ L($G_{pal}$) implies w = $w^R$

**Given:** w ∈ L($G_{pal}$), meaning P ⇒* w **Prove:** w is a palindrome (w = $w^R$)

**Method:** Induction on the length of the derivation

**Basis (One-step derivation P ⇒ w):**

- w must be derived using one of: P → ε, P → 0, P → 1
- Therefore w ∈ {ε, 0, 1}
- All of these are palindromes ✓

**Induction (Derivation takes n+1 steps, n ≥ 1):**

- **Assumption:** Any string derived in n or fewer steps is a palindrome
    
- Since the derivation takes n+1 steps, the first step must use:
    
    - Either P ⇒ 0P0, then 0P0 ⇒* w in n steps, or
    - P ⇒ 1P1, then 1P1 ⇒* w in n steps
- __Case P ⇒ 0P0 ⇒_ w:_*
    
    - This means w = 0x0 for some string x
    - P ⇒* x in n steps (fewer than n+1)
    - By inductive hypothesis: x is a palindrome (x = x^R)
    - Therefore: $w^R$ = (0x0)^R = 0x^R0 = 0x0 = w ✓
- __Case P ⇒ 1P1 ⇒_ w:_* Similar argument ✓
    

**Conclusion:** Every string derivable from P is a palindrome.

**This completes the proof that L($G_{pal}$) = {w | w = $w^R$}!**

---

## Sentential Forms

![[Chapter-5-17.webp]]

### Definition and Purpose

When we derive strings from a grammar, we don't always end up with terminal strings right away. Intermediate strings that contain both variables and terminals are special.

**Sentential Form:**

- A string α ∈ (V ∪ T)* is a **sentential form** if S ⇒* α
- Any string derivable from the start symbol (may contain variables and terminals)

**Left-Sentential Form:**

- α is a **left-sentential form** if S ⇒*_lm α
- Derivable using leftmost derivations

**Right-Sentential Form:**

- α is a **right-sentential form** if S ⇒*_rm α
- Derivable using rightmost derivations

**Important Relationship:**

- L(G) = {all sentential forms that are in T*}
- The language consists of sentential forms containing only terminals

**Why This Matters:** Sentential forms represent the intermediate stages of deriving a string. Understanding them helps us:

- Track the derivation process
- Understand parse trees (coming next)
- Analyze ambiguity in grammars

---

## Example of Sentential Forms

![[Chapter-5-18.webp]]

### Using Grammar G1

Recall:

- E → I | E + E | E * E | (E)
- I → a | b | Ia | Ib | I0 | I1

__Example 1: E_(I + E) is a sentential form_*

Derivation:

```
E ⇒ E * E 
  ⇒ E * (E) 
  ⇒ E * (E + E) 
  ⇒ E * (I + E)
```

- This is a sentential form (derivable from S = E)
- **Not** leftmost or rightmost (we switch between left and right)

**Example 2: a*E is a left-sentential form**

Leftmost derivation:

```
E ⇒_lm E * E
  ⇒_lm I * E
  ⇒_lm a * E
```

- Always expanding the leftmost variable
- This is specifically a left-sentential form

__Example 3: E_(E + E) is a right-sentential form_*

Rightmost derivation:

```
E ⇒_rm E * E
  ⇒_rm E * (E)
  ⇒_rm E * (E + E)
```

- Always expanding the rightmost variable
- This is specifically a right-sentential form

---

## Parse Trees

![[Chapter-5-19.webp]]
### Definition

A **parse tree (or derivation tree)** is a graphical representation of how a string is derived from a grammar.

**Formal Definition:** Let G = (V, T, P, S) be a CFG. A tree is a parse tree if:

1. **Every interior node** is labeled by a variable in V
2. **Each leaf** is labeled by either:
    - A variable, or
    - A terminal, or
    - ε (empty string)
    - **Special rule:** If a leaf is ε, it must be the only child of its parent
3. **The root** is labeled by S (the start symbol)
4. **Parent-child relationship:** If an interior node is labeled A and its children (left to right) are labeled X₁, X₂, ..., Xₙ, then **A → X₁X₂...Xₙ** must be a production in P

**More Generally:**

- A parse tree can have any non-terminal as the root (not just S)
- This represents deriving strings from that particular non-terminal

### Visual Structure

Think of a parse tree as:

- **Root** = where we start (start symbol)
- **Internal nodes** = variables being expanded
- **Leaves** = the final terminals (or ε)
- **Edges** = representing production rules

---

## Parse Tree Examples

![[Chapter-5-20.webp]]
### Example 1: Deriving I+E from E

```
        E
       /|\
      E + E
      |   |
      I   E
```

**Explanation:**

- Root: E (start symbol for this subtree)
- First production: E → E + E
- Left E becomes I using E → I
- Right E remains as E
- Yield: I + E

### Example 2: Palindrome P ⇒ 0110

```
        P
       /|\
      0 P 0
        |
        P
       /|\
      1 P 1
        |
        ε
```

**Explanation:**

- Root: P
- Production P → 0P0 creates outer 0s
- Middle P uses P → 1P1 to create 1s
- Innermost P → ε
- Yield: 0110 (reading leaves left to right)

**Key Insight:** The parse tree shows the **structure** of the derivation, while the yield (leaves read left to right) gives us the actual string.

---

## Parse Trees (General Properties)
![[Chapter-5-21.webp]]

### Components of Parse Trees

**Leaves:**

- Labeled by a terminal symbol, or
- Labeled by ε (empty string)
- Cannot have children

**Interior Nodes:**

- Labeled by a variable
- Must have children
- Children correspond to the right-hand side of a production

**Root:**

- Must be labeled by the start symbol S (for full derivations)
- For sub-derivations, can be any variable

### The Connection to Grammar

Each interior node represents an application of a production rule:

- **Node label** = left-hand side of production (head)
- **Children labels** = right-hand side of production (body)

**Example:** If we have node A with children B, c, D (left to right), then the grammar must have production:

- A → BcD

---

## Example: Parse Tree for Balanced Parentheses
![[Chapter-5-22.webp]]

### Grammar

S → SS | (S) | ()

### Parse Tree for "(())()"

```cpp
        S
       / \
      S   S
     /|\ / \
    ( S )( )
      |
     ( )
```

**Reading the tree:**

1. Root S uses production S → SS (splits into two S's)
2. Left S uses S → (S), creating parentheses around another S
3. That inner S uses S → ()
4. Right S uses S → ()
5. **Yield:** (())()

**Key Observations:**

- Each level shows one production application
- The structure reveals how the string is built recursively
- Different parse trees can produce the same string (this will lead to ambiguity!)

---

## Yield of a Parse Tree

![[Chapter-5-23.webp]]

### Definition

The **yield** of a parse tree is the concatenation of all leaf labels, read from left to right (in preorder traversal).

### Example

For the tree:

```
        S
       / \
      S   S
     /|\ / \
    ( S )( )
      |
     ( )
```

**Yield = (())()**

**How to find the yield:**

1. Perform a preorder traversal (visit node, then children left to right)
2. Collect only the leaf labels
3. Concatenate them in order

**Important Properties:**

- The yield is always a string of terminals (possibly with ε)
- For a complete parse tree with root S, the yield is a string in L(G)
- The yield represents the "output" of the derivation

---

## Inference, Derivation, and Parse Trees

![[Chapter-5-24.webp]]

### The Three Perspectives Are Equivalent

There are three equivalent ways to view string generation:

**1. Recursive Inference (Bottom-up)**

- Start with terminals
- Apply productions backward (body to head)
- Build up to start symbol

**2. Derivations (Top-down)**

- Start with start symbol
- Apply productions forward (head to body)
- Expand until only terminals remain

**3. Parse Trees (Structural)**

- Shows the hierarchical structure
- Each node represents a production application
- Yield gives the terminal string

**These are three views of the same process!**

**Key Theorem (to be proven):**

1. If A ⇒*_lm w, then there exists a parse tree with root A and yield w
2. If there exists a parse tree with root A and yield w, then A ⇒*_lm w

This establishes the equivalence between derivations and parse trees.

---

## From Inference to Trees (Part 1)

![[Chapter-5-25.webp]]
![[Chapter-5-26.webp]]

![[Chapter-5-28.webp]]
![[Chapter-5-29.webp]]


### Theorem Part 1: Leftmost Derivation → Parse Tree

**Statement:** If A ⇒*_lm w (where w is all terminals), then there exists a parse tree with root A and yield w.

**Proof Strategy:** Induction on the length of the derivation

### Basis: One-Step Derivation

If A ⇒_lm a₁a₂...aₙ in one step, then:

- We must have production A → a₁a₂...aₙ
- The parse tree is simply:

```
    A
   /|\...\
  a₁ a₂  aₙ
```

- Yield is clearly a₁a₂...aₙ ✓

### Induction: Multi-Step Derivation

**Assume:** For all derivations of length ≤ n, we can construct a parse tree

**To prove:** For a derivation of length n+1, we can construct a parse tree

**Given:** A ⇒_lm w in n+1 steps

The first step must be: **A ⇒ X₁X₂...Xₖ** for some production

Then: **X₁X₂...Xₖ ⇒*_lm w** in n steps (fewer steps)

Since this is leftmost:

- We expand X₁ completely to w₁ (all terminals)
- Then X₂ completely to w₂
- And so on...
- Where w = w₁w₂...wₖ

**By inductive hypothesis:**

- For each Xᵢ ⇒*_lm wᵢ (in ≤ n steps), there exists a parse tree with root Xᵢ and yield wᵢ

**Construct the final tree:**

```
       A
    /  |  \
   X₁  X₂  Xₖ
   |   |   |
  (tree₁)(tree₂)(treeₖ)
```

- Root: A
- Children: X₁, X₂, ..., Xₖ (corresponding to production A → X₁X₂...Xₖ)
- Each Xᵢ is root of its subtree (from inductive hypothesis)
- **Yield:** w₁w₂...wₖ = w ✓

---

## From Trees to Inference (Part 2)

![[Chapter-5-30.webp]]
![[Chapter-5-31.webp]]

### Theorem Part 2: Parse Tree → Leftmost Derivation

**Statement:** If there exists a parse tree with root A and yield w, then A ⇒*_lm w.

**Proof Strategy:** Induction on the height of the tree

### Basis: Height 1

A tree of height 1 looks like:

```
    A
   /|\...\
  a₁ a₂  aₙ
```

- All children are terminals (no interior nodes below root)
- This means A → a₁a₂...aₙ must be a production
- Therefore: A ⇒_lm a₁a₂...aₙ ✓

### Induction: Height n > 1

**Assume:** For all trees of height < n, if the root is A and yield is w, then A ⇒*_lm w

**To prove:** For a tree of height n, the same holds

**Given tree:**

```
       A
    /  |  \
   X₁  X₂  Xₙ
   |   |   |
  (subtree₁)(subtree₂)(subtreeₙ)
```

Where:

- A → X₁X₂...Xₙ is a production (by definition of parse tree)
- Each subtree has height < n
- Yield of subtree i is wᵢ
- Total yield is w = w₁w₂...wₙ

**By inductive hypothesis:**

- For each subtree with root Xᵢ and yield wᵢ: **Xᵢ ⇒*_lm wᵢ**
- (Note: if Xᵢ is a terminal, then Xᵢ = wᵢ trivially)

**Construct leftmost derivation:**

```
A ⇒_lm X₁X₂...Xₙ              (use production A → X₁X₂...Xₙ)
  ⇒*_lm w₁X₂...Xₙ             (expand X₁ to w₁)
  ⇒*_lm w₁w₂X₃...Xₙ           (expand X₂ to w₂)
  ⇒*_lm w₁w₂w₃X₄...Xₙ         (expand X₃ to w₃)
  ...
  ⇒*_lm w₁w₂...wₙ = w          (expand all)
```

This is leftmost because we always expand the leftmost variable completely before moving to the next! ✓

**Conclusion:** Parse trees and leftmost derivations are equivalent representations.

---

## Applications of Context-Free Grammars

![[Chapter-5-32.webp]]

### Programming Languages

CFGs are the foundation for describing programming language syntax!

**Why CFGs?**

- Programming languages have nested structures (loops within loops, expressions within expressions)
- Regular expressions can't handle nesting
- CFGs naturally express recursive, hierarchical structure

**Common Uses:**

1. **Expression syntax:** operator precedence, parenthesization
2. **Statement structure:** if-then-else, while loops, function calls
3. **Data structures:** arrays, records, objects
4. **Program organization:** functions, classes, modules

**Example: Arithmetic expressions**

```javascript
Expr → Expr + Term | Expr - Term | Term
Term → Term * Factor | Term / Factor | Factor
Factor → Number | (Expr) | Identifier
```

This grammar:

- Handles precedence (* and / before + and -)
- Allows nested parentheses
- Defines valid expression structure

**Compilers use CFGs to:**

- **Parse** source code (check if it's syntactically valid)
- **Build parse trees** (abstract syntax trees)
- **Generate code** based on the structure

---

## If-Else Example

![[Chapter-5-33.webp]]

### The Classic If-Else Problem

**Grammar for if-else statements:**

```
Stmt → if Expr then Stmt
     | if Expr then Stmt else Stmt
     | OtherStmt
```

**The Problem:** Consider this code:

```
if E1 then if E2 then S1 else S2
```

**Two possible interpretations:**

**Interpretation 1:**

```
if E1 then (if E2 then S1 else S2)
```

The else belongs to the inner if.

**Interpretation 2:**

```
if E1 then (if E2 then S1) else S2
```

The else belongs to the outer if.

**This is grammatical ambiguity!** The grammar allows both interpretations.

**Real-world solution (most languages):**

- "else matches the nearest unmatched if"
- This is called the **dangling else problem**
- Resolved through grammar design or parsing conventions

---

## Ambiguous Grammar

![[Chapter-5-34.webp]]

### Definition of Ambiguity

**A CFG is ambiguous if:** There exists a string w ∈ L(G) that has two or more distinct parse trees.

**Equivalently:**

- A string w with multiple leftmost derivations, OR
- A string w with multiple rightmost derivations


### Example

**Grammar:**

```
S → A | AB
A → ε | a | Ab | AA
B → b | bc | Bc | bB
```

**String:** w = "aabb"

**Multiple parse trees exist!** (See diagram on page)

This means the grammar is **ambiguous**—the string has multiple structural interpretations.

**Why This Matters:**

- For programming languages, ambiguity is usually **bad**
- We want each program to have exactly one meaning
- Compilers need unambiguous grammars (or disambiguation rules)

---

## Another Ambiguity Example
![[Chapter-5-35.webp]]

### The String "a" Grammar

**Grammar:**

```
S → A | B
A → aA | ε
B → bB | ε
```

**The string "a" has multiple derivations:**

**Derivation 1 (using A):**

```
S ⇒ A ⇒ aA ⇒ aε = a
```

**Derivation 2 (using A differently):**

```
S ⇒ A ⇒ a (using A → a if that production exists)
```

Notice that even a simple string like "a" can have multiple leftmost derivations, proving ambiguity.

**Key Point:** Ambiguity is a property of the **grammar**, not the language. Sometimes we can rewrite the grammar to eliminate ambiguity.

---

## Ambiguity - Equivalent Statements

![[Chapter-5-36.webp]]

### Three Equivalent Definitions

**A CFG is ambiguous if ANY of these are true:**

1. **There exists a string w with multiple parse trees**
    
    - Different structural representations
2. **There exists a string w with multiple leftmost derivations**
    
    - Different ways to expand always choosing the leftmost variable
3. **There exists a string w with multiple rightmost derivations**
    
    - Different ways to expand always choosing the rightmost variable

### Why These Are Equivalent

![[Chapter-5-37.webp]]

**From Part 1 and Part 2 of our earlier proof:**

- Multiple parse trees ⇔ Multiple leftmost derivations
- If two parse trees differ, they produce different leftmost derivations
- If two leftmost derivations differ, they produce different parse trees

**The same relationship holds for rightmost derivations!**

**Practical Implication:**

- To prove a grammar is ambiguous, we can:
    - Show two different parse trees for some string, OR
    - Show two different leftmost derivations, OR
    - Show two different rightmost derivations
- Any one of these is sufficient proof!

---

## Example of Ambiguity: Arithmetic Expressions

![[Chapter-5-38.webp]]

### The Classic Expression Grammar

**Simple grammar:**

```
E → E + E | E * E | (E) | id
```

**This grammar is ambiguous!**

**Example string:** id + id * id

**Derivation 1 (addition first):**

```
E ⇒ E + E 
  ⇒ id + E
  ⇒ id + E * E
  ⇒ id + id * E
  ⇒ id + id * id
```

**Parse tree 1:**

```
      E
     /|\
    E + E
    |  /|\
   id E * E
      |   |
     id  id
```

**Meaning:** (id + id) * id — **Wrong!** Addition happens first.

**Derivation 2 (multiplication first):**

```
E ⇒ E * E
  ⇒ E + E * E
  ⇒ id + E * E
  ⇒ id + id * E
  ⇒ id + id * id
```

**Parse tree 2:**

```
      E
     /|\
    E * E
   /|\  |
  E + E id
  |   |
 id  id
```

**Meaning:** id + (id * id) — **Correct!** Multiplication has higher precedence.

**The problem:** The grammar doesn't encode operator precedence!

---

## Another Ambiguity Example

![[Chapter-5-39.webp]]
### More Expression Ambiguity

**The same grammar:** E → E + E | E * E | (E) | id

**Another ambiguous string:** id + id + id

**Two parse trees:**

**Tree 1 (left-associative):**

```
      E
     /|\
    E + E
   /|\  |
  E + E id
  |   |
 id  id
```

**Meaning:** (id + id) + id

**Tree 2 (right-associative):**

```
      E
     /|\
    E + E
    |  /|\
   id E + E
      |   |
     id  id
```

**Meaning:** id + (id + id)

**Two problems in this grammar:**

1. **No precedence:** Can't distinguish * from +
2. **No associativity:** Can't determine grouping for repeated operators

Both need to be fixed!

---

## Removing Ambiguity

![[Chapter-5-40.webp]]

### The Challenge

**Important Facts:**

1. There is **no algorithm** to determine if an arbitrary CFG is ambiguous
2. There is **no algorithm** to remove all ambiguity from a CFG
3. **However**, for many practical grammars, we can manually remove ambiguity!

### Strategy for Expressions

**Two sources of ambiguity:**

1. **Precedence not considered:** Operations like + and * treated equally
2. **Identical operators can group from left or right:** No associativity rules

**Solution approach:**

- Create separate grammar levels for different precedence
- Enforce left or right associativity through grammar structure
- Use recursion placement to control grouping

---

## Removing Ambiguity - Step 1

![[Chapter-5-41.webp]]

### Fixing Precedence

**Original ambiguous grammar:**

```
E → E + E | E * E | (E) | id
```

**Improved grammar with precedence:**

```
E → E + T | T
T → T * F | F
F → (E) | id
```

**How this works:**

**Three levels:**

1. **E (Expression):** Handles addition (lowest precedence)
2. **T (Term):** Handles multiplication (higher precedence)
3. **F (Factor):** Handles atoms and parenthesized expressions (highest precedence)

**Example derivation:** id + id * id

```
E ⇒ E + T          (addition at top level)
  ⇒ T + T          (left side reduces to term)
  ⇒ F + T          (term reduces to factor)
  ⇒ id + T         (factor is id)
  ⇒ id + T * F     (right side has multiplication)
  ⇒ id + F * F     (reduce to factors)
  ⇒ id + id * F
  ⇒ id + id * id
```

**Parse tree:**

```
       E
      /|\
     E + T
     |  /|\
     T T * F
     | |   |
     F F  id
     | |
    id id
```

**Result:** Multiplication is forced to occur in a term, which is nested deeper than addition. This enforces precedence! ✓

---

## Removing Ambiguity - Step 2

![[Chapter-5-42.webp]]

### Fixing Associativity

**Current grammar:**

```
E → E + T | T
T → T * F | F
F → (E) | id
```

**This grammar already enforces left-associativity!**

**Why?**

- Notice: E → **E** + T (recursive on the left)
- Notice: T → **T** * F (recursive on the left)

**Example:** id + id + id

```
E ⇒ E + T
  ⇒ (E + T) + T         (first + groups with left E)
  ⇒ (T + T) + T
  ⇒ (F + F) + F
  ⇒ (id + id) + id
```

**Parse tree:**

```
       E
      /|\
     E + T
    /|\  |
   E + T F
   |   | |
   T   F id
   |   |
   F  id
   |
  id
```

**Left-associative:** Operations group from left to right: (id + id) + id

**For right-associativity**, we would use:

- E → T + **E** | T (recursive on the right)

---

## Removing Ambiguity - Complete Solution

![[Chapter-5-43.webp]]

### The Unambiguous Expression Grammar

**Final grammar:**

```
E → E + T | E - T | T
T → T * F | T / F | F
F → (E) | id | number
```

**Properties:** ✓ **Precedence:** * and / have higher precedence than + and - ✓ **Associativity:** All operators are left-associative ✓ **Unambiguous:** Each expression has exactly one parse tree ✓ **Complete:** Handles parentheses to override precedence

**Example:** id + id * id

- **Only one parse tree possible**
- Multiplication happens first (deeper in tree)
- Addition happens second (higher in tree)
- Parsed as: id + (id * id)

**Example:** id + id + id

- **Only one parse tree possible**
- Left-to-right grouping: (id + id) + id

**This is the standard approach used in real programming language grammars!**

---

## Inherently Ambiguous Languages

![[Chapter-5-44.webp]]

### When Ambiguity Cannot Be Removed

**Definition:** A context-free language L is **inherently ambiguous** if **every** CFG that generates L is ambiguous.

**Key Insight:**

- Ambiguity is usually a property of the grammar (how we write it)
- But for some languages, ambiguity is **unavoidable**
- No matter how cleverly we design the grammar, ambiguity remains

### Example of Inherent Ambiguity

**Classic example:**

```
L = {aⁱbʲcᵏ | i = j or j = k, i,j,k ≥ 1}
```

**This language includes:**

- Strings where #a's = #b's (regardless of #c's)
- Strings where #b's = #c's (regardless of #a's)

**Why inherently ambiguous?**

- Strings like a²b²c² are in the language for **two reasons**:
    - #a's = #b's (2 = 2)
    - #b's = #c's (2 = 2)
- Any grammar must have two different derivation paths for such strings
- There's no way to "choose" which constraint to follow

**Practical Impact:**

- Most programming languages are designed to **avoid** inherent ambiguity
- Parser generators require unambiguous grammars
- Language designers carefully craft grammars to be unambiguous

---

## Summary and Key Takeaways

### Context-Free Grammars

**Definition:** G = (V, T, P, S)

- V: Variables (non-terminals)
- T: Terminals
- P: Productions (V → (V ∪ T)*)
- S: Start symbol

### Key Concepts

**Derivations:**

- ⇒ : one-step derivation
- ⇒* : zero or more steps
- ⇒_lm : leftmost derivation
- ⇒_rm : rightmost derivation

**Parse Trees:**

- Graphical representation of derivations
- Root: start symbol
- Leaves: terminals (yield)
- Structure: corresponds to production applications

**Language:**

- L(G) = {w ∈ T* | S ⇒* w}
- All terminal strings derivable from start symbol

### Important Results

**Equivalences:**

- Recursive inference ⟺ Derivations ⟺ Parse trees
- Multiple parse trees ⟺ Multiple leftmost derivations ⟺ Multiple rightmost derivations

**Ambiguity:**

- A grammar is ambiguous if some string has multiple parse trees
- Ambiguity can often be removed by grammar redesign
- Some languages are inherently ambiguous

**Applications:**

- Programming language syntax
- Expression parsing with precedence and associativity
- Compiler design

### What CFGs Can and Cannot Do

**CFGs can express:**

- Nested structures (palindromes, balanced parentheses)
- Equal counts (aⁿbⁿ)
- Recursive patterns
- Most programming language constructs

**CFGs cannot express:**

- Triple equality (aⁿbⁿcⁿ) — beyond CFG power
- Context-sensitive patterns
- Some complex dependencies

**Power hierarchy:**

```
Regular Languages ⊂ Context-Free Languages ⊂ Context-Sensitive Languages ⊂ Recursively Enumerable Languages
```

---

## Study Tips and Practice

### To Master CFGs:

1. **Practice writing grammars:**
    
    - Start with simple languages
    - Build up to complex nested structures
    - Check your work with derivations
2. **Practice derivations:**
    
    - Try both leftmost and rightmost
    - Draw parse trees to verify
    - Look for ambiguity
3. **Understand precedence and associativity:**
    
    - Know how to encode them in grammars
    - Practice with expression grammars
    - Recognize ambiguous vs. unambiguous
4. **Recognize patterns:**
    
    - Equal counts: S → aSb | ε
    - Palindromes: P → aPa | bPb | ε
    - Nested structures: use recursive productions

### Common Mistakes to Avoid:

❌ Confusing terminals and variables ❌ Forgetting the empty string in recursive base cases ❌ Not checking for ambiguity ❌ Mixing up precedence levels ❌ Incorrect associativity in recursive productions

### Key Questions to Ask:

✓ Is this grammar ambiguous? ✓ Does it generate the correct language? ✓ Does it enforce proper precedence? ✓ Is it left- or right-associative? ✓ Can I prove correctness?

---

**End of Notes**