# CSCI 2275 - Final Project: Building Data Structures Out of Other Data Structures

## Overview
Every data structure is defined by an **interface**: the operations it supports. How those operations are **implemented** is a separate choice. In this project, you will compare native and alternative implementations of data structure interfaces:

1. **Natively**, the way they are meant to be built.
2. **Crossed**, with one structure built *on top of the other*, according to the solo/pair requirements below. For example, you might build a priority queue using a BST, or an ordered set using a heap.

You will then **predict** how each version should perform, **benchmark** both under several workloads, and explain where your predictions matched reality and where they didn't.

The goal is to understand *how* data structures work: trace their operations, explain how they preserve their invariants, and connect implementation choices to operation costs. An alternative implementation may be useful for some workloads and costly for others.

You may work **solo or in pairs**. If you work in pairs, please notify me of your pair through Piazza by Friday 11/13.

---

## Learning Objectives
By the end of this project, you will be able to:
1. **Separate interface from implementation**, using abstract classes in C++.
2. **Implement** core data structures correctly, with attention to edge cases.
3. **Predict** operation costs (Big-O) for a data structure under a specific workload.
4. **Benchmark empirically**, with sound methodology, and interpret results against asymptotic predictions.
5. **Explain** when a data structure shines, when it fails, and why.
6. **Communicate** design trade-offs in a demo presentation and individually explain an operation using your own code.

---

## Allowed Tech
- Language: C++ (C++17 or later).
- You must provide simple build/run instructions (a Makefile).
- **No STL containers or container adapters** (`vector`, `list`, `deque`, `stack`, `queue`, `priority_queue`, `set`, `map`, `unordered_*`, etc.). You build everything yourself. This is especially important here, because the crossed versions must use *your own* native classes.
- Allowed STL: `<iostream>`, `<fstream>`, `<string>`, `<chrono>`, `<random>`, `<cassert>`, `<stdexcept>`, `<memory>`, and similar non-container utilities. If you are unsure whether a header is allowed, ask on Piazza.
- You may use any language or tool (Python, Excel, etc.) **for plotting only**.

---

## The Core Rule: Crossed Implementations Use Only the Public Interface
A crossed implementation must store all of its data inside an instance of the *other* native class, and it may only call that class's **public methods**.

For example, `ListOnArray` holds a `DynamicArray` member and may only call `get`, `set`, `pushBack`, and the other public `DynamicArray` methods. It may **not** reach into the array's internal buffer.

This rule is what makes the inefficiencies real. If you find yourself wishing you could peek inside the other class, write down why. That observation belongs in your report.

The adherence to this rule will be verified by offline code review by the professor.

---

## Required Operation Behavior
Native and crossed versions of the same interface must have identical observable behavior. Include tests for the applicable cases below.

