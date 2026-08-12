# Competitive Programming Revision Notes

## Digit DP + Weighted Interval Scheduling

---

# 1. Problem Pattern: First Digit = Last Digit

## Problem
Given an inclusive range `[L, R]`, count how many numbers have:

```
first digit == last digit
```

### Example

```
L = 47
R = 73
```
Numbers in the range:

```
47, 48, ..., 54, 55, 56, ..., 65, 66, ..., 73
```
Valid numbers:

```
55
66
```
Therefore:

```
Answer = 2
```
Other examples:

```
11  -> valid
22  -> valid
101 -> valid
343 -> valid
47  -> invalid
123 -> invalid
98  -> invalid
```

---

# 2. Brute Force Approach
For every number from `L` to `R`:

1. Find its first digit.
2. Find its last digit.
3. Compare them.
4. Increment the answer if they are equal.
Example:

```
static boolean sameFirstLast(int n) {
    int last = n % 10;

    int first = n;
    while (first >= 10)
        first /= 10;

    return first == last;
}
```
Then:

```
for (int i = L; i <= R; i++) {
    if (sameFirstLast(i))
        count++;
}
```

## Complexity

```
O(R - L + 1)
```
This is fine for a small range.

But if:

```
R = 10^18
```
we obviously cannot iterate through every number.

That is where Digit DP is useful.

---

# 3. Digit DP

## Main Idea
Instead of checking every number individually, construct numbers digit by digit and count all possibilities.

We calculate:

```
count(L, R)
=
countUpto(R) - countUpto(L - 1)
```
This is a very important Digit DP technique.

---

# 4. Why `countUpto()`?
Suppose we need:

```
count numbers in [47, 73]
```
It is easier to calculate:

```
numbers from 1 to 73
```
and subtract:

```
numbers from 1 to 46
```
Therefore:

```
answer = countUpto(73) - countUpto(46)
```
In general:

```
answer = countUpto(R) - countUpto(L - 1)
```
This converts a range problem into a prefix problem.

---

# 5. Digit DP State
A common state is:

```
dfs(pos, first, started, tight)
```
Each variable has a specific purpose.

## `pos`
Current digit position.

For:

```
573
```
positions are:

```
0 -> 5
1 -> 7
2 -> 3
```

---

## `first`
The first digit of the number being constructed.

Example:

```
5__
```
Then:

```
first = 5
```
When we reach the final digit, we check:

```
last == first
```

---

## `started`
This tells us whether the actual number has started.

This matters because Digit DP often represents shorter numbers using leading zeros.

For example, while considering numbers up to `573`, the number:

```
55
```
may internally look like:

```
055
```
The first `0` is not actually part of the number.

Therefore:

```
started = false
```
before the first non-zero digit.

Once we choose `5`:

```
started = true
```

---

# 6. The Most Important Concept: `tight`
`tight` tells us whether the number we are currently constructing is still exactly following the upper limit.

Suppose:

```
R = 573
```
We start with:

```
tight = true
```
because we have not chosen anything yet.

---

## Example: Still Tight
Suppose we choose:

```
5
```
We have:

```
5__
```
The first digit is exactly the same as the first digit of `573`.

So we are still restricted by `573`.

```
tight = true
```
For the next digit, we cannot exceed `7`.

---

## Example: No Longer Tight
Suppose instead we choose:

```
4
```
Now we have:

```
4__
```
Since:

```
4__ < 573
```
we can never exceed `573`, regardless of the remaining digits.

For example:

```
400 < 573
499 < 573
456 < 573
```
Therefore:

```
tight = false
```
Once `tight` becomes false, we are free to choose any digit `0..9`.

---

# 7. Why Cache Only `tight == 0`?
This was an important concept discussed earlier.

We use:

```
if (tight == 0)
    dp[pos][first][started][0] = ans;
```
Why?

When:

```
tight = 0
```
we are already below the upper limit.

Therefore, the remaining answer depends only on:

```
position
first digit
started
```
It does NOT depend on the actual upper bound anymore.

So the state can safely be reused.

---

## Why not cache `tight == 1`?
When:

```
tight = 1
```
we are still restricted by the exact digits of the upper bound.

For example, the limits:

```
573
```
and:

