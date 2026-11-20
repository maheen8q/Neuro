# Neuro

An AI-assisted code review and rewriting pipeline. You submit code. Neuro examines it, uses an LLM to rewrite it, runs the result through security and efficiency checks, and hands back the final code together with a report of everything it checked and what it found.

> **Status:** Neuro is moving from NeuroCode, a fixed catalog of hardcoded algorithm patterns for competitive programming, to a general pipeline built around an LLM and security checks. Sections marked *(planned)* describe the target design, not what is implemented today.

## What changed

| Removed | Added |
|---|---|
| Hardcoded algorithm patterns and template-based optimizer passes | LLM connection for rewriting answers *(planned)* |
| Scope limited to competitive programming | Company-level security checks *(planned)* |

The earlier version recognized a small catalog of classic problems (pair sum, sorted search, range sum, Fibonacci, k smallest) and rewrote them from templates. That worked for those shapes only, and only for contest-style code. Neuro replaces the catalog with an LLM and keeps the parts that make a rewrite trustworthy: analysis, sandboxed execution and checking.

## How it works *(planned)*

```
Code ──▶ Examine ─▶ LLM rewrite ─▶ Security checks ─▶ Efficiency ─▶ Checklist + correctness ─▶ Final code
                                                                                          └──▶ Report of work
```

1. **Examine.** Parse the code (libclang AST and control flow graph) and derive time and space complexity by rule.
2. **Rewrite.** Send the code and the findings to an LLM and get back a candidate rewrite.
3. **Security checks.** Check the candidate against security rules before anything else runs.
4. **Efficiency.** Measure the candidate against the original on real runs in the sandbox.
5. **Checklist and correctness.** Record every check that ran and whether the candidate's output matches the original's.
6. **Output.** Return the final code and a report listing every check and its result.

In short: input → problem solve → output. The report is part of the product, so a reader can see what was verified rather than trusting the rewrite.

## Honesty policy for results

These rules carry over from NeuroCode and still apply:

- A rewrite is only shown as final if it compiled and passed the checks. If no candidate passes, the user sees that, not an unverified guess.
- A bug is reported as a bug only when a counterexample exists. Anything else is labelled a "suspicious pattern".
- Speedups are measured on real runs against the user's own original code, not quoted from complexity classes.
- Testing finds counterexamples. It never proves their absence.
- LLM output is a candidate, never a source of truth. Rule-derived analysis and measured results take precedence.

## Stack

| Layer | Technology |
|---|---|
| Frontend | React 18 + TypeScript + Vite, Monaco Editor (with diff view), Tailwind CSS |
| Backend | C++20 + Drogon (HTTP and WebSocket) |
| Parsing | libclang (real Clang AST) |
| Sandbox | Forked subprocess with POSIX rlimits, so untrusted code never runs in the API process |
| Database | SQLite |
| LLM | Provider to be decided *(planned)* |
| Tests | GoogleTest (C++), Vitest (frontend) |

The trained algorithm-classification model (LibTorch) belongs to the NeuroCode build. Whether it stays is open now that the hardcoded catalog is gone.

## Repository layout

```
neuro/
├── README.md
├── docs/
│   ├── architecture.md
│   └── api-spec.md
├── frontend/        React + TypeScript
└── backend/         C++20, Drogon
```

## Getting started

Prerequisites: CMake 3.20+, a C++20 compiler (GCC 11+ or Clang 14+), Drogon 1.9+, SQLite3, nlohmann/json, libsodium, libclang-dev, Node.js 18+. Linux or WSL is strongly recommended. LibTorch 2.x (CPU build) is only needed while the classifier is still in the build.

Backend:

```
cd backend
cmake -B build -DCMAKE_PREFIX_PATH=/path/to/libtorch
cmake --build build -j
./build/neurocode        # http://localhost:8080
```

Frontend:

```
cd frontend
npm install
npm run dev              # http://localhost:5173
```

Check the stack: `curl http://localhost:8080/api/v1/health`

## Open questions

These need answers before the design is final:

- **Why Neuro?** What does it do that an LLM on its own does not, and is it more than "another AI"?
- **MCP.** If Neuro is offered as an MCP server, what does an agent gain by calling it?
- **Vibecoders.** Do people who generate code with AI need this, and what do they need in the first place?
- **Audience.** A survey will show what to actually solve. Neuro should be built for a specific audience, and the survey results can also feed content.

## Known gaps

- The LLM rewrite step and the security checks are not implemented yet.
- The server does not shut down gracefully on interrupt (pure virtual method called). Normal operation is unaffected.
- Job history is kept in memory and is lost on server restart.

## Team

Two people: Maheen, Hassan.