| Operation or case | Required behavior |
|---|---|
| Initial state and size | Structures start empty. `size()` and `numVertices()` return the current count; queries do not change it. |
| Empty removal or peek | `peekMin`, `extractMin`, `min`, `max`, `popBack`, `popFront`, `pop`, `top`, `dequeue`, and `front` throw `std::out_of_range` when empty. |
| Indexed access/removal | Indices are zero-based. `get(i)`, `set(i, value)`, and `removeAt(i)` require `0 <= i < size()`; otherwise throw `std::out_of_range`. |
| Indexed insertion | `insertAt(pos, value)` inserts before the current element at `pos`. Valid positions are `0 <= pos <= size()`; `pos == size()` appends. Otherwise throw `std::out_of_range`. |
| Sequence, list, stack, queue values | Duplicate and negative integer values are allowed. Sequences/lists preserve logical order; stacks are LIFO and queues are FIFO. Removal returns the removed value. |
| Ordered set | Store distinct integer keys. `insert` returns `true` only for a new key; duplicate insertion returns `false` without changing the set. `remove` returns `true` only when a key is removed. `contains` returns `false` for an absent key, including in an empty set. |
| Priority queue (project-specific convention) | Store distinct integer keys, matching the provided `OrderedSet` interface. Inserting an existing key is a no-op in both native and crossed versions. `peekMin` returns the smallest key without removing it; `extractMin` removes and returns it. State this convention in your README. |
| Traversal | `inOrder` visits set keys in ascending order; `forEach` visits list elements in logical order. Empty traversal makes no callback calls. Callbacks must be non-null (otherwise throw `std::invalid_argument`) and must not modify the structure being traversed. |
| Graph vertices | `addVertex()` returns the previous vertex count as the new ID. Valid IDs are `0..numVertices()-1`. Invalid IDs, including BFS/Dijkstra sources, cause `std::out_of_range`. Adding a vertex preserves existing edges. |
| Graph edges | Edges are directed; self-loops are allowed. At most one edge exists per ordered pair `(u, v)`. `addEdge` inserts an edge or replaces its weight. Weights must be nonnegative, including zero; negative weights cause `std::invalid_argument`. Represent an absent edge separately from a zero-weight edge. |
| Graph queries/removal | For valid IDs, `hasEdge` returns whether the edge exists. `removeEdge` returns `true` only if it removes an existing edge; otherwise it returns `false`. |
| Neighbor traversal | `forEachNeighbor` visits each outgoing neighbor and its weight once. Order is unspecified; an isolated vertex makes no callback calls. Apply the same callback rules as other traversals. BFS visit order may therefore differ between representations. |
| Graph algorithm results | BFS identifies all vertices reachable from the source; Dijkstra computes shortest-path distances with source distance zero and an explicitly documented unreachable marker. Document how results are returned or exposed and use the same convention for every representation. |

Invalid arguments must leave the logical contents unchanged. If a call has both an invalid vertex and an invalid weight or callback, validate vertex IDs first.

**Copies and temporary storage:** You may use additional instances of your own underlying native class and scalar bookkeeping. Copies must be independent deep copies exposed through a public copy constructor or method; never access private storage from a crossed class. A `const` query may inspect a temporary copy but must not modify the original. Do not maintain a second persistent representation or use raw auxiliary arrays to bypass the public-interface rule. Include copying, temporary storage, and duplicate checks in your cost analysis and timed operations.

---

## Project Tracks
Choose **one** track. Interface headers are provided in `include/`. For Tracks 1, 2, and 4, implement both native classes and the crossed implementation(s) required below. Track 3 is an exception: compare graph representations using the single `Graph` interface and follow its track-specific solo/pair requirements.

| Workload | Solo | Pair |
|---|---|---|
| Native implementations | Both | Both |
| Crossed implementations | **One** (your choice of direction) | **Both** directions |

Track 4 is solo-only and follows the solo column. Track 3 follows its separate requirements below.

### Track 1: Heap ↔ BST
```cpp
class PriorityQueue {            // min-priority queue of ints
public:
    void insert(int key);
    int  peekMin() const;
    int  extractMin();
    int  size() const;
    ~PriorityQueue();
};

class OrderedSet {               // set of distinct ints, kept in order
public:
    bool insert(int key);     // false if already present
    bool contains(int key) const;
    bool remove(int key);     // false if absent
    int  min() const;
    int  max() const;
    void inOrder(void (*visit)(int)) const;
    int  size() const;
    ~OrderedSet();
};
```
- **Native:** `BinaryHeap : PriorityQueue` (array-based) and `BST : OrderedSet` (unbalanced).
- **Crossed:** `PQOnBST : PriorityQueue` and `SetOnHeap : OrderedSet`.
- **Motivating application:** a hospital triage system (priority queue) and a leaderboard (ordered set).