```
583
```
can produce different possibilities from the same position.

Therefore, the answer is dependent on the limit.

In the standard implementation, we don't cache those states.

### Easy way to remember
Think of the upper bound as a fence.

```
tight = 1
```
You are still touching the fence.

You must be careful.

```
tight = 0
```
You are already inside the fence.

You can move freely.

Therefore:

```
tight == 0 -> reusable state
tight == 1 -> limit-dependent state
```

---

# 8. Updating Tight
The transition is conceptually:

```
newTight = tight && (digit == limitDigit)
```
Meaning:

If we were already tight AND we choose exactly the maximum allowed digit, we remain tight.

Otherwise, we become non-tight.

For example:

Limit:

```
573
```
Current digit limit:

```
7
```
Choose:

```
7
```
Then:

```
tight = true
```
Choose:

```
6
```
Then:

```
tight = false
```

---

# 9. Digit DP Base Case
When:

```
pos == num.length
```
we have finished constructing the number.

If a number was actually started:

```
return 1;
```
Otherwise:

```
return 0;
```
The all-zero path represents no actual number.

---

# 10. Digit DP Code

```
public class Main {

    static long[][][][] dp;
    static char[] num;

    static long dfs(int pos, int first, int started, int tight) {

        if (pos == num.length)
            return started == 1 ? 1 : 0;

        if (tight == 0 && dp[pos][first][started][0] != -1)
            return dp[pos][first][started][0];

        int limit = tight == 1 ? num[pos] - '0' : 9;

        long ans = 0;

        for (int d = 0; d <= limit; d++) {

            int newTight =
                (tight == 1 && d == limit) ? 1 : 0;

            if (started == 0) {

                if (d == 0) {
                    ans += dfs(
                        pos + 1,
                        10,
                        0,
                        newTight
                    );
                } else {
                    ans += dfs(
                        pos + 1,
                        d,
                        1,
                        newTight
                    );
                }

            } else {

                if (pos == num.length - 1) {

                    if (d == first)
                        ans++;

                } else {

                    ans += dfs(
                        pos + 1,
                        first,
                        1,
                        newTight
                    );
                }
            }
        }

        if (tight == 0)
            dp[pos][first][started][0] = ans;

        return ans;
    }

    static long countUpto(long x) {

        if (x <= 0)
            return 0;

        num = Long.toString(x).toCharArray();

        dp = new long[20][11][2][2];

        for (int i = 0; i < 20; i++)
            for (int j = 0; j < 11; j++)
                for (int k = 0; k < 2; k++)
                    for (int l = 0; l < 2; l++)
                        dp[i][j][k][l] = -1;

        return dfs(0, 10, 0, 1);
    }

    public static void main(String[] args) {

        long L = 47;
        long R = 73;

        System.out.println(
            countUpto(R) -
            countUpto(L - 1)
        );
    }
}
```

---

# 11. Complexity of Digit DP
For a `long`:

```
Number of digits <= 19
```
States are roughly:

```
position × firstDigit × started × tight
```
Approximately:

```
19 × 11 × 2 × 2
```
For every state, we try at most 10 digits.

So:

```
Time = O(19 × 11 × 2 × 2 × 10)
```
Effectively constant time per query.

This is dramatically better than iterating up to `10^18`.

---

# 12. How to Recognize Digit DP
Think about Digit DP when:

- `N` can be very large, often up to `10^18`
- You need to count numbers from `0/1` to `N`
- The condition depends on digits
- You need to count numbers satisfying properties such as:

- digit sum
- number of occurrences of a digit
- first digit / last digit
- no consecutive equal digits
- alternating digits
- divisibility by some number
- number of distinct digits
- digit restrictions
Typical state variables include:

```
pos
tight
started
previousDigit
sum
remainder
mask
```
The exact state depends on the condition.

---

# 13. Problem Pattern: One Room + Meetings

## Problem
We are given three arrays:

```
people[i]
start[i]
end[i]
```
Each index represents one meeting.

Example:

```
people = [10, 20, 30]

start  = [1, 2, 6]

end    = [3, 5, 7]
```
There is only **one room**.

Therefore meetings cannot overlap.

The condition is:

```
start(next) > end(previous)
```
Notice the strict `>`.

