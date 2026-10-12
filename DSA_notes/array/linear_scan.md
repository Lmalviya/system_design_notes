# Linear Scan / Single Pass (Array)

> **Quick Revision Core**: Linear Scan processes the array in $O(N)$ time (single or constant passes) by maintaining running variables, states, or invariants. The key challenge is reducing auxiliary space from $O(N)$ to $O(1)$ by condensing past history into a few tracking variables.

---

## 🧠 How to Identify & Think (Pattern Recognition Guide)

### 1. 🔍 Recognition Triggers (When to suspect Linear Scan)
- **Constraint Clue**: $N \ge 10^5$ with tight time limits $\implies$ requires $O(N)$ time, ruling out $O(N^2)$ brute force.
- **Order Matters**: The problem requires preserving element order (e.g., contiguous subarrays, timeline of events, transactions).
- **Localized Decision**: To compute the answer at index `i`, you do **not** need the entire history—you only need a **condensed summary** of what occurred in `nums[0...i-1]` (e.g., running sum, min seen so far, current candidate).
- **Prompt Hints**: *"Solve in $O(N)$ time and $O(1)$ extra space"*, *"Do it in a single pass"*, *"Find contiguous subarray..."*.

---

### 2. 🧩 The 4 Core Archetypes of Linear Scan

