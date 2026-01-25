# Pushdown Automata (PDA)

## Chapter Overview


### Topics Covered

This chapter explores **Pushdown Automata (PDA)**, the computational model that recognizes context-free languages:

1. **Introduction to PDAs** - Basic concepts and motivation
2. **Definition of PDA** - Formal structure and components
3. **The Language of a PDA** - Two acceptance methods
4. **Equivalence of PDAs and CFGs** - Converting between models
5. **Deterministic PDAs** - A restricted but important subclass

**Key Relationship:**

```
Regular Languages ⊂ DPDA Languages ⊂ CFL Languages
        ↓                  ↓                ↓
       DFA              DPDA              PDA
```

---

## 6.0 Introduction

![[Chapter-6-Pushdown-Automata-3.webp]]
![[Chapter-6-Pushdown-Automata-4.webp]]
![[Chapter-6-Pushdown-Automata-5.webp]]

### What is a Pushdown Automaton?

A **Pushdown Automaton (PDA)** is a computational model that extends finite automata with a **stack** data structure.

**Key Concept:** PDA = ε-NFA + Stack

**Intuitive Analogy:** Think of a PDA like a person reading a book (the input tape) while using a stack of plates (the stack memory). As they read, they can:

- Push plates onto the stack (store information)
- Pop plates from the stack (retrieve information)
- Only access the top plate (LIFO - Last In, First Out)
- Make decisions based on what they're reading AND what's on top of the stack

### Basic Properties

**What a PDA can do:**

- Read input symbols from left to right (like an NFA)
- Push symbols onto a stack
- Pop symbols from the stack
- Read only the top of the stack
- Make transitions based on: current state, input symbol, AND top stack symbol

**Two Acceptance Methods:**

1. **Acceptance by Final State**
    
    - Accept when reaching a designated accepting state
    - Stack content doesn't matter
    - Similar to how NFAs accept
2. **Acceptance by Empty Stack**
    
    - Accept when the stack becomes empty
    - Current state doesn't matter
    - Unique to PDAs

**Important:** These two methods are equivalent in power—any language accepted by one method can be accepted by the other!

### Nondeterminism vs. Determinism

**Standard PDA:**

- **Nondeterministic** by default
- Can have multiple possible moves from any configuration
- More powerful than deterministic version

**Deterministic PDA (DPDA):**

- At most one move possible from any configuration
- Similar to parsers used in compilers
- Less powerful than general PDAs
- Still more powerful than DFAs

### Why Do We Need PDAs?

**The Limitation of Finite Automata:** Finite automata (DFA/NFA) cannot recognize languages like:

- L = {0ⁿ1ⁿ | n ≥ 1} (equal numbers of 0s and 1s)
- Palindromes, balanced parentheses, etc.

**Problem:** Finite states cannot "count" or "remember" unbounded amounts of information.

**Solution:** The stack provides **infinite memory** in a structured way:

- Can store unlimited information
- Can "count" by pushing and popping
- Can remember and match patterns

**Example:** For L = {0ⁿ1ⁿ | n ≥ 1}:

1. Push each 0 onto the stack
2. For each 1, pop a 0 from the stack
3. Accept if stack is empty when input ends
4. The stack "counts" the 0s and verifies equal 1s!

---

## 6.1 Definition of PDA

### 6.1.1 Informal Introduction

![[Chapter-6-Pushdown-Automata-6.webp]]

### The Power and Limitations of a Stack

**Advantage - Infinite Memory:**

- The stack can grow arbitrarily large
- Can remember unlimited amounts of information
- No fixed size limit (unlike finite states)

**Weakness - LIFO Access:**

- Can only read/write at the **top** of the stack
- Cannot access middle or bottom elements directly
- Information retrieved in reverse order from storage

**What This Means:**

**✓ Can Accept:**

- L_wwR = {wwᴿ | w ∈ (0 + 1)*} - palindromes with reversal
    - Push first half onto stack
    - Pop and match with second half
    - Works because we access in reverse order!

**✗ Cannot Accept:**

- L = {aⁿbⁿcⁿ | n ≥ 1} - triple equality
    - Need to count a's, b's, AND c's
    - Stack can only track one relationship at a time
    - Would need two independent counters

**Intuitive Understanding:** A stack is like a tunnel - you can only access from one end. Items go in and come out in reverse order. This is perfect for matching patterns like "123...321" but not for "111...222...333" where you need to count three things simultaneously.

### Graphical Model of a PDA

![[Chapter-6-Pushdown-Automata-7.webp]]

**Components of a PDA:**

```
┌─────────────────┐
│  Input Tape     │  ← Read-only, left to right
│  a b c d e ...  │
└────────┬────────┘
         ↓
    ┌────────────┐
    │  Tape      │
    │  Reader    │
    └─────┬──────┘
          ↓
    ┌─────────────┐
    │   Finite    │  ← Control logic
    │   State     │
    │   Control   │
    └──────┬──────┘
           ↓
    ┌──────────────┐
    │    Stack     │
    │   Reader &   │
    │    Writer    │
    └──────┬───────┘
           ↓
    ┌──────────────┐
    │   Top ← X    │  ← Only accessible position
    │      Y       │
    │      Z       │
    │   Bottom     │
    └──────────────┘
```

### Stack Operations

![[Chapter-6-Pushdown-Automata-8.webp]]

The PDA can perform complex operations on the stack by **replacing the top symbol** with a string:

**1. Replace with a Single Symbol:**

- Top: X → Y
- Example: Replace 0 with 1

**2. Replace with a String:**

- Top: X → YZ
- X is replaced by Z, Y is pushed on top
- Example: X → ABC means X becomes C, B goes on top of C, A goes on top of B

**3. Pop (Replace with ε):**

- Top: X → ε
- Removes the top symbol
- Example: Stack "XYZ" becomes "YZ"

**4. Leave Unchanged:**

- Top: X → X
- Useful for reading without modifying

**Important:** In one move, we can:

- Read one input symbol (or ε)
- Replace the top stack symbol with a string (including ε)
- Change state

This is more flexible than just "push" or "pop"!

---

## Example: Accepting L_wwR

![[Chapter-6-Pushdown-Automata-9.webp]]

### Problem: Design a PDA for L_wwR = {wwᴿ | w ∈ (0 + 1)*}

**Examples of strings in L_wwR:**

- "0110" (01·10, where 10 = 01ᴿ)
- "1111" (11·11)
- "010010" (010·010)
- "ε" (empty string)

**Strategy:**

**Phase 1 - Store (State q₀):**

- Copy input symbols onto the stack
- Keep pushing as we read

**Phase 2 - Guess Middle (Nondeterministic choice):**

- At any point, **nondeterministically guess** we've reached the middle
- Transition to state q₁
- **This is the key nondeterministic decision!**

**Phase 3 - Match (State q₁):**

- Compare remaining input with stack contents
- For each input symbol, pop matching symbol from stack
- If symbols don't match → reject (dead end)

**Phase 4 - Accept:**

- If we successfully match all remaining input
- AND the stack empties at the same time
- Then we guessed the middle correctly → accept!

**Why Nondeterminism is Necessary:**

- We don't know where the middle of wwᴿ is
- Could be after 1 symbol, 2 symbols, 3 symbols, etc.
- The PDA tries all possibilities in parallel (nondeterministically)
- Only the correct guess leads to acceptance

**Example Trace for "0110":**

```
Correct path (guess middle after position 2):
Push 0 → Stack: 0
Push 1 → Stack: 10
Guess middle → Switch to matching mode
Match 1 (pop) → Stack: 0
Match 0 (pop) → Stack: empty
Accept! ✓

Wrong path (guess middle after position 1):
Push 0 → Stack: 0
Guess middle → Switch to matching mode
Match 1 ≠ 0 → Reject ✗
```