So:

```
start = 6
end = 6
```
is NOT allowed.

---

# 14. What Are We Actually Maximizing?
We want to accommodate the maximum possible number of people.

Suppose:

```
Meeting A = 10 people
Meeting B = 100 people
```
Even if A finishes earlier, B might be the better choice.

Therefore, simply selecting meetings based on earliest finishing time is not enough.

Each meeting has a different value/weight:

```
weight = people
```
This makes it:

# Weighted Interval Scheduling

---

# 15. Difference Between Activity Selection and Weighted Scheduling

## Activity Selection
Every meeting has equal value.

Example:

```
Meeting A -> value 1
Meeting B -> value 1
Meeting C -> value 1
```
Goal:

```
Maximum number of meetings
```
This can be solved greedily by taking the earliest finishing meeting.

---

## Weighted Interval Scheduling
Meetings have different values.

Example:

```
A -> 10 people
B -> 100 people
C -> 20 people
```
Goal:

```
Maximum total people
```
Greedy can fail.

Therefore we use:

```
DP + Sorting + Binary Search
```

---

# 16. Why Sort by End Time?
We sort meetings by their ending time.

Example:

```
Before:

A: 1 - 5
B: 2 - 3
C: 6 - 8
```
After:

```
B: 2 - 3
A: 1 - 5
C: 6 - 8
```
Now, for meeting `i`, all previous meetings are guaranteed to finish no later than it.

This allows us to efficiently find the last compatible meeting using binary search.

---

# 17. Combining the Three Arrays
Instead of keeping:

```
people[]
start[]
end[]
```
separately, create:

```
static class Meeting {

    int start;
    int end;
    int people;

    Meeting(int start, int end, int people) {
        this.start = start;
        this.end = end;
        this.people = people;
    }
}
```
This keeps all information about a meeting together.

---

# 18. Create Meeting Objects

```
Meeting[] meetings = new Meeting[n];
```
Then:

```
for (int i = 0; i < n; i++) {

    meetings[i] =
        new Meeting(
            start[i],
            end[i],
            people[i]
        );
}
```
Now:

```
meetings[i]
```
contains all three values.

---

# 19. Calculate Total People

```
int total = 0;

for (int i = 0; i < n; i++)
    total += people[i];
```
Why?

The question asks for people who **cannot** be accommodated.

We first find:

```
maximum accommodated people
```
Then:

```
not accommodated
=
total people
-
maximum accommodated
```

---

# 20. Sort Meetings

```
Arrays.sort(
    meetings,
    Comparator.comparingInt(a -> a.end)
);
```
This sorts by:

```
end time
```
from smallest to largest.

---

# 21. DP Meaning
Create:

```
int[] dp = new int[n];
```
Definition:

```
dp[i]
=
maximum number of people that can be accommodated
using meetings 0 through i
```
This definition is extremely important.

Whenever you write a DP array, clearly define what it means.

---

# 22. First DP Value

```
dp[0] = meetings[0].people;
```
With only one meeting available, the best we can do is take that meeting.

---

# 23. Process Every Meeting

```
for (int i = 1; i < n; i++)
```
For every meeting, there are two possibilities:

```
1. Skip it
2. Take it
```

---

# 24. Option 1: Take the Current Meeting
Start with:

```
int include = meetings[i].people;
```
We are taking the current meeting.

But maybe we can also take an earlier compatible meeting.

So we need to find the last meeting that does not overlap.

---

# 25. Find Previous Compatible Meeting
We binary search among:

```
0 ... i-1
```

```
int l = 0;
int r = i - 1;
int prev = -1;
```
`prev = -1` means:

```
No compatible meeting found.
```

---

# 26. Compatibility Condition
The problem says:

```
start(next) > end(previous)
```
Therefore:

```
if (meetings[mid].end < meetings[i].start)
```
is the correct condition.

Not:

```
<=
```
because equality is not allowed.

Example:

```
Previous: 1 - 5
Current:  5 - 8
```
Here:

```
current.start = 5
previous.end = 5
```
But:

```
5 > 5
```
is false.

Therefore they overlap according to the problem's rule.

---

# 27. If Compatible

```
prev = mid;
l = mid + 1;
```
We found a compatible meeting.

