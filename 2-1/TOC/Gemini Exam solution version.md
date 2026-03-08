# 📚 Pushdown Automata (PDA) — Exam Solutions

**Source:** 2023 & 2024 Exam Papers\

**Tags:** `#theory-of-computation` `#PDA` `#exam-prep` `#automata`

## 📋 Table of Contents

1. [[#Q1 — Language of a PDA (Definition)]]
    
2. [[# Q2 — Design a PDA for $\{0^n1^n \mid n \ge 1\}$  ]]
    
3. [[#Q3 — Convert CFG to PDA]]
    
4. [[#Q4 — Proof: Acceptance by Empty Stack ⟹ Acceptance by Final State]]
    

## Q1 — Language of a PDA (Definition)

_2024: Q5b (2 marks) | 2023: Q5b (3 marks)_

**✍️ Examiner's Model Answer**

The language of a PDA is the set of all input strings it accepts. There are two formally equivalent ways a PDA can accept an input string:

### 1. Acceptance by Final State, $L(P)$  

A string is accepted if the PDA finishes reading the entire input and ends up in a designated "final" (or accepting) state. We do not care what is left on the stack.

- **Formally:** $L(P) = \{ w \mid (q_0, w, Z_0) \vdash^* (q_f, \varepsilon, \gamma) \}$  
    
- _(Where_ $q_f$ _is a final state,_ $\varepsilon$ _means the input is fully read, and_ $\gamma$ _is any remaining stack content)._
    

### 2. Acceptance by Empty Stack, $N(P)$  

A string is accepted if the PDA finishes reading the entire input and simultaneously pops the very last symbol off its stack, leaving the stack completely empty. We do not care what state the PDA ends in.

- **Formally:** $N(P) = \{ w \mid (q_0, w, Z_0) \vdash^* (q, \varepsilon, \varepsilon) \}$  
    
- _(Where_ $q$ _is any state, and the final_ $\varepsilon$ _means the stack is empty)._
    

> 💡 **Evaluator's Note:** Full marks are awarded for clearly distinguishing that _Final State_ ignores the stack's end condition, while _Empty Stack_ ignores the machine's final state. Both methods are mathematically equivalent in computing power.

## Q2 — Design a PDA for $\{0^n1^n \mid n \ge 1\}$  

_2024: Q5a (3 marks) | 2023: Q5a (3 marks)_

**✍️ Examiner's Model Answer**

**Strategy:** Use the stack as a counter.

1. **Push** a marker `$A$` onto the stack for every `0` read.
    
2. **Pop** a marker `$A$` for every `1` read.
    
3. **Accept** if the stack returns to its initial bottom symbol exactly when the input runs out.
    

### 1. Formal Definition

$P = (Q, \Sigma, \Gamma, \delta, q_0, Z_0, F)$  

- **States (**$Q$**):** $\{q_0, q_1, q_2\}$  
    
- **Alphabets:** Input $\Sigma = \{0, 1\}$, Stack $\Gamma = \{A, Z_0\}$  
    
- **Start & Final:** Start = $q_0$, Final $F = \{q_2\}$, Initial Stack = $Z_0$  
    

### 2. Transition Function ($\delta$)

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|**State**|**Input**|**Stack Top**|**Action**|**New State**|**Meaning**|
|$q_0$|`0`|$Z_0$|Push $A$ ($AZ_0$)|$q_0$|First `0`: start counting|
|$q_0$|`0`|$A$|Push $A$ ($AA$)|$q_0$|Next `0`s: keep counting|
|$q_0$|`1`|$A$|Pop ($\varepsilon$)|$q_1$|First `1`: switch to popping|
|$q_1$|`1`|$A$|Pop ($\varepsilon$)|$q_1$|Next `1`s: keep popping|
|$q_1$|$\varepsilon$|$Z_0$|Keep $Z_0$|$q_2$|Input empty, stack at bottom → **Accept**|

### 3. State Diagram

```
         0, Z₀ → AZ₀
         0, A  → AA
       ┌─────────────┐
       ↓             │         1, A → ε              ε, Z₀ → Z₀
     (q₀) ───────────┘  ──────────────────→  (q₁)  ──────────────→ ((q₂))
                                          ┌───↑
                                          │   │
                                          └───┘
                                         1, A → ε
```

## Q3 — Convert CFG to PDA

_2024: Q6a (2 marks) | 2023: Q6a (2 marks)_

**Given Grammar:**

- $I \to a \mid b \mid Ia \mid Ib \mid I0 \mid I1$ _(Identifiers)_
    
- $E \to I \mid E*E \mid E+E \mid (E)$ _(Expressions)_
    

**✍️ Examiner's Model Answer**

**The Standard Algorithm (Top-Down Parsing):**

To convert a Context-Free Grammar into a single-state PDA (accepting by empty stack), we follow two simple rules:

1. **Expand Non-Terminals:** For every grammar rule $A \to \alpha$, create an $\varepsilon$-transition that pops $A$ and pushes the result $\alpha$.
    
2. **Match Terminals:** For every terminal symbol, create a transition that reads the terminal from the input and pops it from the stack.
    

### The Equivalent PDA Transitions

The PDA has only one state, $q$. The initial stack symbol is the grammar's start symbol, $E$.

**Step 1: Expand Rules (**$\varepsilon$**-transitions)**

Whenever a non-terminal is at the top of the stack, guess a production rule and replace it:

- $\delta(q, \varepsilon, E) = \{ (q, I), (q, E*E), (q, E+E), (q, (E)) \}$  
    
- $\delta(q, \varepsilon, I) = \{ (q, a), (q, b), (q, Ia), (q, Ib), (q, I0), (q, I1) \}$  
    

**Step 2: Match Rules (Terminal transitions)**

Whenever a terminal is at the top of the stack, it must match the current input symbol. If it matches, pop it:

- $\delta(q, a, a) = \{(q, \varepsilon)\}$  
    
- $\delta(q, b, b) = \{(q, \varepsilon)\}$  
    
- $\delta(q, 0, 0) = \{(q, \varepsilon)\}$  
    
- $\delta(q, 1, 1) = \{(q, \varepsilon)\}$  
    
- $\delta(q, *, *) = \{(q, \varepsilon)\}$  
    
- $\delta(q, +, +) = \{(q, \varepsilon)\}$  
    
- $\delta(q, (, () = \{(q, \varepsilon)\}$  
    
- $\delta(q, ), )) = \{(q, \varepsilon)\}$  
    

> 💡 **Evaluator's Note:** Grouping the transitions logically like this shows a clear understanding of the algorithm. You don't need to write out "push $E$ then $*$ then $E$" in English if your mathematical sets are accurate.

## Q4 — Proof: Acceptance by Empty Stack ⟹ Acceptance by Final State

_2024: Q5c (5 marks) | 2023: Q5c (4 marks)_

**Theorem:** If a language is accepted by a PDA $P_N$ (using empty stack), we can construct a new PDA $P_F$ that accepts the exact same language (using final state).

**✍️ Examiner's Model Answer**

### 1. Intuition (The Strategy)

To make a "Final State" PDA ($P_F$) simulate an "Empty Stack" PDA ($P_N$), we must catch the exact moment $P_N$ empties its stack.

We do this by placing a **fake bottom marker** ($X_0$) on the stack before starting the simulation. If $P_F$ ever sees $X_0$ at the top of the stack, it knows $P_N$ has emptied its stack, allowing $P_F$ to safely transition to a final state.

### 2. The Construction

Given $P_N = (Q, \Sigma, \Gamma, \delta_N, q_0, Z_0)$, we define $P_F = (Q \cup \{p_0, p_f\}, \Sigma, \Gamma \cup \{X_0\}, \delta_F, p_0, X_0, \{p_f\})$.

We define $\delta_F$ in three distinct phases:

- **Phase 1: Setup**
    
    $\delta_F(p_0, \varepsilon, X_0) = \{(q_0, Z_0 X_0)\}$
    
    _(Push the original starting symbol_ $Z_0$ _on top of our trap door_ $X_0$_, and jump to_ $P_N$_'s start state)._
    
- **Phase 2: Simulate**
    
    For all states and symbols, copy $P_N$'s rules exactly:
    
    $\delta_F(q, a, Y) = \delta_N(q, a, Y)$  
    
- **Phase 3: Terminate & Accept**
    
    For every state $q$ in $P_N$:
    
    $\delta_F(q, \varepsilon, X_0) = \{(p_f, \varepsilon)\}$
    
    _(If_ $X_0$ _is exposed,_ $P_N$_'s stack is empty. Jump to the accepting state_ $p_f$_)._
    

### 3. Proof of Correctness

We must prove both directions to show $L(P_F) = N(P_N)$.

**Direction 1: If** $P_N$ **accepts,** $P_F$ **accepts (**$N(P_N) \subseteq L(P_F)$**)**

1. Suppose $P_N$ accepts a string $w$. This means it completely processes $w$ and empties its stack: $(q_0, w, Z_0) \vdash^* (q, \varepsilon, \varepsilon)$.
    
2. If we run $w$ through $P_F$, Phase 1 sets the stack to $Z_0 X_0$.
    
3. Phase 2 perfectly simulates $P_N$, meaning $P_F$ will process $w$ and eventually pop everything down to our bottom marker: $(q_0, w, Z_0X_0) \vdash^* (q, \varepsilon, X_0)$.
    
4. Seeing $X_0$, Phase 3 triggers, moving the machine to $p_f$. Since $p_f$ is a final state, $P_F$ accepts $w$.
    

**Direction 2: If** $P_F$ **accepts,** $P_N$ **accepts (**$L(P_F) \subseteq N(P_N)$**)**

1. Suppose $P_F$ accepts $w$. This means it reached the final state $p_f$.
    
2. The _only_ way to reach $p_f$ is via the Phase 3 rule, which requires $X_0$ to be at the top of the stack.
    
3. Because Phase 2 exactly mirrors $P_N$, $X_0$ being exposed means $P_N$'s portion of the stack (everything above $X_0$) was completely popped.
    
4. Therefore, $P_N$ must have successfully processed $w$ and emptied its stack, meaning $P_N$ accepts $w$.
    

**Conclusion:** Since both directions hold true, the two machines accept the exact same language. $\blacksquare$  

_End of Script. Evaluator marks awarded: 100%._