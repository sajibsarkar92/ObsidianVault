Here are the comprehensive notes for the **Longest Common Subsequence (LCS) Problem** and its high-yield exam variants.

---

# 🧬 Topic: Longest Common Subsequence (LCS) & Variants

## 1. The Base Problem: Standard LCS

### **Problem Statement**

Given two strings `s1` (length $M$) and `s2` (length $N$), find the length of the longest subsequence that appears in both strings.

- **Subsequence:** A sequence that appears in the same relative order but not necessarily contiguously.
    
- _Example:_ `s1 = "ABCDE"`, `s2 = "ACE"`. LCS is `"ACE"`, Length = 3.
    

### **Intuition & Logic**

We compare the two strings character by character, usually from the end (or beginning). Let `i` be the index in `s1` and `j` be the index in `s2`.

1. **Match:** If `s1[i] == s2[j]`, this character is definitely part of the LCS. We add 1 to our count and move both pointers back (`i-1`, `j-1`).
    
2. **Mismatch:** If `s1[i] != s2[j]`, we have a conflict. We can't match these two characters together. We have two choices:
    
    - Skip the character in `s1` and try to match `s1[0...i-1]` with `s2[0...j]`.
        
    - Skip the character in `s2` and try to match `s1[0...i]` with `s2[0...j-1]`.
        
    - We take the **maximum** of these two options.
        

### **Recurrence Relation**

Let `dp[i][j]` be the LCS length for `s1[0...i-1]` and `s2[0...j-1]`.

$$dp[i][j] = \begin{cases} 0 & \text{if } i=0 \text{ or } j=0 \\ 1 + dp[i-1][j-1] & \text{if } s1[i-1] == s2[j-1] \\ \max(dp[i-1][j], dp[i][j-1]) & \text{if } s1[i-1] \neq s2[j-1] \end{cases}$$

### **C++ Implementation**

C++

```
#include <iostream>
#include <vector>
#include <string>
#include <algorithm>

using namespace std;

int longestCommonSubsequence(string text1, string text2) {
    int m = text1.length();
    int n = text2.length();
    
    // dp[i][j] stores LCS of text1[0..i-1] and text2[0..j-1]
    // Size is (m+1) x (n+1) to handle empty string base cases at index 0
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            // Check if characters match (Note: 0-based index in string is i-1)
            if (text1[i - 1] == text2[j - 1]) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                // Mismatch: Max of skipping char from text1 OR skipping char from text2
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[m][n];
}
```

### **How It Works**

1. **Table Setup:** We use `m+1` and `n+1` so `dp[0][...]` represents an empty string prefix. The LCS of anything with an empty string is 0.
    
2. **Traversal:** We fill row by row.
    
3. **Result:** `dp[m][n]` contains the answer for the full strings.
    

---

## 2. Variant: Printing the LCS (Reconstruction)

### **The Variant**

Instead of just returning the _length_ (e.g., 3), return the actual string (e.g., "ACE").

### **Logic Shift**

We don't need a new DP calculation. We simply **backtrack** from the bottom-right corner `dp[m][n]` to `dp[0][0]`.

- If `s1[i-1] == s2[j-1]`, this character was part of the LCS. Add it to result, move diagonally up-left (`i-1`, `j-1`).
    
- If mismatch, move in the direction of the larger value (`max(up, left)`). If they are equal, either path works.
    

### **C++ Implementation**

C++

```cpp
string printLCS(string s1, string s2) {
    int m = s1.length();
    int n = s2.length();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    // 1. Build the Table (Standard LCS)
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1[i - 1] == s2[j - 1]) dp[i][j] = 1 + dp[i - 1][j - 1];
            else dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
        }
    }

    // 2. Backtrack to find the String
    string lcs = "";
    int i = m, j = n;
    while (i > 0 && j > 0) {
        // If characters match, it's part of LCS
        if (s1[i - 1] == s2[j - 1]) {
            lcs += s1[i - 1];
            i--; j--;
        }
        // If mismatch, go to the larger neighbor
        else if (dp[i - 1][j] > dp[i][j - 1]) {
            i--; // Move Up
        } else {
            j--; // Move Left
        }
    }
    
    reverse(lcs.begin(), lcs.end()); // We built it backwards
    return lcs;
}
```

---

## 3. Variant: Longest Common Substring (Contiguous)

### **The Variant**

Find the longest string that is a **substring** (contiguous) of both `s1` and `s2`.

- _Example:_ `s1 = "ABCDE"`, `s2 = "ABFDE"`.
    
    - LCS is "ABDE" (Length 4).
        
    - LC Substring is "AB" or "DE" (Length 2).
        