### Track 2: Dynamic Array ↔ Linked List
```cpp
class Sequence {                 // index-based access
public:
    int  get(int i) const;
    void set(int i, int value);
    void pushBack(int value);
    int  popBack();
    int  size() const;
    ~Sequence();
};

class List {                     // position-based access
public:
    void pushFront(int value);
    int  popFront();
    void insertAt(int pos, int value);
    int  removeAt(int pos);
    void forEach(void (*visit)(int)) const;
    int  size() const;
    ~List();
};
```
- **Native:** `DynamicArray : Sequence` (with doubling) and `LinkedList : List` (doubly linked).
- **Crossed:** `ArrayOnList : Sequence` and `ListOnArray : List`. `ListOnArray` must store elements in order in the array.
- **Bonus:** Write a second `ListOnArray` using the *cursor-based* technique (an array of nodes with `next` indices and a free list), and compare it against the first one.
- **Motivating application:** a text editor buffer (insertions everywhere) and a music playlist with shuffle (random access).

### Track 3: Graphs, Adjacency Matrix ↔ Adjacency List
```cpp
class Graph {                    // directed, weighted, vertices 0..V-1
public:
    int  addVertex();                 // returns new id
    void addEdge(int u, int v, int w);
    bool removeEdge(int u, int v);
    bool hasEdge(int u, int v) const;
    void forEachNeighbor(int u, void (*visit)(int v, int w)) const;
    int  numVertices() const;
    ~Graph();
};
// Also implement, ONCE, using only the Graph interface:
//   bfs(const Graph&, int src)  and  dijkstra(const Graph&, int src)
```
- **Implementations:** `MatrixGraph : Graph`, built on *your* `DynamicArray`, and `ListGraph : Graph`, built on *your* `DynamicArray` of *your* `LinkedList`s.
- Because both implementations satisfy the same interface, **your BFS and Dijkstra code must be identical for both**. Only the graph underneath changes.
- **Pairs:** also implement `TreeGraph : Graph`, where each vertex stores its neighbors in a BST. This sits between the matrix and the list.
- **Motivating application:** the pathfinder and social network apps from previous semesters.

### Track 4: Stack ↔ Queue (Only for students working individually)
```cpp
class Stack { public: void push(int)=0; int pop()=0; int top() const=0; int size() const=0; ~Stack()=default; };
class Queue { public: void enqueue(int)=0; int dequeue()=0; int front() const=0; int size() const=0; ~Queue()=default; };
```
- **Native:** `ArrayStack : Stack` and `CircularQueue : Queue`.
- **Crossed:** Choose **one**: `QueueFromTwoStacks : Queue` or `StackFromQueues : Stack`. Together with both native classes, this makes **three required classes**. Implementing the other crossed direction is optional and may qualify for the existing bonus.
- **Storage:** `QueueFromTwoStacks` may compose two instances of your native `ArrayStack`; `StackFromQueues` may compose one or two instances of your native `CircularQueue`. Access them only through their public interfaces.
- **Analysis:** Explain the worst-case cost of each operation and the amortized cost over a sequence of operations for your chosen crossed implementation. Trace a short sequence that illustrates when elements move between the underlying structures.
- **Motivating application:** browser back/forward navigation and theme-park ride queues.

---

## Required Benchmark Scenarios
Benchmark **at least four workloads** for your track (five for Track 3, as listed below). Compare implementations of the **same interface**, using identical inputs and operation sequences. Benchmark only the classes you are required to implement; optional implementations may be added. No implementation is required to win: explain the results you actually observe.

| Track | Required workloads and comparisons |
|---|---|
| 1 | Random inserts and sorted inserts: compare native/crossed versions separately for each implemented interface. `contains`-heavy: compare ordered-set versions. Mixed insert/`extractMin`: compare priority-queue versions. If a solo student implements only one crossed direction, benchmark all four workloads, but the other interface has only its native baseline. Use distinct keys for insertion workloads. |
| 2 | Sequential `get(i)` over all positions: compare sequence versions. `pushFront`-heavy and middle insertion: compare list versions. Append-only then traversal: compare sequence versions using `pushBack`/`get`, and list versions using `insertAt(size(), value)`/`forEach`, in separate comparisons. A solo student's unchosen crossed direction has only its native baseline. |
| 3 | Sparse graph + BFS; dense graph + BFS; `hasEdge`-heavy; growing graph using `addVertex`; sparse graph + Dijkstra. Compare every required graph representation using identical vertices, edges, weights, and sources. |
| 4 | Fill then drain; alternating one insertion and one removal; repeated long bursts of insertions then removals; maximum single-operation latency during a documented sequence. Compare your chosen crossed implementation against its matching native class. Use `push`/`pop` for stacks and `enqueue`/`dequeue` for queues. |