But we want the **latest** compatible meeting because it gives us the largest possible set of earlier options.

Therefore we continue searching to the right.

---

# 28. If Not Compatible

```
r = mid - 1;
```
The meeting ends too late.

So we need to search earlier meetings.

---

# 29. Add Previous DP
After binary search:

```
if (prev != -1)
    include += dp[prev];
```
Suppose:

```
Current meeting = 40 people
```
and:

```
dp[prev] = 70
```
Then:

```
include = 40 + 70 = 110
```
Meaning:

```
Take the current meeting
+
best possible compatible schedule before it
```

---

# 30. Option 2: Skip the Current Meeting
If we skip meeting `i`, the answer is simply:

```
dp[i - 1]
```
because we use the best answer from all previous meetings.

---

# 31. Choose Better Option
The central recurrence is:

```
dp[i] = Math.max(dp[i - 1], include);
```
Equivalent mathematical idea:

```
dp[i] =
max(
    skip current meeting,
    take current meeting + best compatible schedule
)
```
This is the heart of Weighted Interval Scheduling.

---

# 32. Final Answer
After processing all meetings:

```
dp[n - 1]
```
contains:

```
maximum people accommodated
```
But the question asks:

```
minimum people NOT accommodated
```
Therefore:

```
return total - dp[n - 1];
```

---

# 33. Complete Weighted Interval Scheduling Code

```
import java.util.*;

public class Main {

    static class Meeting {

        int start;
        int end;
        int people;

        Meeting(int start, int end, int people) {
            this.start = start;
            this.end = end;
            this.people = people;
        }
    }

    static int minimumPeopleNotAccommodated(
            int[] people,
            int[] start,
            int[] end) {

        int n = people.length;

        Meeting[] meetings = new Meeting[n];

        int total = 0;

        for (int i = 0; i < n; i++) {

            meetings[i] =
                new Meeting(
                    start[i],
                    end[i],
                    people[i]
                );

            total += people[i];
        }

        // Sort by ending time
        Arrays.sort(
            meetings,
            Comparator.comparingInt(a -> a.end)
        );

        int[] dp = new int[n];

        dp[0] = meetings[0].people;

        for (int i = 1; i < n; i++) {

            // Option 1: Take current meeting
            int include = meetings[i].people;

            // Find latest compatible meeting
            int left = 0;
            int right = i - 1;
            int prev = -1;

            while (left <= right) {

                int mid = (left + right) / 2;

                if (meetings[mid].end < meetings[i].start) {

                    prev = mid;
                    left = mid + 1;

                } else {

                    right = mid - 1;
                }
            }

            if (prev != -1)
                include += dp[prev];

            // Option 2: Skip current meeting
            dp[i] =
                Math.max(
                    dp[i - 1],
                    include
                );
        }

        return total - dp[n - 1];
    }
}
```

---

# 34. Complexity
Sorting:

```
O(n log n)
```
For every meeting, binary search takes:

```
O(log n)
```
For all meetings:

```
O(n log n)
```
Therefore:

```
Time = O(n log n)
Space = O(n)
```

---

# 35. How to Recognize Weighted Interval Scheduling
Look for a problem containing:

```
start time
end time
value / profit / people / weight
```
and:

```
Only one activity can happen at a time
```
and the goal is:

```
maximize total value
```
Think:

```
Weighted Interval Scheduling
```
Typical solution:

```
Sort by end
        ↓
DP
        ↓
Binary search previous compatible interval
```

---

# 36. The Most Important Recurrence to Memorize
For Weighted Interval Scheduling:

```
dp[i] =
max(
    dp[i-1],
    value[i] + dp[previousCompatible]
)
```
Interpretation:

```
Don't take current
OR
Take current + best compatible previous solution
```

---

# 37. Standard Problems Similar to This

## Weighted Interval Scheduling
Classic problem.

Pattern:

```
DP + Binary Search
```

---

## LeetCode 1235 — Maximum Profit in Job Scheduling
Given:

```
startTime
endTime
profit
```
Maximize total profit.

Very similar to this problem.

---

## CSES — Projects
Given projects with:

```
start
end
reward
```
Choose non-overlapping projects with maximum reward.