### **Logic Shift**

The "Match" logic is the same ($1 + \text{diagonal}$), but the "Mismatch" logic changes drastically.

- **Match:** `dp[i][j] = 1 + dp[i-1][j-1]`
    
- **Mismatch:** If characters don't match, the contiguous chain is **broken**. We must reset `dp[i][j] = 0`.
    
- **Result:** The answer is not `dp[m][n]`, but the **maximum value found anywhere** in the table.
    

### **C++ Implementation**

C++

```
int longestCommonSubstring(string s1, string s2) {
    int m = s1.length();
    int n = s2.length();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));
    int maxLen = 0;

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (s1[i - 1] == s2[j - 1]) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
                maxLen = max(maxLen, dp[i][j]); // Update global max
            } else {
                dp[i][j] = 0; // Reset count on mismatch
            }
        }
    }
    return maxLen;
}
```

---

## 4. Variant: LCS of 3 Strings

### **The Variant**

Find the length of the longest subsequence common to three strings: `s1`, `s2`, and `s3`.

### **Logic Shift**

We simply add a third dimension.

- State: `dp[i][j][k]`
    
- Logic:
    
    - If `s1[i] == s2[j] == s3[k]`: `1 + dp[i-1][j-1][k-1]`
        
    - Else: `max(dp[i-1][j][k], dp[i][j-1][k], dp[i][j][k-1])`
        

### **C++ Implementation**

C++

```
int lcs3(string s1, string s2, string s3) {
    int n1 = s1.length(), n2 = s2.length(), n3 = s3.length();
    // 3D Array initialized to 0
    int dp[n1+1][n2+1][n3+1]; 
    memset(dp, 0, sizeof(dp));

    for (int i = 1; i <= n1; i++) {
        for (int j = 1; j <= n2; j++) {
            for (int k = 1; k <= n3; k++) {
                if (s1[i-1] == s2[j-1] && s2[j-1] == s3[k-1]) {
                    dp[i][j][k] = 1 + dp[i-1][j-1][k-1];
                } else {
                    dp[i][j][k] = max({dp[i-1][j][k], 
                                       dp[i][j-1][k], 
                                       dp[i][j][k-1]});
                }
            }
        }
    }
    return dp[n1][n2][n3];
}
```

---

## 5. Variant: Shortest Common Supersequence

### **The Variant**

Find the length of the shortest string that contains both `s1` and `s2` as subsequences.

- _Example:_ `AGGTAB`, `GXTXAYB` -> `AGGXTXAYB`.
    

### **Logic Shift**

This is a pure math derivation from LCS.

To make the supersequence, we take all characters from `s1` and all from `s2`. However, the characters that are in the LCS are counted twice (once in `s1`, once in `s2`), but we only need to write them once in the supersequence.

- **Formula:** `Length = Length(s1) + Length(s2) - LCS(s1, s2)`
    

### **C++ Snippet**

C++

```
int shortestCommonSupersequence(string s1, string s2) {
    int lenLCS = longestCommonSubsequence(s1, s2);
    return s1.length() + s2.length() - lenLCS;
}
```

---

## 6. Variant: Longest Palindromic Subsequence

### **The Variant**

Find the longest subsequence of string `s` that is a palindrome.

- _Example:_ `s = "BBABCBCAB"`. Answer is 7 (`"BABCBAB"`).
    

### **Logic Shift**

We don't need a new algorithm! A palindrome reads the same forwards and backwards.

Therefore, the Longest Palindromic Subsequence of `s` is simply the **LCS of `s` and `Reverse(s)`**.

### **C++ Snippet**

C++

```
int longestPalindromicSubsequence(string s) {
    string t = s;
    reverse(t.begin(), t.end());
    // Reuse the standard LCS function
    return longestCommonSubsequence(s, t);
}
```

---

## 7. Variant: Space Optimized LCS

### **The Variant**

The standard LCS uses $O(M \times N)$ space. If $M=10000, N=10000$, this is 100MB of RAM. Can we do it with less?

### **Logic Shift**

Notice that to calculate row `i`, we ONLY need data from row `i-1`. We don't need row `i-2` or anything before that.

- We can maintain just **two rows**: `prev` and `curr`.
    
- After computing `curr`, `prev` becomes `curr`, and we reuse the memory.
    
- Space Complexity: $O(\min(M, N))$.
    

### **C++ Implementation**

C++

