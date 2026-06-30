# Complete DSA Revision and LeetCode Practice Guide

A detailed interview-prep roadmap covering core data structures, algorithms, patterns, solving approaches, examples, common mistakes, and LeetCode practice problems.

> Scope note: DSA is too broad to be literally infinite. This guide focuses on the patterns most commonly useful for coding interviews and LeetCode-style preparation: arrays, strings, linked lists, stacks/queues, trees, graphs, dynamic programming, greedy, bit manipulation, math, and advanced data structures.

## How to Use This File

- **First pass:** read the explanation and solve 2 easy/medium problems per topic.

- **Second pass:** solve the remaining practice list without looking at solutions first.

- **Revision pass:** for each topic, write the invariant, complexity, and failure cases from memory.

- **Interview pass:** explain your approach verbally before coding; most interviews evaluate reasoning, not only final code.


## Index


### Arrays and Hashing

- [Array Traversal, Simulation, and In-Place Updates](#arrays-traversal-simulation)
- [Hash Map / Frequency Counting](#hash-map-frequency)
- [Prefix Sum](#prefix-sum-basic)
- [Prefix Sum + Hash Map](#prefix-sum-hash-map)
- [Difference Array and Range Updates](#difference-array)
- [Kadane's Algorithm / Maximum Subarray DP](#kadane)
- [Two Pointers](#two-pointers)
- [Fixed-Size Sliding Window](#sliding-window-fixed)
- [Variable-Size Sliding Window](#sliding-window-variable)
- [Sorting as an Algorithmic Strategy](#sorting-and-array-greedy)
- [Intervals and Line Sweep](#intervals-line-sweep)
- [Matrix Traversal](#matrix-traversal)
- [Cyclic Sort / Index Placement](#cyclic-sort-index-placement)

### Strings

- [String Frequency, Anagrams, and Character Counting](#string-frequency-anagrams)
- [Palindromes](#palindromes)
- [KMP / Prefix Function / Z-Algorithm](#string-matching-kmp)
- [Rolling Hash / Rabin-Karp](#rolling-hash)
- [Parsing, Encoding, and String Simulation](#parsing-and-stack-strings)

### Linked Lists

- [Linked List Reversal](#linked-list-reversal)
- [Fast and Slow Pointers](#linked-list-fast-slow)
- [Dummy Node, Merge, and Pointer Rewiring](#linked-list-merge-dummy)
- [Random Pointer and Deep Copy](#linked-list-random-copy)

### Stacks and Queues

- [Stack Basics, Parentheses, and Nested State](#stack-parentheses)
- [Monotonic Stack](#monotonic-stack)
- [Queue and BFS Level Processing](#queue-bfs)
- [Deque and Monotonic Queue](#monotonic-queue)

### Heaps and Priority Queues

- [Heap / Priority Queue / Top-K](#heap-top-k)
- [Two Heaps](#two-heaps)

### Binary Search

- [Binary Search on Index / Boundary](#binary-search-boundary)
- [Binary Search on Answer](#binary-search-answer)
- [Rotated Sorted Array](#rotated-array-search)

### Trie

- [Trie / Prefix Tree](#trie-prefix-tree)

### Backtracking

- [Subsets, Combinations, and Permutations](#backtracking-subsets-permutations)
- [Constraint Backtracking](#backtracking-constraints)

### Trees

- [Binary Tree Traversal](#tree-traversal)
- [Height, Depth, Diameter, and Balanced Trees](#tree-height-diameter)
- [Tree Path Problems](#tree-paths)
- [Binary Search Tree](#bst)
- [Lowest Common Ancestor](#lca)
- [Tree Construction and Serialization](#tree-construction-serialization)
- [Tree DP](#tree-dp)

### Graphs

- [Graph Representation](#graph-representation)
- [DFS/BFS and Connected Components](#graph-dfs-bfs-components)
- [Cycle Detection](#cycle-detection)
- [Topological Sort](#topological-sort)
- [Union-Find / Disjoint Set Union](#union-find)
- [Minimum Spanning Tree: Kruskal and Prim](#mst)
- [BFS Shortest Path in Unweighted Graphs](#bfs-shortest-path)
- [Multi-Source BFS](#multi-source-bfs)
- [Dijkstra's Algorithm](#dijkstra)
- [Bellman-Ford Algorithm](#bellman-ford)
- [Floyd-Warshall / All-Pairs Shortest Path](#floyd-warshall)
- [0-1 BFS](#zero-one-bfs)
- [Tarjan's Algorithm: Bridges and Strongly Connected Components](#tarjan-bridges-scc)
- [Bipartite Graph / Graph Coloring](#bipartite-graph)
- [Eulerian Path and Hierholzer's Algorithm](#eulerian-path)

### Dynamic Programming

- [1D Dynamic Programming](#dp-1d)
- [Knapsack DP: 0/1 and Unbounded](#dp-knapsack)
- [Longest Increasing Subsequence](#dp-lis)
- [LCS, Edit Distance, and Two-String DP](#dp-lcs-edit-distance)
- [Grid DP](#dp-grid)
- [Palindrome DP and Partition DP](#dp-palindrome-partition)
- [Interval DP](#dp-interval)
- [Bitmask DP](#dp-bitmask)

### Greedy

- [Greedy Algorithms](#greedy-core)
- [Greedy Intervals and Scheduling](#greedy-intervals)

### Bit Manipulation

- [Bit Manipulation Basics](#bit-manipulation)
- [Bitmask Subsets](#bitmask-subsets)

### Math

- [Math, GCD, Sieve, and Modular Arithmetic](#math-number-theory)

### Advanced Data Structures

- [Fenwick Tree / Binary Indexed Tree](#fenwick-tree)
- [Segment Tree](#segment-tree)
- [Ordered Set / TreeMap](#ordered-set-treemap)

---

## General Problem-Solving Framework


1. **Read constraints first.** They usually decide the algorithm:
   - `n <= 20`: backtracking / bitmask DP may be valid.
   - `n <= 10^3`: O(n²) may pass.
   - `n <= 10^5`: usually O(n log n) or O(n).
   - Graph with non-negative weights: Dijkstra.
   - Graph with negative edges or K-edge constraint: Bellman-Ford-style relaxation.
2. **Classify the input shape.**
   - Array/string: sliding window, two pointers, prefix sums, sorting, hash map.
   - Linked list: pointer rewiring, fast/slow, dummy node.
   - Tree: DFS/postorder, BFS level order, path recursion.
   - Graph: BFS/DFS, topological sort, DSU, shortest path.
   - Optimization/counting: dynamic programming or greedy.
3. **Define the invariant.** State what remains true after each iteration or recursive call.
4. **Start brute force, then remove repeated work.**
   - Repeated range sums -> prefix sum.
   - Repeated membership checks -> hash map/set.
   - Repeated overlapping recursion -> memoization/DP.
   - Repeated min/max extraction -> heap/deque.
5. **Validate edge cases before coding.**
   Empty input, one element, duplicates, negative values, disconnected graph, cycles, all invalid values, maximum constraints.
6. **Complexity must match constraints.**
   If your first solution is O(n²), check whether `n` allows it. If not, identify the repeated work and cache/index it.


---

<a id="arrays-traversal-simulation"></a>

## 1. Array Traversal, Simulation, and In-Place Updates

**Category:** Arrays and Hashing


### Explanation

This is the base layer for almost every DSA problem. You scan an array, maintain state, mutate safely, or simulate operations exactly as described. The interview signal is not the algorithm itself, but your ability to preserve invariants and avoid index bugs.


### When to Use

Use when the problem statement is procedural: rotate, move, replace, count, compare adjacent elements, or simulate steps without requiring a more advanced structure.


### How to Solve

- Clarify whether the array can be modified in-place.
- Define the loop invariant: after index `i`, what part of the array is already correct?
- Use write pointers when compacting/removing elements.
- Use temporary variables when overwriting data that is still needed.
- Check empty array, one element, and all-same-value cases.

### Example

`nums = [0,1,0,3,12]`: keep a `write` pointer for non-zero elements. Each non-zero value is written to `nums[write]`, then fill the rest with zeroes.


### Common Mistakes

- Mutating while iterating without a write pointer
- Off-by-one loops
- Ignoring whether order must be preserved

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Move Zeroes](https://leetcode.com/problems/move-zeroes/) | Easy |
| 2 | [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) | Easy |
| 3 | [Rotate Array](https://leetcode.com/problems/rotate-array/) | Medium |
| 4 | [Product of Array Except Self](https://leetcode.com/problems/product-of-array-except-self/) | Medium |
| 5 | [Next Permutation](https://leetcode.com/problems/next-permutation/) | Medium |
| 6 | [Plus One](https://leetcode.com/problems/plus-one/) | Easy |
| 7 | [Sort Colors](https://leetcode.com/problems/sort-colors/) | Medium |

---

<a id="hash-map-frequency"></a>

## 2. Hash Map / Frequency Counting

**Category:** Arrays and Hashing


### Explanation

Hash maps convert repeated searching into constant-time lookup. This pattern is central for counting, detecting duplicates, grouping by a key, and remembering the last position of a value.


### When to Use

Use when you need fast membership, frequency, first/last occurrence, pair lookup, or grouping by a transformed key.


### How to Solve

- Choose the key carefully: raw value, character, tuple, normalized string, remainder, or prefix sum.
- Store the minimum information needed: count, index, boolean, list, or aggregate.
- Update the map in the correct order: sometimes lookup before insert prevents using the same element twice.
- For grouping, normalize equivalent objects into the same key.
- Analyze memory as O(number of distinct keys).

### Example

`Two Sum`: for each `x`, check whether `target - x` exists in the map. If yes, return both indices. Otherwise store `x -> index`.


### Common Mistakes

- Updating before lookup in pair problems
- Using a list scan instead of a map
- Forgetting negative numbers and zero as valid keys

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Two Sum](https://leetcode.com/problems/two-sum/) | Easy |
| 2 | [Contains Duplicate](https://leetcode.com/problems/contains-duplicate/) | Easy |
| 3 | [Valid Anagram](https://leetcode.com/problems/valid-anagram/) | Easy |
| 4 | [Group Anagrams](https://leetcode.com/problems/group-anagrams/) | Medium |
| 5 | [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | Medium |
| 6 | [Longest Consecutive Sequence](https://leetcode.com/problems/longest-consecutive-sequence/) | Medium |
| 7 | [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | Medium |

---

<a id="prefix-sum-basic"></a>

## 3. Prefix Sum

**Category:** Arrays and Hashing


### Explanation

Prefix sum stores cumulative totals so any range sum can be answered in O(1). It is one of the first optimizations you should try when a brute-force solution repeatedly recomputes subarray sums.


### When to Use

Use when many queries or nested loops repeatedly ask for sum/count over contiguous ranges.


### How to Solve

- Build `prefix` where `prefix[i + 1] = prefix[i] + nums[i]`.
- Range sum `[l, r]` becomes `prefix[r + 1] - prefix[l]`.
- For 2D grids, build a 2D prefix matrix with inclusion-exclusion.
- For binary arrays, prefix counts can represent number of ones/zeros.
- Watch inclusive vs exclusive boundaries.

### Example

`nums = [2,4,1,7]`, `prefix = [0,2,6,7,14]`. Sum from index 1 to 3 is `prefix[4] - prefix[1] = 12`.


### Common Mistakes

- Not using an initial zero prefix
- Mixing inclusive and exclusive indices
- Integer overflow in languages with fixed-width ints

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/) | Easy |
| 2 | [Find Pivot Index](https://leetcode.com/problems/find-pivot-index/) | Easy |
| 3 | [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | Medium |
| 4 | [Contiguous Array](https://leetcode.com/problems/contiguous-array/) | Medium |
| 5 | [Range Sum Query 2D - Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/) | Medium |
| 6 | [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) | Medium |

---

<a id="prefix-sum-hash-map"></a>

## 4. Prefix Sum + Hash Map

**Category:** Arrays and Hashing


### Explanation

This pattern finds subarrays with a target property by storing previous prefix states. Instead of checking every left boundary, you ask: has the prefix value I need already appeared?


### When to Use

Use for subarray sum equals K, divisible subarrays, equal zero/one counts, or longest subarray with a target balance.


### How to Solve

- Let `curr` be the prefix value at the current index.
- Derive the needed previous prefix: for sum K, need `curr - K`.
- Store counts if the problem asks for number of subarrays.
- Store earliest index if the problem asks for maximum length.
- Initialize the map with prefix state before the array, usually `{0: 1}` or `{0: -1}`.

### Example

`[1,2,3]`, target `3`: at prefix `3`, previous prefix `0` exists, so subarray `[1,2]` works.


### Common Mistakes

- Forgetting initial prefix zero
- Storing latest index when earliest index is needed
- Confusing count-map and index-map versions

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | Medium |
| 2 | [Continuous Subarray Sum](https://leetcode.com/problems/continuous-subarray-sum/) | Medium |
| 3 | [Contiguous Array](https://leetcode.com/problems/contiguous-array/) | Medium |
| 4 | [Subarrays Divisible by K](https://leetcode.com/problems/subarrays-divisible-by-k/) | Medium |
| 5 | [Binary Subarrays With Sum](https://leetcode.com/problems/binary-subarrays-with-sum/) | Medium |
| 6 | [Make Sum Divisible by P](https://leetcode.com/problems/make-sum-divisible-by-p/) | Medium |

---

<a id="difference-array"></a>

## 5. Difference Array and Range Updates

**Category:** Arrays and Hashing


### Explanation

A difference array makes many range increments cheap. Instead of updating every element in `[l, r]`, you mark the start and end boundary, then reconstruct the final values using a prefix sum.


### When to Use

Use when many operations add/remove values over ranges, especially when direct updates would be O(n * q).


### How to Solve

- Create `diff` initialized with zeros.
- For update `[l, r] += x`, do `diff[l] += x` and `diff[r + 1] -= x` if in bounds.
- Prefix-sum `diff` to recover actual values.
- For interval capacity problems, treat starts as +load and ends as -load.
- For 2D range updates, extend the same idea with four corner updates.

### Example

Bookings: add `+seats` at start flight and `-seats` after end flight. Prefix sum gives total booked seats per flight.


### Common Mistakes

- Forgetting the `r + 1` boundary
- Using closed intervals where the problem uses half-open intervals
- Not sorting events for line sweep variants

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Corporate Flight Bookings](https://leetcode.com/problems/corporate-flight-bookings/) | Medium |
| 2 | [Car Pooling](https://leetcode.com/problems/car-pooling/) | Medium |
| 3 | [Shifting Letters II](https://leetcode.com/problems/shifting-letters-ii/) | Medium |
| 4 | [Minimum Number of K Consecutive Bit Flips](https://leetcode.com/problems/minimum-number-of-k-consecutive-bit-flips/) | Hard |
| 5 | [Maximum Population Year](https://leetcode.com/problems/maximum-population-year/) | Easy |
| 6 | [My Calendar II](https://leetcode.com/problems/my-calendar-ii/) | Medium |

---

<a id="kadane"></a>

## 6. Kadane's Algorithm / Maximum Subarray DP

**Category:** Arrays and Hashing


### Explanation

Kadane's algorithm finds the best contiguous subarray by deciding at each element whether to extend the previous subarray or start fresh. It is a small dynamic programming pattern disguised as array scanning.


### When to Use

Use for maximum/minimum contiguous sum, best profit with one transaction, or circular maximum subarray variants.


### How to Solve

- Maintain `current_best_ending_here` and `global_best`.
- At each value: `current = max(value, current + value)`.
- Update global answer after processing the current value.
- For circular arrays, compare normal max with total sum minus minimum subarray.
- Handle all-negative arrays carefully.

### Example

`[-2,1,-3,4,-1,2,1,-5,4]`: best subarray is `[4,-1,2,1]` with sum `6`.


### Common Mistakes

- Returning zero for all-negative arrays unless empty subarray is allowed
- Forgetting circular edge case where all values are negative
- Using sliding window on arrays with negative values

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) | Medium |
| 2 | [Best Time to Buy and Sell Stock](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) | Easy |
| 3 | [Maximum Sum Circular Subarray](https://leetcode.com/problems/maximum-sum-circular-subarray/) | Medium |
| 4 | [Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/) | Medium |
| 5 | [Maximum Subarray Sum with One Deletion](https://leetcode.com/problems/maximum-subarray-sum-with-one-deletion/) | Medium |
| 6 | [House Robber](https://leetcode.com/problems/house-robber/) | Medium |

---

<a id="two-pointers"></a>

## 7. Two Pointers

**Category:** Arrays and Hashing


### Explanation

Two pointers reduce pair or range search from O(n²) to O(n) when the data has order or when movement can be justified monotonically.


### When to Use

Use on sorted arrays, palindromes, pair sums, partitioning, container area, and merging from two ends.


### How to Solve

- Identify what moving the left or right pointer changes.
- Use sorted order to eliminate impossible candidates.
- For pair sums, move left if sum is too small and right if too large.
- For partitioning, keep valid/invalid regions explicit.
- Stop when pointers cross or when all candidates have been eliminated.

### Example

`Two Sum II`: sorted array. If `nums[l] + nums[r] > target`, decreasing `r` is the only useful move.


### Common Mistakes

- Using two pointers without sorted/monotonic structure
- Skipping duplicate handling in 3Sum/4Sum
- Wrong pointer updates causing infinite loops

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | Medium |
| 2 | [3Sum](https://leetcode.com/problems/3sum/) | Medium |
| 3 | [Container With Most Water](https://leetcode.com/problems/container-with-most-water/) | Medium |
| 4 | [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | Hard |
| 5 | [Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) | Easy |
| 6 | [Squares of a Sorted Array](https://leetcode.com/problems/squares-of-a-sorted-array/) | Easy |
| 7 | [Sort Colors](https://leetcode.com/problems/sort-colors/) | Medium |

---

<a id="sliding-window-fixed"></a>

## 8. Fixed-Size Sliding Window

**Category:** Arrays and Hashing


### Explanation

A fixed-size sliding window maintains an aggregate over exactly `k` consecutive elements. You add the new right element and remove the old left element in O(1).


### When to Use

Use when the problem explicitly asks for length `k`, `k` consecutive elements, or a fixed-size substring/subarray.


### How to Solve

- Build the first window.
- Slide one step at a time: add incoming element, remove outgoing element.
- Update the answer after every complete window.
- Use frequency arrays/maps for fixed-size string windows.
- Keep window length exactly `k`.

### Example

For max average subarray of length 4, maintain the sum of the last 4 elements instead of recomputing each subarray.


### Common Mistakes

- Updating answer before the first full window exists
- Forgetting to remove outgoing element
- Confusing fixed-window with variable-window constraints

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/) | Easy |
| 2 | [Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Medium |
| 3 | [Permutation in String](https://leetcode.com/problems/permutation-in-string/) | Medium |
| 4 | [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | Hard |
| 5 | [Maximum Number of Vowels in a Substring of Given Length](https://leetcode.com/problems/maximum-number-of-vowels-in-a-substring-of-given-length/) | Medium |
| 6 | [Minimum Recolors to Get K Consecutive Black Blocks](https://leetcode.com/problems/minimum-recolors-to-get-k-consecutive-black-blocks/) | Easy |

---

<a id="sliding-window-variable"></a>

## 9. Variable-Size Sliding Window

**Category:** Arrays and Hashing


### Explanation

A variable-size window grows with the right pointer and shrinks from the left until it satisfies a constraint. The key is that the constraint must be monotonic enough for shrinking to make progress.


### When to Use

Use for longest/shortest subarray or substring satisfying constraints such as at most K distinct, sum at least target with positive numbers, or no repeated characters.


### How to Solve

- Expand right and update state.
- While the window is invalid, remove `left` and increment left.
- When valid, update max/min answer depending on the problem.
- Use maps for distinct counts and duplicate counts.
- Confirm whether negative numbers break the monotonic assumption.

### Example

`Longest Substring Without Repeating Characters`: expand until duplicate appears; shrink left until the duplicate is removed.


### Common Mistakes

- Using it for arbitrary negative sums
- Shrinking too early or too late
- Not deleting zero-count map keys

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Medium |
| 2 | [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) | Medium |
| 3 | [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | Medium |
| 4 | [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Hard |
| 5 | [Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets/) | Medium |
| 6 | [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) | Medium |
| 7 | [Longest Subarray of 1's After Deleting One Element](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/) | Medium |

---

<a id="sorting-and-array-greedy"></a>

## 10. Sorting as an Algorithmic Strategy

**Category:** Arrays and Hashing


### Explanation

Sorting exposes order. Once values are ordered, many problems become greedy, two-pointer, interval, or duplicate-handling problems.


### When to Use

Use when relative ordering matters more than original positions, or when you need to group equal/nearby values.


### How to Solve

- Ask whether losing original order is allowed.
- Sort by the field that makes the greedy decision locally obvious.
- For custom objects, sort by start time, end time, value, or ratio depending on the invariant.
- After sorting, scan once and maintain the best state.
- Handle ties intentionally.

### Example

In interval removal problems, sorting by end time lets you keep the interval that leaves maximum room for future intervals.


### Common Mistakes

- Sorting when original order is required
- Wrong tie-breaker
- Assuming O(n log n) is acceptable without checking constraints

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium |
| 2 | [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium |
| 3 | [Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) | Medium |
| 4 | [Meeting Rooms III](https://leetcode.com/problems/meeting-rooms-iii/) | Hard |
| 5 | [Largest Number](https://leetcode.com/problems/largest-number/) | Medium |
| 6 | [Boats to Save People](https://leetcode.com/problems/boats-to-save-people/) | Medium |

---

<a id="intervals-line-sweep"></a>

## 11. Intervals and Line Sweep

**Category:** Arrays and Hashing


### Explanation

Interval problems are about overlap, coverage, gaps, or active counts over time. Line sweep converts ranges into sorted events and processes changes in active state.


### When to Use

Use when input consists of start/end pairs: meetings, bookings, balloons, calendar ranges, CPU tasks, or coverage.


### How to Solve

- Sort intervals by start or end depending on the question.
- For merging, compare current start with previous end.
- For active counts, create events: start `+1`, end `-1`.
- Choose whether endpoints are inclusive or exclusive.
- For greedy removals, prefer intervals with earlier end.

### Example

`[[1,3],[2,6],[8,10]]`: sorted by start, `[1,3]` overlaps `[2,6]`, so merge into `[1,6]`.


### Common Mistakes

- Endpoint convention bugs
- Sorting by start when end-time greedy is needed
- Not handling nested intervals

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium |
| 2 | [Insert Interval](https://leetcode.com/problems/insert-interval/) | Medium |
| 3 | [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium |
| 4 | [Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) | Medium |
| 5 | [My Calendar I](https://leetcode.com/problems/my-calendar-i/) | Medium |
| 6 | [My Calendar II](https://leetcode.com/problems/my-calendar-ii/) | Medium |
| 7 | [Employee Free Time](https://leetcode.com/problems/employee-free-time/) | Hard |

---

<a id="matrix-traversal"></a>

## 12. Matrix Traversal

**Category:** Arrays and Hashing


### Explanation

Matrix problems are graph problems on a grid, simulation problems, or boundary-management problems. The core skill is converting `(row, col)` movement into safe loops.


### When to Use

Use when input is a 2D grid and you need search, count, transform, rotate, spiral, or dynamic programming.


### How to Solve

- Define directions as `[(1,0),(-1,0),(0,1),(0,-1)]` or diagonals if needed.
- Always check bounds before reading cells.
- Use visited marking to avoid repeated processing.
- For in-place changes, decide whether to mutate immediately or mark first.
- Separate traversal logic from business condition.

### Example

For flood fill, start from `(sr, sc)`, visit all neighbors with the original color, and recolor them.


### Common Mistakes

- Swapping row/column dimensions
- Mutating cells before finishing condition checks
- Missing diagonal/non-diagonal distinction

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Flood Fill](https://leetcode.com/problems/flood-fill/) | Easy |
| 2 | [Number of Islands](https://leetcode.com/problems/number-of-islands/) | Medium |
| 3 | [Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | Medium |
| 4 | [Set Matrix Zeroes](https://leetcode.com/problems/set-matrix-zeroes/) | Medium |
| 5 | [Spiral Matrix](https://leetcode.com/problems/spiral-matrix/) | Medium |
| 6 | [Rotate Image](https://leetcode.com/problems/rotate-image/) | Medium |
| 7 | [Game of Life](https://leetcode.com/problems/game-of-life/) | Medium |

---

<a id="cyclic-sort-index-placement"></a>

## 13. Cyclic Sort / Index Placement

**Category:** Arrays and Hashing


### Explanation

When array values are from a known range such as `1..n`, each value has a natural target index. You can place values into their correct positions in O(n) without extra memory.


### When to Use

Use for missing number, duplicate number, first missing positive, or problems where values map directly to indices.


### How to Solve

- While `nums[i]` belongs in the array and is not already in its target position, swap it there.
- Do not increment `i` after a swap until the new value at `i` is checked.
- After placement, scan for the first mismatch.
- Guard against duplicates to avoid infinite swaps.
- Use sign marking as an alternative when swaps are risky.

### Example

`[3,4,-1,1]`: place value `1` at index `0`, value `3` at index `2`; first index where `nums[i] != i+1` gives answer `2`.


### Common Mistakes

- Infinite loop on duplicate values
- Not checking value bounds
- Assuming zero-based when values are one-based

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Missing Number](https://leetcode.com/problems/missing-number/) | Easy |
| 2 | [Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/) | Easy |
| 3 | [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) | Medium |
| 4 | [First Missing Positive](https://leetcode.com/problems/first-missing-positive/) | Hard |
| 5 | [Set Mismatch](https://leetcode.com/problems/set-mismatch/) | Easy |
| 6 | [Find All Duplicates in an Array](https://leetcode.com/problems/find-all-duplicates-in-an-array/) | Medium |

---

<a id="string-frequency-anagrams"></a>

## 14. String Frequency, Anagrams, and Character Counting

**Category:** Strings


### Explanation

Most string counting problems reduce to comparing frequency vectors. For lowercase English letters, a fixed array of size 26 is faster and simpler than a hash map.


### When to Use

Use when order does not matter but character counts do: anagrams, ransom note, grouping, replacement, or permutations.


### How to Solve

- Choose frequency representation: 26-array, 128/256 ASCII array, Unicode map, or sorted string.
- For anagrams, compare counts rather than repeatedly sorting if many checks are needed.
- For grouping, use a tuple/count-string as a map key.
- Track mismatch count for faster sliding-window checks.
- Be explicit about case sensitivity and allowed characters.

### Example

`eat`, `tea`, and `ate` share the same sorted key `aet` or the same 26-count vector.


### Common Mistakes

- Assuming only lowercase without reading constraints
- Sorting every window unnecessarily
- Using mutable arrays as hash keys

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Valid Anagram](https://leetcode.com/problems/valid-anagram/) | Easy |
| 2 | [Group Anagrams](https://leetcode.com/problems/group-anagrams/) | Medium |
| 3 | [Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Medium |
| 4 | [Permutation in String](https://leetcode.com/problems/permutation-in-string/) | Medium |
| 5 | [Ransom Note](https://leetcode.com/problems/ransom-note/) | Easy |
| 6 | [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Hard |

---

<a id="palindromes"></a>

## 15. Palindromes

**Category:** Strings


### Explanation

Palindrome problems exploit symmetry. You either compare from both ends, expand around a center, or use DP when many substring states are needed.


### When to Use

Use when the problem asks for mirrored strings, longest palindromic substring/subsequence, palindrome partitions, or near-palindromes.


### How to Solve

- For validation, use two pointers from both ends.
- For substring search, expand around each possible center.
- For deletion variants, branch once when a mismatch occurs.
- For subsequence variants, use DP on intervals.
- Normalize the string if ignoring case/non-alphanumeric characters.

### Example

`babad`: expanding around centers finds `bab` or `aba` as a longest palindromic substring.


### Common Mistakes

- Confusing substring and subsequence
- Not handling even-length centers
- Over-branching deletion cases

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Valid Palindrome](https://leetcode.com/problems/valid-palindrome/) | Easy |
| 2 | [Valid Palindrome II](https://leetcode.com/problems/valid-palindrome-ii/) | Easy |
| 3 | [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/) | Medium |
| 4 | [Palindromic Substrings](https://leetcode.com/problems/palindromic-substrings/) | Medium |
| 5 | [Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/) | Medium |
| 6 | [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/) | Medium |
| 7 | [Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/) | Hard |

---

<a id="string-matching-kmp"></a>

## 16. KMP / Prefix Function / Z-Algorithm

**Category:** Strings


### Explanation

KMP and Z-algorithm avoid rechecking characters during pattern matching. They precompute how much of the current match can be reused after a mismatch.


### When to Use

Use for substring search, repeated pattern detection, prefix-suffix analysis, and problems where naive matching is too slow.


### How to Solve

- For KMP, build `lps[i]`: length of longest proper prefix equal to suffix for pattern prefix ending at `i`.
- During matching, on mismatch, jump using `lps` instead of resetting to zero.
- For Z-algorithm, compute longest substring starting at `i` matching the prefix.
- Use these when O(n*m) is too slow.
- Test with repeated characters like `aaaaab`.

### Example

Pattern `ababaca`: if matching fails after `ababa`, KMP knows that prefix `aba` can still be reused.


### Common Mistakes

- Incorrect LPS fallback loop
- Not distinguishing proper prefix from whole string
- Overusing KMP where built-in search is acceptable in interviews unless algorithm is expected

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | Easy |
| 2 | [Repeated Substring Pattern](https://leetcode.com/problems/repeated-substring-pattern/) | Easy |
| 3 | [Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/) | Hard |
| 4 | [Longest Happy Prefix](https://leetcode.com/problems/longest-happy-prefix/) | Hard |
| 5 | [Repeated String Match](https://leetcode.com/problems/repeated-string-match/) | Medium |
| 6 | [Count Beautiful Splits in an Array](https://leetcode.com/problems/count-beautiful-splits-in-an-array/) | Medium |

---

<a id="rolling-hash"></a>

## 17. Rolling Hash / Rabin-Karp

**Category:** Strings


### Explanation

Rolling hash assigns numeric fingerprints to substrings so equality checks become fast. It is useful when comparing many substrings or detecting duplicates.


### When to Use

Use for duplicate substring, repeated DNA sequences, substring search, and binary-search-on-length string problems.


### How to Solve

- Choose a base and modulus, or use 64-bit unsigned overflow carefully.
- Precompute prefix hash and powers.
- Hash substring `[l, r]` in O(1).
- Use double hashing or verify actual strings to reduce collision risk.
- Often combine with binary search on answer length.

### Example

Longest duplicate substring: binary search length `L`; use hashes to check whether any length-`L` substring repeats.


### Common Mistakes

- Ignoring collision risk
- Negative modulo bugs
- Using substring copies inside the supposedly O(n) check

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Repeated DNA Sequences](https://leetcode.com/problems/repeated-dna-sequences/) | Medium |
| 2 | [Longest Duplicate Substring](https://leetcode.com/problems/longest-duplicate-substring/) | Hard |
| 3 | [Find the Index of the First Occurrence in a String](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | Easy |
| 4 | [Distinct Echo Substrings](https://leetcode.com/problems/distinct-echo-substrings/) | Hard |
| 5 | [Longest Happy Prefix](https://leetcode.com/problems/longest-happy-prefix/) | Hard |
| 6 | [Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/) | Hard |

---

<a id="parsing-and-stack-strings"></a>

## 18. Parsing, Encoding, and String Simulation

**Category:** Strings


### Explanation

Parsing problems require controlled reading of tokens, numbers, signs, brackets, and delimiters. Most mistakes come from trying to handle all cases inside one messy loop.


### When to Use

Use when strings represent expressions, paths, compressed text, encoded forms, logs, or nested structures.


### How to Solve

- Define token types: number, word, operator, bracket, slash, sign.
- Use a stack for nested structures.
- Accumulate multi-digit numbers explicitly.
- Separate parsing state from output construction.
- Test empty input, nested input, and invalid-looking edge cases allowed by constraints.

### Example

`3[a2[c]]` decodes by pushing previous string and repeat count at `[`, then expanding at `]`.


### Common Mistakes

- Treating each digit as a separate number
- Not resetting current buffer after pushing
- Incorrect operator precedence handling

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Decode String](https://leetcode.com/problems/decode-string/) | Medium |
| 2 | [Basic Calculator](https://leetcode.com/problems/basic-calculator/) | Hard |
| 3 | [Basic Calculator II](https://leetcode.com/problems/basic-calculator-ii/) | Medium |
| 4 | [Simplify Path](https://leetcode.com/problems/simplify-path/) | Medium |
| 5 | [String to Integer (atoi)](https://leetcode.com/problems/string-to-integer-atoi/) | Medium |
| 6 | [Remove All Adjacent Duplicates In String](https://leetcode.com/problems/remove-all-adjacent-duplicates-in-string/) | Easy |

---

<a id="linked-list-reversal"></a>

## 19. Linked List Reversal

**Category:** Linked Lists


### Explanation

Reversal is the core linked-list operation. Because nodes only point forward, you must preserve the next node before changing the current pointer.


### When to Use

Use when reversing a full list, a sublist, groups of k, or when comparing list halves.


### How to Solve

- Maintain `prev`, `curr`, and `next_node`.
- Before changing `curr.next`, store `next_node = curr.next`.
- Set `curr.next = prev`, then move both pointers forward.
- For sublist reversal, use a dummy node and reconnect boundaries.
- For k-group reversal, first confirm there are at least k nodes.

### Example

`1 -> 2 -> 3`: after one step, `1 -> null`, `prev = 1`, `curr = 2`.


### Common Mistakes

- Losing the rest of the list by overwriting `next`
- Not reconnecting reversed sublist boundaries
- Forgetting dummy node for head changes

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Reverse Linked List](https://leetcode.com/problems/reverse-linked-list/) | Easy |
| 2 | [Reverse Linked List II](https://leetcode.com/problems/reverse-linked-list-ii/) | Medium |
| 3 | [Reverse Nodes in k-Group](https://leetcode.com/problems/reverse-nodes-in-k-group/) | Hard |
| 4 | [Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/) | Easy |
| 5 | [Swap Nodes in Pairs](https://leetcode.com/problems/swap-nodes-in-pairs/) | Medium |
| 6 | [Reorder List](https://leetcode.com/problems/reorder-list/) | Medium |

---

<a id="linked-list-fast-slow"></a>

## 20. Fast and Slow Pointers

**Category:** Linked Lists


### Explanation

Fast/slow pointers exploit different movement speeds. They find middle nodes, detect cycles, locate cycle entry, and split linked lists.


### When to Use

Use when you need the middle, cycle detection, cycle start, or balanced splitting without extra memory.


### How to Solve

- Move `slow` one step and `fast` two steps.
- If `fast` reaches null, there is no cycle.
- If they meet, reset one pointer to head to find cycle entry.
- For middle selection, decide whether you need first middle or second middle.
- For list splitting, cut the list after finding the midpoint.

### Example

In a cycle, fast gains one node per iteration over slow, so eventually they meet inside the loop.


### Common Mistakes

- Wrong middle for even-length lists
- Dereferencing `fast.next` without null check
- Not breaking the list before merge sort

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Linked List Cycle](https://leetcode.com/problems/linked-list-cycle/) | Easy |
| 2 | [Linked List Cycle II](https://leetcode.com/problems/linked-list-cycle-ii/) | Medium |
| 3 | [Middle of the Linked List](https://leetcode.com/problems/middle-of-the-linked-list/) | Easy |
| 4 | [Happy Number](https://leetcode.com/problems/happy-number/) | Easy |
| 5 | [Palindrome Linked List](https://leetcode.com/problems/palindrome-linked-list/) | Easy |
| 6 | [Reorder List](https://leetcode.com/problems/reorder-list/) | Medium |

---

<a id="linked-list-merge-dummy"></a>

## 21. Dummy Node, Merge, and Pointer Rewiring

**Category:** Linked Lists


### Explanation

A dummy node removes special handling for operations that may change the head. It is especially useful for merging, deleting, partitioning, and building new lists.


### When to Use

Use when constructing or modifying linked lists while the head may change.


### How to Solve

- Create `dummy` and `tail`.
- Attach nodes to `tail.next` and advance `tail`.
- For deletion, use `prev` to skip nodes.
- Return `dummy.next`.
- Be careful whether you are reusing nodes or creating new ones.

### Example

Merging two sorted lists: repeatedly attach the smaller current node to `tail`, then attach the remaining list.


### Common Mistakes

- Returning dummy instead of dummy.next
- Not advancing tail
- Creating new nodes when in-place rewiring is expected

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Merge Two Sorted Lists](https://leetcode.com/problems/merge-two-sorted-lists/) | Easy |
| 2 | [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Hard |
| 3 | [Remove Nth Node From End of List](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) | Medium |
| 4 | [Partition List](https://leetcode.com/problems/partition-list/) | Medium |
| 5 | [Add Two Numbers](https://leetcode.com/problems/add-two-numbers/) | Medium |
| 6 | [Odd Even Linked List](https://leetcode.com/problems/odd-even-linked-list/) | Medium |

---

<a id="linked-list-random-copy"></a>

## 22. Random Pointer and Deep Copy

**Category:** Linked Lists


### Explanation

Deep-copy linked-list problems require preserving relationships between old nodes and new nodes. A map from original node to copied node is the cleanest method.


### When to Use

Use when nodes have extra pointers such as random, child, or arbitrary graph-like links.


### How to Solve

- First pass: create a copy node for every original node and store `old -> new`.
- Second pass: assign `next` and `random` using the map.
- Alternative O(1) space: interleave copied nodes between originals, assign randoms, then separate.
- Handle null pointers explicitly.
- Ensure the result has no references to original nodes.

### Example

If `old.random = x`, then `copy.random = map[x]`.


### Common Mistakes

- Shallow copying random pointers
- Not handling null random
- Corrupting original list in interleaving method

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/) | Medium |
| 2 | [Flatten a Multilevel Doubly Linked List](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/) | Medium |
| 3 | [Clone Graph](https://leetcode.com/problems/clone-graph/) | Medium |
| 4 | [Design Linked List](https://leetcode.com/problems/design-linked-list/) | Medium |
| 5 | [LRU Cache](https://leetcode.com/problems/lru-cache/) | Medium |

---

<a id="stack-parentheses"></a>

## 23. Stack Basics, Parentheses, and Nested State

**Category:** Stacks and Queues


### Explanation

Stacks model last-in-first-out behavior. They are ideal for matching brackets, undoing operations, nested parsing, and maintaining previous states.


### When to Use

Use when the most recent unmatched item must be resolved first.


### How to Solve

- Push opening brackets or previous states.
- On closing token, validate against the stack top.
- For nested values, push the current context and reset.
- At the end, ensure the stack is empty if all opens must close.
- Use sentinel values to simplify length calculations.

### Example

`({[]})`: every closing bracket must match the most recent opening bracket.


### Common Mistakes

- Popping from empty stack
- Only counting brackets instead of matching types
- Forgetting final non-empty stack check

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Valid Parentheses](https://leetcode.com/problems/valid-parentheses/) | Easy |
| 2 | [Min Stack](https://leetcode.com/problems/min-stack/) | Medium |
| 3 | [Evaluate Reverse Polish Notation](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | Medium |
| 4 | [Decode String](https://leetcode.com/problems/decode-string/) | Medium |
| 5 | [Simplify Path](https://leetcode.com/problems/simplify-path/) | Medium |
| 6 | [Longest Valid Parentheses](https://leetcode.com/problems/longest-valid-parentheses/) | Hard |

---

<a id="monotonic-stack"></a>

## 24. Monotonic Stack

**Category:** Stacks and Queues


### Explanation

A monotonic stack keeps elements in increasing or decreasing order so you can find the next greater/smaller element efficiently.


### When to Use

Use when each element needs to know the nearest greater/smaller value to the left or right.


### How to Solve

- Decide what the stack stores: values or indices.
- While the current value breaks monotonic order, pop and resolve answers for popped indices.
- Push the current index after resolving.
- Use circular processing for circular next-greater problems.
- For histogram problems, add sentinel bars.

### Example

Daily Temperatures: when a warmer day appears, pop colder previous days and set their wait time.


### Common Mistakes

- Using values when duplicate indices are needed
- Wrong comparison direction
- Forgetting sentinel for remaining stack elements

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Daily Temperatures](https://leetcode.com/problems/daily-temperatures/) | Medium |
| 2 | [Next Greater Element I](https://leetcode.com/problems/next-greater-element-i/) | Easy |
| 3 | [Next Greater Element II](https://leetcode.com/problems/next-greater-element-ii/) | Medium |
| 4 | [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/) | Hard |
| 5 | [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | Hard |
| 6 | [Sum of Subarray Minimums](https://leetcode.com/problems/sum-of-subarray-minimums/) | Medium |
| 7 | [Remove K Digits](https://leetcode.com/problems/remove-k-digits/) | Medium |

---

<a id="queue-bfs"></a>

## 25. Queue and BFS Level Processing

**Category:** Stacks and Queues


### Explanation

A queue processes items in first-in-first-out order, which is exactly what breadth-first search needs for shortest path in unweighted graphs or level-order traversal.


### When to Use

Use when processing by distance, levels, time steps, waves, or nearest-source expansion.


### How to Solve

- Initialize queue with starting nodes.
- Mark visited when enqueuing, not when dequeuing, to avoid duplicates.
- Process one level at a time when distance/time matters.
- For grids, use direction arrays.
- Return when the target is first reached in unweighted BFS.

### Example

Rotting oranges: all initially rotten oranges are enqueued. Each BFS level represents one minute.


### Common Mistakes

- Marking visited too late
- Mixing level count with node count
- Using DFS for shortest unweighted path

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) | Medium |
| 2 | [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | Medium |
| 3 | [Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Medium |
| 4 | [Open the Lock](https://leetcode.com/problems/open-the-lock/) | Medium |
| 5 | [Perfect Squares](https://leetcode.com/problems/perfect-squares/) | Medium |
| 6 | [Word Ladder](https://leetcode.com/problems/word-ladder/) | Hard |

---

<a id="monotonic-queue"></a>

## 26. Deque and Monotonic Queue

**Category:** Stacks and Queues


### Explanation

A monotonic queue is a deque that keeps candidate elements ordered so the best element of the current window is always at the front.


### When to Use

Use for sliding-window maximum/minimum and constrained DP where you need the best previous state inside a window.


### How to Solve

- Store indices, not just values.
- Before adding current index, pop worse candidates from the back.
- Remove front index if it leaves the window.
- The front is the current max/min.
- For DP, store indices of candidate DP values.

### Example

Sliding window maximum: remove all smaller values behind the current value; they can never become maximum while current remains in window.


### Common Mistakes

- Not removing out-of-window indices
- Using a heap and forgetting lazy deletion
- Storing values and losing duplicate positions

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | Hard |
| 2 | [Shortest Subarray with Sum at Least K](https://leetcode.com/problems/shortest-subarray-with-sum-at-least-k/) | Hard |
| 3 | [Constrained Subsequence Sum](https://leetcode.com/problems/constrained-subsequence-sum/) | Hard |
| 4 | [Jump Game VI](https://leetcode.com/problems/jump-game-vi/) | Medium |
| 5 | [Maximum Number of Robots Within Budget](https://leetcode.com/problems/maximum-number-of-robots-within-budget/) | Hard |

---

<a id="heap-top-k"></a>

## 27. Heap / Priority Queue / Top-K

**Category:** Heaps and Priority Queues


### Explanation

A heap gives repeated access to the smallest or largest item in O(log n). It is the standard structure for top-k, streaming median support, scheduling, and graph shortest paths.


### When to Use

Use when you repeatedly need the current min/max while inserting new candidates.


### How to Solve

- Use a min-heap for smallest-first processing.
- For top-k largest, keep a min-heap of size k.
- For k-way merge, push the current item from each list and advance the source of the popped item.
- For lazy deletion, mark removed values and clean heap top when needed.
- Remember Python's `heapq` is a min-heap.

### Example

Top K Frequent: push `(frequency, value)` into a min-heap and keep only k elements.


### Common Mistakes

- Sorting the whole list when k is small
- Not bounding heap size
- Forgetting tie-breaking when heap entries compare complex objects

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Medium |
| 2 | [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | Medium |
| 3 | [Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/) | Medium |
| 4 | [Merge k Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Hard |
| 5 | [Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Medium |
| 6 | [Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) | Easy |
| 7 | [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | Medium |

---

<a id="two-heaps"></a>

## 28. Two Heaps

**Category:** Heaps and Priority Queues


### Explanation

Two heaps split data into lower and upper halves. This lets you maintain a median or balance two groups under streaming updates.


### When to Use

Use when you need median, running balance, or repeatedly compare lower-half max against upper-half min.


### How to Solve

- Use a max-heap for the lower half and min-heap for the upper half.
- Maintain size invariant: sizes differ by at most one.
- Maintain order invariant: every lower-half value <= every upper-half value.
- Rebalance after each insertion/deletion.
- For sliding window median, combine heaps with lazy deletion maps.

### Example

MedianFinder: if lower has one extra element, median is its max; if equal sizes, median is average of both tops.


### Common Mistakes

- Not rebalancing after deletion
- Forgetting lazy deletion in sliding window
- Letting lower max exceed upper min

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | Hard |
| 2 | [Sliding Window Median](https://leetcode.com/problems/sliding-window-median/) | Hard |
| 3 | [IPO](https://leetcode.com/problems/ipo/) | Hard |
| 4 | [Minimum Cost to Hire K Workers](https://leetcode.com/problems/minimum-cost-to-hire-k-workers/) | Hard |
| 5 | [The Number of the Smallest Unoccupied Chair](https://leetcode.com/problems/the-number-of-the-smallest-unoccupied-chair/) | Medium |

---

<a id="binary-search-boundary"></a>

## 29. Binary Search on Index / Boundary

**Category:** Binary Search


### Explanation

Binary search finds a target or boundary in a sorted search space. The hard part is not the midpoint; it is defining the invariant and termination condition.


### When to Use

Use when the array/search space is sorted or when a monotonic predicate exists.


### How to Solve

- Define whether you are finding exact target, first true, last false, lower bound, or upper bound.
- Use `while l < r` for boundary search.
- Compute `mid = l + (r - l) // 2` in fixed-width languages.
- Move only the side that can be safely eliminated.
- After the loop, validate the candidate if exact existence matters.

### Example

First bad version: predicate `isBad(mid)` is false then true. Find the first index where it becomes true.


### Common Mistakes

- Using binary search without monotonicity
- Infinite loop from wrong midpoint
- Returning insertion index without validation

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Binary Search](https://leetcode.com/problems/binary-search/) | Easy |
| 2 | [Search Insert Position](https://leetcode.com/problems/search-insert-position/) | Easy |
| 3 | [First Bad Version](https://leetcode.com/problems/first-bad-version/) | Easy |
| 4 | [Find First and Last Position of Element in Sorted Array](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) | Medium |
| 5 | [Search a 2D Matrix](https://leetcode.com/problems/search-a-2d-matrix/) | Medium |
| 6 | [Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array/) | Medium |

---

<a id="binary-search-answer"></a>

## 30. Binary Search on Answer

**Category:** Binary Search


### Explanation

Binary search on answer works when you cannot directly construct the answer, but can test whether a candidate answer is feasible. This is common in minimization/maximization problems.


### When to Use

Use when constraints are large and the answer space is ordered: if `x` works, all larger values work, or vice versa.


### How to Solve

- Identify the answer range `[lo, hi]`.
- Write a `can(x)` function that returns true/false.
- Prove monotonicity before coding.
- For minimize feasible answer: if `can(mid)` true, move right boundary to mid.
- For maximize feasible answer: if `can(mid)` true, move left boundary upward.

### Example

Koko eating bananas: if speed `k` finishes in time, any larger speed also finishes, so binary-search the minimum feasible speed.


### Common Mistakes

- Weak feasibility function
- Wrong boundary for min vs max
- Overflow in multiplication inside `can()`

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Koko Eating Bananas](https://leetcode.com/problems/koko-eating-bananas/) | Medium |
| 2 | [Capacity To Ship Packages Within D Days](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) | Medium |
| 3 | [Split Array Largest Sum](https://leetcode.com/problems/split-array-largest-sum/) | Hard |
| 4 | [Minimized Maximum of Products Distributed to Any Store](https://leetcode.com/problems/minimized-maximum-of-products-distributed-to-any-store/) | Medium |
| 5 | [Magnetic Force Between Two Balls](https://leetcode.com/problems/magnetic-force-between-two-balls/) | Medium |
| 6 | [Minimum Number of Days to Make m Bouquets](https://leetcode.com/problems/minimum-number-of-days-to-make-m-bouquets/) | Medium |

---

<a id="rotated-array-search"></a>

## 31. Rotated Sorted Array

**Category:** Binary Search


### Explanation

A rotated sorted array is two sorted halves joined at a pivot. Binary search still works if you determine which half is sorted at each step.


### When to Use

Use when a sorted array has been rotated and you need search/minimum, possibly with duplicates.


### How to Solve

- Compare `nums[l]`, `nums[mid]`, and `nums[r]`.
- If left half is sorted, check whether target lies inside it.
- Otherwise, right half is sorted.
- For minimum, move toward the unsorted side.
- With duplicates, you may need to shrink boundaries one by one.

### Example

`[4,5,6,7,0,1,2]`: if `mid=7`, left side `[4,5,6,7]` is sorted; target `0` must be on the right.


### Common Mistakes

- Using normal binary search directly
- Not handling duplicates
- Confusing pivot-finding with target-search

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) | Medium |
| 2 | [Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) | Medium |
| 3 | [Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/) | Medium |
| 4 | [Find Minimum in Rotated Sorted Array II](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/) | Hard |
| 5 | [Search a 2D Matrix II](https://leetcode.com/problems/search-a-2d-matrix-ii/) | Medium |

---

<a id="trie-prefix-tree"></a>

## 32. Trie / Prefix Tree

**Category:** Trie


### Explanation

A trie stores strings character by character, sharing common prefixes. It gives efficient prefix queries and word lookup.


### When to Use

Use when you need prefix search, autocomplete, dictionary matching, wildcard search, or many word lookups.


### How to Solve

- Each node maps character -> child node.
- Mark `is_word` at terminal nodes.
- For insert/search, traverse characters one by one.
- For prefix query, success means the prefix path exists.
- For memory-heavy cases, use dictionaries/maps instead of fixed arrays if alphabet is sparse.

### Example

Insert `cat` and `car`: both share path `c -> a`, then branch into `t` and `r`.


### Common Mistakes

- Forgetting terminal marker
- Returning true for prefix when full-word search is required
- Memory blowup from large alphabets

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Implement Trie (Prefix Tree)](https://leetcode.com/problems/implement-trie-prefix-tree/) | Medium |
| 2 | [Design Add and Search Words Data Structure](https://leetcode.com/problems/design-add-and-search-words-data-structure/) | Medium |
| 3 | [Word Search II](https://leetcode.com/problems/word-search-ii/) | Hard |
| 4 | [Replace Words](https://leetcode.com/problems/replace-words/) | Medium |
| 5 | [Map Sum Pairs](https://leetcode.com/problems/map-sum-pairs/) | Medium |
| 6 | [Search Suggestions System](https://leetcode.com/problems/search-suggestions-system/) | Medium |

---

<a id="backtracking-subsets-permutations"></a>

## 33. Subsets, Combinations, and Permutations

**Category:** Backtracking


### Explanation

Backtracking explores a decision tree and undoes choices after recursive calls. It is used when you must generate all valid configurations.


### When to Use

Use when the output itself can be exponential: subsets, permutations, combinations, partitions, and all possible valid arrangements.


### How to Solve

- Define the recursive state: index, current path, used flags, remaining target.
- Choose an option, recurse, then undo the choice.
- Use `start` index for combinations/subsets to avoid duplicates.
- Use `used[]` for permutations.
- Sort input when duplicate-skipping is required.

### Example

Subsets of `[1,2]`: choose or skip each element to produce `[]`, `[1]`, `[2]`, `[1,2]`.


### Common Mistakes

- Not undoing state
- Duplicate outputs from repeated values
- Using backtracking for problems that need DP pruning

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Subsets](https://leetcode.com/problems/subsets/) | Medium |
| 2 | [Subsets II](https://leetcode.com/problems/subsets-ii/) | Medium |
| 3 | [Permutations](https://leetcode.com/problems/permutations/) | Medium |
| 4 | [Permutations II](https://leetcode.com/problems/permutations-ii/) | Medium |
| 5 | [Combinations](https://leetcode.com/problems/combinations/) | Medium |
| 6 | [Combination Sum](https://leetcode.com/problems/combination-sum/) | Medium |
| 7 | [Combination Sum II](https://leetcode.com/problems/combination-sum-ii/) | Medium |

---

<a id="backtracking-constraints"></a>

## 34. Constraint Backtracking

**Category:** Backtracking


### Explanation

Constraint backtracking searches possible assignments while pruning invalid states early. It is the standard approach for N-Queens, Sudoku, and board search problems.


### When to Use

Use when every decision must satisfy constraints against previous decisions.


### How to Solve

- Represent constraints in O(1) sets/arrays where possible.
- At each step, generate only valid candidates.
- Apply choice, update constraints, recurse, then revert.
- Use pruning to stop branches that cannot lead to a solution.
- Pick the next variable strategically if possible.

### Example

N-Queens: place one queen per row; use columns, main diagonals, and anti-diagonals sets to validate O(1).


### Common Mistakes

- Checking whole board every time
- Forgetting to remove constraints on backtrack
- Not pruning impossible branches

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [N-Queens](https://leetcode.com/problems/n-queens/) | Hard |
| 2 | [N-Queens II](https://leetcode.com/problems/n-queens-ii/) | Hard |
| 3 | [Sudoku Solver](https://leetcode.com/problems/sudoku-solver/) | Hard |
| 4 | [Word Search](https://leetcode.com/problems/word-search/) | Medium |
| 5 | [Restore IP Addresses](https://leetcode.com/problems/restore-ip-addresses/) | Medium |
| 6 | [Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) | Medium |

---

<a id="tree-traversal"></a>

## 35. Binary Tree Traversal

**Category:** Trees


### Explanation

Tree traversal is how you visit nodes in a structured order. Preorder, inorder, postorder, and level-order each encode a different dependency relationship.


### When to Use

Use when you need to inspect every node, collect values, compute properties, or transform a tree.


### How to Solve

- Preorder: process node before children.
- Inorder: left, node, right; gives sorted order in BST.
- Postorder: process children before node; useful for height and deletion.
- Level-order: BFS by depth.
- Choose recursive or iterative based on constraints and language recursion limits.

### Example

Inorder traversal of a BST returns values in ascending sorted order.


### Common Mistakes

- Using preorder when child results are needed first
- Forgetting null base case
- Recursion depth overflow on skewed trees

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/) | Easy |
| 2 | [Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/) | Easy |
| 3 | [Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/) | Easy |
| 4 | [Binary Tree Level Order Traversal](https://leetcode.com/problems/binary-tree-level-order-traversal/) | Medium |
| 5 | [Binary Tree Zigzag Level Order Traversal](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/) | Medium |
| 6 | [Average of Levels in Binary Tree](https://leetcode.com/problems/average-of-levels-in-binary-tree/) | Easy |

---

<a id="tree-height-diameter"></a>

## 36. Height, Depth, Diameter, and Balanced Trees

**Category:** Trees


### Explanation

Many tree problems are postorder computations. You get information from left and right children, combine it at the current node, and return one value upward.


### When to Use

Use when the answer depends on subtree heights, depths, balance, diameter, or longest path.


### How to Solve

- Write a helper that returns height or a structured result.
- Update a nonlocal/global answer for paths passing through the current node.
- Return only the value the parent needs.
- Use `-1` sentinel to propagate imbalance early.
- Distinguish node count vs edge count.

### Example

Diameter at a node is `left_height + right_height`; height returned upward is `1 + max(left, right)`.


### Common Mistakes

- Returning diameter instead of height
- Counting edges/nodes inconsistently
- Recomputing height repeatedly causing O(n²)

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Maximum Depth of Binary Tree](https://leetcode.com/problems/maximum-depth-of-binary-tree/) | Easy |
| 2 | [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) | Easy |
| 3 | [Balanced Binary Tree](https://leetcode.com/problems/balanced-binary-tree/) | Easy |
| 4 | [Minimum Depth of Binary Tree](https://leetcode.com/problems/minimum-depth-of-binary-tree/) | Easy |
| 5 | [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) | Hard |
| 6 | [Longest Univalue Path](https://leetcode.com/problems/longest-univalue-path/) | Medium |

---

<a id="tree-paths"></a>

## 37. Tree Path Problems

**Category:** Trees


### Explanation

Path problems track information from root to current node or combine paths from child subtrees. The definition of path direction is critical.


### When to Use

Use for root-to-leaf sums, any downward path sum, maximum path, path collection, and prefix-sum-on-tree problems.


### How to Solve

- Clarify whether paths must start at root, end at leaf, or can start/end anywhere.
- For root-to-leaf, carry current path/sum downward.
- For any downward path sum, use prefix sum map along the current recursion stack.
- For max path through a node, ignore negative child contributions.
- Backtrack path state after leaving a node.

### Example

Path Sum III: current prefix sum minus target tells how many previous ancestors form a valid downward path.


### Common Mistakes

- Confusing root-to-leaf with any path
- Not decrementing prefix count on backtrack
- Allowing paths to fork incorrectly

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Path Sum](https://leetcode.com/problems/path-sum/) | Easy |
| 2 | [Path Sum II](https://leetcode.com/problems/path-sum-ii/) | Medium |
| 3 | [Path Sum III](https://leetcode.com/problems/path-sum-iii/) | Medium |
| 4 | [Sum Root to Leaf Numbers](https://leetcode.com/problems/sum-root-to-leaf-numbers/) | Medium |
| 5 | [Binary Tree Maximum Path Sum](https://leetcode.com/problems/binary-tree-maximum-path-sum/) | Hard |
| 6 | [Smallest String Starting From Leaf](https://leetcode.com/problems/smallest-string-starting-from-leaf/) | Medium |

---

<a id="bst"></a>

## 38. Binary Search Tree

**Category:** Trees


### Explanation

A BST maintains ordering: left subtree values are smaller and right subtree values are larger. This allows search, validation, kth order, and range operations.


### When to Use

Use when tree ordering can eliminate branches or inorder order matters.


### How to Solve

- Use bounds `(low, high)` to validate a BST.
- Use inorder traversal for sorted values.
- For kth smallest, stop inorder after k visits.
- For search/insert/delete, move left or right based on value.
- For LCA in BST, exploit split point.

### Example

In a valid BST, every node must satisfy all ancestor bounds, not just direct parent comparison.


### Common Mistakes

- Only comparing with immediate children
- Ignoring duplicate policy
- Traversing both sides when BST property can prune

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Validate Binary Search Tree](https://leetcode.com/problems/validate-binary-search-tree/) | Medium |
| 2 | [Kth Smallest Element in a BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | Medium |
| 3 | [Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) | Medium |
| 4 | [Search in a Binary Search Tree](https://leetcode.com/problems/search-in-a-binary-search-tree/) | Easy |
| 5 | [Insert into a Binary Search Tree](https://leetcode.com/problems/insert-into-a-binary-search-tree/) | Medium |
| 6 | [Delete Node in a BST](https://leetcode.com/problems/delete-node-in-a-bst/) | Medium |
| 7 | [Convert Sorted Array to Binary Search Tree](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/) | Easy |

---

<a id="lca"></a>

## 39. Lowest Common Ancestor

**Category:** Trees


### Explanation

LCA finds the deepest node that is an ancestor of two target nodes. In a general binary tree, the answer emerges from postorder search.


### When to Use

Use when you need relationship between two nodes, distance between nodes, or subtree ownership.


### How to Solve

- If current node is null, return null.
- If current node is one of the targets, return it.
- Search left and right.
- If both sides return non-null, current node is LCA.
- If only one side returns non-null, propagate that result upward.

### Example

If `p` is found in left subtree and `q` is found in right subtree, the current node is their lowest common ancestor.


### Common Mistakes

- Using BST logic on a non-BST
- Not handling case where one node is ancestor of the other
- Assuming both nodes exist when constraints do not guarantee it

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Lowest Common Ancestor of a Binary Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) | Medium |
| 2 | [Lowest Common Ancestor of a Binary Search Tree](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) | Medium |
| 3 | [Lowest Common Ancestor of a Binary Tree III](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree-iii/) | Medium |
| 4 | [Step-By-Step Directions From a Binary Tree Node to Another](https://leetcode.com/problems/step-by-step-directions-from-a-binary-tree-node-to-another/) | Medium |
| 5 | [Find Distance in a Binary Tree](https://leetcode.com/problems/find-distance-in-a-binary-tree/) | Medium |

---

<a id="tree-construction-serialization"></a>

## 40. Tree Construction and Serialization

**Category:** Trees


### Explanation

Tree construction reconstructs structure from traversal/order data. Serialization converts a tree to a string/list and back while preserving null positions.


### When to Use

Use when building trees from preorder/inorder/postorder, encoding trees, or comparing/subtree matching.


### How to Solve

- Use preorder first element as root; inorder splits left/right subtrees.
- Build a map from value to inorder index for O(1) splits.
- Use recursion boundaries rather than slicing arrays for efficiency.
- For serialization, include null markers.
- For level-order serialization, push null children when needed to preserve shape.

### Example

Preorder `[3,9,20,15,7]`, inorder `[9,3,15,20,7]`: root is `3`; inorder splits into `[9]` and `[15,20,7]`.


### Common Mistakes

- Array slicing causing O(n²)
- Forgetting null markers
- Assuming values are unique when not guaranteed

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Construct Binary Tree from Preorder and Inorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) | Medium |
| 2 | [Construct Binary Tree from Inorder and Postorder Traversal](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/) | Medium |
| 3 | [Serialize and Deserialize Binary Tree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) | Hard |
| 4 | [Subtree of Another Tree](https://leetcode.com/problems/subtree-of-another-tree/) | Easy |
| 5 | [Find Duplicate Subtrees](https://leetcode.com/problems/find-duplicate-subtrees/) | Medium |
| 6 | [Verify Preorder Serialization of a Binary Tree](https://leetcode.com/problems/verify-preorder-serialization-of-a-binary-tree/) | Medium |

---

<a id="tree-dp"></a>

## 41. Tree DP

**Category:** Trees


### Explanation

Tree DP computes multiple states per node based on child states. It is common when each node has choices such as take/skip, camera/no camera, or robbed/not robbed.


### When to Use

Use when local choices affect child choices and the tree has no cycles.


### How to Solve

- Define states clearly for a node.
- Compute child states first with postorder DFS.
- Combine child states according to constraints.
- Return all states the parent needs.
- Avoid greedy assumptions unless proven.

### Example

House Robber III: if you rob a node, you cannot rob children; if you skip it, children may be robbed or skipped.


### Common Mistakes

- Returning only one value when parent needs multiple states
- Greedy local decisions
- Not handling null node states consistently

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [House Robber III](https://leetcode.com/problems/house-robber-iii/) | Medium |
| 2 | [Binary Tree Cameras](https://leetcode.com/problems/binary-tree-cameras/) | Hard |
| 3 | [Maximum Sum BST in Binary Tree](https://leetcode.com/problems/maximum-sum-bst-in-binary-tree/) | Hard |
| 4 | [Longest Univalue Path](https://leetcode.com/problems/longest-univalue-path/) | Medium |
| 5 | [Diameter of Binary Tree](https://leetcode.com/problems/diameter-of-binary-tree/) | Easy |
| 6 | [Sum of Distances in Tree](https://leetcode.com/problems/sum-of-distances-in-tree/) | Hard |

---

<a id="graph-representation"></a>

## 42. Graph Representation

**Category:** Graphs


### Explanation

Graph problems start with representation. Edges may be directed/undirected, weighted/unweighted, dense/sparse, implicit/explicit, and may require adjacency lists, matrices, or edge lists.


### When to Use

Use before choosing BFS, DFS, Dijkstra, Union-Find, topological sort, or dynamic programming on graph states.


### How to Solve

- Identify nodes: integers, strings, grid cells, words, bitmasks, or states.
- Identify edge direction and weight.
- Build adjacency list for sparse graphs.
- Use edge list for Bellman-Ford and Kruskal.
- Use matrix for dense all-pairs problems or Floyd-Warshall.

### Example

Flights `[u,v,w]` naturally become directed weighted edges: `adj[u].append((v,w))`.


### Common Mistakes

- Treating directed graph as undirected
- Ignoring self-loops/multiple edges
- Building O(n²) graph when implicit neighbor generation is cheaper

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Clone Graph](https://leetcode.com/problems/clone-graph/) | Medium |
| 2 | [Evaluate Division](https://leetcode.com/problems/evaluate-division/) | Medium |
| 3 | [Find if Path Exists in Graph](https://leetcode.com/problems/find-if-path-exists-in-graph/) | Easy |
| 4 | [All Paths From Source to Target](https://leetcode.com/problems/all-paths-from-source-to-target/) | Medium |
| 5 | [Reorder Routes to Make All Paths Lead to the City Zero](https://leetcode.com/problems/reorder-routes-to-make-all-paths-lead-to-the-city-zero/) | Medium |
| 6 | [Keys and Rooms](https://leetcode.com/problems/keys-and-rooms/) | Medium |

---

<a id="graph-dfs-bfs-components"></a>

## 43. DFS/BFS and Connected Components

**Category:** Graphs


### Explanation

DFS and BFS explore reachable nodes. Connected-component problems count or label groups where every node can reach every other node through allowed edges.


### When to Use

Use for islands, provinces, reachability, rooms/keys, graph traversal, and component counting.


### How to Solve

- Build neighbors or generate them on the fly.
- Maintain a visited set.
- Start traversal from every unvisited node to count components.
- For grids, mutate visited cells or use a visited matrix.
- Choose DFS for simple exploration and BFS for shortest levels.

### Example

Number of Islands: each unvisited land cell starts a DFS/BFS that marks its whole island.


### Common Mistakes

- Not starting from all nodes
- Recursive DFS stack overflow
- Forgetting to mark visited before recursing/enqueuing

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Number of Islands](https://leetcode.com/problems/number-of-islands/) | Medium |
| 2 | [Max Area of Island](https://leetcode.com/problems/max-area-of-island/) | Medium |
| 3 | [Number of Provinces](https://leetcode.com/problems/number-of-provinces/) | Medium |
| 4 | [Keys and Rooms](https://leetcode.com/problems/keys-and-rooms/) | Medium |
| 5 | [Find if Path Exists in Graph](https://leetcode.com/problems/find-if-path-exists-in-graph/) | Easy |
| 6 | [Surrounded Regions](https://leetcode.com/problems/surrounded-regions/) | Medium |
| 7 | [Pacific Atlantic Water Flow](https://leetcode.com/problems/pacific-atlantic-water-flow/) | Medium |

---

<a id="cycle-detection"></a>

## 44. Cycle Detection

**Category:** Graphs


### Explanation

Cycle detection depends on graph type. Directed graphs need recursion states; undirected graphs need parent tracking or Union-Find.


### When to Use

Use when validating trees, prerequisites, safe states, redundant edges, or detecting impossible dependency schedules.


### How to Solve

- For directed DFS, use colors: unvisited, visiting, visited.
- A back-edge to a visiting node means a cycle.
- For undirected DFS, ignore the edge back to parent.
- Union-Find detects cycles while adding undirected edges.
- Topological sort can also reveal directed cycles.

### Example

Course Schedule: if a prerequisite chain reaches a course already in the current DFS path, the schedule is impossible.


### Common Mistakes

- Using undirected parent logic on directed graph
- Not distinguishing visiting vs fully visited
- Forgetting disconnected components

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Course Schedule](https://leetcode.com/problems/course-schedule/) | Medium |
| 2 | [Find Eventual Safe States](https://leetcode.com/problems/find-eventual-safe-states/) | Medium |
| 3 | [Redundant Connection](https://leetcode.com/problems/redundant-connection/) | Medium |
| 4 | [Detect Cycles in 2D Grid](https://leetcode.com/problems/detect-cycles-in-2d-grid/) | Medium |
| 5 | [Graph Valid Tree](https://leetcode.com/problems/graph-valid-tree/) | Medium |
| 6 | [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) | Medium |

---

<a id="topological-sort"></a>

## 45. Topological Sort

**Category:** Graphs


### Explanation

Topological sort orders directed acyclic graph nodes so every prerequisite appears before dependent nodes. It is dependency scheduling.


### When to Use

Use for courses, build systems, recipes, alien dictionaries, prerequisite reachability, and DAG DP.


### How to Solve

- Build directed edges from prerequisite to dependent.
- Compute indegree of every node.
- Push all zero-indegree nodes into a queue.
- Pop, append to order, and reduce neighbors' indegree.
- If processed count < n, a cycle exists.

### Example

Course order: courses with no prerequisites can be taken first; removing them may unlock more courses.


### Common Mistakes

- Reversing edge direction
- Not adding isolated nodes
- Not detecting cycle when queue empties early

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Course Schedule](https://leetcode.com/problems/course-schedule/) | Medium |
| 2 | [Course Schedule II](https://leetcode.com/problems/course-schedule-ii/) | Medium |
| 3 | [Find All Possible Recipes from Given Supplies](https://leetcode.com/problems/find-all-possible-recipes-from-given-supplies/) | Medium |
| 4 | [Sort Items by Groups Respecting Dependencies](https://leetcode.com/problems/sort-items-by-groups-respecting-dependencies/) | Hard |
| 5 | [Course Schedule IV](https://leetcode.com/problems/course-schedule-iv/) | Medium |
| 6 | [Minimum Height Trees](https://leetcode.com/problems/minimum-height-trees/) | Medium |

---

<a id="union-find"></a>

## 46. Union-Find / Disjoint Set Union

**Category:** Graphs


### Explanation

Union-Find tracks dynamic connectivity. It supports near-constant-time `find` and `union` operations using path compression and union by rank/size.


### When to Use

Use for components, redundant connections, Kruskal MST, accounts merging, and connectivity as edges are added.


### How to Solve

- Initialize each node as its own parent.
- Use `find(x)` with path compression.
- Use `union(a,b)` to merge roots.
- Track component count or size when needed.
- For grid DSU, map `(r,c)` to `r * cols + c`.

### Example

When adding an edge `(u,v)`, if `find(u) == find(v)`, the edge forms a cycle.


### Common Mistakes

- No path compression causing slow operations
- Not handling string nodes with maps
- Incorrect component count update on already-connected nodes

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Redundant Connection](https://leetcode.com/problems/redundant-connection/) | Medium |
| 2 | [Number of Provinces](https://leetcode.com/problems/number-of-provinces/) | Medium |
| 3 | [Accounts Merge](https://leetcode.com/problems/accounts-merge/) | Medium |
| 4 | [Satisfiability of Equality Equations](https://leetcode.com/problems/satisfiability-of-equality-equations/) | Medium |
| 5 | [Most Stones Removed with Same Row or Column](https://leetcode.com/problems/most-stones-removed-with-same-row-or-column/) | Medium |
| 6 | [Number of Operations to Make Network Connected](https://leetcode.com/problems/number-of-operations-to-make-network-connected/) | Medium |
| 7 | [Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) | Medium |

---

<a id="mst"></a>

## 47. Minimum Spanning Tree: Kruskal and Prim

**Category:** Graphs


### Explanation

An MST connects all nodes with minimum total edge cost in an undirected weighted graph. Kruskal sorts edges and uses DSU; Prim grows the tree with a heap.


### When to Use

Use when the problem asks for minimum cost to connect all points/nodes with undirected edges.


### How to Solve

- Kruskal: sort edges by weight, add if endpoints are in different DSU components.
- Prim: start from any node, repeatedly add the cheapest edge to an unvisited node.
- Stop after `n - 1` edges.
- If not all nodes are connected, no MST exists.
- Do not use MST for shortest path; they solve different problems.

### Example

Connecting points by Manhattan distance: each possible edge has cost; choose edges that connect all points with minimum total cost.


### Common Mistakes

- Confusing MST with shortest path
- Adding edges that form cycles
- Building all O(n²) edges when constraints require optimization

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Min Cost to Connect All Points](https://leetcode.com/problems/min-cost-to-connect-all-points/) | Medium |
| 2 | [Connecting Cities With Minimum Cost](https://leetcode.com/problems/connecting-cities-with-minimum-cost/) | Medium |
| 3 | [Optimize Water Distribution in a Village](https://leetcode.com/problems/optimize-water-distribution-in-a-village/) | Hard |
| 4 | [Find Critical and Pseudo-Critical Edges in Minimum Spanning Tree](https://leetcode.com/problems/find-critical-and-pseudo-critical-edges-in-minimum-spanning-tree/) | Hard |
| 5 | [Redundant Connection](https://leetcode.com/problems/redundant-connection/) | Medium |

---

<a id="bfs-shortest-path"></a>

## 48. BFS Shortest Path in Unweighted Graphs

**Category:** Graphs


### Explanation

BFS gives shortest path when every edge has equal cost. The first time you reach a node, you reached it with the fewest edges.


### When to Use

Use for shortest number of moves, word transformations, lock rotations, grid steps, and unweighted state-space search.


### How to Solve

- Represent each state compactly.
- Start BFS from the initial state.
- Generate valid neighbors.
- Mark visited when enqueueing.
- Return distance when target is dequeued or first generated.

### Example

Word Ladder: each word is a node; edges connect words differing by one character. BFS finds minimum transformations.


### Common Mistakes

- Using DFS for shortest path
- Not marking states visited
- Generating neighbors too slowly for word problems

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Word Ladder](https://leetcode.com/problems/word-ladder/) | Hard |
| 2 | [Open the Lock](https://leetcode.com/problems/open-the-lock/) | Medium |
| 3 | [Shortest Path in Binary Matrix](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Medium |
| 4 | [Minimum Genetic Mutation](https://leetcode.com/problems/minimum-genetic-mutation/) | Medium |
| 5 | [Snakes and Ladders](https://leetcode.com/problems/snakes-and-ladders/) | Medium |
| 6 | [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | Medium |

---

<a id="multi-source-bfs"></a>

## 49. Multi-Source BFS

**Category:** Graphs


### Explanation

Multi-source BFS starts from all sources simultaneously. It finds nearest distance to any source and models spreading processes.


### When to Use

Use for rotting, fire spread, distance to nearest zero/gate, and problems where many origins expand at once.


### How to Solve

- Enqueue all initial sources with distance 0.
- Mark them visited immediately.
- Run normal BFS.
- Each level represents distance/time from the nearest source.
- If some target cells remain unvisited, handle impossible case.

### Example

01 Matrix: all zero cells are sources; BFS fills each one-cell with distance to nearest zero.


### Common Mistakes

- Running BFS separately from every cell
- Not enqueueing all sources before BFS starts
- Wrong minute count when last level has no new changes

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [01 Matrix](https://leetcode.com/problems/01-matrix/) | Medium |
| 2 | [Rotting Oranges](https://leetcode.com/problems/rotting-oranges/) | Medium |
| 3 | [Walls and Gates](https://leetcode.com/problems/walls-and-gates/) | Medium |
| 4 | [As Far from Land as Possible](https://leetcode.com/problems/as-far-from-land-as-possible/) | Medium |
| 5 | [Map of Highest Peak](https://leetcode.com/problems/map-of-highest-peak/) | Medium |
| 6 | [Shortest Bridge](https://leetcode.com/problems/shortest-bridge/) | Medium |

---

<a id="dijkstra"></a>

## 50. Dijkstra's Algorithm

**Category:** Graphs


### Explanation

Dijkstra finds shortest paths from a source in a graph with non-negative edge weights. It greedily expands the currently known closest node using a priority queue.


### When to Use

Use for weighted shortest path when all edge weights are non-negative.


### How to Solve

- Build adjacency list with `(neighbor, weight)`.
- Initialize `dist[source] = 0`, others infinity.
- Push `(0, source)` into a min-heap.
- Pop the smallest distance; skip stale heap entries.
- Relax outgoing edges: if `d + w < dist[v]`, update and push.

### Example

Network Delay Time: shortest time from source to every node is a classic single-source shortest path problem.


### Common Mistakes

- Using Dijkstra with negative weights
- Not skipping stale heap states
- Marking visited too early when using modified state

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Medium |
| 2 | [Path With Minimum Effort](https://leetcode.com/problems/path-with-minimum-effort/) | Medium |
| 3 | [Swim in Rising Water](https://leetcode.com/problems/swim-in-rising-water/) | Hard |
| 4 | [Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/) | Medium |
| 5 | [Number of Ways to Arrive at Destination](https://leetcode.com/problems/number-of-ways-to-arrive-at-destination/) | Medium |
| 6 | [Minimum Obstacle Removal to Reach Corner](https://leetcode.com/problems/minimum-obstacle-removal-to-reach-corner/) | Hard |
| 7 | [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Medium |

---

<a id="bellman-ford"></a>

## 51. Bellman-Ford Algorithm

**Category:** Graphs


### Explanation

Bellman-Ford finds shortest paths even when negative edge weights exist, as long as there is no reachable negative cycle. It relaxes every edge repeatedly.


### When to Use

Use for graphs with negative weights, path constraints by number of edges/stops, or when you need negative-cycle detection.


### How to Solve

- Keep an edge list `(u, v, w)`.
- Initialize distances with source as 0.
- Repeat relaxation `V - 1` times: update `dist[v]` from `dist[u] + w`.
- If an additional pass can still relax an edge, a negative cycle is reachable.
- For at-most-K-stops problems, use a copy of distances per iteration to prevent using too many edges.

### Example

Cheapest Flights Within K Stops: each iteration represents using one more flight edge. Copying the previous distance array enforces the stop limit.


### Common Mistakes

- Using in-place relaxation for K-stop constraints
- Running Dijkstra on negative weights
- Forgetting negative-cycle detection where required

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Cheapest Flights Within K Stops](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Medium |
| 2 | [Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Medium |
| 3 | [Path with Maximum Probability](https://leetcode.com/problems/path-with-maximum-probability/) | Medium |
| 4 | [Find the City With the Smallest Number of Neighbors at a Threshold Distance](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/) | Medium |
| 5 | [Minimum Cost to Make at Least One Valid Path in a Grid](https://leetcode.com/problems/minimum-cost-to-make-at-least-one-valid-path-in-a-grid/) | Hard |

---

<a id="floyd-warshall"></a>

## 52. Floyd-Warshall / All-Pairs Shortest Path

**Category:** Graphs


### Explanation

Floyd-Warshall computes shortest paths between every pair of nodes using dynamic programming over intermediate nodes. It works with negative edges but not negative cycles.


### When to Use

Use when `n` is small-to-medium and many all-pairs distance queries are needed.


### How to Solve

- Initialize `dist[i][j]` with direct edge weights and `dist[i][i] = 0`.
- For each intermediate `k`, try improving every `i -> j` via `k`.
- Transition: `dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`.
- Time complexity is O(n³), so check constraints.
- Can be adapted for transitive closure/reachability.

### Example

If going from city `i` to `j` through city `k` is cheaper than current known route, update the distance.


### Common Mistakes

- Using O(n³) when n is too large
- Not initializing unreachable distances to infinity
- Overflow when adding infinity

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Find the City With the Smallest Number of Neighbors at a Threshold Distance](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/) | Medium |
| 2 | [Course Schedule IV](https://leetcode.com/problems/course-schedule-iv/) | Medium |
| 3 | [Evaluate Division](https://leetcode.com/problems/evaluate-division/) | Medium |
| 4 | [Network Delay Time](https://leetcode.com/problems/network-delay-time/) | Medium |
| 5 | [Minimum Cost to Convert String I](https://leetcode.com/problems/minimum-cost-to-convert-string-i/) | Medium |
| 6 | [Design Graph With Shortest Path Calculator](https://leetcode.com/problems/design-graph-with-shortest-path-calculator/) | Hard |

---

<a id="zero-one-bfs"></a>

## 53. 0-1 BFS

**Category:** Graphs


### Explanation

0-1 BFS is shortest path for graphs whose edge weights are only 0 or 1. It uses a deque instead of a priority queue.


### When to Use

Use when moving along preferred direction costs 0/1, removing obstacles costs 1, or all weights are binary.


### How to Solve

- Use deque and distance array.
- For weight 0 edge, push neighbor to front.
- For weight 1 edge, push neighbor to back.
- Only relax when the new distance is smaller.
- This gives O(V + E) instead of heap-based O(E log V).

### Example

Minimum obstacle removal: moving into an empty cell costs 0; moving into an obstacle costs 1 and goes to the back of the deque.


### Common Mistakes

- Using normal BFS when weights differ
- Using Dijkstra when 0-1 BFS is simpler
- Marking visited instead of allowing distance relaxation

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Minimum Obstacle Removal to Reach Corner](https://leetcode.com/problems/minimum-obstacle-removal-to-reach-corner/) | Hard |
| 2 | [Minimum Cost to Make at Least One Valid Path in a Grid](https://leetcode.com/problems/minimum-cost-to-make-at-least-one-valid-path-in-a-grid/) | Hard |
| 3 | [Shortest Path in a Grid with Obstacles Elimination](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/) | Hard |
| 4 | [Sliding Puzzle](https://leetcode.com/problems/sliding-puzzle/) | Hard |
| 5 | [Open the Lock](https://leetcode.com/problems/open-the-lock/) | Medium |

---

<a id="tarjan-bridges-scc"></a>

## 54. Tarjan's Algorithm: Bridges and Strongly Connected Components

**Category:** Graphs


### Explanation

Tarjan-style DFS uses discovery time and low-link values to identify critical edges, articulation points, or strongly connected components.


### When to Use

Use when removing an edge/node changes connectivity, or when directed graph components are mutually reachable.


### How to Solve

- Track discovery time `disc[u]` and lowest reachable time `low[u]`.
- DFS through neighbors while ignoring the direct parent edge for bridges.
- After visiting child `v`, update `low[u] = min(low[u], low[v])`.
- If `low[v] > disc[u]`, edge `(u,v)` is a bridge.
- For SCC, use a stack and component extraction when root low-link is found.

### Example

A bridge is an edge where the child subtree cannot reach any ancestor of the current node without using that edge.


### Common Mistakes

- Incorrect parent-edge handling
- Not updating low-link on back edges
- Applying undirected bridge logic to directed SCC problems

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Critical Connections in a Network](https://leetcode.com/problems/critical-connections-in-a-network/) | Hard |
| 2 | [Number of Connected Components in an Undirected Graph](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) | Medium |
| 3 | [Redundant Connection](https://leetcode.com/problems/redundant-connection/) | Medium |
| 4 | [Find Critical and Pseudo-Critical Edges in Minimum Spanning Tree](https://leetcode.com/problems/find-critical-and-pseudo-critical-edges-in-minimum-spanning-tree/) | Hard |
| 5 | [Course Schedule](https://leetcode.com/problems/course-schedule/) | Medium |

---

<a id="bipartite-graph"></a>

## 55. Bipartite Graph / Graph Coloring

**Category:** Graphs


### Explanation

A graph is bipartite if nodes can be colored with two colors so every edge connects opposite colors. Odd cycles break bipartiteness.


### When to Use

Use for grouping conflicts, possible bipartition, two-team assignment, and parity constraints.


### How to Solve

- Initialize color for each unvisited component.
- BFS/DFS; assign opposite color to neighbors.
- If a neighbor already has the same color, return false.
- Process all components, not just node 0.
- Union-Find with enemy grouping can solve advanced variants.

### Example

Possible bipartition: people who dislike each other must be in opposite groups.


### Common Mistakes

- Ignoring disconnected components
- Using visited without color
- Not handling self-loops

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Is Graph Bipartite?](https://leetcode.com/problems/is-graph-bipartite/) | Medium |
| 2 | [Possible Bipartition](https://leetcode.com/problems/possible-bipartition/) | Medium |
| 3 | [Flower Planting With No Adjacent](https://leetcode.com/problems/flower-planting-with-no-adjacent/) | Medium |
| 4 | [Divide Nodes Into the Maximum Number of Groups](https://leetcode.com/problems/divide-nodes-into-the-maximum-number-of-groups/) | Hard |
| 5 | [Satisfiability of Equality Equations](https://leetcode.com/problems/satisfiability-of-equality-equations/) | Medium |

---

<a id="eulerian-path"></a>

## 56. Eulerian Path and Hierholzer's Algorithm

**Category:** Graphs


### Explanation

An Eulerian path uses every edge exactly once. Hierholzer's algorithm builds the path by DFS-consuming edges and appending nodes after exhausting outgoing edges.


### When to Use

Use for itinerary reconstruction, using all tickets/edges exactly once, and path reconstruction with lexical order constraints.


### How to Solve

- Build adjacency list of outgoing edges.
- Sort destinations reverse if using stack/pop to get lexical order.
- DFS while edges remain from current node.
- Append node to route after consuming all outgoing edges.
- Reverse the final postorder route.

### Example

Reconstruct Itinerary: every ticket must be used once, and lexical order decides which outgoing edge to consume first.


### Common Mistakes

- Treating it as shortest path
- Appending before consuming all edges
- Not sorting for lexical constraints

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Reconstruct Itinerary](https://leetcode.com/problems/reconstruct-itinerary/) | Hard |
| 2 | [Valid Arrangement of Pairs](https://leetcode.com/problems/valid-arrangement-of-pairs/) | Hard |
| 3 | [Cracking the Safe](https://leetcode.com/problems/cracking-the-safe/) | Hard |
| 4 | [Find the Town Judge](https://leetcode.com/problems/find-the-town-judge/) | Easy |
| 5 | [All Paths From Source to Target](https://leetcode.com/problems/all-paths-from-source-to-target/) | Medium |

---

<a id="dp-1d"></a>

## 57. 1D Dynamic Programming

**Category:** Dynamic Programming


### Explanation

1D DP solves problems where the answer for index `i` depends on previous indices. The core task is defining `dp[i]` precisely.


### When to Use

Use for climbing, decoding, robbing, segmenting strings, and linear recurrence problems.


### How to Solve

- Define `dp[i]` in words before coding.
- Identify base cases.
- Write transition from smaller states to larger states.
- Optimize space if only recent states are needed.
- Check whether `i` means position, prefix length, or last chosen index.

### Example

Climbing stairs: `dp[i] = dp[i-1] + dp[i-2]` because the last move was either 1 or 2 steps.


### Common Mistakes

- Vague DP state
- Wrong base case
- Confusing index with prefix length

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Climbing Stairs](https://leetcode.com/problems/climbing-stairs/) | Easy |
| 2 | [Min Cost Climbing Stairs](https://leetcode.com/problems/min-cost-climbing-stairs/) | Easy |
| 3 | [House Robber](https://leetcode.com/problems/house-robber/) | Medium |
| 4 | [Decode Ways](https://leetcode.com/problems/decode-ways/) | Medium |
| 5 | [Word Break](https://leetcode.com/problems/word-break/) | Medium |
| 6 | [Perfect Squares](https://leetcode.com/problems/perfect-squares/) | Medium |
| 7 | [Integer Break](https://leetcode.com/problems/integer-break/) | Medium |

---

<a id="dp-knapsack"></a>

## 58. Knapsack DP: 0/1 and Unbounded

**Category:** Dynamic Programming


### Explanation

Knapsack DP chooses items under capacity constraints. 0/1 knapsack allows each item once; unbounded knapsack allows reuse.


### When to Use

Use when decisions involve picking/not picking items with weights, values, sums, or capacities.


### How to Solve

- Define `dp[c]` as best value/possibility for capacity or sum `c`.
- For 0/1 knapsack, iterate capacity backward to avoid reusing the same item.
- For unbounded knapsack, iterate capacity forward to allow reuse.
- For counting combinations, order of loops controls duplicate counting.
- Use boolean DP for possible sums.

### Example

Coin Change is unbounded: each coin can be reused, so update capacities forward.


### Common Mistakes

- Wrong loop direction
- Counting permutations when combinations are required
- Not initializing impossible states correctly

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Coin Change](https://leetcode.com/problems/coin-change/) | Medium |
| 2 | [Coin Change II](https://leetcode.com/problems/coin-change-ii/) | Medium |
| 3 | [Partition Equal Subset Sum](https://leetcode.com/problems/partition-equal-subset-sum/) | Medium |
| 4 | [Target Sum](https://leetcode.com/problems/target-sum/) | Medium |
| 5 | [Last Stone Weight II](https://leetcode.com/problems/last-stone-weight-ii/) | Medium |
| 6 | [Ones and Zeroes](https://leetcode.com/problems/ones-and-zeroes/) | Medium |
| 7 | [Combination Sum IV](https://leetcode.com/problems/combination-sum-iv/) | Medium |

---

<a id="dp-lis"></a>

## 59. Longest Increasing Subsequence

**Category:** Dynamic Programming


### Explanation

LIS asks for the longest ordered subsequence satisfying increasing constraints. It has an O(n²) DP and an O(n log n) patience-sorting method.


### When to Use

Use when maintaining relative order but not necessarily contiguity, often with increasing/decreasing conditions.


### How to Solve

- O(n²): `dp[i]` is LIS ending at `i`; check all `j < i`.
- O(n log n): maintain `tails[len]`, the smallest possible tail for a subsequence length.
- Use binary search to replace first tail >= current value.
- For reconstruction, store predecessors.
- For 2D variants, sort one dimension and run LIS on the other.

### Example

For `[10,9,2,5,3,7,101,18]`, LIS length is 4, e.g. `[2,3,7,18]`.


### Common Mistakes

- Confusing subsequence with subarray
- Incorrect sorting in Russian Doll Envelopes
- Using `>` vs `>=` incorrectly for strictness

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Longest Increasing Subsequence](https://leetcode.com/problems/longest-increasing-subsequence/) | Medium |
| 2 | [Number of Longest Increasing Subsequence](https://leetcode.com/problems/number-of-longest-increasing-subsequence/) | Medium |
| 3 | [Russian Doll Envelopes](https://leetcode.com/problems/russian-doll-envelopes/) | Hard |
| 4 | [Maximum Length of Pair Chain](https://leetcode.com/problems/maximum-length-of-pair-chain/) | Medium |
| 5 | [Longest String Chain](https://leetcode.com/problems/longest-string-chain/) | Medium |
| 6 | [Increasing Triplet Subsequence](https://leetcode.com/problems/increasing-triplet-subsequence/) | Medium |

---

<a id="dp-lcs-edit-distance"></a>

## 60. LCS, Edit Distance, and Two-String DP

**Category:** Dynamic Programming


### Explanation

Two-string DP compares prefixes of two strings. The state usually means the best answer for `text1[:i]` and `text2[:j]`.


### When to Use

Use for common subsequence, edit distance, interleaving, matching, deletion distance, and wildcard/regex matching.


### How to Solve

- Define `dp[i][j]` using prefix lengths, not raw indices.
- If characters match, transition diagonally.
- If they do not match, use insert/delete/replace/skip transitions depending on problem.
- Initialize first row/column for empty prefixes.
- Optimize space to two rows if only previous row is needed.

### Example

LCS: if `a[i-1] == b[j-1]`, then `dp[i][j] = 1 + dp[i-1][j-1]`; else take max of skipping one char.


### Common Mistakes

- Wrong empty-prefix initialization
- Confusing substring with subsequence
- Off-by-one in character access

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Longest Common Subsequence](https://leetcode.com/problems/longest-common-subsequence/) | Medium |
| 2 | [Edit Distance](https://leetcode.com/problems/edit-distance/) | Medium |
| 3 | [Delete Operation for Two Strings](https://leetcode.com/problems/delete-operation-for-two-strings/) | Medium |
| 4 | [Interleaving String](https://leetcode.com/problems/interleaving-string/) | Medium |
| 5 | [Distinct Subsequences](https://leetcode.com/problems/distinct-subsequences/) | Hard |
| 6 | [Regular Expression Matching](https://leetcode.com/problems/regular-expression-matching/) | Hard |
| 7 | [Wildcard Matching](https://leetcode.com/problems/wildcard-matching/) | Hard |

---

<a id="dp-grid"></a>

## 61. Grid DP

**Category:** Dynamic Programming


### Explanation

Grid DP computes optimal/count values cell by cell, often from top/left or all valid previous directions.


### When to Use

Use for path counts, min path sum, obstacles, dungeon health, falling paths, and matrix-based recurrence.


### How to Solve

- Define `dp[r][c]` as best/count ways to reach or leave cell `(r,c)`.
- Initialize first row/column carefully.
- Transition from allowed previous cells.
- For obstacles, zero out blocked states.
- For memory optimization, use one row if only previous row is needed.

### Example

Unique Paths: `dp[r][c] = dp[r-1][c] + dp[r][c-1]`.


### Common Mistakes

- Wrong obstacle initialization
- Forgetting boundaries
- Using DFS without memoization and getting exponential time

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Unique Paths](https://leetcode.com/problems/unique-paths/) | Medium |
| 2 | [Unique Paths II](https://leetcode.com/problems/unique-paths-ii/) | Medium |
| 3 | [Minimum Path Sum](https://leetcode.com/problems/minimum-path-sum/) | Medium |
| 4 | [Dungeon Game](https://leetcode.com/problems/dungeon-game/) | Hard |
| 5 | [Minimum Falling Path Sum](https://leetcode.com/problems/minimum-falling-path-sum/) | Medium |
| 6 | [Cherry Pickup II](https://leetcode.com/problems/cherry-pickup-ii/) | Hard |
| 7 | [Triangle](https://leetcode.com/problems/triangle/) | Medium |

---

<a id="dp-palindrome-partition"></a>

## 62. Palindrome DP and Partition DP

**Category:** Dynamic Programming


### Explanation

Palindrome DP precomputes whether substrings are palindromes. Partition DP then uses those valid substrings to minimize cuts or generate partitions.


### When to Use

Use when the problem asks about palindromic substrings, minimum cuts, or partitioning a string into valid pieces.


### How to Solve

- Build `isPal[l][r]` from smaller substrings.
- Single characters are palindromes; length-2 depends on equality.
- For partitioning, let `dp[i]` be best answer for prefix ending at `i`.
- Try every previous cut `j` where `s[j:i]` is valid.
- For enumeration, combine backtracking with palindrome precheck.

### Example

Minimum cuts: if `s[j:i]` is palindrome, `dp[i] = min(dp[i], dp[j] + 1)` depending on indexing.


### Common Mistakes

- Checking palindrome repeatedly O(n³)
- Bad base case for zero cuts
- Confusing cut count with partition count

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Palindromic Substrings](https://leetcode.com/problems/palindromic-substrings/) | Medium |
| 2 | [Longest Palindromic Substring](https://leetcode.com/problems/longest-palindromic-substring/) | Medium |
| 3 | [Longest Palindromic Subsequence](https://leetcode.com/problems/longest-palindromic-subsequence/) | Medium |
| 4 | [Palindrome Partitioning](https://leetcode.com/problems/palindrome-partitioning/) | Medium |
| 5 | [Palindrome Partitioning II](https://leetcode.com/problems/palindrome-partitioning-ii/) | Hard |
| 6 | [Minimum Insertion Steps to Make a String Palindrome](https://leetcode.com/problems/minimum-insertion-steps-to-make-a-string-palindrome/) | Hard |

---

<a id="dp-interval"></a>

## 63. Interval DP

**Category:** Dynamic Programming


### Explanation

Interval DP solves problems where the answer for a range depends on splitting that range into smaller ranges or choosing boundaries.


### When to Use

Use for burst balloons, matrix-chain-like problems, polygon triangulation, merging stones, and optimal game strategies.


### How to Solve

- Define `dp[l][r]` for interval `[l, r]`.
- Solve smaller intervals before larger intervals.
- Try each possible split or last action inside the interval.
- Add boundary contribution if choosing an element last.
- Be careful with open vs closed interval definitions.

### Example

Burst Balloons: choose which balloon bursts last in an interval, because its neighbors are then known.


### Common Mistakes

- Choosing first action when last action simplifies the state
- Wrong interval length iteration
- Boundary padding mistakes

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Burst Balloons](https://leetcode.com/problems/burst-balloons/) | Hard |
| 2 | [Minimum Score Triangulation of Polygon](https://leetcode.com/problems/minimum-score-triangulation-of-polygon/) | Medium |
| 3 | [Stone Game](https://leetcode.com/problems/stone-game/) | Medium |
| 4 | [Stone Game II](https://leetcode.com/problems/stone-game-ii/) | Medium |
| 5 | [Predict the Winner](https://leetcode.com/problems/predict-the-winner/) | Medium |
| 6 | [Minimum Cost to Cut a Stick](https://leetcode.com/problems/minimum-cost-to-cut-a-stick/) | Hard |
| 7 | [Strange Printer](https://leetcode.com/problems/strange-printer/) | Hard |

---

<a id="dp-bitmask"></a>

## 64. Bitmask DP

**Category:** Dynamic Programming


### Explanation

Bitmask DP represents a subset of items as bits in an integer. It is useful when `n` is small but choices over subsets are exponential.


### When to Use

Use for traveling-salesman-like problems, assignment, visiting all nodes, compatibility matching, and subset states.


### How to Solve

- Represent selected items as `mask`.
- Use bit operations to test/add/remove items.
- Define state like `dp[mask][last]` or `dp[mask]`.
- Transition by adding one unvisited item.
- Check constraints: usually n <= 20, often much smaller.

### Example

Shortest Path Visiting All Nodes: BFS/DP state is `(node, visited_mask)`.


### Common Mistakes

- Using bitmask when n is too large
- Forgetting that `1 << n` grows exponentially
- Incorrect bit tests

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Shortest Path Visiting All Nodes](https://leetcode.com/problems/shortest-path-visiting-all-nodes/) | Hard |
| 2 | [Partition to K Equal Sum Subsets](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/) | Medium |
| 3 | [Maximum Compatibility Score Sum](https://leetcode.com/problems/maximum-compatibility-score-sum/) | Medium |
| 4 | [Minimum XOR Sum of Two Arrays](https://leetcode.com/problems/minimum-xor-sum-of-two-arrays/) | Hard |
| 5 | [Can I Win](https://leetcode.com/problems/can-i-win/) | Medium |
| 6 | [Number of Ways to Wear Different Hats to Each Other](https://leetcode.com/problems/number-of-ways-to-wear-different-hats-to-each-other/) | Hard |

---

<a id="greedy-core"></a>

## 65. Greedy Algorithms

**Category:** Greedy


### Explanation

Greedy algorithms make locally optimal choices that can be proven globally optimal. The interview burden is proof: why does the local choice not block a better future?


### When to Use

Use when sorting plus local choice, earliest finish, farthest reach, smallest/largest available item, or exchange argument applies.


### How to Solve

- Identify the greedy choice.
- Sort or heapify so the best local candidate is available.
- Prove with exchange argument or invariant.
- Track only the state needed to validate future choices.
- If local choice affects future in complex ways, consider DP instead.

### Example

Jump Game: track the farthest reachable index. If current index exceeds it, you are stuck.


### Common Mistakes

- Using greedy where DP is required
- No proof of correctness
- Sorting by wrong criterion

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Jump Game](https://leetcode.com/problems/jump-game/) | Medium |
| 2 | [Jump Game II](https://leetcode.com/problems/jump-game-ii/) | Medium |
| 3 | [Gas Station](https://leetcode.com/problems/gas-station/) | Medium |
| 4 | [Candy](https://leetcode.com/problems/candy/) | Hard |
| 5 | [Partition Labels](https://leetcode.com/problems/partition-labels/) | Medium |
| 6 | [Queue Reconstruction by Height](https://leetcode.com/problems/queue-reconstruction-by-height/) | Medium |
| 7 | [Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Medium |

---

<a id="greedy-intervals"></a>

## 66. Greedy Intervals and Scheduling

**Category:** Greedy


### Explanation

Interval scheduling greedy choices often rely on earliest finishing time, minimum overlap, or maximizing remaining room.


### When to Use

Use when selecting/removing intervals, scheduling meetings, shooting arrows, or assigning resources.


### How to Solve

- For maximum non-overlapping intervals, sort by end time.
- For merging, sort by start time.
- For minimum removals, keep interval with smaller end when overlap occurs.
- For resource counts, use heap or line sweep.
- Validate endpoint inclusivity.

### Example

Non-overlapping intervals: when two intervals overlap, remove the one with the later end because it restricts future choices more.


### Common Mistakes

- Sorting by start for selection problems
- Endpoint convention mistakes
- Not considering nested intervals

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium |
| 2 | [Minimum Number of Arrows to Burst Balloons](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) | Medium |
| 3 | [Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium |
| 4 | [Meeting Rooms III](https://leetcode.com/problems/meeting-rooms-iii/) | Hard |
| 5 | [Remove Covered Intervals](https://leetcode.com/problems/remove-covered-intervals/) | Medium |
| 6 | [Maximum Number of Events That Can Be Attended](https://leetcode.com/problems/maximum-number-of-events-that-can-be-attended/) | Medium |

---

<a id="bit-manipulation"></a>

## 67. Bit Manipulation Basics

**Category:** Bit Manipulation


### Explanation

Bit manipulation treats integers as binary sets or flags. It is useful for parity, uniqueness, subsets, masks, and low-level optimizations.


### When to Use

Use when constraints mention bits, XOR, powers of two, subsets, masks, or O(1) memory tricks.


### How to Solve

- Use `x & 1` for parity and `x >> 1` to shift.
- Use `x & (x - 1)` to remove the lowest set bit.
- Use XOR for canceling equal values.
- Use `mask | (1 << i)` to set a bit.
- Use `mask & (1 << i)` to test a bit.

### Example

`a ^ a = 0` and `a ^ 0 = a`, so XOR of all numbers finds the unique number when every other number appears twice.


### Common Mistakes

- Operator precedence bugs
- Signed integer edge cases
- Confusing bit index with value

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Single Number](https://leetcode.com/problems/single-number/) | Easy |
| 2 | [Number of 1 Bits](https://leetcode.com/problems/number-of-1-bits/) | Easy |
| 3 | [Counting Bits](https://leetcode.com/problems/counting-bits/) | Easy |
| 4 | [Reverse Bits](https://leetcode.com/problems/reverse-bits/) | Easy |
| 5 | [Power of Two](https://leetcode.com/problems/power-of-two/) | Easy |
| 6 | [Bitwise AND of Numbers Range](https://leetcode.com/problems/bitwise-and-of-numbers-range/) | Medium |
| 7 | [Sum of Two Integers](https://leetcode.com/problems/sum-of-two-integers/) | Medium |

---

<a id="bitmask-subsets"></a>

## 68. Bitmask Subsets

**Category:** Bit Manipulation


### Explanation

Every subset of `n` items can be represented by an `n`-bit mask. Enumerating masks is a compact alternative to recursive subset generation.


### When to Use

Use when n is small and you need to iterate all subsets or store visited combinations.


### How to Solve

- Loop `mask` from `0` to `(1 << n) - 1`.
- For each bit `i`, check if selected using `mask & (1 << i)`.
- Use submask enumeration: `sub = mask; sub = (sub - 1) & mask`.
- Combine with DP when subset results build on smaller subsets.
- Avoid if n is large.

### Example

For `n=3`, mask `101` means item 0 and item 2 are selected.


### Common Mistakes

- Exponential blowup
- Off-by-one in mask range
- Not handling empty subset

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Subsets](https://leetcode.com/problems/subsets/) | Medium |
| 2 | [Subsets II](https://leetcode.com/problems/subsets-ii/) | Medium |
| 3 | [Partition to K Equal Sum Subsets](https://leetcode.com/problems/partition-to-k-equal-sum-subsets/) | Medium |
| 4 | [Maximum Product of Word Lengths](https://leetcode.com/problems/maximum-product-of-word-lengths/) | Medium |
| 5 | [Beautiful Arrangement](https://leetcode.com/problems/beautiful-arrangement/) | Medium |
| 6 | [Number of Squareful Arrays](https://leetcode.com/problems/number-of-squareful-arrays/) | Hard |

---

<a id="math-number-theory"></a>

## 69. Math, GCD, Sieve, and Modular Arithmetic

**Category:** Math


### Explanation

Math problems test whether you can reduce computation using number properties. Common tools include GCD, modulo, prime sieve, fast exponentiation, and combinatorics.


### When to Use

Use when brute-force simulation is too slow and constraints involve divisibility, primes, remainders, or large counts.


### How to Solve

- Use Euclid's algorithm for GCD.
- Use modulo carefully after every addition/multiplication when counts are large.
- Use sieve for many prime queries up to N.
- Use fast exponentiation for powers.
- Look for invariants such as parity, gcd, remainder classes, or monotonic formulas.

### Example

GCD by Euclid: `gcd(a,b) = gcd(b, a % b)` until `b = 0`.


### Common Mistakes

- Overflow before modulo
- Using primality test repeatedly instead of sieve
- Ignoring zero/negative edge cases

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Count Primes](https://leetcode.com/problems/count-primes/) | Medium |
| 2 | [Pow(x, n)](https://leetcode.com/problems/powx-n/) | Medium |
| 3 | [Excel Sheet Column Number](https://leetcode.com/problems/excel-sheet-column-number/) | Easy |
| 4 | [Happy Number](https://leetcode.com/problems/happy-number/) | Easy |
| 5 | [Ugly Number](https://leetcode.com/problems/ugly-number/) | Easy |
| 6 | [Ugly Number II](https://leetcode.com/problems/ugly-number-ii/) | Medium |
| 7 | [Greatest Common Divisor of Strings](https://leetcode.com/problems/greatest-common-divisor-of-strings/) | Easy |

---

<a id="fenwick-tree"></a>

## 70. Fenwick Tree / Binary Indexed Tree

**Category:** Advanced Data Structures


### Explanation

A Fenwick tree supports prefix sums and point updates in O(log n) with compact code. It is commonly used for frequency tables and inversion counting.


### When to Use

Use when you need dynamic prefix sums, order counts, rank queries, or count of smaller elements.


### How to Solve

- Use 1-indexed tree array.
- Update index `i`: while `i <= n`, add delta and `i += i & -i`.
- Query prefix `i`: accumulate while `i > 0`, then `i -= i & -i`.
- For arbitrary values, coordinate-compress first.
- Range sum is `query(r) - query(l-1)`.

### Example

Count smaller after self: scan from right, query how many compressed values smaller than current have appeared, then update current value count.


### Common Mistakes

- Forgetting coordinate compression
- 0-index/1-index confusion
- Using Fenwick for non-invertible operations

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Range Sum Query - Mutable](https://leetcode.com/problems/range-sum-query-mutable/) | Medium |
| 2 | [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) | Hard |
| 3 | [Reverse Pairs](https://leetcode.com/problems/reverse-pairs/) | Hard |
| 4 | [Create Sorted Array through Instructions](https://leetcode.com/problems/create-sorted-array-through-instructions/) | Hard |
| 5 | [Count of Range Sum](https://leetcode.com/problems/count-of-range-sum/) | Hard |

---

<a id="segment-tree"></a>

## 71. Segment Tree

**Category:** Advanced Data Structures


### Explanation

A segment tree answers range queries and updates over intervals in O(log n). It is more flexible than Fenwick because it can support min/max/gcd/custom aggregates and lazy propagation.


### When to Use

Use for dynamic range min/max/sum queries, interval updates, booking conflicts, and sweep-line compression.


### How to Solve

- Build tree over an array or compressed coordinate range.
- Each node stores aggregate information for its interval.
- Point update changes one leaf and recomputes ancestors.
- Range query combines relevant node aggregates.
- Use lazy propagation for range updates.

### Example

Range Sum Query Mutable: update one index, then query sum over `[l,r]` by combining segment intervals.


### Common Mistakes

- Overengineering when prefix sums are enough
- Lazy propagation bugs
- Not coordinate-compressing huge ranges

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Range Sum Query - Mutable](https://leetcode.com/problems/range-sum-query-mutable/) | Medium |
| 2 | [My Calendar I](https://leetcode.com/problems/my-calendar-i/) | Medium |
| 3 | [My Calendar II](https://leetcode.com/problems/my-calendar-ii/) | Medium |
| 4 | [The Skyline Problem](https://leetcode.com/problems/the-skyline-problem/) | Hard |
| 5 | [Falling Squares](https://leetcode.com/problems/falling-squares/) | Hard |
| 6 | [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) | Hard |

---

<a id="ordered-set-treemap"></a>

## 72. Ordered Set / TreeMap

**Category:** Advanced Data Structures


### Explanation

Ordered sets/maps keep keys sorted and support floor/ceiling/range operations. In Java this is TreeMap/TreeSet; in C++ set/map; in Python you often need `bisect` with sorted lists or a third-party sorted container.


### When to Use

Use when you need nearest smaller/larger value, interval lookup, calendar booking, or dynamic sorted order.


### How to Solve

- Use floor/ceiling to find neighboring intervals or values.
- For calendars, check previous interval end and next interval start.
- For sliding constraints, keep active window values ordered.
- Remove elements as they leave the window.
- If language lacks ordered set, use heap, bisect, or Fenwick after coordinate compression.

### Example

Contains Duplicate III: maintain a sliding ordered set and check whether any value lies within `[x-k, x+k]`.


### Common Mistakes

- Using hash set when order is required
- Not removing stale window elements
- Assuming Python set is ordered

### Practice Problems

| # | Problem | Difficulty |
|---:|---|---|
| 1 | [Contains Duplicate III](https://leetcode.com/problems/contains-duplicate-iii/) | Hard |
| 2 | [My Calendar I](https://leetcode.com/problems/my-calendar-i/) | Medium |
| 3 | [My Calendar II](https://leetcode.com/problems/my-calendar-ii/) | Medium |
| 4 | [My Calendar III](https://leetcode.com/problems/my-calendar-iii/) | Hard |
| 5 | [Count of Range Sum](https://leetcode.com/problems/count-of-range-sum/) | Hard |
| 6 | [Snapshot Array](https://leetcode.com/problems/snapshot-array/) | Medium |

---


## Recommended Study Order

### Phase 1: Core Coding Patterns
1. Array traversal and hash maps
2. Prefix sums
3. Two pointers
4. Sliding window
5. Stack and queue
6. Binary search
7. Linked list basics

### Phase 2: Tree and Graph Foundations
1. Tree traversal
2. Tree height/path/BST/LCA
3. Graph BFS/DFS
4. Connected components
5. Cycle detection
6. Topological sort
7. Union-Find

### Phase 3: High-Value Interview Algorithms
1. Dijkstra
2. Bellman-Ford
3. Floyd-Warshall
4. MST
5. Backtracking
6. Greedy
7. 1D/2D DP

### Phase 4: Advanced Patterns
1. Interval DP
2. Bitmask DP
3. Trie + Word Search
4. Fenwick Tree
5. Segment Tree
6. Tarjan bridges/SCC
7. Rolling hash / KMP

## What to Memorize vs Understand

### Memorize
- Binary search boundary templates.
- DFS/BFS visited handling.
- Dijkstra heap relaxation.
- Union-Find `find` with path compression.
- Prefix sum and difference array formulas.
- Common DP loop directions for 0/1 vs unbounded knapsack.

### Understand
- Why two pointers are valid only with monotonic movement.
- Why BFS gives shortest path only for equal-weight edges.
- Why Dijkstra fails with negative weights.
- Why Bellman-Ford uses `V - 1` relaxations.
- Why topological sort only works on DAGs.
- Why greedy needs a proof, not just intuition.
- How to define DP state before writing transitions.

## Final Interview Checklist

Before submitting a solution, verify:

- Does the algorithm match the constraints?
- Did you handle empty/single-element input?
- Did you handle duplicates?
- Did you handle negative values if relevant?
- Are graph edges directed or undirected?
- Are weights negative, zero, one, or arbitrary non-negative?
- Did you mark visited at the correct time?
- Is your binary search invariant clear?
- Does your DP state have correct base cases?
- Can you explain your time and space complexity in one sentence?


## Source and Practice Reference Notes

- Practice links point to LeetCode problem pages using stable problem slugs where available.

- The roadmap is curated around common coding-interview patterns, not around a single LeetCode study plan.

- For graph shortest-path revision, compare the algorithm choice: BFS for unweighted graphs, Dijkstra for non-negative weighted graphs, Bellman-Ford for negative weights or edge-count-limited relaxation, Floyd-Warshall for all-pairs shortest paths with smaller `n`.