**Methodology requirements:**
- Use at least **four input sizes spanning at least two orders of magnitude**. Choose feasible sizes for your track; for example, 100, 500, 2,000, and 10,000. Large quadratic workloads and dense matrices may need smaller sizes. Explain your choices.
- Define `n` and the operation counts for every workload. For non-graph workloads, use `n` inserted elements or a starting size of `n`, and specify the number and mix of subsequent operations. For graphs, use `n = V`, report both `V` and `E`, and state the number of edge queries. Sparse graphs should have `E` proportional to `V`; dense graphs should have `E` proportional to `V²`. State the constants or generation rule.
- For growing graphs, time a sequence of `n` calls to `addVertex` starting from an empty graph. For other workloads, exclude initial population unless population is the operation being measured. Include all work performed by each tested operation, including copying and resizing.
- If a run exceeds **60 seconds**, stop it and report a timeout. If memory limits prevent a run, report that separately. Do not plot either as a measured runtime.
- Run each measurement **at least five times** from equivalent initial states and report the **median**. For the latency workload, record the maximum individual-operation latency in each repetition, then report the median of those five maxima; explain timing overhead.
- Time with `std::chrono::steady_clock`. Exclude input generation, correctness checks, plotting, and file/console I/O from timed regions. Retain and verify operation results outside the timed region so the compiler cannot discard the measured work.
- Use a fixed random seed and the same generated inputs for all compared implementations. Document operation counts, initial states, and graph generation rules so runs are reproducible.
- Plot **runtime vs. n on log-log axes**, with one line per comparable implementation. Label units and plot single-operation latency separately from total runtime. Slopes can suggest growth trends, but finite measurements do not prove a Big-O bound; explain logarithmic factors, amortized costs, and constant factors where relevant.
- Report CPU, RAM, compiler/version, and compiler flags. Use the same flags for all runs, preferably `-O2`.

---

## Milestones & Deliverables

### Milestone 1 — Proposal (Due Nov 13, 2026, 11:59 p.m.). Feedback by Nov 16, 2026.
- **Track** and team members.
- **Prediction table:** For every operation × implementation, give the Big-O you expect, with a one-line justification. For each required workload, predict which implementation will win and by roughly how much.
- Predictions are graded on **reasoning, not correctness**. Being wrong and explaining why later is a great outcome.

**Submit** `docs/proposal.md` to your repository.

### Milestone 2 — Project Presentation (During lectures, week of Nov 30 – Dec 4, 2026)
- Your project must be **ready by your assigned presentation time**: all required implementations and graph algorithms for your track must work, tests must pass, and required benchmarks and plots must be complete.
- Include at least **three passing unit tests per class**, with additional cases as needed to cover the required operation behavior.
- Have `docs/design.md` ready with each class's invariants, operation cost table, and explanation of operation mappings.
- Deliver the five-minute demo described below and be prepared for individual questions about your code. A polished final report is not required at presentation time.
- After presenting, you may clean up code, address feedback, and finish the report before the final deadline. This period is not additional time to complete missing core functionality or benchmark results.

### Milestone 3 — Final Checkpoint (Due Dec 4, 2026, 11:59 p.m.)
Submit your cleaned-up code, tests, benchmark driver and results, documentation, and final **2–4-page report, including plots**, by this deadline.

**Repo structure (required):**
```
final-project/
  include/            # provided interface headers (do not modify)
  src/                # your implementations
  tests/              # unit tests
  bench/              # benchmark driver + raw results (CSV)
  docs/
    proposal.md
    design.md
    report.md         # 2–4 pages, including plots
    genai_usage.md    # prompts/outputs if GenAI used
  Makefile            # targets: all, test, bench, clean
  README.md           # build/run/test/benchmark instructions
```