```
int lcsOptimized(string text1, string text2) {
    int m = text1.length();
    int n = text2.length();
    
    vector<int> prev(n + 1, 0);
    vector<int> curr(n + 1, 0);

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (text1[i - 1] == text2[j - 1]) {
                curr[j] = 1 + prev[j - 1];
            } else {
                curr[j] = max(prev[j], curr[j - 1]);
            }
        }
        // Move current row to previous for next iteration
        prev = curr;
    }
    return prev[n];
}
```
---

## 8. Variant: Minimum Insertions/Deletions to Convert String A to B

### **The Variant**

You are given two strings, `s1` and `s2`. You want to transform `s1` into `s2` using only two operations:

1. **Delete** a character from `s1`.
    
2. **Insert** a character into `s1`.
    
    Find the **minimum number of operations** required.
    

- _Example:_ `s1 = "heap"`, `s2 = "pea"`.
    
    - Delete 'h' $\rightarrow$ "eap"
        
    - Delete 'p' $\rightarrow$ "ea" (Wait, this is wrong logic).
        
    - Correct Path: Delete 'h' ("eap"), Delete 'p' ("ea"), Insert 'p' ("pea")? No.
        
    - Optimal: LCS is "ea".
        
    - "heap" $\rightarrow$ delete 'h' $\rightarrow$ "eap" $\rightarrow$ delete 'p' $\rightarrow$ "ea" $\rightarrow$ insert 'p' $\rightarrow$ "pea". Total 3 ops.
        

### **Logic Shift (The Math)**

You don't need a new DP algorithm. This is purely derived from LCS.

- The characters in the LCS are the ones we **keep**.
    
- Everything in `s1` that is _not_ in the LCS must be **deleted**.
    
    - Deletions = `Length(s1) - LCS`
        
- Everything in `s2` that is _not_ in the LCS must be **inserted**.
    
    - Insertions = `Length(s2) - LCS`
        
- **Total Operations** = `(m - LCS) + (n - LCS)` = `m + n - 2*LCS`.
    

### **C++ Implementation**

C++

```
int minOperations(string s1, string s2) {
    int m = s1.length();
    int n = s2.length();
    int lcs = longestCommonSubsequence(s1, s2); // Use standard LCS function
    
    int deletions = m - lcs;
    int insertions = n - lcs;
    
    return deletions + insertions;
}
```

---

## 9. Variant: Edit Distance (Levenshtein Distance)

### **The Variant**

This is the "Super-LCS". You want to convert `s1` to `s2` with minimum operations, but now you have **three** allowed operations:

1. **Insert** a character.
    
2. **Delete** a character.
    
3. **Replace** a character (Change 'a' to 'b').
    

### **Logic Shift**

In LCS, if `s1[i] != s2[j]`, we just skipped. Here, we must fix the mismatch.

- **Match:** If `s1[i] == s2[j]`, cost is 0. Move to `dp[i-1][j-1]`.
    
- **Mismatch:** We take the minimum of 3 options + 1 cost:
    
    1. `dp[i][j-1]` (Insert into s1 / Skip in s2)
        
    2. `dp[i-1][j]` (Delete from s1 / Skip in s1)
        
    3. `dp[i-1][j-1]` (Replace `s1[i]` with `s2[j]`)
        

### **C++ Implementation**

C++

```
int minDistance(string word1, string word2) {
    int m = word1.length();
    int n = word2.length();
    vector<vector<int>> dp(m + 1, vector<int>(n + 1));

    for (int i = 0; i <= m; i++) {
        for (int j = 0; j <= n; j++) {
            // Base Case: If one string is empty, we must insert/delete all chars of the other
            if (i == 0) dp[i][j] = j; // Insert j chars
            else if (j == 0) dp[i][j] = i; // Delete i chars
            
            // Standard Logic
            else if (word1[i - 1] == word2[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1]; // No cost
            } else {
                dp[i][j] = 1 + min({
                    dp[i - 1][j],    // Delete
                    dp[i][j - 1],    // Insert
                    dp[i - 1][j - 1] // Replace
                });
            }
        }
    }
    return dp[m][n];
}
```

---

## 10. Variant: Longest Repeating Subsequence

### **The Variant**

Given a string `s`, find the length of the longest subsequence that appears **twice** in it.

- _Constraint:_ The two subsequences cannot use the same character at the same index. (The $i$-th character of the string cannot be used as the $i$-th character of both subsequences).
    
- _Example:_ `s = "AABEBCDD"`.
    
    - Repeating Subsequence: "ABD" (Indices 0, 2, 6 and 1, 4, 7).
        

### **Logic Shift**

