# Monotonic Stack Revision Guide

This is a revision-friendly cheat sheet. When you revisit it, you should be able to answer four things quickly:

1. When should I use this pattern?
2. What is the core idea?
3. What template should I write?
4. Which problems use it?

---

## 0. Quick decision tree
Ask these questions in order:

- Am I finding a nearest element?
  - Think: Previous/Next Smaller or Greater
- Am I finding a boundary around an element?
  - Think: Previous smaller + Next smaller
- Is every element contributing independently?
  - Think: Contribution counting
- Does only the latest survivor matter?
  - Think: Collision simulation
- Am I removing previous worse elements greedily?
  - Think: Greedy monotonic stack

If any of these fit, start thinking about a monotonic stack.

---

## 1. Nearest Smaller / Greater

### Use when
- Next Greater / Smaller
- Previous Greater / Smaller
- Daily Temperatures
- Stock Span

### Core idea
Keep a stack of candidates so that the top is the most relevant one for the current element.

### Template
```java
for (int i = 0; i < n; i++) {
    while (!stack.isEmpty() && violatesMonotonicity) {
        stack.pop();
    }

    answer[i] = stack.isEmpty() ? boundary : stack.peek();
    stack.push(i);
}
```

### Memory trick
- For next greater, pop while current > top
- For next smaller, pop while current < top

### Common pitfalls
- Wrong direction of traversal
- Wrong comparison sign
- Not handling empty stack

### Problems
- Next Greater Element I
- Next Greater Element II
- Daily Temperatures
- Online Stock Span

---

## 2. Boundary Problems

### Use when
You need the range of influence of an element, usually determined by the nearest smaller element on both sides.

### Core idea
Find:
- previous smaller
- next smaller

Then the element is bounded by them.

### Template
```text
left = i - prevSmaller
right = nextSmaller - i
```

### Why it works
The element cannot extend beyond the nearest smaller blocker on either side.

### Problems
- Largest Rectangle in Histogram
- Sum of Subarray Minimums
- Sum of Subarray Maximums
- Sum of Subarray Ranges

---

## 3. Contribution Counting

### Use when
The problem asks for the total contribution of each element to all subarrays or all valid structures.

### Core idea
Instead of thinking about all subarrays directly, count how many valid choices each element gets on the left and right.

### Formula
```text
contribution of element = value × leftChoices × rightChoices
```

### Key relation
If you know the boundaries:
```text
leftChoices = i - prev
rightChoices = next - i
```

### Problems
- Sum of Subarray Minimums
- Sum of Subarray Maximums
- Sum of Subarray Ranges

---

## 4. Width / Area Problems

### Use when
You need the largest rectangle, largest area, or width determined by a current height.

### Core idea
The current element acts as the limiting height. The width is controlled by the nearest smaller elements on both sides.

### Formula
```text
width = nextSmaller - prevSmaller - 1
area = height × width
```

### Classic problem
- Largest Rectangle in Histogram

### Remember
This is a special case of boundary logic with area calculation.

---

## 5. Collision Simulation

### Use when
Only the latest surviving element matters, and previous elements get removed after collisions.

### Core idea
A stack is perfect because only the current top matters for the next collision.

### Mental model
The problem is like a simulation where weaker elements disappear.

### Problems
- Asteroid Collision
- Remove Adjacent Duplicates
- Valid Parentheses

### Remember
You are not trying to find the global answer immediately; you are simulating local interactions.

---

## 6. Greedy Removal

### Use when
You need to remove previous worse elements to keep a better structure.

### Core idea
When a better element arrives, remove previous weaker ones greedily.

### Template
```java
while (!stack.isEmpty() && current is better && can remove) {
    stack.pop();
}
stack.push(current);
```

### Classic problem
- Remove K Digits

### Remember
This is not just “stacking”; it is “stack while maintaining the best possible prefix”.

---

## 7. Monotonic Increasing Stack

### Shape
```text
1 3 5 8
```

### Use for
- Previous Smaller
- Next Smaller
- Histogram problems
- Sum of minimums problems

### Rule
Pop while:
```text
current < top
```

---

## 8. Monotonic Decreasing Stack

### Shape
```text
9 7 5 2
```

### Use for
- Previous Greater
- Next Greater
- Sum of maximums problems

### Rule
Pop while:
```text
current > top
```

---

## 9. Duplicate Handling

This is one of the most important details.

### Rule
Never use two strict or two non-strict conditions at the same time. Exactly one side must own duplicates.

### For minimums
```text
previous: >=
next: >
```

### For maximums
```text
previous: <=
next: <
```

### Why it matters
It prevents off-by-one mistakes and makes your logic consistent.

---

## 10. Direction of Traversal

### Need previous element?
```text
Left → Right
```

### Need next element?
```text
Right → Left
```

### Memory trick
- Previous = scan normally
- Next = scan backwards

---

## 11. What should you store?

### Usually store
```java
Stack<Integer>
```
with indices, because indices give:
- value
- distance
- width

### If only values matter
```java
Stack<Character>
```
or
```java
Stack<Integer>
```
is enough.

---

## 12. Core formulas to remember

### Boundary
```text
left = i - prev
right = next - i
```

### Contribution
```text
value × left × right
```

### Width / Area
```text
width = next - prev - 1
area = height × width
```

---

## 13. Common templates

### Previous Smaller
```java
while (!stack.isEmpty() && arr[stack.peek()] >= arr[i]) {
    stack.pop();
}
```

### Next Smaller
```java
while (!stack.isEmpty() && arr[stack.peek()] > arr[i]) {
    stack.pop();
}
```

### Previous Greater
```java
while (!stack.isEmpty() && arr[stack.peek()] <= arr[i]) {
    stack.pop();
}
```

### Next Greater
```java
while (!stack.isEmpty() && arr[stack.peek()] < arr[i]) {
    stack.pop();
}
```

---

## 14. Revision checklist
Before solving a new problem, ask:

1. Do I need the nearest smaller/greater?
2. Do I need a left boundary and a right boundary?
3. Is every element contributing independently?
4. Is the latest survivor the only thing that matters?
5. Do I need to remove previous worse elements greedily?

If yes, the monotonic stack is likely the right tool.

---

## 15. Problems you have already covered
- Next Greater Element → Nearest Greater
- Previous/Next Smaller → Boundary
- Sum of Subarray Minimums → Contribution
- Sum of Subarray Ranges → Contribution + Max + Min
- Largest Rectangle in Histogram → Boundary + Width
- Asteroid Collision → Simulation
- Remove K Digits → Greedy Removal

---

## 16. What to practice next
### Easy
- Next Greater Element I
- Next Greater Element II
- Daily Temperatures
- Online Stock Span

### Medium
- Maximal Rectangle
- 132 Pattern
- Number of Visible People in a Queue
- Buildings With an Ocean View

### Hard
- Trapping Rain Water (stack approach)

The goal is not to memorize every problem, but to recognize the pattern quickly and apply the correct template.