---

## 6.1.2 Formal Definition

![[Chapter-6-Pushdown-Automata-10.webp]]
![[Chapter-6-Pushdown-Automata-11.webp]]

### The 7-Tuple Definition

A **Pushdown Automaton** is formally defined as:

**P = (Q, Σ, Γ, δ, q₀, Z₀, F)**

**Components:**

**1. Q - Finite Set of States**

- Like in NFAs/DFAs
- Control logic states
- Example: Q = {q₀, q₁, q₂}

**2. Σ - Input Alphabet**

- Set of symbols that can appear in input strings
- Example: Σ = {0, 1}

**3. Γ - Stack Alphabet**

- Set of symbols that can be pushed onto the stack
- May overlap with Σ, but often different
- Example: Γ = {0, 1, Z₀}

**4. δ - Transition Function**

- Most complex part!
- __δ: Q × (Σ ∪ {ε}) × Γ → 2^(Q × Γ_)_*
- Input: (current state, input symbol or ε, top stack symbol)
- Output: set of (next state, replacement string for top symbol)

**Notation:** δ(q, a, X) = {(p₁, γ₁), (p₂, γ₂), ...}

- q ∈ Q (current state)
- a ∈ Σ ∪ {ε} (input symbol to consume, or ε for no input)
- X ∈ Γ (current top stack symbol)
- pᵢ ∈ Q (next state)
- γᵢ ∈ Γ* (string to replace X on the stack)

**Understanding γ (the replacement string):**

|γ value|Effect|Example|
|---|---|---|
|ε|Pop X (remove top)|Stack "XYZ" → "YZ"|
|X|No change|Stack stays "XYZ"|
|YZ|Replace X with Z, push Y|Stack "XYZ" → "YZZ"|
|ABC|Replace X with C, push B then A|Stack "XYZ" → "ABCYZ"|

**5. q₀ - Start State**

- Initial state (q₀ ∈ Q)
- Where execution begins

**6. Z₀ - Start Stack Symbol**

- Initial stack contents (Z₀ ∈ Γ)
- Stack begins with just this symbol
- Often used as a "bottom marker"

**7. F - Set of Accepting States**

- F ⊆ Q
- For acceptance by final state method
- May be empty for acceptance by empty stack

---

## Example 6.2: Formal PDA for L_wwR

![[Chapter-6-Pushdown-Automata-12.webp]]
![[Chapter-6-Pushdown-Automata-13.webp]]

### Formal Definition

**P = ({q₀, q₁, q₂}, {0, 1}, {0, 1, Z₀}, δ, q₀, Z₀, {q₂})**

**Transition Function δ:**

**Phase 1 - Initial Pushing (from start):**

- δ(q₀, 0, Z₀) = {(q₀, 0Z₀)} — Read 0, push 0 on top of Z₀
- δ(q₀, 1, Z₀) = {(q₀, 1Z₀)} — Read 1, push 1 on top of Z₀

**Phase 2 - Continuing to Push:**

- δ(q₀, 0, 0) = {(q₀, 00)} — Read 0 when 0 on top, push another 0
- δ(q₀, 0, 1) = {(q₀, 01)} — Read 0 when 1 on top, push 0
- δ(q₀, 1, 0) = {(q₀, 10)} — Read 1 when 0 on top, push 1
- δ(q₀, 1, 1) = {(q₀, 11)} — Read 1 when 1 on top, push another 1

**Phase 3 - Guessing Middle (ε-transitions):**

- δ(q₀, ε, Z₀) = {(q₁, Z₀)} — Guess middle with empty string (only Z₀)
- δ(q₀, ε, 0) = {(q₁, 0)} — Guess middle when 0 on top
- δ(q₀, ε, 1) = {(q₁, 1)} — Guess middle when 1 on top

**Phase 4 - Matching:**

- δ(q₁, 0, 0) = {(q₁, ε)} — Match 0 with 0 on stack, pop
- δ(q₁, 1, 1) = {(q₁, ε)} — Match 1 with 1 on stack, pop

**Phase 5 - Accepting:**

- δ(q₁, ε, Z₀) = {(q₂, Z₀)} — When stack has only Z₀, accept

### Understanding the Design

**Key Ideas:**

1. **Z₀ as Bottom Marker:**
    
    - Never removed during normal operation
    - Signals when we've matched everything
    - Distinguishes "empty of data" from "completely empty"
2. **Nondeterministic Guessing:**
    
    - ε-transitions from q₀ to q₁ can fire at ANY time
    - PDA explores all possible middle positions
    - Only correct guess leads to acceptance