**Final deliverables:**
- **Working code** that builds from a clean clone with `make`.
- **Tests** that run with `make test`, and **benchmarks** that run with `make bench` and write CSV output.
- **Report (2–4 pages).** See the outline below.
- **Short demo (5 min).** See the Presentation Day section below.

---

## Report Outline (2–4 pages)
1. **Implementations.** Briefly describe each class, its invariants, and how each crossed implementation maps its operations onto the other structure. Trace one representative operation step by step, connecting the work performed to its time and auxiliary-space costs. For Track 3, trace an operation across the different graph representations.
2. **Predictions.** Restate your proposal's prediction table, revised if your understanding changed.
3. **Results.** Include log-log plots for each workload and a table of measured times.
4. **Predictions vs. reality.** For each workload: did the result match your prediction? Explain any surprises. Constant factors, cache behavior, memory allocation, and degenerate inputs are all fair game.
5. **When does each structure shine or fail?** Name a real application where the crossed version would be acceptable, and one where it would be a disaster.
6. **Limitations & future work.** For example, how would a balanced BST, a better hash function, or a different representation change your results?

---

## Assessment (Points-Based)
Total: **100 points**

| Criterion | Points | Expectations |
|---|---:|---|
| Proposal predictions | 10 | Complete prediction table with justified Big-O for every operation. |
| Implementation correctness | 25 | All classes pass functional tests, handle edge cases (empty structure, single element, duplicates, out-of-range access), and are free of memory leaks and undefined behavior. |
| Design & OOP | 10 | Correct use of the provided interfaces, polymorphism, and composition. Crossed implementations respect the public-interface rule. |
| Benchmark methodology | 15 | Required workloads, sizes, repetitions, and reproducibility. Correct log-log plots. |
| Analysis & discussion | 20 | Sound asymptotic analysis, an honest comparison of predictions against measurements, and insightful explanations of surprises. |
| Code quality & docs | 10 | Readability, structure, comments, and README clarity. |
| Presentation/demo | 10 | Concise, compelling, and demonstrates correctness and performance. |
| **Total** | **100** | |
| Bonus | Up to +5 | Track 2's cursor-based list or an additional well-analyzed implementation. |

**Academic integrity violations: up to −100 and referral to honor code.**

---

## GenAI Usage Policy
GenAI tools may be used for **scaffolding, brainstorming, refactoring, documentation, benchmark plotting scripts, and Makefiles**. They may **not** be used to write the native or crossed data structure implementations, required BFS/Dijkstra implementations, or Big-O predictions. You must author tests and benchmark logic yourself; GenAI may help format plots from your measured results. For debugging, you may request conceptual explanations, but not generated replacement implementation code. Scaffolding and refactoring assistance must stay outside the prohibited implementation and analysis work.

If GenAI is used:
- Include a `docs/genai_usage.md` with **all prompts** and **key outputs** used.
- Add a **reflection** on how you validated or corrected GenAI suggestions.
- Cite any generated text or snippets in code comments.
- You are responsible for correctness and academic integrity.

---

## Submission
Submit your work to the GitHub repository by **Dec 4, 2026, 11:59 p.m.**

---

## Presentation Day
- **5 minutes** per team.
- Show:
  1. Your track and one crossed implementation. Explain how it maps one structure onto the other.
  2. **One plot where your prediction was wrong or surprising**, and your explanation.
  3. One interesting bug you found and fixed.
  4. A short operation trace through your code: explain the changes to the structure, the invariant preserved, and the resulting cost. For Track 3, use a graph operation or algorithm step.
- Be ready for 2 minutes of Q&A on implementation details and trade-offs. Each team member must be able to explain the code and analysis; questions may be directed to either member.

---

_Disclaimer: Generative AI was used to help draft the text of this assignment._
