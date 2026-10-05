# The Solver Architecture

The Solver uses a modular engine architecture designed around three primary goals:

- keep solving logic independent from user interfaces
- make every deduction testable and explainable
- support deterministic, replayable solving behavior

The first supported puzzle type is **Nonograms**, but the architecture is intended to leave room for additional logic-puzzle modules over time.

---

## Overview

```text
Puzzle Definition
       |
       v
Board State
       |
       v
Board Orchestrator
       |
       v
Line Evaluator
       |
       v
Deduction Strategies
       |
       v
Proposed Moves
       |
       v
Board Update
       |
       +------> Explanation / Transcript
       |
       +------> Re-evaluation Queue
```

The solver engine is responsible for puzzle reasoning.

Interfaces such as command-line, web, or mobile applications interact with the engine but do not contain the core solving logic.

This separation keeps the solving system reusable, testable, and easier to evolve.

---

## Core Solve Flow

At a high level, solving proceeds as an iterative evaluation loop.

### 1. Puzzle Initialization

A puzzle definition provides:

- board dimensions
- row clues
- column clues
- initial board state

The board is initialized with all known puzzle constraints and any preexisting cell states.

### 2. Line Selection

The Board Orchestrator selects a row or column that should be evaluated.

Selection may eventually be influenced by factors such as:

- recent board changes
- unresolved cells
- estimated information value
- prior progress
- evaluation cost

The initial implementation can remain simpler while preserving a clean boundary for future prioritization logic.

### 3. Line Evaluation

The selected line is sent to the Stateless Line Evaluator.

The evaluator receives the current facts for that line, including:

- line length
- clue sequence
- current cell state

The evaluator does not own or mutate the full board.

Its responsibility is to determine what new facts can be logically inferred from the supplied line state.

### 4. Deduction Strategy Evaluation

Applicable deduction strategies analyze the line.

Each strategy attempts to identify deterministic information supported by the current line state and clues.

Strategies may operate independently or as part of an ordered evaluation pipeline.

The strategy layer is intentionally modular so deduction behaviors can be:

- tested independently
- added incrementally
- revised without changing unrelated logic
- explained by rule type

### 5. Proposed Moves

The evaluator returns proposed changes rather than directly modifying board state.

A proposal may include:

- cell position
- proposed value
- reasoning source
- explanation metadata

Conceptually:

```text
Line: Row 8
Cell: 6
Value: Filled
Reason: overlap across all legal placements
```

This makes deduction output explicit and inspectable.

### 6. Board Update

The Board Orchestrator applies valid proposed moves to the board.

Only previously unknown cells should normally change.

If a proposed move conflicts with an already known fact, the solver should treat the situation as a contradiction rather than silently overwriting state.

### 7. Re-evaluation

When board cells change, affected rows and columns become candidates for reevaluation.

For example:

- a change in Row 8 affects the corresponding columns
- a change in Column 5 affects the corresponding rows

The solve loop continues until one of the defined stopping conditions is reached.

---

## Component Responsibilities

### Puzzle Definition

Responsible for:

- board dimensions
- row clues
- column clues
- puzzle identity or metadata
- initial board state

The puzzle definition describes the problem but does not perform solving.

---

### Board State

Responsible for:

- storing known cell values
- exposing row and column states
- applying validated updates
- detecting completion
- preserving current puzzle facts

The board represents the current known truth of the puzzle.

It should not contain deduction logic.

---

### Board Orchestrator

Responsible for:

- selecting lines for evaluation
- passing line state to the evaluator
- receiving proposed moves
- validating and applying board updates
- scheduling affected lines for reevaluation
- detecting solve progress or stagnation
- coordinating transcript and replay events

The Board Orchestrator owns solve progression.

It does not need to understand the internal details of each deduction strategy.

---

### Stateless Line Evaluator

Responsible for:

- receiving one line at a time
- evaluating the line against its clues
- applying available deduction strategies
- returning proposed cell changes
- returning explanation metadata
- avoiding direct mutation of full-board state

The evaluator is intentionally stateless.

Given the same:

- line length
- clues
- current state

it should produce the same logical result.

This makes line evaluation easier to test, reproduce, and debug.

---

### Deduction Strategies

Responsible for:

- implementing specific logical deduction patterns
- identifying facts that are guaranteed by the current line state
- producing only deterministic conclusions
- associating deductions with an identifiable reasoning source

Each strategy should be small enough to test independently.

The public architecture intentionally treats deduction strategies conceptually rather than documenting implementation details.

---

### Explanation / Transcript Layer

Responsible for capturing enough information to explain how the board changed.

A solve event may eventually include:

- line identifier
- previous state
- updated state
- affected cells
- deduction category
- supporting reasoning
- sequence number
- timestamp or replay order

The transcript is not merely a UI feature.

It supports:

- debugging
- regression analysis
- deterministic replay
- user-facing explanations
- future teacher mode

---

## Why the Solver Engine Is Separate From the UI

Embedding solving logic directly into a user interface would make early development simpler, but it would create several long-term problems.

### Tight Coupling

A web, mobile, or desktop interface could become responsible for both presentation and puzzle reasoning.

That would make the solving engine harder to reuse.

### Duplicate Logic

