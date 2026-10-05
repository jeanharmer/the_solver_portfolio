# The Solver

> An explainable puzzle-solving engine designed to model human-style logical deduction and make every solving step understandable.

The Solver is a software engineering project focused on building a clean, modular, and explainable solving engine.

The first supported puzzle type is **Nonograms**.

Rather than treating a puzzle as a black-box search problem, The Solver is designed around explicit logical deductions that can be recorded, explained, tested, and eventually replayed.

The project is also being developed as a technical portfolio piece demonstrating architecture, reasoning-engine design, testing strategy, and iterative product development.

---

## ✨ Highlights

- Human-explainable logical deductions
- Modular solving strategy pipeline
- Stateless line evaluation
- Separation between solving logic and user interfaces
- Deterministic, testable solving behavior
- Support for move explanations and future replay
- Regression testing built from real puzzle-solving cases
- Architecture designed to support additional puzzle types over time
- Private production implementation with a public technical showcase

---

## 🧩 Why an Explainable Solver?

Many puzzle solvers are primarily concerned with one question:

> Can the puzzle be solved?

The Solver asks additional questions:

- **Why is this move valid?**
- **What logical rule produced it?**
- **What information was available at the time?**
- **Can the same decision be reproduced later?**
- **Can the reasoning be shown to a human player?**

Those questions shape the architecture.

The goal is not merely to produce a completed board. The goal is to create a solving engine whose decisions can be inspected and understood.

---

## 🏗️ Architecture

The Solver separates puzzle intelligence from the applications that may eventually consume it.

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
Board Update + Explanation
       |
       +------> Solve Transcript
       |
       +------> Future Clients
                 |
                 +--> CLI
                 +--> Web
                 +--> Mobile
```

The **solver engine is the product**.

Command-line, web, and mobile interfaces are clients of the engine rather than places where solving logic lives.

This separation keeps the core logic easier to test, reuse, and evolve.

More detailed architecture documentation will be maintained in:

`docs/architecture.md`

---

## 🧠 Solving Model

Nonograms are solved by repeatedly evaluating rows and columns using the information currently known about the board.

At a high level:

1. The board orchestrator selects a line for evaluation.
2. The line evaluator receives:
   - the line length
   - the puzzle clues
   - the current known cell state
3. Applicable deduction strategies analyze the line.
4. The evaluator proposes only logically supported changes.
5. The board applies those changes.
6. Affected lines become candidates for reevaluation.
7. The process repeats until:
   - the puzzle is solved,
   - no further deduction is available, or
   - another solving capability is required.

Individual solving strategies remain independent of board-level coordination wherever practical.

This keeps reasoning units small and testable.

---

## 🔎 Explainability

A solving move is more useful when the engine can explain what caused it.

Conceptually, a move can include information such as:

```text
Line: Row 8
Previous state: ....#..........
Updated state:  ...###.........

Reason:
The known filled cell belongs to a clue whose legal positions
overlap across these cells.
```

The long-term goal is for solving activity to support:

- move-by-move explanations
- deterministic replay
- solve transcripts
- debugging and regression analysis
- educational or teacher-style presentation

The explanation model is therefore treated as part of the engine design rather than a UI-only feature.

---

## 🧪 Testing Strategy

Development is driven heavily by small regression cases extracted from real puzzle-solving situations.

Each case captures information such as:

- line length
- clues
- current line state
- expected deduction
- expected resulting state

This approach allows individual deduction behaviors to be tested independently before they are integrated into full-board solving.

Real puzzles are also used as higher-level benchmarks.

A difficult Nonogram nicknamed **Poppies** has served as an important development benchmark because it contains a variety of deduction situations that exposed gaps in earlier versions of the evaluator.

The current solver logic has progressed far enough to complete that benchmark through the project's manual proof-of-concept workflow.

---

## 🧰 Technology Stack

### Core

- **Python**
- Modular solver-engine architecture
- Structured puzzle and line-state representations

### Engineering

- Git
- GitHub
- Regression-driven development
- Incremental proof-of-concept testing

### Planned Clients

- Command-line interface
- Web portfolio demonstration
- Mobile application

The clients will consume the solver engine rather than duplicate solving logic.

---

## 🗂️ Project Structure

The public portfolio repository intentionally contains documentation rather than the full solver implementation.

```text
the_solver_portfolio/
├── README.md
├── docs/
│   └── architecture.md
└── examples/
    └── ...
```

The `examples` area may contain sanitized solving examples and transcripts as the public showcase evolves.

---

## 🔒 Source Availability

The production Solver implementation is maintained in a **private repository**.

This public repository serves as a technical portfolio and project showcase documenting the project's:

- architecture
- engineering decisions
- solving model
- testing strategy
- development milestones
- explainability goals

Detailed deduction algorithms, proprietary solving logic, internal test assets, and production source code are intentionally not included.

---

## 📌 Project Status

**Active development**

Current development has focused primarily on the Nonogram solving engine and line-evaluation logic.

Completed or demonstrated capabilities include:

- stateless line evaluation
- multiple deterministic deduction strategies
- regression cases derived from real puzzles
- explanation-oriented move design
- successful completion of the Poppies benchmark through the manual proof-of-concept workflow

The next major architectural step is broader board-level orchestration so that line deductions can drive a complete automated solve loop.

---

## 🗺️ Roadmap

Planned work includes:

- Board Orchestrator implementation
- automated row/column reevaluation
- move transcript and replay support
- richer explanation metadata
- expanded regression coverage
- CLI client
- lightweight web portfolio demonstration
- teacher-style explanation mode
- exploration of additional logic-puzzle modules

---

## 👩‍💻 About This Project

The Solver began with a deceptively simple question:

> Can a puzzle solver reason in a way that feels understandable to a human?

That question evolved into a broader software engineering exercise involving:

- rule-engine design
- state management
- orchestration
- explainability
- regression testing
- architectural separation
- product thinking

The project is intentionally being built as more than a puzzle script.

The goal is a reusable solving engine whose behavior can be tested, inspected, explained, and eventually taught.