This is a modification of **LCS(s, s)**.

Normally, `LCS(s, s)` is just the length of `s` (diagonal matches).

To find repeating parts, we run LCS on `s` and `s`, but we add a condition: **`i != j`**.

- We can only match `s[i]` with `s[j]` if `i` and `j` are different indices.
    

### **C++ Implementation**

C++

```
int longestRepeatingSubsequence(string s) {
    int n = s.length();
    vector<vector<int>> dp(n + 1, vector<int>(n + 1, 0));

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= n; j++) {
            // Key Change: Match only if indices differ
            if (s[i - 1] == s[j - 1] && i != j) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[n][n];
}
```

---

## 11. Variant: Longest Common Increasing Subsequence (LCIS)

### **The Variant**

This combines **LCS** and **LIS** (Longest Increasing Subsequence).

Given two arrays/strings, find a subsequence that is:

1. **Common** to both.
    
2. Strictly **Increasing**.
    

### **Logic Shift**

- **Naive Approach:** Find LCS ($O(N^2)$), then find LIS of that? No, because LCS is not unique.
    
- **Correct $O(N^2)$ Approach:**
    
    We iterate through `s1` (index `i`). For each `s1[i]`, we scan through `s2` (index `j`).
    
    We maintain a variable `current_max` which tracks the length of the LCIS ending before `j` using values smaller than `s1[i]`.
    

### **C++ Implementation**

C++

```
int LCIS(vector<int>& arr1, vector<int>& arr2) {
    int n = arr1.size();
    int m = arr2.size();
    
    // dp[j] stores the length of LCIS ending with arr2[j]
    vector<int> dp(m, 0);

    for (int i = 0; i < n; i++) {
        int current_max = 0; // Tracks LCIS length of elements smaller than arr1[i]
        
        for (int j = 0; j < m; j++) {
            // Case 1: Standard LIS update logic inside LCS loop
            // If arr1[i] > arr2[j], then arr2[j] can be a predecessor to arr1[i]
            if (arr1[i] > arr2[j]) {
                if (dp[j] > current_max) {
                    current_max = dp[j];
                }
            }
            
            // Case 2: Match Found
            // If they match, we can append arr1[i] to the best sequence found so far
            else if (arr1[i] == arr2[j]) {
                dp[j] = current_max + 1;
            }
        }
    }

    // The answer is the max value in the dp array
    int ans = 0;
    for (int x : dp) ans = max(ans, x);
    return ans;
}
```

---

## 12. Variant: LCS on Permutations (O(N log N) Optimization)

### **The Variant**

Standard LCS is $O(N^2)$. If $N = 10^5$, this is too slow.

However, if the two arrays are **Permutations** of numbers $1 \dots N$ (each number appears exactly once), we can solve it in $O(N \log N)$.

### **Logic Shift**

Since elements are unique, we can map the elements of `s1` to their indices: `pos[s1[i]] = i`.

Now, replace every element in `s2` with its position in `s1`.

The problem reduces to finding the **Longest Increasing Subsequence (LIS)** of this transformed `s2`.

- _Why?_ An increasing sequence of indices in `s2` corresponds to a sequence appearing in the same relative order in `s1`.
    

### **C++ Implementation**

C++

```
// Requires a standard O(N log N) LIS function
int lengthOfLIS(vector<int>& nums); 

int lcsPermutation(vector<int>& P, vector<int>& Q) {
    int n = P.size();
    // Map value -> index in P
    vector<int> posInP(n + 1);
    for (int i = 0; i < n; i++) {
        posInP[P[i]] = i;
    }

    // Transform Q based on positions in P
    vector<int> mappedQ;
    for (int x : Q) {
        // If x exists in P (always true for permutation)
        mappedQ.push_back(posInP[x]);
    }

    // The LCS of P and Q is exactly the LIS of mappedQ
    return lengthOfLIS(mappedQ);
}
```

---

## 13. Variant: LCS with K Mismatches

### **The Variant**

Find the longest common subsequence, but you are allowed to change at most **K** characters in the strings to make them match.

### **Logic Shift**

We add a dimension to the state: `dp[i][j][k]`.

- `k` represents the number of changes ("mismatches used") allowed.
    
- If `s1[i] == s2[j]`: `1 + dp[i-1][j-1][k]`
    
- If `s1[i] != s2[j]`:
    
    1. **Skip s1:** `dp[i-1][j][k]`
        
    2. **Skip s2:** `dp[i][j-1][k]`
        
    3. **Use a mismatch (Change char):** `1 + dp[i-1][j-1][k-1]` (Only if `k > 0`)
        

