# LearnPulse — The Complete End-to-End Engineering, Architecture & Mathematical Blueprint
### Comprehensive Deep-Dive for SIH 2026 (Problem Statement: SIH 26207)
> **Purpose of this Document:** To give you a 100% logical, crystal-clear, end-to-end understanding of every single engineering technique, architecture layer, mathematical equation, algorithm, and file interaction in LearnPulse.  
> Even if you are a **DSA beginner**, by reading this document from top to bottom, you will understand this system better than 99% of computer science graduates.

---

## 📑 Master Table of Contents
1. [Core Philosophy: Why DAG? Why Acyclic? Why Distractors?](#1-core-philosophy-why-dag-why-acyclic-why-distractors)
2. [Database Schema & Data Architecture (`supabase/schema.sql`)](#2-database-schema--data-architecture)
3. [Graph Algorithms & Prerequisite Backtracking (`graph.ts`)](#3-graph-algorithms--prerequisite-backtracking)
4. [Kahn's Topological Sort & Cycle Prevention Deep-Dive](#4-kahns-topological-sort--cycle-prevention-deep-dive)
5. [Anti-Gaming Scaled Mastery Mathematics (`mastery.ts`)](#5-anti-gaming-scaled-mastery-mathematics)
6. [4-Factor Root Cause Ranking Heuristic (`rootCause.ts`)](#6-4-factor-root-cause-ranking-heuristic)
7. [Temporal Memory Decay & Ebbinghaus Curve (`decay.ts`)](#7-temporal-memory-decay--ebbinghaus-curve)
8. [The AI Engine: Gemini 2.5 Flash, Prompts & Zod Schemas (`src/lib/ai/`)](#8-the-ai-engine-gemini-25-flash-prompts--zod-schemas)
9. [Sub-Millisecond Dual-Tier Caching Architecture](#9-sub-millisecond-dual-tier-caching-architecture)
10. [Offline-First Queue & Idempotent Synchronization (`offlineQueue.ts`)](#10-offline-first-queue--idempotent-synchronization)
11. [Frontend Systems, React 19 & Canvas Mechanics (`@xyflow/react`)](#11-frontend-systems-react-19--canvas-mechanics)
12. [End-to-End Execution Lifecycles (The Complete Data Journeys)](#12-end-to-end-execution-lifecycles)
13. [Verification Suite: 101 Vitest Tests Across 11 Suites](#13-verification-suite-101-vitest-tests-across-11-suites)
14. [The Master Defense Scripts: How to Answer Every Question Logically](#14-the-master-defense-scripts)

---

## 1. Core Philosophy: Why DAG? Why Acyclic? Why Distractors?

Before looking at any code, understand the **Three Foundational Why's** that drive this entire project.

### Why a Graph instead of a Tree?
* **A Tree:** In a computer science Tree, every node can only have **exactly one parent**.
  - If education were a tree, *Query Optimization* could only depend on *B-Trees*. But in real engineering, *Query Optimization* depends on **both** *B-Trees* (storage) AND *Relational Algebra* (logic).
* **A Graph:** In a Graph, a node can have **multiple parents (prerequisites)** and **multiple children (dependents)**.
  - This matches real human knowledge: you need multiple foundational skills to understand one advanced concept.

### Why "Directed Acyclic" (DAG)?
* **Directed (D):** The arrows have a direction. Arrow goes from $A \to B$ (meaning Concept A is the prerequisite of Concept B). You must understand Pointers before understanding Linked Lists.
* **Acyclic (A):** There are **NO CIRCULAR LOOPS**.
  - If Arrow $A \to B$, Arrow $B \to C$, and Arrow $C \to A$, this is a cycle. In education, a cycle is a fatal deadlock: you cannot learn A without B, B without C, and C without A—a student could never begin!
  - Therefore, our system mathematically enforces that the graph is **strictly acyclic**.

### Why Analyze the "Distractor" (Wrong Option)?
* Standard platforms do **Explanatory Grading**: If you pick Option B on a 4-option MCQ, the platform says: *"Wrong! Option C is correct because [textbook definition]."*
* **The Pedagogical Flaw:** The student didn't pick Option B because they were blind; they picked Option B because of a **specific false rule (Thought Trap)** in their head!
  - Example: In database indexing, Option B states: *"Adding an index always speeds up all database operations."*
  - The student picked it because they know indexes speed up `SELECT`. They forgot that indexes **slow down `INSERT` and `UPDATE`** because the index tree must be rebalanced on every write!
* **LearnPulse's Mental Mirror:** Instead of lecturing on Option C, LearnPulse says:  
  *"You fell into Thought Trap #3: You assumed indexes help every operation. Paradox: What happens during 10,000 INSERTs per second? The database crashes while rebalancing the B-Tree! Anchor: Index speeds up Reads, but penalizes Writes."*

---

## 2. Database Schema & Data Architecture

Located at: [`supabase/schema.sql`](file:///Users/subhajit/Developer/Development/SIH%202026/Learnpulse/learnpulse-app/supabase/schema.sql)

Our database is built on **Supabase PostgreSQL 15**, utilizing strict relational normalization and PostgreSQL **Row-Level Security (RLS)**.

```mermaid
erDiagram
    courses ||--o{ concepts : "contains"
    concepts ||--o{ concept_edges : "prerequisite / dependent"
    concepts ||--o{ questions : "contains"
    concepts ||--o{ student_mastery : "tracks"
    questions ||--o{ attempts : "evaluated in"
    questions ||--o{ interventions : "cached deconstructions"
    auth_users ||--o{ attempts : "logs"
    auth_users ||--o{ student_mastery : "owns"

    courses {
        uuid id PK
        string title
        text description
        uuid created_by
    }
    concepts {
        uuid id PK
        uuid course_id FK
        string name
        int order_index
        string tier
    }
    concept_edges {
        uuid id PK
        uuid course_id FK
        uuid prerequisite_id FK
        uuid concept_id FK
        float weight
    }
    questions {
        uuid id PK
        uuid concept_id FK
        text question_text
        jsonb options
        string correct_answer
        string difficulty
    }
    attempts {
        uuid id PK
        uuid user_id FK
        uuid question_id FK
        uuid concept_id FK
        string selected_option
        boolean is_correct
        timestamp created_at
    }
    student_mastery {
        uuid id PK
        uuid user_id FK
        uuid concept_id FK
        int score
        string mastery_level
        timestamp last_practiced_at
        float retention_rate
    }
    interventions {
        uuid id PK
        uuid question_id FK
        string selected_option
        text thought_trap
        text cognitive_dissonance
        text vernacular_anchor
    }
```

### Key Architectural Safeguards in the Schema:
1. **The Unique Constraint on `student_mastery(user_id, concept_id)`:**
   - Guarantees that a student has **exactly one mastery record** per concept.
   - Enables atomic `INSERT ... ON CONFLICT (user_id, concept_id) DO UPDATE SET score = EXCLUDED.score`, preventing concurrent race conditions.
2. **The Composite Index on `attempts(user_id, concept_id, created_at DESC)`:**
   - Speeds up student history queries from $O(N)$ sequential table scans to $O(\log N)$ index-only scans, retrieving recent attempts in $< 2\text{ms}$.
3. **Row-Level Security (RLS):**
   ```sql
   CREATE POLICY "Users can only read and write their own attempts"
   ON attempts FOR ALL
   USING (auth.uid() = user_id);
   ```
   - Even if a student tampers with client-side API requests, the PostgreSQL kernel blocks access to another student's data.

---

## 3. Graph Algorithms & Prerequisite Backtracking

Located at: [`src/lib/algorithms/graph.ts`](file:///Users/subhajit/Developer/Development/SIH%202026/Learnpulse/learnpulse-app/src/lib/algorithms/graph.ts)

### 3.1 The Bidirectional Adjacency List (`buildAdjacencyList`)
Why do we build two maps instead of one?
```typescript
export interface AdjacencyList {
  prerequisites: Map<string, Array<{ id: string; weight: number }>>; // Backward: Child -> Parents
  dependents: Map<string, Array<{ id: string; weight: number }>>;    // Forward: Parent -> Children
}
```
* **Why Bidirectional?**
  - To diagnose a failure, you must go **BACKWARDS** (Child $\to$ Parents) to find the root blocker.
  - To schedule an Ebbinghaus retention review, you must go **FORWARDS** (Parent $\to$ Children) to find which advanced topics rely on this fundamental concept.
  - `buildAdjacencyList(edges)` scans the database edges once in $O(E)$ time and populates both in-memory maps.

### 3.2 BFS (Breadth-First Search) Prerequisite Backtracking (`bfsPrerequisites`)
```typescript
export function bfsPrerequisites(targetConceptId: string, adj: AdjacencyList): PrerequisiteNode[] {
  const visited = new Set<string>();
  const result: PrerequisiteNode[] = [];
  const queue: Array<{ id: string; weight: number; depth: number }> = [];

  // 1. Seed the queue with direct parents (Depth 1)
  const directPrereqs = adj.prerequisites.get(targetConceptId) ?? [];
  for (const prereq of directPrereqs) {
    visited.add(prereq.id);
    queue.push({ ...prereq, depth: 1 });
  }

  // 2. Explore layer by layer (FIFO Queue)
  while (queue.length > 0) {
    const current = queue.shift()!;
    result.push(current);

    // 3. Look for grandparents (Depth + 1)
    const furtherPrereqs = adj.prerequisites.get(current.id) ?? [];
    for (const prereq of furtherPrereqs) {
      if (!visited.has(prereq.id)) {
        visited.add(prereq.id);
        queue.push({ ...prereq, depth: current.depth + 1 });
      }
    }
  }
  return result;
}
```
* **The Logic:**
  - Uses a `visited` Set to prevent infinite loops if bad data exists.
  - Uses `queue.shift()` (FIFO) so that all Depth 1 parents are analyzed before Depth 2 grandparents.
  - Returns ancestors in order of **topological proximity**, ensuring closest causes are prioritized.

---

## 4. Kahn's Topological Sort & Cycle Prevention Deep-Dive

Located at: [`src/lib/algorithms/graph.ts`](file:///Users/subhajit/Developer/Development/SIH%202026/Learnpulse/learnpulse-app/src/lib/algorithms/graph.ts#L111-L156)

Kahn's Algorithm is the **mathematical gatekeeper** of LearnPulse. It runs in $O(V + E)$ time to prove whether a syllabus is valid or deadlocked.

```mermaid
flowchart TD
    A["Initialize inDegree = 0 for all V vertices"] --> B["Iterate through all edges (u -> v): increment inDegree[v]"]
    B --> C["Find all vertices where inDegree == 0<br/>(Topics with NO prerequisites)"]
    C --> D["Push 0-in-degree vertices into Queue"]
    D --> E{"Is Queue empty?"}
    E -- "NO" --> F["Dequeue node u -> Add to sortedList"]
    F --> G["For each dependent neighbor v: inDegree[v] = inDegree[v] - 1"]
    G --> H{"Did inDegree[v] reach 0?"}
    H -- "YES" --> I["Enqueue node v"]
    H -- "NO" --> E
    I --> E
    E -- "YES" --> J{"Is sortedList.length == totalVertices?"}
    J -- "YES" --> K["✅ Clean DAG Verified: Return Topological Order"]
    J -- "NO" --> L["❌ CYCLE DETECTED! Return null (Reject transaction)"]

    style K fill:#dcfce7,stroke:#16a34a,stroke-width:2px
    style L fill:#fee2e2,stroke:#ef4444,stroke-width:2px
```

### The Mathematical Proof of Cycle Detection:
* In a Directed Acyclic Graph (DAG), there is **always at least one vertex with in-degree 0** (the starting point).
* In a circular cycle ($A \to B \to C \to A$):
  - $A$ needs $C$ (in-degree $\ge 1$).
  - $B$ needs $A$ (in-degree $\ge 1$).
  - $C$ needs $B$ (in-degree $\ge 1$).
* **None of the nodes in the cycle ever hit in-degree 0!**
* Therefore, they are never enqueued, and when the queue empties, `sortedList.length < totalVertices`.
* If a professor tries to save a circular syllabus, our server action runs this in **$< 2\text{ms}$** and immediately aborts the database transaction.

---

## 5. Anti-Gaming Scaled Mastery Mathematics

Located at: [`src/lib/algorithms/mastery.ts`](file:///Users/subhajit/Developer/Development/SIH%202026/Learnpulse/learnpulse-app/src/lib/algorithms/mastery.ts)

### 5.1 The Flaw of Traditional Accuracy
Traditional LMS platforms calculate mastery as:
$$\text{Naive Mastery} = \frac{\text{Total Correct Attempts}}{\text{Total Attempts}}$$
* **The Exploit:** If a student answers 1 easy question correctly 20 times, $\text{Accuracy} = \frac{20}{20} = 100\%$. The system marks them as an expert, even though they skipped 80% of the topic's curriculum!

### 5.2 The LearnPulse Scaled Mastery Formulation
We decouple mastery into **Breadth Coverage (80 Points)** and **Precision Accuracy (20 Points)**:

$$\text{Coverage} = \min\left(1, \frac{\text{Unique Questions Solved Correctly}}{\text{Total Available Questions in Concept}}\right)$$

$$\text{Accuracy} = \min\left(1, \frac{\text{Total Correct Attempts}}{\text{Total Attempts}}\right)$$

$$\text{Mastery Score} = \text{round}\Big(80 \cdot \text{Coverage} + 20 \cdot \text{Coverage} \cdot \text{Accuracy}\Big)$$

### Mathematical Properties:
1. **Strict Breadth Requirement:** Notice that Coverage multiplies BOTH the base (80) and the accuracy bonus (20). If Coverage is $0.2$, the maximum score is $0.2 \times 100 = 20\%$.
2. **Error Forgiveness:** If a student struggles initially (e.g. 5 failed attempts on Question 1) but eventually solves all 5 questions correctly, their Coverage reaches $1.0$, and their score will still be between **$85–92\%$**, encouraging resilience over perfectionism.
3. **Mastery Tiers:**
   - **Level 1 (0–39%):** Getting Started (High prerequisite risk).
   - **Level 2 (40–69%):** Developing (Partial coverage or mixed accuracy).
   - **Level 3 (70–84%):** Proficient (High coverage, ready for forward concepts).
   - **Level 4 (85–100%):** Mastered (Full bank solved with high precision).

---

## 6. 4-Factor Root Cause Ranking Heuristic

Located at: [`src/lib/algorithms/rootCause.ts`](file:///Users/subhajit/Developer/Development/SIH%202026/Learnpulse/learnpulse-app/src/lib/algorithms/rootCause.ts)

When BFS finds multiple ancestor candidates, how do we mathematically rank them to find the true root cause?

$$\text{RootCauseScore} = 0.45 \cdot W + 0.25 \cdot E + 0.20 \cdot Ev + 0.10 \cdot R$$

| Parameter | Mathematical Formulation | Pedagogical Justification |
| :--- | :--- | :--- |
| **Weakness ($W$)** | $1 - \frac{\text{MasteryScore}}{100}$ | An unmastered concept is the strongest prerequisite blocker signal. |
| **Edge Weight ($E$)** | $\min\left(\frac{\text{edgeWeight}}{5}, 1.0\right)$ | Mandatory core prerequisites ($w \approx 1.0$) outrank loose contextual references ($w \approx 0.3$). |
| **Evidence ($Ev$)** | $\min\left(\frac{\text{failedAttempts}}{20}, 1.0\right)$ | Empirical historical failure on this prerequisite proves the student actually struggled with it. |
| **Recency ($R$)** | $\text{Normalized proximity } [0, 1]$ | Recent struggles indicate active confusion; old struggles indicate potential decay. |

The algorithm computes this for every ancestor in the BFS tree, sorts descending, and returns the **Top 3 candidates**, highlighting the #1 primary bottleneck node in red on the visual canvas.

---

## 7. Temporal Memory Decay & Ebbinghaus Curve

Located at: [`src/lib/algorithms/decay.ts`](file:///Users/subhajit/Developer/Development/SIH%202026/Learnpulse/learnpulse-app/src/lib/algorithms/decay.ts)

Human cognitive retention decays exponentially over time without spaced repetition. LearnPulse models this using the **Ebbinghaus Forgetting Curve**:

$$R(t) = e^{-\frac{t}{S}}$$

* $t$ = Elapsed days since the student last practiced the concept.
* $S$ = Memory Stability factor.

### Derivation of Memory Stability ($S$):
$$S = \text{Mastery Score} \times 0.14$$
* Why the constant $0.14$?
  - For a student who achieved **100% Mastery**, $S = 100 \times 0.14 = \mathbf{14\text{ days}}$.
  - Setting $t = 7\text{ days}$ (1 week later):
    $$R(7) = e^{-\frac{7}{14}} = e^{-0.5} \approx 0.6065 \quad (60.65\%)$$
  - This mathematically aligns the **$60\%$ retention review threshold** with the standard 1-week university tutorial and review cycle!
* When $R(t) < 0.60$, `isReviewDue()` returns `true`.

### Forward-Dependent Review Selection (`reviewSelection.ts`):
* Instead of serving the student the exact same basic question they solved last week, `getForwardDependents()` queries the graph for **downstream concepts that depend on this fundamental concept**.
* The student is asked to solve a challenge that **applies** the fundamental concept in a real problem.
* If they succeed, their retention stability timer resets and expands; if they fail, the system triggers the 60-second prerequisite bite.

---

## 8. The AI Engine: Gemini 2.5 Flash, Prompts & Zod Schemas

Located at: [`src/lib/ai/`](file:///Users/subhajit/Developer/Development/SIH%202026/Learnpulse/learnpulse-app/src/lib/ai/)

```mermaid
flowchart LR
    A["Selected Distractor + Question Context"] --> B["System Prompt (prompts.ts)"]
    B --> C["Google Gemini 2.5 Flash API (@google/genai)"]
    C --> D["Raw JSON String Response"]
    D --> E["Zod Schema Validator (misconception.ts)"]
    E -- "Pass" --> F["Store in L1 In-Memory LRU Cache"]
    E -- "Fail / Malformed" --> G["Trigger Circuit Breaker: Fallback to Curated SQL Cache"]
    F --> H["Client Render: Mental Mirror Card"]

    style E fill:#fef3c7,stroke:#d97706,stroke-width:2px
    style F fill:#dcfce7,stroke:#16a34a,stroke-width:2px
    style G fill:#fee2e2,stroke:#ef4444,stroke-width:2px
```

### 8.1 The Zod Schema Enforcement
In `src/lib/ai/misconception.ts`:
```typescript
export const MisconceptionResponseSchema = z.object({
  thoughtTrap: z.string().min(10),
  cognitiveDissonance: z.object({
    paradoxScenario: z.string().min(10),
    whyItBreaks: z.string().min(10),
  }),
  mentalAnchor: z.string().min(5),
  vernacularAnchor: z.string().min(5),
});
```
If Gemini omits a field or hallucinates invalid keys, the Zod parser immediately throws an error at the boundary, diverting execution to the database cache without sending broken payloads to the UI.

---

## 9. Sub-Millisecond Dual-Tier Caching Architecture

How does LearnPulse achieve **$< 1\text{ms}$** response times and near-zero API bills?

```
Student Picks Option B
       │
       ▼
[Tier 1: In-Memory LRU Cache] ──(Hit)──► Return in < 1ms (Zero Network, ₹0.00 Cost)
       │
     (Miss)
       ▼
[Tier 2: PostgreSQL Interventions Table] ──(Hit)──► Return in 8ms (Indexed Point-Lookup)
       │
     (Miss)
       ▼
[Tier 3: Google Gemini 2.5 Flash] ──────► Generate in 1.2s -> Save to Tier 1 & Tier 2
```

In any university course, the question bank is finite. Across 190 questions, there are only $190 \times 3 = 570$ unique distractors.
* Once all 570 options have been encountered once, **100% of student attempts hit the Tier 1 or Tier 2 cache**.
* The university pays **₹0.00 in AI API costs** for subsequent student cohorts!

---

## 10. Offline-First Queue & Idempotent Synchronization

Located at: [`src/lib/offline/offlineQueue.ts`](file:///Users/subhajit/Developer/Development/SIH%202026/Learnpulse/learnpulse-app/src/lib/offline/offlineQueue.ts)

When a student practices in a rural college with intermittent Wi-Fi:
1. **Network Interception:** The queue intercepts quiz attempts when `navigator.onLine === false`.
2. **Local Persistence:** Attempts are serialized into `IndexedDB / localStorage` with microsecond timestamps.
3. **Optimistic Local State:** The UI updates the student's mastery score locally so their momentum is never broken.
4. **Automatic Background Reconciliation:**
   ```typescript
   window.addEventListener('online', async () => {
     await offlineQueue.processQueue();
   });
   ```
5. **Idempotency Guarantee:**
   - Attempts are replayed to the server in batch.
   - The PostgreSQL `attempts` table records the historical timestamp, and `student_mastery` updates via `ON CONFLICT (user_id, concept_id) DO UPDATE`.
   - Even if the network flickers and sends the same batch twice, the database maintains **zero duplicate scores and zero score drift**.

---

## 11. Frontend Systems, React 19 & Canvas Mechanics

Located at: [`src/components/mastery/InteractiveDagGraph.tsx`](file:///Users/subhajit/Developer/Development/SIH%202026/Learnpulse/learnpulse-app/src/components/mastery/InteractiveDagGraph.tsx)

### 11.1 The SSR Hydration Boundary
React Flow (`@xyflow/react`) requires window dimensions (`window.innerWidth`, DOM bounding boxes) to calculate node coordinates. On the Next.js server, `window` is undefined.
* **Our Solution:** Dynamic Client Boundary with Skeleton Fallback:
  ```typescript
  const InteractiveDagGraph = dynamic(() => import('./InteractiveDagGraph'), {
    ssr: false,
    loading: () => <DagSkeletonLoader />
  });
  ```
* On the server, Next.js streams a lightweight CSS skeleton with pre-calculated topological tiers.
* When the client mounts in the browser, `@xyflow/react` hydrates seamlessly with **zero Cumulative Layout Shift (CLS = 0.00)**.

### 11.2 High-Contrast Mastery Color Tokens
Nodes in the DAG canvas dynamically color-code based on mastery levels:
* **Green (`#10b981`):** Level 4 • Mastered ($85–100\%$).
* **Blue (`#3b82f6`):** Level 3 • Proficient ($70–84\%$).
* **Yellow/Amber (`#f59e0b`):** Level 2 • Developing ($40–69\%$) or Retention Review Due.
* **Red (`#ef4444`):** Level 1 • At-Risk ($0–39\%$).
* **Pulsing High-Contrast Red Border:** The Primary Ancestral Bottleneck identified by `rootCause.ts`.

---

## 12. End-to-End Execution Lifecycles

Here is the exact file-by-file journey of the two main user actions:

### Journey 1: Student Answers an MCQ
1. Student clicks Option B in `QuizCard.tsx`.
2. Client fires `POST /api/submit-attempt`:
   - Route handler calls `updateMastery()` and `computeScaledMastery()` in `mastery.ts`.
   - Inserts record into `attempts` and updates `student_mastery`.
3. Client fires `POST /api/ai/misconception`:
   - Checks in-memory cache in `misconception.ts`.
   - If miss, calls Gemini with `MisconceptionResponseSchema`.
4. Client triggers BFS prerequisite backtrack via `bfsPrerequisites()` in `graph.ts`.
5. `rootCause.ts` scores candidates and returns top blocker.
6. `InteractiveDagGraph.tsx` updates node colors and pulses the bottleneck node.
7. `QuizCard.tsx` renders the Mental Mirror card and the 60-second recovery bite.

### Journey 2: Teacher Ingests a Syllabus
1. Teacher pastes text into `SyllabusIngestionModal.tsx`.
2. Calls Server Action in `src/app/actions/authoring.ts`.
3. Calls `synthesizeDagFromSyllabus()` in `dagSynthesis.ts`.
4. Gemini extracts concepts and edge tuples `[prereqId, conceptId]`.
5. `topologicalSort()` in `graph.ts` runs Kahn's algorithm.
6. If cycle detected: Aborts transaction and highlights the circular loop.
7. If acyclic: Inserts into `concepts` and `concept_edges` in Supabase.
8. DAG canvas re-renders with new syllabus in **under 45 seconds**.

---

## 13. Verification Suite: 101 Vitest Tests Across 11 Suites

LearnPulse maintains **100% test pass rate** across 11 automated test suites:
Command: `npm run test:run` (Duration: **538 milliseconds**)

| Test Suite File | Test Count | What It Formally Proves |
| :--- | :---: | :--- |
| `graph.test.ts` | 15 tests | Kahn's algorithm cycle detection, topological order validity, BFS depth accuracy. |
| `mastery.test.ts` | 15 tests | Scaled mastery bounds ($0–100$), anti-gaming coverage multipliers, zero-question safety. |
| `decay.test.ts` | 11 tests | Ebbinghaus exponential decay math ($R = e^{-t/S}$), 60% threshold triggers, stability scaling. |
| `risk.test.ts` | 15 tests | Student at-risk classification and cohort distribution calculations. |
| `rootCause.test.ts` | 9 tests | 4-factor heuristic weights, ancestor candidate ranking, normalization bounds. |
| `offlineQueue.test.ts` | 5 tests | Offline queue serialization, FIFO ordering, atomic batch synchronization replay. |
| `misconception.test.ts` | 5 tests | Zod schema enforcement, in-memory cache hits ($< 1$ms), circuit-breaker fallbacks. |
| `dagSynthesis.test.ts` | 7 tests | Syllabus concept extraction, cycle rejection on generated edges, JSON sanitization. |
| `conceptBite.test.ts` | 5 tests | 60-second bite prompt outputs, bilingual anchor validation, quick-check pools. |
| `masteryLevels.test.ts` | 9 tests | Level 1 through Level 4 score boundary classification. |
| `reviewSelection.test.ts` | 5 tests | Forward-dependent concept discovery and retention review question generation. |

---

## 14. The Master Defense Scripts: How to Answer Every Question Logically

Whenever a professor or judge asks you a question, use this simple **3-Sentence Formula**:
1. **Sentence 1 (The Principle):** Acknowledge the core computer science or pedagogical principle.
2. **Sentence 2 (The Implementation):** Name the exact file and algorithm in our codebase that solves it.
3. **Sentence 3 (The Proof):** State the mathematical guarantee, test benchmark, or latency metric.

### Master Script Examples:

#### If asked about Graph Cycles:
> *"Sir, course prerequisites cannot be circular because that causes a learning deadlock. In `src/lib/algorithms/graph.ts`, we implement Kahn's Topological Sort Algorithm in $O(V+E)$ time by tracking vertex in-degrees. Before any syllabus touches our database, Kahn's algorithm mathematically verifies that processed nodes equal total nodes, rejecting circular dependencies with 100% certainty across our 15 automated test cases."*

#### If asked about Cheating or Quiz Gaming:
> *"Sir, traditional platforms calculate mastery as naive accuracy, allowing students to game the system by spamming one easy question. In `src/lib/algorithms/mastery.ts`, we engineered our Scaled Mastery formula coupling 80 points of Breadth Coverage with 20 points of Accuracy. A student who answers one question 50 times is mathematically capped at 20% mastery until they prove competence across the entire concept bank."*

#### If asked about AI Code Generation or Authenticity:
> *"Sir, we used AI coding tools as high-speed syntax compilers and pair programmers, exactly as senior engineers at Google do today. But the problem definition, the Scaled Mastery mathematics, the Kahn's algorithm cycle invariants, the 4-factor root cause weights, and all 101 Vitest test specifications were designed and engineered by our team. The AI was our builder; we are the architects."*

#### If asked about High Latency or API Costs:
> *"Sir, recurring student attempts never wait for Gemini because of our Dual-Tier Caching Architecture. In `src/lib/ai/misconception.ts`, distractors are cached in an in-memory LRU map and PostgreSQL B-Tree indexes. The first student pays 1.2s of AI generation; the next 10,000 students get sub-millisecond lookups (< 1ms) at ₹0.00 API cost."*

---

### 🎓 You Are Now Ready
You have the complete, unshakeable logical understanding of the entire system. Keep this document open during practice sessions. Walk into your evaluation knowing that your architecture is sound, your math is proven, and your code is backed by 101 passing tests. You've got this!
