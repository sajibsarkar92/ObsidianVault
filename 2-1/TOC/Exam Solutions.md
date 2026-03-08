# 📚 Pushdown Automata (PDA) — Exam Solutions

> **Source:** Questions from 2023 & 2024 exam papers (identical content, minor mark variations) **Tags:** #theory-of-computation #PDA #exam-prep #automata

---

## 📋 Table of Contents

1. [[#Q1 — Language of a PDA (Definition)]]
2. [[#Q2 — Design a PDA for ${0^n1^n \mid n \ge 1}$]]
3. [[#Q3 — Convert CFG to PDA]]
4. [[#Q4 — Proof: Acceptance by Empty Stack ⟹ Acceptance by Final State]]

---

## Q1 — Language of a PDA (Definition)

> **2024:** Q5b — 2 marks | **2023:** Q5b — 3 marks

### ✍️ Model Answer

The **language of a PDA** is defined as the set of all input strings that the PDA _accepts_. There are two standard modes of acceptance:

**1. Acceptance by Final State — $L(P)$**

A string $w \in \Sigma^*$ is accepted by PDA $P$ if, starting from the initial configuration $(q_0, w, Z_0)$, the PDA can reach a configuration of the form $(q_f, \varepsilon, \gamma)$ where:

- $q_f \in F$ is a final (accepting) state,
- the entire input $w$ has been consumed (empty remaining input $\varepsilon$),
- $\gamma$ is any stack content (we don't care what's on the stack).

Formally: $$L(P) = { w \mid (q_0, w, Z_0) \vdash^* (q_f, \varepsilon, \gamma), ; q_f \in F, ; \gamma \in \Gamma^* }$$

**2. Acceptance by Empty Stack — $N(P)$**

A string $w \in \Sigma^*$ is accepted by PDA $P$ if, starting from $(q_0, w, Z_0)$, the PDA can reach a configuration where the input is exhausted **and** the stack is completely empty:

$$N(P) = { w \mid (q_0, w, Z_0) \vdash^* (q, \varepsilon, \varepsilon), ; q \in Q }$$

> 💡 **Note:** Both modes are equivalent in power — any language accepted by one can be accepted by the other. This equivalence is proven formally in Q4.

---

## Q2 — Design a PDA for ${0^n1^n \mid n \ge 1}$

> **2024:** Q5a — 3 marks | **2023:** Q5a — 3 marks

### ✍️ Model Answer

**Language:** All strings with $n$ zeros followed by $n$ ones, where $n \ge 1$. **Strategy:** Push a symbol onto the stack for each `0` read. Pop a symbol for each `1` read. Accept when stack returns to the initial stack symbol (or goes empty).

---

#### Formal Definition

$$P = (Q, \Sigma, \Gamma, \delta, q_0, Z_0, F)$$

Where:

- $Q = {q_0, q_1, q_2}$
- $\Sigma = {0, 1}$
- $\Gamma = {A, Z_0}$
- Start state: $q_0$
- Initial stack symbol: $Z_0$
- Final states: $F = {q_2}$

---

#### Transition Function $\delta$

|State|Input|Stack Top|New State|Stack Action|Meaning|
|---|---|---|---|---|---|
|$q_0$|$0$|$Z_0$|$q_0$|Push $AZ_0$|First 0: push A|
|$q_0$|$0$|$A$|$q_0$|Push $AA$|Subsequent 0s: push A|
|$q_0$|$1$|$A$|$q_1$|Pop $A$|Switch to popping phase|
|$q_1$|$1$|$A$|$q_1$|Pop $A$|Continue popping|
|$q_1$|$\varepsilon$|$Z_0$|$q_2$|Keep $Z_0$|Stack back to bottom → accept|

---

#### Step-by-Step Trace for $w = 0011$

|Step|State|Remaining Input|Stack (top→bottom)|Action|
|---|---|---|---|---|
|1|$q_0$|$0011$|$Z_0$|Read `0`, push $A$|
|2|$q_0$|$011$|$AZ_0$|Read `0`, push $A$|
|3|$q_0$|$11$|$AAZ_0$|Read `1`, pop $A$, go to $q_1$|
|4|$q_1$|$1$|$AZ_0$|Read `1`, pop $A$|
|5|$q_1$|$\varepsilon$|$Z_0$|$\varepsilon$-move to $q_2$ ✅|

> 💡 **Key Insight:** The stack acts as a counter. Every `0` increments the counter (push), every `1` decrements it (pop). The string is valid iff the counter hits exactly zero when the input ends.

---

#### State Diagram (Text Representation)

```
         0, Z₀ / AZ₀
         0, A  / AA
    ┌────────────────┐
    ↓                │         1, A / ε              ε, Z₀ / Z₀
  →(q₀)─────────────┘  ──────────────────→  (q₁)  ──────────────→ ((q₂))
                            1, A / ε         loop
```

---

## Q3 — Convert CFG to PDA

> **2024:** Q6a — 2 marks | **2023:** Q6a — 2 marks

### Given Grammar

$$I \to a \mid b \mid Ia \mid Ib \mid I0 \mid I1$$ $$E \to I \mid E*E \mid E+E \mid (E)$$

> 💡 **Note on the Grammar:** $I$ generates identifiers (strings of letters/digits starting with a letter). $E$ generates expressions using operators `+`, `*` and parentheses.

---

### ✍️ Model Answer

#### The Standard Algorithm: CFG → PDA

To convert any CFG $G = (V, T, P, S)$ to an equivalent PDA, we use the following standard construction:

**Construct PDA** $M = ({q}, T, V \cup T, \delta, q, S)$ with a **single state** $q$, where transitions are:

1. **For each production rule $A \to \alpha$** in the grammar: $$\delta(q, \varepsilon, A) = {(q, \alpha)}$$ _(Pop $A$ off the stack, push the RHS $\alpha$ onto the stack — leftmost symbol on top)_
    
2. **For each terminal symbol $a \in T$**: $$\delta(q, a, a) = {(q, \varepsilon)}$$ _(If top of stack matches current input terminal, pop both)_
    

---

#### Applying the Algorithm

**Step 1 — Identify all components:**

- Non-terminals: $V = {I, E}$
- Terminals: $T = {a, b, 0, 1, *, +, (, )}$
- Start symbol: $S = E$

**Step 2 — Write $\varepsilon$-transitions for each production rule:**

From grammar rules for $I$:

$$\delta(q, \varepsilon, I) \ni (q, a)$$ $$\delta(q, \varepsilon, I) \ni (q, b)$$ $$\delta(q, \varepsilon, I) \ni (q, Ia) \quad \text{(push } a \text{ then } I \text{ — } I \text{ is on top)}$$ $$\delta(q, \varepsilon, I) \ni (q, Ib)$$ $$\delta(q, \varepsilon, I) \ni (q, I0)$$ $$\delta(q, \varepsilon, I) \ni (q, I1)$$

From grammar rules for $E$:

$$\delta(q, \varepsilon, E) \ni (q, I)$$ $$\delta(q, \varepsilon, E) \ni (q, E*E) \quad \text{(push } E \text{ then } * \text{ then } E \text{ — leftmost } E \text{ on top)}$$ $$\delta(q, \varepsilon, E) \ni (q, E+E)$$ $$\delta(q, \varepsilon, E) \ni (q, (E))$$

**Step 3 — Write terminal-matching transitions (one per terminal):**

$$\delta(q, a, a) = (q, \varepsilon)$$ $$\delta(q, b, b) = (q, \varepsilon)$$ $$\delta(q, 0, 0) = (q, \varepsilon)$$ $$\delta(q, 1, 1) = (q, \varepsilon)$$ $$\delta(q, *, *) = (q, \varepsilon)$$ $$\delta(q, +, +) = (q, \varepsilon)$$ $$\delta(q, (, () = (q, \varepsilon)$$ $$\delta(q, ), )) = (q, \varepsilon)$$

**Step 4 — Acceptance:**

The PDA accepts by **empty stack**. When the stack becomes empty, the input has been derived from start symbol $E$ and is accepted.

---

#### Summary Table of PDA

|Transition|Input|Stack Top|Push (new stack top)|
|---|---|---|---|
|$\delta(q, \varepsilon, E)$|$\varepsilon$|$E$|$I$ or $E*E$ or $E+E$ or $(E)$|
|$\delta(q, \varepsilon, I)$|$\varepsilon$|$I$|$a$ or $b$ or $Ia$ or $Ib$ or $I0$ or $I1$|
|$\delta(q, a, a)$|$a$|$a$|$\varepsilon$ (pop)|
|$\delta(q, b, b)$|$b$|$b$|$\varepsilon$ (pop)|
|$\delta(q, 0, 0)$|$0$|$0$|$\varepsilon$ (pop)|
|$\delta(q, 1, 1)$|$1$|$1$|$\varepsilon$ (pop)|
|… and similarly for `*`, `+`, `(`, `)`||||

> 💡 **Key Insight:** This is a _top-down parsing_ PDA. It simulates a leftmost derivation: the stack holds the "future" of the derivation. When a non-terminal is on top, it gets expanded; when a terminal is on top, it gets matched against input.

---

## Q4 — Proof: $N(P_N) \Rightarrow L(P_F)$

> **2024:** Q5c — 5 marks | **2023:** Q5c — 4 marks

### Statement to Prove

> **Theorem:** If $L = N(P_N)$ for some PDA $P_N = (Q, \Sigma, \Gamma, \delta_N, q_0, Z_0)$, then there exists a PDA $P_F$ such that $L = L(P_F)$.

In plain English: _If a language is accepted by empty stack, then there is another PDA that accepts the same language by final state._

---

### ✍️ Model Answer

#### Intuition

$P_N$ accepts a string $w$ when it empties its stack. We need to build $P_F$ that accepts $w$ when it enters a final state. The idea is:

- Add a **new start state** $p_0$ that sets up the stack with a special "bottom marker" $X_0$.
- Let $P_F$ simulate $P_N$ exactly.
- Add a **new final state** $p_f$ that is reached via $\varepsilon$-transitions whenever the stack has only $X_0$ left (meaning $P_N$ would have emptied its stack).

> 💡 The marker $X_0$ prevents $P_F$ from accidentally emptying its stack during intermediate steps of a computation.

---

#### Construction of $P_F$

Define: $$P_F = (Q', \Sigma, \Gamma', \delta_F, p_0, X_0, {p_f})$$

Where:

- $Q' = Q \cup {p_0, p_f}$ (two new states added)
- $\Gamma' = \Gamma \cup {X_0}$ (new bottom-of-stack marker added)
- $p_0$ is the new start state
- $p_f$ is the new (and only) accepting state
- $X_0 \notin \Gamma$ is the new initial stack symbol

---

#### Defining $\delta_F$

The transition function $\delta_F$ is defined in three parts:

**Part 1 — Initialisation (from new start state):**

$$\delta_F(p_0, \varepsilon, X_0) = {(q_0, Z_0 X_0)}$$

> _On $\varepsilon$-input, push the original start symbol $Z_0$ on top of $X_0$. This transfers control to $P_N$'s start state $q_0$ with the original stack setup, but with our bottom marker $X_0$ underneath._

**Part 2 — Simulate $P_N$ exactly:**

For all $q \in Q$, $a \in \Sigma \cup {\varepsilon}$, $Y \in \Gamma$:

$$\delta_F(q, a, Y) \supseteq \delta_N(q, a, Y)$$

> _All original transitions of $P_N$ are preserved in $P_F$ unchanged._

**Part 3 — Detect empty stack and move to final state:**

For all $q \in Q$:

$$\delta_F(q, \varepsilon, X_0) \ni (p_f, \varepsilon)$$

> _Whenever any state $q$ of $P_N$ sees the bottom marker $X_0$ on top of the stack (meaning $P_N$'s stack is now effectively empty), take an $\varepsilon$-transition to the new accepting state $p_f$._

---

#### Correctness Proof

We now prove that $L(P_F) = N(P_N) = L$.

**($\Rightarrow$) If $w \in N(P_N)$, then $w \in L(P_F)$:**

Suppose $w \in N(P_N)$. Then: $$(q_0, w, Z_0) \vdash^*_{P_N} (q, \varepsilon, \varepsilon) \quad \text{for some } q \in Q$$

In $P_F$, the computation proceeds as:

1. From $(p_0, w, X_0)$, apply Part 1: go to $(q_0, w, Z_0 X_0)$
    
2. Simulate $P_N$ step by step using Part 2: $(q_0, w, Z_0 X_0) \vdash^*_{P_F} (q, \varepsilon, X_0)$
    
    _(The $X_0$ remains at the bottom throughout since $P_N$'s transitions only operate on $\Gamma$, not $X_0$)_
    
3. Now top of stack is $X_0$. Apply Part 3: $(q, \varepsilon, X_0) \vdash_{P_F} (p_f, \varepsilon, \varepsilon)$
    

Since $p_f \in F$, the string $w$ is accepted by $P_F$. Hence $w \in L(P_F)$.

---

**($\Leftarrow$) If $w \in L(P_F)$, then $w \in N(P_N)$:**

Suppose $w \in L(P_F)$. Then $P_F$ reaches $p_f$ on input $w$.

- $p_f$ is only reachable via Part 3, which fires only when the stack top is $X_0$.
- $X_0$ appears on top only when $P_N$'s portion of the stack is completely empty.
- By Part 1, $P_F$ starts by simulating $P_N$ from $(q_0, w, Z_0)$.
- Therefore, at the point $X_0$ is on top, $P_N$ must be in a configuration $(q, \varepsilon, \varepsilon)$.

This means $(q_0, w, Z_0) \vdash^*_{P_N} (q, \varepsilon, \varepsilon)$, so $w \in N(P_N)$.

---

#### Conclusion

Since $L(P_F) \subseteq N(P_N)$ and $N(P_N) \subseteq L(P_F)$, we have:

$$L(P_F) = N(P_N) = L$$

Therefore, for every PDA $P_N$ that accepts $L$ by empty stack, there exists a PDA $P_F$ (constructed as above) that accepts the same language $L$ by final state. $\blacksquare$

---

> 💡 **Exam Tip:** Always write both directions of the proof ($\Rightarrow$ and $\Leftarrow$) even for a 4–5 mark question. State the construction clearly and then argue correctness. Markers look for: the formal definition of $P_F$, the three-part $\delta_F$, and at least an outline of why the construction works.

---

## 🧠 Quick Revision Summary

|Topic|Key Point|
|---|---|
|Language of PDA|Two modes: final state $L(P)$ and empty stack $N(P)$ — both equivalent|
|PDA for $0^n1^n$|Push on `0`, pop on `1`, accept when stack returns to $Z_0$|
|CFG → PDA|Single-state PDA: $\varepsilon$-transitions for productions, terminal-match transitions|
|Empty stack → Final state|Add $p_0$, $p_f$, and $X_0$; detect empty stack via $X_0$ on top|

---

_Last updated: exam prep 2024_