Supporting more than one client could require duplicating deduction behavior across applications.

### Harder Testing

Testing solver behavior through a UI would be slower and less precise than testing the engine directly.

### Reduced Explainability

Reasoning metadata can be lost when deduction logic is mixed directly into display code.

### Limited Extensibility

A separate engine can support future clients without requiring the puzzle logic to be rewritten.

The intended model is:

```text
                Solver Engine
                     |
        +------------+------------+
        |            |            |
       CLI          Web         Mobile
```

The clients ask the engine to solve or evaluate.

They do not implement the solving rules themselves.

---

## Explainability Model

Explainability is a first-class architectural concern.

A valid solving move should ideally answer:

- what changed?
- where did it change?
- which deduction produced the change?
- what evidence supported the deduction?
- what was the board state before the move?
- can the move be reproduced later?

The goal is to avoid a black-box interaction such as:

```text
Input board
   |
   v
Solved board
```

Instead, The Solver aims for:

```text
Input board
   |
   v
Deduction
   |
   v
Board change
   |
   v
Explanation
   |
   v
Next deduction
```

This supports both engineering diagnostics and eventual educational behavior.

---

## Determinism and Replay

Where practical, the solver should behave deterministically.

Given the same:

- puzzle definition
- board state
- strategy set
- evaluation order

the solver should be able to reproduce the same solve sequence.

This enables replay.

Conceptually:

```text
Initial Board
    |
    v
Move 001
    |
    v
Move 002
    |
    v
Move 003
    |
    v
...
    |
    v
Solved Board
```

Replay makes it easier to:

- investigate regressions
- compare solver versions
- inspect why a particular path occurred
- demonstrate solving behavior
- build teaching tools later

---

## Testing Strategy

The Solver is developed using layered testing.

### Strategy-Level Tests

Individual deduction strategies are tested against small, targeted line states.

Each case can define:

- line length
- clues
- initial state
- expected state
- expected move type

These tests isolate specific reasoning behavior.

### Regression Cases

Real puzzle-solving situations are converted into repeatable test cases.

This is important because many useful edge cases emerge only during real solves.

Regression cases help prevent future changes from breaking previously working behavior.

### Full-Puzzle Benchmarks

Selected puzzles are used as integration benchmarks.

A difficult Nonogram nicknamed **Poppies** has been used as a major benchmark because it contains varied deduction situations that exercised multiple parts of the evaluator.

The current logic has successfully completed that puzzle through the manual proof-of-concept workflow.

### Future Replay Tests

As transcript support matures, entire solve sequences can become replayable regression artifacts.

This will allow testing not only final correctness, but also the sequence of reasoning steps.

---

## Stop Conditions

The Board Orchestrator should recognize several possible outcomes.

### Puzzle Solved

All required cells are resolved and all row and column clues are satisfied.

### No Further Progress

No available strategy can produce additional facts.

This indicates either:

- the puzzle requires a deduction not yet implemented
- a broader reasoning step is needed
- the solver architecture requires another capability

A stalled solve is useful diagnostic information.

### Contradiction Detected

A proposed move conflicts with an existing board fact or produces an invalid line state.

The solver should surface the contradiction rather than attempting to continue silently.

### Invalid Puzzle State

The supplied puzzle or board state violates known constraints.

Validation should distinguish invalid input from an unsolved but valid puzzle.

---

## State Mutation Principle

The solver is designed to minimize uncontrolled mutation.

The preferred pattern is:

```text
Current State
     |
     v
Evaluation
     |
     v
Proposed Changes
     |
     v
Validation
     |
     v
New State
```

This pattern improves:

- testability
- traceability
- debugging
- replay
- explanation quality

The evaluator reasons about state.

The orchestrator controls state changes.

---

## Design Principle

The central design principle is:

> The engine should know how to solve; clients should only know how to present and interact with the solve.

A related rule is:

> Deduction logic proposes facts. The board owns truth.

These boundaries help keep responsibilities clear as the system grows.

---

## Extensibility

Nonograms are the first puzzle type, but the architecture should not unnecessarily lock the project to one puzzle family.

Reusable concepts may include:

- puzzle definition
- board or state model
- orchestrator
- strategy pipeline
- proposed moves
- explanations
- transcripts
- replay

Puzzle-specific logic belongs inside the relevant puzzle module.

This allows future exploration of additional logic puzzles without forcing unrelated rules into the Nonogram implementation.

The goal is extensibility without premature abstraction.

---

## Future Architecture Work

Planned architecture work includes:

- Board Orchestrator implementation
- automated line scheduling
- line-priority or weighting strategy
- richer explanation metadata
- full solve transcripts
- deterministic replay
- contradiction diagnostics
- broader regression coverage
- CLI client integration
- lightweight web demonstration
- mobile client exploration
- teacher-mode explanation workflows
- support for additional puzzle modules where appropriate

---

## Summary

The Solver architecture can be summarized as:

```text
Puzzle State
     +
Stateless Evaluation
     +
Explicit Proposed Moves
     +
Controlled Board Updates
     +
Explainable Transcript
     =
Testable, replayable puzzle solving
```

The architecture is intentionally designed so the solving engine remains the stable center of the system while interfaces, explanation features, and additional puzzle types can evolve around it.