3. **State Meanings:**
    
    - **q₀:** Still reading first half (or haven't decided)
    - **q₁:** Committed to matching mode
    - **q₂:** Success! (accepting state)

**Example Execution for "01":**

```
(q₀, 01, Z₀) ⊢ (q₀, 1, 0Z₀)    [push 0]
             ⊢ (q₀, ε, 10Z₀)   [push 1]
             ⊢ (q₁, ε, 10Z₀)   [guess middle - WRONG]
             Dead end (can't match ε)

Alternative path:
(q₀, 01, Z₀) ⊢ (q₀, 1, 0Z₀)    [push 0]
             ⊢ (q₁, 1, 0Z₀)    [guess middle after '0']
             Dead end (1 ≠ 0)

Alternative path:
(q₀, 01, Z₀) ⊢ (q₁, 01, Z₀)    [guess middle immediately]
             Dead end (0 ≠ Z₀)

Hmm, "01" is not in L_wwR! It's not a palindrome.
```

**Example Execution for "0110":**

```
(q₀, 0110, Z₀) ⊢ (q₀, 110, 0Z₀)     [push 0]
               ⊢ (q₀, 10, 10Z₀)     [push 1]
               ⊢ (q₁, 10, 10Z₀)     [guess middle HERE ✓]
               ⊢ (q₁, 0, 0Z₀)       [match 1]
               ⊢ (q₁, ε, Z₀)        [match 0]
               ⊢ (q₂, ε, Z₀)        [accept!]
```

---

## 6.1.3 Graphical Notation for PDAs

![[Chapter-6-Pushdown-Automata-14.webp]]
![[Chapter-6-Pushdown-Automata-15.webp]]**

### Transition Diagrams

Like NFAs, we can draw PDAs as state diagrams. The notation is extended to include stack operations.

**Arc Notation:** `a, X/α`

**Meaning:** On an arc from state p to state q:

- **a:** Input symbol to read (or ε for no input)
- **X:** Current top stack symbol
- **α:** String to replace X with

**Formal Relationship:**

- Arc labeled `a, X/α` from p to q
- Means: (q, α) ∈ δ(p, a, X)

### Example 6.3: Transition Diagram for L_wwR
![[Chapter-6-Pushdown-Automata-15.webp]]

```
         ε, Z₀/Z₀
         ε, 0/0      ┌──────────┐  ε, Z₀/Z₀
         ε, 1/1   ┌──│    q₁    │──────┐
    ┌──────────────┘  └──────────┘      │
    │                                    ↓
    │  0, 0/ε                    ┌──────────┐
    │  1, 1/ε                    │    q₂    │ (accept)
    │                            └──────────┘
┌──────────┐
│   q₀     │ (start)
└──────────┘
    ↑ │
    │ │ 0, Z₀/0Z₀
    │ │ 1, Z₀/1Z₀
    │ │ 0, 0/00
    │ │ 0, 1/01
    │ │ 1, 0/10
    │ │ 1, 1/11
    │ └──────────┐
    └────────────┘
    (self-loops)
```

### Where is the Nondeterminism?

**Key Nondeterministic Choices:**

1. **Self-loops in q₀:** Can always choose to keep pushing
2. **ε-transitions to q₁:** Can choose to "guess middle" at any time
3. **Multiple options:** At many configurations, multiple transitions are possible

**Example from state q₀ with "0" on top of stack and "0" as next input:**

- Option 1: δ(q₀, 0, 0) = {(q₀, 00)} — Keep pushing
- Option 2: δ(q₀, ε, 0) = {(q₁, 0)} — Guess middle (don't consume input)

Both are possible! The PDA explores both paths.

**Visual Identification:**

- Multiple arcs leaving the same state with compatible labels
- ε-transitions provide additional choices
- Any configuration where |δ(q, a, X)| > 1 is nondeterministic

---

## 6.1.4 Instantaneous Descriptions

![[Chapter-6-Pushdown-Automata-16.webp]]
![[Chapter-6-Pushdown-Automata-17.webp]]



### Configuration Representation

An **Instantaneous Description (ID)** is a snapshot of the PDA's current configuration.

**ID = (q, w, γ)**

**Components:**

- **q:** Current state (q ∈ Q)
- **w:** Remaining input to be read (w ∈ Σ*)
- **γ:** Current stack contents, top at left (γ ∈ Γ*)

**Example IDs:**

- (q₀, 0110, Z₀) — At start, full input, stack has only Z₀
- (q₀, 110, 0Z₀) — Read one 0, stack has 0 on top of Z₀
- (q₁, 10, 10Z₀) — In matching mode, stack has "10Z₀" (top to bottom)

### The Move Relation (⊢)

**Notation:** (q, aw, Xβ) ⊢ (p, w, αβ)

**Meaning:**

- If (p, α) ∈ δ(q, a, X)
- Then we can move from (q, aw, Xβ) to (p, w, αβ)

**Breaking it down:**

- Read input symbol **a** (or ε if a = ε)
- Current state **q** → next state **p**
- Top of stack **X** is replaced by **α**
- Rest of stack **β** remains unchanged
- Remaining input **aw** → **w** (consumed **a**)

**Closure:** ⊢* means "zero or more moves"

- (q, w, γ) ⊢* (q, w, γ) — Reflexive (zero moves)
- If (q, w, γ) ⊢* (p, x, α) and (p, x, α) ⊢ (r, y, β) then (q, w, γ) ⊢* (r, y, β) — Transitive

### Example 6.4: Moves for Input "1111"

![[Chapter-6-Pushdown-Automata-18.webp]]
**Accepting path:**

```
(q₀, 1111, Z₀) 
⊢ (q₀, 111, 1Z₀)      [push 1]
⊢ (q₀, 11, 11Z₀)      [push 1]
⊢ (q₁, 11, 11Z₀)      [guess middle]
⊢ (q₁, 1, 1Z₀)        [match 1, pop]
⊢ (q₁, ε, Z₀)         [match 1, pop]
⊢ (q₂, ε, Z₀)         [accept]
```

**Key observations:**

- Input decreases left to right (consumed)
- Stack changes based on operations
- Multiple other paths exist (dead ends not shown)
- Only one path needs to accept!

**Dead-end paths (examples):**

```
(q₀, 1111, Z₀) ⊢ (q₁, 1111, Z₀) [guess middle too early]
Dead end: can't match 1 with Z₀

(q₀, 1111, Z₀) ⊢ (q₀, 111, 1Z₀) ⊢ (q₀, 11, 11Z₀) 
              ⊢ (q₀, 1, 111Z₀) ⊢ (q₀, ε, 1111Z₀)
              ⊢ (q₁, ε, 1111Z₀)
Dead end: no input left to match, but stack not empty
```

---

## Theorems About IDs

![[Chapter-6-Pushdown-Automata-19.webp]]
![[Chapter-6-Pushdown-Automata-20.webp]]
### Theorem 6.5: Adding to ID

**Statement:** If P = (Q, Σ, Γ, δ, q₀, Z₀, F) is a PDA and (q, x, α) ⊢* (p, y, β), then for any w ∈ Σ* and γ ∈ Γ*: __(q, xw, αγ) ⊢_ (p, yw, βγ)_*

**Intuitive Meaning:**

- If we can make a sequence of moves
- We can add "stuff" to the end of the input
- We can add "stuff" to the bottom of the stack
- The same sequence of moves still works!

**Why This Works:**

- PDAs only read input left-to-right
- PDAs only access top of stack
- Adding to the right of input doesn't interfere
- Adding to bottom of stack doesn't interfere

**Example:** If (q, 01, XY) ⊢* (p, ε, Z), then:

- (q, 01**10**, XY**ABC**) ⊢* (p, **10**, Z**ABC**)
- The moves for "01" and "XY" are unaffected by the additions

**Note:** The reverse is NOT true in general!

- Can't always remove stuff and preserve moves
- Removed parts might be needed!

### Theorem 6.6: Removing from ID

**Statement:** If P = (Q, Σ, Γ, δ, q₀, Z₀, F) is a PDA and (q, xw, α) ⊢* (p, yw, β), then: __(q, x, α) ⊢_ (p, y, β)_*

**Intuitive Meaning:**

- If a sequence of moves works with extra input **w** at the end
- The same moves work without that extra input
- Because the PDA doesn't touch **w** during those moves!

**Why This Works:**

- **w** appears unchanged in both IDs
- Means it was never read during the moves
- So its presence or absence doesn't matter

**Example:** If (q, 01**10**, XY) ⊢* (p, ε**10**, Z), then:

- (q, 01, XY) ⊢* (p, ε, Z)
- The "10" wasn't touched, so removing it doesn't matter

**Together:** Theorems 6.5 and 6.6 help us reason about PDA behavior

- Can focus on relevant parts of input/stack
- Can generalize from specific examples

---

## 6.2 The Language of a PDA

![[Chapter-6-Pushdown-Automata-21.webp]]

### Two Acceptance Methods

**Important Facts:**

1. **Two equivalent definitions:**
    
    - Acceptance by **final state**
    - Acceptance by **empty stack**
2. **Equivalence theorem:**
    
    - L has a PDA accepting by final state ⟺ L has a PDA accepting by empty stack
    - The class of languages is the same!
3. **For a specific PDA:**
    
    - L(P) (by final state) ≠ N(P) (by empty stack) usually
    - Same machine, different acceptance criteria → different languages
4. **Conversions:**
    
    - Can convert PDA accepting by final state → PDA accepting by empty stack
    - Can convert PDA accepting by empty stack → PDA accepting by final state

**Intuitive Comparison:**

|Aspect|Final State|Empty Stack|
|---|---|---|
|Accept when|In accepting state|Stack is empty|
|Stack matters?|No|Yes (must be empty)|
|State matters?|Yes (must be in F)|No (any state)|
|Like...|NFA acceptance|Unique to PDAs|

---

## 6.2.1 Acceptance by Final State

![[Chapter-6-Pushdown-Automata-22.webp]]**

### Definition

For PDA P = (Q, Σ, Γ, δ, q₀, Z₀, F):

__L(P) = {w | (q₀, w, Z₀) ⊢_ (q, ε, α) for some q ∈ F and any α ∈ Γ_}**

**Breaking it down:**

- **w:** An input string
- **Start:** (q₀, w, Z₀) — begin in start state with full input and initial stack
- **End:** (q, ε, α) — end with:
    - **q ∈ F:** Some accepting state (must be in F!)
    - **ε:** All input consumed
    - **α:** Any stack contents (don't care what's on stack)

**Key Points:**

- Must consume all input
- Must end in an accepting state
- Stack contents are **irrelevant**
- Similar to NFA acceptance

**Example:** For the L_wwR PDA from Example 6.2:

- F = {q₂}
- String "1111" is accepted because: (q₀, 1111, Z₀) ⊢* (q₂, ε, Z₀)
    - All input consumed ✓
    - State q₂ ∈ F ✓

### Example 6.7: Proof that PDA Accepts L_wwR

**Claim:** The PDA from Example 6.2 accepts exactly L_wwR

**Proof idea:**

- **(⊆) L(P) ⊆ L_wwR:** If PDA accepts w, then w ∈ L_wwR
- **(⊇) L_wwR ⊆ L(P):** If w ∈ L_wwR, then PDA accepts w

**Detailed proof:** See textbook (induction on string length)

---

## 6.2.2 Acceptance by Empty Stack

![[Chapter-6-Pushdown-Automata-23.webp]]
![[Chapter-6-Pushdown-Automata-24.webp]]


### Definition

For PDA P = (Q, Σ, Γ, δ, q₀, Z₀):

__N(P) = {w | (q₀, w, Z₀) ⊢_ (q, ε, ε) for any q ∈ Q}_*

**Breaking it down:**

- **w:** An input string
- **Start:** (q₀, w, Z₀) — begin normally
- **End:** (q, ε, ε) — end with:
    - **q:** Any state (don't care about state!)
    - **ε:** All input consumed
    - **ε:** Stack is empty (this is what matters!)

**Key Points:**

- Must consume all input
- Must empty the stack completely
- Final state is **irrelevant**
- Can drop F from tuple, making it a **6-tuple**

**Notation:** For empty stack acceptance, sometimes write:

- P = (Q, Σ, Γ, δ, q₀, Z₀) — no F needed!

### Example 6.8: Modifying L_wwR PDA for Empty Stack

**Original transition (final state version):**

```
δ(q₁, ε, Z₀) = {(q₂, Z₀)}  [Keep Z₀, enter accepting state]
```

**Modified transition (empty stack version):**

```
δ(q₁, ε, Z₀) = {(q₂, ε)}   [Pop Z₀, empty the stack!]
```

**What changed:**

- Instead of keeping Z₀ and changing state
- We **pop Z₀** to empty the stack
- Stack becomes empty → accept by empty stack!

**Accepting sequence for "1111":**

```
(q₀, 1111, Z₀) 
⊢* (q₁, ε, Z₀)    [after matching]
⊢ (q₂, ε, ε)      [pop Z₀, stack empty → accept!]
```

---

## 6.2.3 From Empty Stack to Final State


**

### Theorem 6.9: Conversion Algorithm

**Statement:** If L = N(P_N) for some PDA P_N, then there exists a PDA P_F such that L = L(P_F).

**Translation:** Any language accepted by empty stack can also be accepted by final state.

### The Construction

![[Chapter-6-Pushdown-Automata-25.webp]]**

**Idea:** Wrap the original PDA with:

- New start state p₀ that **pushes a bottom marker X₀**
- New accepting state p_f that **detects when X₀ is exposed**

**Graphical representation:**

```
     ┌────────────────────────────────────┐
     │         Original PDA P_N           │
p₀ ──┤  (accepts by empty stack)          ├── p_f
     │                                    │
     └────────────────────────────────────┘
  
  ε, X₀/Z₀X₀         When X₀ exposed         ε, X₀/ε
  (setup)           (P_N emptied stack)      (accept)
```

**Given:** P_N = (Q, Σ, Γ, δ_N, q₀, Z₀)

**Construct:** P_F = (Q ∪ {p₀, p_f}, Σ, Γ ∪ {X₀}, δ_F, p₀, X₀, {p_f})

**Transition function δ_F:**

1. **δ_F(p₀, ε, X₀) = {(q₀, Z₀X₀)}**
    
    - From new start state, push original start symbol Z₀
    - Keep bottom marker X₀ below it
    - Enter original start state q₀
2. **For all q ∈ Q, a ∈ Σ ∪ {ε}, Y ∈ Γ:**
    
    - **δ_F(q, a, Y) = δ_N(q, a, Y)**
    - Copy all original transitions!
    - Simulate P_N exactly
3. **For all q ∈ Q:**
    
    - **δ_F(q, ε, X₀) = {(p_f, ε)}**
    - When X₀ is exposed (stack otherwise empty), jump to accepting state
    - Pop X₀ (though this doesn't matter for final state acceptance)

### Why This Works

**Key Insight:** X₀ acts as a "bottom detector"

**Behavior:**

1. Push X₀ at the bottom initially
2. Run original PDA normally
3. When original PDA empties its stack, X₀ becomes exposed
4. Detecting X₀ means: "original PDA would have accepted by empty stack"
5. Transition to p_f → accept by final state!

**Example:** For string w in L(P_N):

```
(p₀, w, X₀)          [start]
⊢ (q₀, w, Z₀X₀)      [setup: push Z₀, keep X₀ below]
⊢* (q, ε, X₀)        [simulate P_N until it empties Z₀...Γ*]
⊢ (p_f, ε, ε)        [X₀ exposed, accept!]
```

**Result:** w ∈ L(P_F) ⟺ w ∈ N(P_N)

**Proof of correctness:** See textbook (shows the two languages are equal)

---

## Example 6.10: If-Else Error Detection
![[Chapter-6-Pushdown-Automata-28.webp]]

### Problem Description

**Design a PDA to detect if-else balance errors.**

**Language:**

- **i** represents "if"
- **e** represents "else"
- Accept when #else > #if (too many else's)

**Strategy:**

- Push Z for each "if" seen
- Pop Z for each "else" seen
- If stack empties before input ends → too many else's → accept!

### PDA by Empty Stack

![[Chapter-6-Pushdown-Automata-29.webp]]


**P_N = ({q}, {i, e}, {Z}, δ_N, q, Z)**

**Transition diagram:**

```
        ┌──────────┐
     ───│    q     │
   start└──────────┘
         ↑ │
    i,Z/ZZ│ │e,Z/ε
         │ ↓
        (loops)
```

**Transitions:**

- **δ_N(q, i, Z) = {(q, ZZ)}** — When see "if", push Z (count it)
- **δ_N(q, e, Z) = {(q, ε)}** — When see "else", pop Z (balance one if)

**How it works:**

- Start with one Z on stack
- Each "i" adds a Z (increment counter)
- Each "e" removes a Z (decrement counter)
- Stack empties when: #e = #i + 1 (one more else than ifs plus initial)

**Example: w = "iee"**

```
(q, iee, Z)
⊢ (q, ee, ZZ)    [read i, push Z]
⊢ (q, e, Z)      [read e, pop Z]
⊢ (q, ε, ε)      [read e, pop Z, stack empty!]
Accept by empty stack! ✓
```

**Example: w = "eei"**

```
(q, eei, Z)
⊢ (q, ei, ε)     [read e, pop Z, stack empty]
⊢ ???            [read e, but stack is empty - ERROR]
Reject (no valid transition)
```

Wait, this doesn't work right... Let me reconsider. Actually, once the stack is empty, the PDA accepts and stops processing. So "eei" would accept after "ee", which is correct for detecting the error at that point!

### PDA by Final State

![[Chapter-6-Pushdown-Automata-30.webp]]
**P_F = ({p, q, r}, {i, e}, {Z, X₀}, δ_F, p, X₀, {r})**

**Transition diagram:**

```
  ┌───┐  ε,X₀/ZX₀  ┌───┐         ┌───┐
  │ p │─────────→  │ q │────────→│ r │
  └───┘            └───┘  ε,X₀/ε └───┘
   start             ↑ │          (accept)
                i,Z/ZZ│ │e,Z/ε
                     │ ↓
                    (loops)
```

**Transitions:**

- **δ_F(p, ε, X₀) = {(q, ZX₀)}** — Setup: push Z, keep X₀
- **δ_F(q, i, Z) = {(q, ZZ)}** — Same as before
- **δ_F(q, e, Z) = {(q, ε)}** — Same as before
- **δ_F(q, ε, X₀) = {(r, ε)}** — When X₀ exposed, accept!

**Example: w = "iee"**

```
(p, iee, X₀)
⊢ (q, iee, ZX₀)   [setup]
⊢ (q, ee, ZZX₀)   [read i]
⊢ (q, e, ZX₀)     [read e]
⊢ (q, ε, X₀)      [read e, Z₀ exposed]
⊢ (r, ε, ε)       [accept by final state!]
```

**This demonstrates the conversion from empty stack to final state!**

---

## 6.2.4 From Final State to Empty Stack

![[Chapter-6-Pushdown-Automata-31.webp]]

### Theorem 6.11: Reverse Conversion

**Statement:** If L = L(P_F) for some PDA P_F, then there exists a PDA P_N such that L = N(P_N).

**Translation:** Any language accepted by final state can also be accepted by empty stack.

### The Construction

![[Chapter-6-Pushdown-Automata-31.webp]]

**Idea:**

- Add new start state that pushes bottom marker X₀
- When P_F enters accepting state, add transitions to pop everything

**Graphical representation:**

```
     ┌────────────────────────────────────┐
     │       Original PDA P_F             │
p₀ ──┤  (accepts by final state)          ├── p
     │                                    │
     └────────────────────────────────────┘
  
  ε,X₀/Z₀X₀      From every accepting        ε,any/ε
  (setup)        state, can start popping    (clear stack)
```

**Given:** P_F = (Q, Σ, Γ, δ_F, q₀, Z₀, F)

**Construct:** P_N = (Q ∪ {p₀, p}, Σ, Γ ∪ {X₀}, δ_N, p₀, X₀)

**Transition function δ_N:**

1. **δ_N(p₀, ε, X₀) = {(q₀, Z₀X₀)}**
    
    - Setup: push original start symbol, keep X₀
2. **For all transitions in P_F:**
    
    - Copy them to δ_N (simulate P_F)
3. **For all q ∈ F (accepting states) and Y ∈ Γ ∪ {X₀}:**
    
    - **δ_N(q, ε, Y) = {(p, ε)}**
    - From any accepting state, can jump to p and start popping
4. **For all Y ∈ Γ ∪ {X₀}:**
    
    - **δ_N(p, ε, Y) = {(p, ε)}**
    - State p pops everything until stack is empty

### Why This Works

**Key Insight:** When P_F accepts (final state), we manually empty the stack

**Behavior:**

1. Simulate P_F exactly
2. When P_F would accept (in state from F):
    - ε-transition to popping state p
3. State p has ε-transitions to pop any symbol
4. Keep popping until stack is completely empty
5. Empty stack → accept by empty stack!

**Example:** For string w in L(P_F):

```
(p₀, w, X₀)           [start]
⊢ (q₀, w, Z₀X₀)       [setup]
⊢* (q_f, ε, αX₀)      [P_F accepts, q_f ∈ F, α is leftover]
⊢ (p, ε, X₀)          [pop all of α]
⊢ (p, ε, ε)           [pop X₀, empty!]
Accept by empty stack! ✓
```

**Result:** w ∈ N(P_N) ⟺ w ∈ L(P_F)

**Together with Theorem 6.9:**

- Empty stack ⟺ Final state (same class of languages!)
- The choice is a matter of convenience

---

## 6.3 Equivalence of PDAs and CFGs

![[Chapter-6-Pushdown-Automata-32.webp]]

### The Three Equivalences

We need to prove:

1. **Languages defined by CFGs** (Context-Free Languages)
2. **Languages accepted by PDAs (final state)**
3. **Languages accepted by PDAs (empty stack)**

**Status:**

- ✓ Equivalence of (2) and (3) — **Proven** (Theorems 6.9 and 6.11)
- Need to prove: (1) ⟺ (2) or (1) ⟺ (3)

**Strategy:**

- Show: CFG → PDA (Theorem 6.13)
- Show: PDA → CFG (Theorem 6.14)
- This completes the circle!

**Fundamental Result:**

```
Context-Free Grammars ⟺ Pushdown Automata
```

This is one of the most important theorems in formal language theory!

---

## 6.3.1 From Grammars to PDAs

![[Chapter-6-Pushdown-Automata-33.webp]]
![[Chapter-6-Pushdown-Automata-34.webp]]
![[Chapter-6-Pushdown-Automata-35.webp]]

### Construction Algorithm

**Given:** CFG G = (V, T, Q, S)

**Construct:** PDA P that accepts L(G) by empty stack

**P = ({q}, T, V ∪ T, δ, q, S)**

**Key idea:** Use the stack to simulate leftmost derivations!

**Transition function δ:**

**1. For each variable A ∈ V:**

- **For each production A → β:**
    - **δ(q, ε, A) = {(q, β)}**
- Effect: Replace A on stack with right-hand side β
- This simulates expanding A in a derivation!

**2. For each terminal a ∈ T:**

- **δ(q, a, a) = {(q, ε)}**
- Effect: Match terminal on stack with input symbol
- Pop it when they match

### How It Works

**Stack initially contains:** S (start symbol)

**Process:**

1. **Expand variables:** When variable A is on top of stack:
    
    - Nondeterministically choose a production A → β
    - Replace A with β on stack
    - This simulates one derivation step!
2. **Match terminals:** When terminal a is on top of stack:
    
    - Read a from input
    - Pop a from stack (they match!)
3. **Accept:** When stack empties and input is consumed
    
    - We've successfully simulated a derivation S ⇒* w

**Intuition:** The PDA performs a **leftmost derivation** using the stack!

- Stack contains the sentential form
- Always expand the leftmost variable (because it's on top!)
- Match terminals as they're exposed

### Theorem 6.13

**Statement:** If P is constructed from G as above, then **N(P) = L(G)**.

**Proof idea:**

- **(⊆)** If P accepts w by empty stack, then w ∈ L(G)
    - Each sequence of moves corresponds to a leftmost derivation
- **(⊇)** If w ∈ L(G), then P accepts w
    - Each leftmost derivation corresponds to a sequence of moves

**Detailed proof:** See textbook (induction on derivation length)

---

## Example 6.12: Expression Grammar to PDA

![[Chapter-6-Pushdown-Automata-35.webp]]
### Grammar (from Chapter 5)

```
I → a | b | Ia | Ib | I0 | I1
E → I | E*E | E+E | (E)
```

### Constructed PDA

**P = ({q}, {a,b,0,1,(,),+,_}, {I,E,a,b,0,1,(,),+,_}, δ, q, E)**

**Transition function:**

**a) For variable I:**

```
δ(q, ε, I) = {(q, a), (q, b), (q, Ia), (q, Ib), (q, I0), (q, I1)}
```

One choice for each production for I!

**b) For variable E:**

```
δ(q, ε, E) = {(q, I), (q, E+E), (q, E*E), (q, (E))}
```

One choice for each production for E!

**c) For each terminal d ∈ {a,b,0,1,(,),+,*}:**

```
δ(q, d, d) = {(q, ε)}
```

Match and pop!

### Example Derivation: Accept "a+b"

**Leftmost derivation in grammar:**

```
E ⇒ E+E ⇒ I+E ⇒ a+E ⇒ a+I ⇒ a+b
```

**Corresponding PDA moves:**

```
(q, a+b, E)        [start: stack = E]
⊢ (q, a+b, E+E)    [expand E → E+E]
⊢ (q, a+b, I+E)    [expand E → I]
⊢ (q, a+b, a+E)    [expand I → a]
⊢ (q, +b, +E)      [match 'a']
⊢ (q, b, E)        [match '+']
⊢ (q, b, I)        [expand E → I]
⊢ (q, b, b)        [expand I → b]
⊢ (q, ε, ε)        [match 'b', stack empty!]
Accept! ✓
```

**Key observations:**

- Each nondeterministic choice = choosing a production
- Order matches leftmost derivation exactly
- Stack empties ⟺ successful derivation

---

## 6.3.2 From PDAs to Grammars

![[Chapter-6-Pushdown-Automata-36.webp]]![[Chapter-6-Pushdown-Automata-37.webp]]

### Theorem 6.14: The Reverse Construction

**Statement:** Let P = (Q, Σ, Γ, δ, q₀, Z₀) be a PDA. Then there exists a CFG G such that **L(G) = N(P)**.

**Translation:** Every PDA (accepting by empty stack) has an equivalent grammar!

### The Construction

**Key Idea:** Variables represent "stack consumption"

**Nonterminal [pXq]** means:

- "Starting in state p with X on top of stack"
- "Ending in state q with X popped (and everything pushed on top of X also popped)"
- "Consuming some input w in the process"

**Intuition:** [pXq] generates all strings that:

- Take PDA from state p to state q
- Net effect on stack: pop X (and whatever was temporarily pushed above it)

### Grammar Construction

**G = (V, T, P, S)** where:

**Variables V:**

- Start symbol **S**
- All symbols of form **[pXq]** where p,q ∈ Q, X ∈ Γ

**Terminals T:**

- Same as PDA input alphabet Σ

**Productions P:**

**1. Start productions:**

- For all states p ∈ Q:
    - **S → [q₀Z₀p]**
- Meaning: Starting configuration, can end in any state p with stack emptied

**2. Main productions:**

- For each transition δ(q, a, X) containing (r, Y₁Y₂...Yₖ):
    - Where a ∈ Σ ∪ {ε}
    - Where k ≥ 0 (k=0 means pop X)
- For all state sequences r₁, r₂, ..., rₖ:
    - **[qXrₖ] → a[rY₁r₁][r₁Y₂r₂]...[rₖ₋₁Yₖrₖ]**

**Understanding the main production:**

```
[qXrₖ] → a[rY₁r₁][r₁Y₂r₂]...[rₖ₋₁Yₖrₖ]
  ↑       ↑      ↑       ↑           ↑
  |       |      |       |           |
  |       |      |       |           └─ Pop Yₖ: rₖ₋₁ → rₖ
  |       |      |       └─ Pop Y₂: r₁ → r₂  
  |       |      └─ Pop Y₁: r → r₁
  |       └─ Read input symbol a (or ε)
  └─ Pop X: start in q, end in rₖ
```

**Special case k=0 (popping X):**

- δ(q, a, X) contains (r, ε)
- Production: **[qXr] → a**
- Meaning: Read a, pop X, go from q to r

**Special case k=1:**

- δ(q, a, X) contains (r, Y)
- For all states p:
    - **[qXp] → a[rYp]**
- Meaning: Replace X with Y, eventually pop Y

**Special case k=2:**

- δ(q, a, X) contains (r, Y₁Y₂)
- For all states r₁, r₂:
    - **[qXr₂] → a[rY₁r₁][r₁Y₂r₂]**
- Meaning: X becomes Y₂, push Y₁, eventually pop both

---

## Example 6.15: Converting PDA to Grammar

![[Chapter-6-Pushdown-Automata-38.webp]]
![[Chapter-6-Pushdown-Automata-39.webp]]

### The If-Else PDA

**Recall from Example 6.10:**

```
P_N = ({q}, {i, e}, {Z}, δ_N, q, Z)

Transitions:
- δ_N(q, i, Z) = {(q, ZZ)}
- δ_N(q, e, Z) = {(q, ε)}
```

### Step 1: Identify Variables

**Only possible variables:** [qZq]

- Only one state q
- Only one stack symbol Z
- So [qZq] is the only [pXr] symbol possible!

### Step 2: Create Productions

**Production 1 - Start symbol:**

- S → [qZq]
- (For all states p, S → [q₀Z₀p], but only q exists)

**Production 2 - From δ_N(q, i, Z) = {(q, ZZ)}:**

- Transition: state q, read i, pop Z, push ZZ, go to q
- k = 2 (push two symbols)
- For all r₁, r₂ (but only q exists):
    - [qZq] → i[qZq][qZq]
- This is the only production from this transition!

**Production 3 - From δ_N(q, e, Z) = {(q, ε)}:**

- Transition: state q, read e, pop Z, push nothing, go to q
- k = 0 (pop Z entirely)
- [qZq] → e

### Step 3: Simplify

**Replace [qZq] with simple variable A:**

```
S → A
A → iAA
A → e
```

**Further simplification:**

```
S → iSS | e
```

**Final grammar:**

```
G = ({S}, {i, e}, {S → iSS | e}, S)
```

### Verification

**This grammar generates strings like:**

- e (one else)
- iee (if, else, else)
- iieeee (if, if, else, else, else, else)

**Pattern:** More elses than ifs (the error condition!)

**Key insight:** The grammar captures the same language as the PDA!

- Reflects the stack behavior
- Nested structures in grammar = stack push/pop in PDA

---

## 6.4 Deterministic PDAs
![[Chapter-6-Pushdown-Automata-40.webp]]

### 6.4.1 Definition of DPDA

**Intuitive definition:** A PDA is **deterministic** if there is never a choice of moves in any situation.

**Formal definition:** PDA P = (Q, Σ, Γ, δ, q₀, Z₀, F) is a **DPDA** if and only if:

**Condition 1 - At most one move:**

- For all q ∈ Q, a ∈ Σ ∪ {ε}, X ∈ Γ:
    - **|δ(q, a, X)| ≤ 1**
- At most one element in the set
- Means: at most one choice of next move

**Condition 2 - No ε-move conflicts:**

- For all q ∈ Q, a ∈ Σ, X ∈ Γ:
    - If δ(q, a, X) is nonempty, then **δ(q, ε, X) must be empty**
- Can't have both a regular move AND an ε-move with same state/stack
- Must choose: read input OR make ε-move, not both

### Why Both Conditions?

**Condition 1 alone isn't enough:**

```
δ(q, 0, X) = {(p, Y)}   ← only one choice with input 0
δ(q, ε, X) = {(r, Z)}   ← only one choice with ε
```

But we have TWO choices from (q, 0..., X...)!

- Read the 0 and go to p
- Make ε-move and go to r This is nondeterministic!

**Condition 2 prevents this conflict.**

### Examples

**Deterministic:**

```
δ(q, 0, X) = {(p, XX)}   ← exactly one choice
δ(q, 1, X) = {(r, ε)}    ← exactly one choice  
δ(q, ε, Y) = {(s, Y)}    ← only when different stack symbol
```

**Nondeterministic:**

```
δ(q, 0, X) = {(p, XX), (r, X)}   ← two choices! (violates condition 1)

δ(q, 0, X) = {(p, XX)}
δ(q, ε, X) = {(r, ε)}             ← conflict! (violates condition 2)
```

---

## Example 6.16: L_wcwR with DPDA

![[Chapter-6-Pushdown-Automata-41.webp]]
![[Chapter-6-Pushdown-Automata-42.webp]]
### Why L_wwR Doesn't Work

**Recall L_wwR = {wwᴿ | w ∈ (0+1)*}:**

- No center marker
- Must **guess** where the middle is
- Nondeterminism is essential!

**Claim:** There is **no DPDA** for L_wwR.

**Intuition:** Without a clear middle marker, a deterministic PDA can't know when to switch from pushing to popping mode.

### The Modified Language

**L_wcwR = {wcwᴿ | w ∈ (0+1)*}**

**Examples:**

- "0c0" (w = "0")
- "01c10" (w = "01")
- "110c011" (w = "110")
- "c" (w = ε)

**Key difference:** The **c** is a center marker!

- No guessing needed
- When we see c, we know to switch modes
- Deterministic PDA is possible!

### DPDA for L_wcwR

![[Chapter-6-Pushdown-Automata-42.webp]]

**P = ({q₀, q₁, q₂}, {0,1,c}, {0,1,Z₀}, δ, q₀, Z₀, {q₂})**

**Transition diagram:**

```
         c, Z₀/Z₀
         c, 0/0      ┌──────────┐  ε, Z₀/Z₀
         c, 1/1   ┌──│    q₁    │──────┐
    ┌──────────────┘  └──────────┘      │
    │                                    ↓
    │  0, 0/ε                    ┌──────────┐
    │  1, 1/ε                    │    q₂    │ (accept)
    │                            └──────────┘
┌──────────┐
│   q₀     │ (start)
└──────────┘
    ↑ │
    │ │ 0, Z₀/0Z₀
    │ │ 1, Z₀/1Z₀
    │ │ 0, 0/00
    │ │ 0, 1/01
    │ │ 1, 0/10
    │ │ 1, 1/11
    │ └──────────┐
    └────────────┘
```

**Key differences from L_wwR PDA:**

- No ε-transitions from q₀ to q₁ (no guessing!)
- Transitions labeled with **c** instead of ε
- Completely deterministic!

**Verification of determinism:**

**Condition 1 - At most one move:** ✓

- From q₀: exactly one transition for each (state, input, stack) triple
- From q₁: exactly one transition for each triple
- No multiple choices!

**Condition 2 - No ε-move conflicts:** ✓

- Only ε-move is δ(q₁, ε, Z₀) = {(q₂, Z₀)}
- No regular move from (q₁, anything, Z₀)
- No conflicts!

### Execution Example: "01c10"

```
(q₀, 01c10, Z₀)
⊢ (q₀, 1c10, 0Z₀)      [push 0]
⊢ (q₀, c10, 10Z₀)      [push 1]
⊢ (q₁, 10, 10Z₀)       [read c, switch to matching]
⊢ (q₁, 0, 0Z₀)         [match 1, pop]
⊢ (q₁, ε, Z₀)          [match 0, pop]
⊢ (q₂, ε, Z₀)          [accept]
```

**Every move is deterministic** — no choices needed!

---

## 6.4.2 Regular Languages and DPDAs

![[Chapter-6-Pushdown-Automata-43.webp]]

### Theorem 6.17: Regular Languages ⊆ DPDA Languages

**Statement:** If L is a regular language, then L = L(P) for some DPDA P (accepting by final state).

**Translation:** Every regular language can be recognized by a DPDA.

**Proof idea:**

- Start with DFA A that accepts L
- Construct DPDA P that simulates A
- Use stack, but never actually use it!

### Construction

**Given:** DFA A = (Q, Σ, δ_A, q₀, F)

**Construct:** DPDA P = (Q, Σ, {Z₀}, δ_P, q₀, Z₀, F)

**Transition function:**

- For all states p, q ∈ Q and a ∈ Σ such that δ_A(q, a) = p:
    - **δ_P(q, a, Z₀) = {(p, Z₀)}**

**What this means:**

- Same states as DFA
- Same transitions as DFA
- Stack always contains just Z₀ (never changes!)
- Stack is ignored — purely for PDA formalism

**Result:** This DPDA is deterministic and accepts the same language as the DFA.

**Consequence:**

```
Regular Languages ⊆ DPDA Languages
```

---

## 6.4.3 DPDAs and Context-Free Languages

![[Chapter-6-Pushdown-Automata-45.webp]]
![[Chapter-6-Pushdown-Automata-46.webp]]

### The Language Hierarchy

DPDAs occupy a middle ground:

```
Regular Languages ⊂ DPDA Languages ⊂ CFL Languages
        ↑                  ↑                ↑
       DFA              DPDA              PDA
```

**All inclusions are proper (strict)!**

### DPDA Languages Properly Include Regular Languages

**Proof:** L_wcwR is accepted by a DPDA but is not regular.

**Why L_wcwR is not regular:**

- Apply pumping lemma
- For string 0ⁿc0ⁿ, pumping the 0's before c
- Creates imbalance: 0ⁿ⁺ᵏc0ⁿ (not in language)
- Contradiction! Not regular.

**But we showed a DPDA for it!**

**Conclusion:** DPDA Languages ⊃ Regular Languages (strictly larger)

### CFLs Properly Include DPDA Languages

**Claim:** Some CFLs cannot be accepted by any DPDA.

**Example:** L_wwR = {wwᴿ | w ∈ (0+1)*}

**L_wwR is a CFL:**

- We've seen a (nondeterministic) PDA for it
- So it's context-free ✓

**L_wwR cannot be accepted by any DPDA:**

- Informal argument (see textbook p. 255):
- Without a center marker, a DPDA cannot know when to switch modes
- Must commit to either pushing or popping
- Cannot backtrack from wrong decision
- Nondeterminism is essential!

**Formal proof:** More complex, beyond scope (see advanced texts)

**Conclusion:** CFL ⊃ DPDA Languages (strictly larger)

### Summary Diagram

```
┌─────────────────────────────────────────┐
│      Context-Free Languages (CFL)       │
│  ┌───────────────────────────────────┐  │
│  │    DPDA Languages                 │  │
│  │  ┌─────────────────────────────┐  │  │
│  │  │   Regular Languages         │  │  │
│  │  │                             │  │  │
│  │  │  Examples:                  │  │  │
│  │  │  - (0+1)*                   │  │  │
│  │  │  - a*b*                     │  │  │
│  │  └─────────────────────────────┘  │  │
│  │                                   │  │
│  │  Examples (DPDA but not RL):      │  │
│  │  - 0ⁿc0ⁿ (L_wcwR)                 │  │
│  │  - Balanced parentheses           │  │
│  └───────────────────────────────────┘  │
│                                          │
│  Examples (CFL but not DPDA):            │
│  - wwᴿ (L_wwR)                           │
│  - Ambiguous languages                   │
└─────────────────────────────────────────┘
```

---

## 6.4.4 DPDAs and Acceptance by Empty Stack

![[Chapter-6-Pushdown-Automata-44.webp]]
### Theorem 6.19: Prefix Property Restriction

**Statement:** A language L is N(P) for some DPDA P if and only if:

1. L has the **prefix property**, AND
2. L = L(P') for some DPDA P'

**Prefix Property Definition:** Language L has the prefix property if there are no two different strings x and y in L such that x is a prefix of y.

**In other words:**

- If x ∈ L and y ∈ L
- Then neither can be a prefix of the other
- No string in L is a "beginning part" of another string in L

### Examples

**Has prefix property:**

- L = {0ⁿ1ⁿ | n ≥ 1} — none is a prefix of another ✓
    - "01", "0011", "000111", ... (all different lengths, but...)
    - "01" is not a prefix of "0011" (different patterns)

**Does NOT have prefix property:**

- L = {0*, 0ⁿ1ⁿ} — "0" is prefix of "00", "000", etc. ✗
- L = {a, ab, abc} — "a" is prefix of "ab", "ab" is prefix of "abc" ✗

### Why This Restriction?

**Problem with empty stack acceptance and determinism:**

- Suppose x ∈ L is a prefix of y ∈ L
- DPDA reads x, empties stack → must accept
- But input has more symbols (rest of y)
- DPDA cannot "un-accept" or continue
- Determinism broken!

**With final state:**

- Can read x, reach accepting state, but continue reading
- Can then reach another accepting state for y
- No problem!

**Conclusion:** Empty stack acceptance is more restrictive for DPDAs than final state acceptance.

---

## 6.4.5 DPDAs and Ambiguous Grammars

**INSERT PAGE 47 OF PDF HERE**

### Theorem 6.20: Empty Stack → Unambiguous Grammar

**Statement:** If L = N(P) for some DPDA P, then L has an unambiguous CFG.

**Translation:** Every language accepted by a DPDA (empty stack) has an unambiguous grammar.

**Significance:**

- DPDAs naturally avoid ambiguity
- Determinism in PDA → unambiguity in grammar
- Practical importance for parsing!

**Proof:** See textbook (uses the PDA-to-grammar construction)

### Theorem 6.21: Final State → Unambiguous Grammar

**Statement:** If L = L(P) for some DPDA P, then L has an unambiguous CFG.

**Translation:** Every language accepted by a DPDA (final state) has an unambiguous grammar.

**Proof:** See textbook

### Implications

**Positive results:**

- DPDA Languages ⊆ Unambiguous CFL
- Languages with DPDAs have unambiguous grammars
- Useful for compiler design!

**Negative results:**

- Some CFLs are inherently ambiguous
- Those cannot have DPDAs
- Example: {aⁱbʲcᵏ | i=j or j=k}

### Practical Importance

**For programming languages:**

- Want unambiguous grammars (unique parse trees)
- Want deterministic parsing (efficient, no backtracking)
- Design languages to be DPDA-recognizable
- Most programming languages are DPDA languages!

**Parser design:**

- LL parsers ≈ DPDAs (top-down)
- LR parsers ≈ DPDAs (bottom-up)
- Both are deterministic!

---

## Summary and Key Takeaways

### Pushdown Automata Fundamentals

**Definition:**

- PDA = ε-NFA + Stack (LIFO memory)
- 7-tuple: (Q, Σ, Γ, δ, q₀, Z₀, F)
- Can push, pop, and read top of stack

**Power:**

- More powerful than NFAs/DFAs
- Can recognize non-regular languages (0ⁿ1ⁿ, palindromes, etc.)
- Still limited (cannot do 0ⁿ1ⁿ2ⁿ)

### Two Acceptance Methods

**Final State:**

- L(P) = {w | (q₀, w, Z₀) ⊢* (q, ε, α), q ∈ F}
- Like NFA acceptance
- Stack content irrelevant

**Empty Stack:**

- N(P) = {w | (q₀, w, Z₀) ⊢* (q, ε, ε)}
- Unique to PDAs
- Final state irrelevant

**Equivalence:** Both methods recognize the same class of languages!

- Can convert between them (Theorems 6.9 and 6.11)

### Equivalence with CFGs

**Fundamental Theorem:**

```
Context-Free Languages ⟺ PDA Languages
```

**Conversions:**

- CFG → PDA: Stack simulates leftmost derivations
- PDA → CFG: Variables [pXq] represent stack consumption

**Implication:** Three equivalent definitions of CFL!

1. Generated by CFG
2. Accepted by PDA (final state)
3. Accepted by PDA (empty stack)

### Deterministic PDAs

**Definition:**

- At most one move from any configuration
- No ε-move conflicts with regular moves

**Language Hierarchy:**

```
Regular ⊂ DPDA ⊂ CFL
```

All inclusions are strict!

**Properties:**

- Every DPDA language has unambiguous grammar
- Some CFLs have no DPDA (e.g., L_wwR)
- Some CFLs have no unambiguous grammar (inherently ambiguous)

### Important Languages

|Language|Regular?|DPDA?|CFL?|Notes|
|---|---|---|---|---|
|(0+1)*|✓|✓|✓|Basic regular|
|0ⁿ1ⁿ|✗|✓|✓|Needs counting|
|0ⁿc0ⁿ (L_wcwR)|✗|✓|✓|Has center marker|
|wwᴿ (L_wwR)|✗|✗|✓|Needs nondeterminism|
|0ⁿ1ⁿ2ⁿ|✗|✗|✗|Beyond CFL|

### Practical Applications

**Compiler Design:**

- Programming language syntax is typically DPDA-recognizable
- Parsers implement DPDAs
- LL and LR parsers are deterministic

**Why DPDAs matter:**

- Efficient (no backtracking)
- Unambiguous (unique parse trees)
- Predictable behavior

---

## Study Tips and Practice

### To Master PDAs:

1. **Understand the stack:**
    
    - Practice tracing stack contents
    - Remember: LIFO (Last In, First Out)
    - Know all operations: push, pop, replace
2. **Practice designing PDAs:**
    
    - Start with simple languages (0ⁿ1ⁿ)
    - Understand when to push vs. pop
    - Identify where nondeterminism is needed
3. **Master IDs and moves:**
    
    - Write out move sequences
    - Track state, input, and stack together
    - Use ⊢* notation correctly
4. **Understand both acceptance methods:**
    
    - Know when to use each
    - Practice converting between them
    - Understand their equivalence
5. **Know the conversions:**
    
    - CFG → PDA (simulate derivations)
    - PDA → CFG (variables = stack consumption)
    - Practice with examples
6. **Recognize determinism:**
    
    - Check both conditions
    - Identify ε-move conflicts
    - Know when DPDA is possible

### Common Mistakes to Avoid:

❌ Forgetting the stack is LIFO (not random access) ❌ Confusing final state vs. empty stack acceptance ❌ Not recognizing nondeterministic choices ❌ Trying to build DPDA for L_wwR (impossible!) ❌ Forgetting bottom marker in conversions ❌ Mixing up stack alphabet Γ and input alphabet Σ

### Key Questions to Ask:

✓ Can a DFA/NFA recognize this? (Is it regular?) ✓ Do I need nondeterminism? (L_wwR vs. L_wcwR) ✓ Where should I push? Where should I pop? ✓ What acceptance method makes sense? ✓ Is this language deterministic? (DPDA possible?) ✓ Does this have an unambiguous grammar?

### Practice Problems:

1. Design PDA for {0ⁱ1ʲ | i < j}
2. Design PDA for balanced parentheses
3. Convert given CFG to PDA
4. Convert given PDA to CFG
5. Prove L_wcwR is not regular
6. Show why L_wwR needs nondeterminism
7. Identify whether given PDA is deterministic

---

## Important Theorems Summary

**Theorem 6.5:** Can add to input/stack in IDs **Theorem 6.6:** Can remove unchanged input from IDs **Theorem 6.9:** Empty stack → Final state conversion **Theorem 6.11:** Final state → Empty stack conversion **Theorem 6.13:** CFG → PDA (N(P) = L(G)) **Theorem 6.14:** PDA → CFG (L(G) = N(P)) **Theorem 6.17:** Regular languages ⊆ DPDA languages **Theorem 6.19:** DPDA empty stack requires prefix property **Theorem 6.20:** DPDA empty stack → unambiguous grammar **Theorem 6.21:** DPDA final state → unambiguous grammar

---

**End of Pushdown Automata Notes**