| Archetype | How to Spot It | Core Strategy | Classic Examples |
| :--- | :--- | :--- | :--- |
| **1. Voting / Elimination** | Frequency $> n/k$ threshold; dominant items. | Cancel groups of $k$ distinct elements. Majority survives. | LC 169 ($> n/2$), LC 229 ($> n/3$) |
| **2. Resetting Accumulator (Kadane's)** | Contiguous subarray with max/min sum/product. | Accumulate running metric; **discard/reset** when past becomes a liability. | LC 53 (Max Subarray), LC 152 (Max Product) |
| **3. Running Extremes / Invariants** | Best pair $(i, j)$ where $i < j$ (e.g., buy before sell). | Maintain `min_so_far` or `max_so_far`; compute profit at every $j$. | LC 121 (Stock I), LC 1014 (Sightseeing Pair) |
| **4. Two-Pass (Prefix + Suffix)** | Result for `i` depends on all elements *except* `i`. | Pass 1 captures left-side state; Pass 2 captures right-side state. | LC 238 (Product Except Self), LC 42 (Rain Water) |

---

### 3. 🎯 4-Step Mental Framework (How to tackle an unseen problem)

1. **Step 1: Identify the minimal history needed**
   - Ask: *"What is the absolute minimum information I need from the left to evaluate the current element?"*
   - If it's just a number (e.g., smallest price so far, running sum), you don't need an array/hash map—use simple variables ($O(1)$ space).
2. **Step 2: Define the Reset / Discard Condition**
   - Ask: *"When does the past history hurt more than it helps?"*
   - Negative running sum $\rightarrow$ reset to `0` (Kadane's).
   - Candidate vote hits `0` $\rightarrow$ pick new candidate (Boyer-Moore).
3. **Step 3: Decide if you need a 2nd pass**
   - Is an answer guaranteed? If **no**, a verification pass is required (e.g., LC 229, LC 169 without guarantees).
   - Does answer depend on both left and right sides? $\rightarrow$ Prefix pass then Suffix pass (or 2-pointers).
4. **Step 4: Check boundary & edge cases**
   - All negative numbers? (Does your initialized variable break? e.g., `maxSum = 0` vs `nums[0]`).
   - Single-element arrays, empty arrays, or no valid answer existing.

---

## 📋 Problem Template (Copy for new problems)

```markdown
## [Problem Name] (LC #xxx)
- **Problem**: <1-line summary>
- **Approaches**:
  | Approach | Time | Space | Key Idea |
  | :--- | :--- | :--- | :--- |
- **Optimal Intuition**: <1-2 bullet points>
- **Code**: <concise snippet>
- **Gotchas / Follow-ups**: <edge cases or variations>
```

---

## 1. Majority Element (LC 169)

- **Problem**: Find the element that appears more than $\lfloor n / 2 \rfloor$ times in an array of size $n$ (guaranteed to exist).

### Approaches Comparison

| Approach | Time | Space | Key Takeaway |
| :--- | :--- | :--- | :--- |
| **Sorting** | $O(N \log N)$ | $O(1)$ | Majority element always lands at index `n / 2`. |
| **Hash Map** | $O(N)$ | $O(N)$ | Count frequencies; return element with count $> n / 2$. |
| **Boyer-Moore Voting** *(Optimal)* | $O(N)$ | $O(1)$ | Pair-wise cancellation: majority element always survives. |

### 💡 Optimal Intuition (Boyer-Moore Voting)
- **Concept**: Think of it as a battle royale. Every non-majority element cancels out one occurrence of the majority candidate. Because majority count $> n/2$, it will remain the last candidate standing.
- **Logic**:
  1. If `count == 0`, pick current element as `candidate`.
  2. If current element equals `candidate`, increment `count`.
  3. Otherwise, decrement `count`.

### 💻 Code

```cpp
int majorityElement(vector<int>& nums) {
    int candidate = 0, count = 0;
    for (int num : nums) {
        if (count == 0) {
            candidate = num;
        }
        count += (num == candidate) ? 1 : -1;
    }
    return candidate;
}
```

### ⚠️ Gotchas & Follow-ups
- **Not Guaranteed?**: If the problem doesn't guarantee a majority exists, a **2nd pass** is mandatory to count candidate occurrences and verify `count > n / 2`.
- **Follow-up (LC 229 - Majority Element II)**: Find all elements appearing $> \lfloor n / 3 \rfloor$ times.
  - At most **2** such elements can exist.
  - Maintain **2 candidates** and **2 counters**, then run a verification pass.

---

## 2. Maximum Subarray (LC 53)

- **Problem**: Given an integer array `nums`, find the contiguous subarray with the largest sum and return its sum.

### Approaches Comparison

| Approach | Time | Space | Key Takeaway |
| :--- | :--- | :--- | :--- |
| **Brute Force** | $O(N^2)$ | $O(1)$ | Try all subarrays $(i, j)$; causes Time Limit Exceeded (TLE). |
| **DP (Tabulation)** | $O(N)$ | $O(N)$ | `dp[i] = max(nums[i], dp[i-1] + nums[i])` represents max sum ending at `i`. |
| **Kadane's Algorithm** *(Optimal)* | $O(N)$ | $O(1)$ | Space-optimized DP: discard negative prefix sum and start fresh. |

### 💡 Optimal Intuition (Kadane's Algorithm)
- **Concept**: A prefix with a negative sum is dead weight—it can only reduce the sum of any future subarray. Hence, whenever running sum drops below `0`, reset it.
- **Logic**:
  1. Maintain `current_sum` (running sum) and `max_sum` (global best).
  2. For each number `x`:
     - Add `x` to `current_sum`.
     - Update `max_sum = max(max_sum, current_sum)`.
     - If `current_sum < 0`, reset `current_sum = 0`.

### 💻 Code

```cpp
int maxSubArray(vector<int>& nums) {
    int maxSum = nums[0];
    int currentSum = 0;

    for (int x : nums) {
        currentSum += x;
        maxSum = max(maxSum, currentSum);
        if (currentSum < 0) {
            currentSum = 0; // discard negative running sum
        }
    }
    return maxSum;
}
```

### ⚠️ Gotchas & Follow-ups
- **All Negative Array**: Initializing `maxSum = 0` fails when all numbers are negative (e.g. `[-3, -2, -1]` -> answer is `-1`, not `0`). Always initialize `maxSum = nums[0]` and update `maxSum` *before* resetting `currentSum = 0`.
- **Follow-up: Return the Subarray**: Track `start`, `end`, and a `temp_start` pointer that updates to `i + 1` whenever `currentSum` resets to `0`.
- **Follow-up (LC 918 - Maximum Sum Circular Subarray)**: Answer is `max(max_subarray, total_sum - min_subarray)` (except when all elements are negative).

---

## 3. Majority Element II (LC 229)

- **Problem**: Given an integer array `nums` of size $n$, find all elements that appear more than $\lfloor n / 3 \rfloor$ times.

### Approaches Comparison

| Approach | Time | Space | Key Takeaway |
| :--- | :--- | :--- | :--- |
| **Hash Map** | $O(N)$ | $O(N)$ | Count frequencies; return elements with count $> \lfloor n / 3 \rfloor$. |
| **Extended Boyer-Moore** *(Optimal)* | $O(N)$ | $O(1)$ | Track at most 2 candidates (triplet cancellation), followed by a mandatory verification pass. |

### 💡 Optimal Intuition (Extended Boyer-Moore)
- **Why at most 2 elements? (The 100% Pie Analogy)**:
  - To qualify, an element must take up **strictly more than $33.33\%$** ($> n / 3$) of the array.
  - If you had 3 such elements, together they would require $> 100\%$ of the array ($34\% + 34\% + 34\% = 102\%$), which is impossible.
  - Therefore, there can be **at most 2** qualifying elements (possible counts: 0, 1, or 2).
- **Why Triplet Cancellation works?**:
  - In LC 169 ($> 50\%$), we canceled **pairs** of 2 distinct elements.
  - Here ($> 33.33\%$), whenever we see 3 distinct elements, we cancel that **triplet $(a, b, c)$**.
  - **The Guarantee**: In an array of size $n$, you can form at most $\lfloor n / 3 \rfloor$ triplets of distinct elements. Since our true majority element appears **$> n / 3$** times, it has strictly more occurrences than triplets that can ever be formed—so it is **guaranteed to survive**!
- **Algorithm (2 Passes)**:
  1. **Pass 1 (Find Candidates via Triplet Elimination)**:
     - Maintain 2 candidate slots (`cand1`, `cand2`) and 2 counts (`count1`, `count2`).
     - **Order of checks matters**:
       - `num == cand1` $\rightarrow$ `count1++` (reinforces candidate 1)
       - `num == cand2` $\rightarrow$ `count2++` (reinforces candidate 2)
       - `count1 == 0` $\rightarrow$ `cand1 = num, count1 = 1` (claims empty slot 1)
       - `count2 == 0` $\rightarrow$ `cand2 = num, count2 = 1` (claims empty slot 2)
       - Else $\rightarrow$ `count1--, count2--` (encountered a 3rd distinct number: cancel 1 of each to eliminate a triplet)
  2. **Pass 2 (Verification Pass)**:
     - Count exact occurrences of `cand1` and `cand2`.
     - Add to result only if actual count $> \lfloor n / 3 \rfloor$.

### 💻 Code

```cpp
vector<int> majorityElement(vector<int>& nums) {
    int cand1 = 0, cand2 = 0;
    int count1 = 0, count2 = 0;

    // Pass 1: Find potential candidates
    for (int num : nums) {
        if (num == cand1) {
            count1++;
        } else if (num == cand2) {
            count2++;
        } else if (count1 == 0) {
            cand1 = num;
            count1 = 1;
        } else if (count2 == 0) {
            cand2 = num;
            count2 = 1;
        } else {
            count1--;
            count2--;
        }
    }

    // Pass 2: Verify candidates
    count1 = 0;
    count2 = 0;
    for (int num : nums) {
        if (num == cand1) count1++;
        else if (num == cand2) count2++;
    }

    vector<int> result;
    int threshold = nums.size() / 3;
    if (count1 > threshold) result.push_back(cand1);
    if (count2 > threshold) result.push_back(cand2);
    return result;
}
```

### ⚠️ Gotchas & Follow-ups
- **Critical If-Else Order**: Must check `num == cand1` and `num == cand2` **before** checking `count == 0`, otherwise both candidates will latch onto the same duplicate value.
- **Verification Pass is Mandatory**: Elements may not exist at all (e.g. `[1, 2, 3]` leaves candidates from Pass 1, but none have count $> n/3$).
- **Generalization ($> \lfloor n / k \rfloor$)**: At most $k - 1$ majority elements can exist. Maintain $k - 1$ candidates and counters ($O(N \cdot k)$ time, $O(k)$ space).

---