Same core pattern.

---

# 38. Difficulty Comparison

## First Digit = Last Digit

### Brute force

```
Easy
```

### Digit DP

```
Medium → Hard
```
The difficulty comes from recognizing and implementing:

```
pos
first
started
tight
```

---

## Meeting Room Problem
Recognizing basic Activity Selection:

```
Easy
```
Recognizing Weighted Interval Scheduling:

```
Medium
```
Implementing:

```
Sorting + DP + Binary Search
```
is typically:

```
Medium
```

---

# 39. Important Difference: Greedy vs DP
This is a very important CP lesson.

If every meeting has equal value:

```
Meeting A = 1
Meeting B = 1
Meeting C = 1
```
and we want the maximum number of meetings:

```
Greedy
```
works.

Sort by earliest ending time.

But if:

```
A = 10 people
B = 100 people
C = 20 people
```
we cannot simply choose the earliest-ending meeting.

The values are different.

Therefore:

```
Weighted
→ DP
```

---

# 40. Quick Revision Cheat Sheet

## Digit DP
When you see:

```
Count numbers from L to R
```
with digit-related conditions and huge `R`:

```
Use:
count(R) - count(L-1)
```
Typical state:

```
dp(pos, tight, started, ...)
```
Remember:

```
pos     = current digit
tight   = still restricted by limit?
started = has actual number started?
first   = first digit
```
Important:

```
tight = 1
→ restricted

tight = 0
→ free
→ state can be reused
```

---

## Weighted Interval Scheduling
When you see:

```
start
end
value
```
with non-overlapping intervals and maximum total value:

```
Sort by end
       ↓
DP
       ↓
Binary search previous compatible interval
```
Recurrence:

```
dp[i] =
max(
    dp[i-1],
    value[i] + dp[prev]
)
```
For this problem specifically:

```
start > previousEnd
```
so compatibility is:

```
previousEnd < currentStart
```
Finally:

```
minimum not accommodated
=
total people
-
maximum accommodated people
```

---

# 41. Two Mental Templates to Memorize

## Template 1 — Digit DP
When you see:

> "Count numbers ≤ N satisfying some digit property"
Think:

```
Build number digit by digit.

State:
(pos, tight, started, required information)

At each position:
try digit 0..limit.

If tight:
limit = digit of N
else:
limit = 9

At the end:
check condition.
```
For a range:

```
answer = solve(R) - solve(L-1)
```

---

## Template 2 — Weighted Intervals
When you see:

> "Choose non-overlapping intervals to maximize total value"
Think:

```
Create intervals
        ↓
Sort by end
        ↓
dp[i]
        ↓
Find previous compatible interval
using binary search
        ↓
max(skip, take)
```
Formula:

```
dp[i] =
max(
    dp[i-1],
    value[i] + dp[prev]
)
```

---

# 42. Final Recognition Guide
| If the problem says... | Think... |
|---|---|
| Count numbers from L to R | Prefix counting |
| R can be 10¹⁸ | Digit DP |
| Property depends on digits | Digit DP |
| First/last digit condition | Digit DP |
| Digit sum condition | Digit DP |
| Previous digit matters | Digit DP |
| `tight` appears | Digit DP |
| Start + End + Value | Interval Scheduling |
| Intervals cannot overlap | Interval DP |
| Maximize profit/reward/people | Weighted Interval Scheduling |
| Equal value for every interval | Possibly Greedy |
| Different value for each interval | DP |
| Need previous non-overlapping interval | Binary Search |
| Sort by ending time | Weighted Interval Scheduling |

---

# 43. The Two Key Lessons

### Lesson 1 — Digit DP
Don't think:

> "I need to check every number."
Think:

> "I need to count all possible digit combinations without explicitly generating every number."
The key states are:

```
position
tight
started
property-specific information
```

---

### Lesson 2 — Weighted Interval Scheduling
Don't think:

> "I need to greedily choose meetings."
Think:

> "For every meeting, I have two choices: take it or skip it."
Then:

```
Take:
current value + best compatible previous answer

Skip:
previous DP answer
```
Therefore:

```
dp[i] = max(skip, take)
```
These two patterns are worth memorizing because many apparently different CP problems reduce to these exact ideas.
