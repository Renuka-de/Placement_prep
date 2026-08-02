# Sliding Window + Two Pointers — Revision Guide

This note helps you revise the common patterns instead of memorizing individual problems.

---

## 1. First Questions to Ask

Before writing code, ask these 4 questions:

1. What makes a window valid?
2. What state do I maintain to check validity?
3. When should the left pointer move?
4. How does each valid window contribute to the answer?

---

## 2. Core Idea

A sliding window keeps a range of elements and moves pointers to avoid rechecking from scratch.

Typical structure:

```python
left = 0
for right in range(n):
    include(nums[right])

    while window is invalid:
        remove(nums[left])
        left += 1

    update answer
```

---

## 3. Pattern Cheat Sheet

### Pattern A — Longest / Maximum Valid Window

Goal: find the longest window satisfying a condition.

Use when:
- longest substring/subarray
- maximum length

Template:

```python
left = 0
for right in range(n):
    add(nums[right])
    while invalid:
        remove(nums[left])
        left += 1
    ans = max(ans, right - left + 1)
```

Examples:
- Longest Substring Without Repeating Characters
- Longest Repeating Character Replacement
- Max Consecutive Ones III
- Fruit Into Baskets

---

### Pattern B — Count All Valid Subarrays

Goal: count every valid subarray.

Use when:
- “count subarrays with at most K …”
- “count subarrays satisfying property”

Template:

```python
left = 0
for right in range(n):
    add(nums[right])
    while invalid:
        remove(nums[left])
        left += 1
    ans += right - left + 1
```

Why it works:
- Once the window is shrunk to a valid state, every subarray ending at right and starting from left to right is valid.

---

### Pattern C — Exactly K = AtMost(K) - AtMost(K-1)

This is one of the most important interview tricks.

Use when:
- “exactly K distinct”
- “exactly K odd numbers”
- “exactly K zeros”

Formula:

```python
exactly_k = at_most(k) - at_most(k - 1)
```

Key idea:
- If the property is monotonic, this works very well.

---

### Pattern D — Frequency Window

Maintain counts in a frequency map.

Typical operations:

```python
freq[ch] += 1
# ...
freq[ch] -= 1
```

Use when the condition depends on:
- character frequency
- distinct count
- replacement problems

Examples:
- Longest Repeating Character Replacement
- Fruits Into Baskets
- Minimum Window Substring

---

### Pattern E — Last Seen Index

Instead of shrinking the window, remember the last position of each value.

Use when:
- duplicate characters matter
- you need to know when a character was last seen

Template:

```python
last_seen[ch] = i
left = max(left, last_seen[ch] + 1)
```

Examples:
- Longest Substring Without Repeating Characters
- Substrings containing all three characters

---

### Pattern F — Fixed Window

Window size stays constant.

Use when:
- fixed size K
- need to slide one step at a time

Template:

```python
window_size = k
# expand and remove oldest element each step
```

Examples:
- Maximum Average Subarray
- Sliding Window Maximum
- Find All Anagrams

---

### Pattern G — Minimum Window

Goal: find the smallest valid window.

Use when:
- minimum length is required

Template:

```python
left = 0
for right in range(n):
    add(nums[right])
    while window is valid:
        update answer with current window
        remove(nums[left])
        left += 1
```

Examples:
- Minimum Window Substring
- Minimum Size Subarray Sum

---

### Pattern H — Complement Counting

Sometimes it is easier to count invalid windows and subtract from total.

Use when:
- “at least K” is easier to transform

Idea:

```python
answer = total - invalid
```

Good example:
- subarrays with at least K

Warning:
- not always useful, especially when the complement becomes messy

---

### Pattern I — Stale Maximum Trick

In some frequency-based problems, you can maintain a stale maximum frequency.

Use when:
- only need an upper bound on the most frequent character

Key idea:
- max frequency only increases or stays same
- never need to recompute from scratch

Example:
- Longest Repeating Character Replacement

---

## 4. Two Pointers Common Patterns

Two pointers are often used when the problem has a sorted or monotonic structure.

### Pattern 1 — Pair Sum / Two Sum Style

Use when:
- array is sorted
- looking for pair with target sum

Template:

```python
left, right = 0, n - 1
while left < right:
    if nums[left] + nums[right] == target:
        # found
    elif nums[left] + nums[right] < target:
        left += 1
    else:
        right -= 1
```

---

### Pattern 2 — Fast and Slow Pointer

Use when:
- linked list cycle
- removing duplicates
- array partitioning

Template:

```python
slow = 0
for fast in range(n):
    if condition:
        nums[slow] = nums[fast]
        slow += 1
```

---

### Pattern 3 — Reverse / Mirror Pointer

Use when:
- checking palindrome
- comparing from both ends

Template:

```python
left, right = 0, n - 1
while left < right:
    if nums[left] != nums[right]:
        break
    left += 1
    right -= 1
```

---

## 5. Problem → Pattern Map

- Longest Substring Without Repeating Characters → Last Seen / Sliding Window
- Longest Repeating Character Replacement → Frequency + Longest Window
- Max Consecutive Ones III → Longest Valid Window
- Binary Subarrays With Sum → AtMost − AtMost
- Count Nice Subarrays → AtMost − AtMost
- Fruits Into Baskets → Frequency + At Most Distinct
- Minimum Window Substring → Minimum Valid Window
- Sliding Window Maximum → Fixed Window + Monotonic Queue

---

## 6. Quick Revision Checklist

When you see a new problem, check:

- What makes the window valid?
- What values must be tracked?
- When should left move?
- Is the goal longest, shortest, or count?
- Can I solve it using at-most or exactly-K trick?

---

## 7. Golden Rule

Before coding, always answer:

1. What defines a valid window?
2. What information do I need to maintain that validity?
3. When should I move the left pointer?
4. How does each valid window contribute to the answer?

If you can answer these four, most sliding window and two-pointer problems become much easier.