### **C++ Implementation**

C++

```
int lcsKMismatches(string s1, string s2, int K) {
    int n = s1.length();
    int m = s2.length();
    
    // dp[i][j][k]
    // Use vector for ease, though raw array is faster
    // Dimensions: [n+1][m+1][K+1]
    vector<vector<vector<int>>> dp(n + 1, vector<vector<int>>(m + 1, vector<int>(K + 1, 0)));

    for (int i = 1; i <= n; i++) {
        for (int j = 1; j <= m; j++) {
            for (int k = 0; k <= K; k++) {
                if (s1[i - 1] == s2[j - 1]) {
                    // Match: extend without using 'k' budget
                    dp[i][j][k] = 1 + dp[i - 1][j - 1][k];
                } else {
                    // Option 1 & 2: Skip characters (Standard LCS logic)
                    int res = max(dp[i - 1][j][k], dp[i][j - 1][k]);
                    
                    // Option 3: Force match by using 1 change budget
                    if (k > 0) {
                        res = max(res, 1 + dp[i - 1][j - 1][k - 1]);
                    }
                    dp[i][j][k] = res;
                }
            }
        }
    }
    return dp[n][m][K];
}
```


Here is the additional **Word-Based LCS** variant, often framed as "Plagiarism Detection" or "File Diffing" in exams.

---

## 14. Variant: LCS of Words (Sentence Comparison / Diff Tool)

### **The Variant**

Instead of comparing two _strings_ character by character (e.g., "apple" vs "apply"), you are comparing two _sentences_ or _files_ word by word (or line by line).

- **Input:**
    
    - Sentence 1: "The quick brown fox jumps"
        
    - Sentence 2: "The brown fox runs fast"
        
- **Goal:** Find the Longest Common Subsequence of **words**.
    
    - Result: `["The", "brown", "fox"]` (Length 3).
        
- **Real-world Context:** This is exactly how the `diff` command in Linux or Git works (comparing lines of code instead of characters).
    

### **Logic Shift**

The logic is mathematically identical to standard LCS. The difference is simply the **atomic unit** of comparison.

1. **Preprocessing:** You must first tokenize (split) the input strings into arrays (vectors) of words.
    
2. **Comparison:** Instead of checking `s1[i] == s2[j]`, you check `word_array1[i] == word_array2[j]`.
    

### **C++ Implementation**

C++

```
#include <iostream>
#include <vector>
#include <string>
#include <sstream>
#include <algorithm>

using namespace std;

// Helper function to split a sentence into a vector of words
vector<string> splitToWords(string s) {
    vector<string> words;
    stringstream ss(s);
    string word;
    while (ss >> word) { // Automatically handles spaces
        words.push_back(word);
    }
    return words;
}

int lcsWords(string text1, string text2) {
    // 1. Preprocessing: Tokenize strings into words
    vector<string> w1 = splitToWords(text1);
    vector<string> w2 = splitToWords(text2);
    
    int m = w1.size();
    int n = w2.size();
    
    // 2. Standard LCS DP table
    vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            // Key Change: Compare string objects (words) instead of chars
            if (w1[i - 1] == w2[j - 1]) {
                dp[i][j] = 1 + dp[i - 1][j - 1];
            } else {
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    
    return dp[m][n];
}
```

---

## Summary of LCS Variants for Exam

|**Variant**|**Key Logic Change**|**Complexity**|
|---|---|---|
|**Standard LCS**|Base problem.|$O(N^2)$|
|**Print LCS**|Backtrack from `dp[n][m]` to `dp[0][0]`.|$O(N^2)$|
|**LC Substring**|Reset to 0 on mismatch. Answer is global max.|$O(N^2)$|
|**LCS of 3**|3D Array `dp[i][j][k]`.|$O(N^3)$|
|**Shortest Supersequence**|`Len(A) + Len(B) - LCS`.|$O(N^2)$|
|**Min Ops (Insert/Delete)**|Derived from LCS length.|$O(N^2)$|
|**Longest Palindromic Subseq**|`LCS(s, reverse(s))`.|$O(N^2)$|
|**Edit Distance**|Min of (Insert, Delete, Replace).|$O(N^2)$|
|**LCS on Permutations**|Map indices + LIS.|$O(N \log N)$|
|**Word/Line LCS**|Split string into tokens, then run LCS.|$O(N^2)$|

**Next Step:** Are you ready to move on to **LIS (Longest Increasing Subsequence)** and its variants (like Box Stacking and Building Bridges)?