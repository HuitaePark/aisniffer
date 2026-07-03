# AISniffer

**AISniffer** is a context-aware static analysis tool for detecting **AI-smell signals** in pull requests.

AISniffer does **not** determine whether code was written by AI.
Instead, it detects code patterns often seen in generated or copy-pasted code, such as contextless implementation, shallow exception handling, superficial tests, and project convention drift.

> AI usage is not the problem.
> Merging unverified, contextless code is.

## What is AI Smell?

**AI smell** is not proof of AI authorship.

It is a practical term for maintainability and review risks that often appear in generated code:

* contextless implementation
* shallow exception handling
* superficial tests
* project convention drift
* unclear responsibility boundaries
* generic error handling
* missing domain-specific validation
* code that looks correct but does not fit the existing codebase

AISniffer treats these patterns as **review signals**, not accusations.

## Why AISniffer?

Modern AI coding tools can generate working code quickly.
However, generated code may still miss project-specific context:

* existing exception models
* DTO and mapper conventions
* service-layer responsibility boundaries
* test strategies
* logging policies
* domain rules
* security assumptions
* architectural constraints

Traditional linters and static analyzers are good at detecting predefined rule violations.

AISniffer focuses on a different question:

> Does this pull request fit the context of the existing project?

## What AISniffer Does

AISniffer analyzes pull request changes and reports review signals such as:

* direct use of generic exceptions
* exception handling that differs from the existing project pattern
* manual validation logic where the project usually uses validation annotations
* service methods that mix validation, mapping, persistence, and response creation
* new success-path tests without meaningful failure-path tests
* naming or package-structure drift from the existing codebase
* code that appears isolated from nearby project conventions

AISniffer should not comment:

> This code was written by AI.

Instead, it should comment:

> This change appears to bypass the project’s existing exception model. Please review whether it should use the existing `BusinessException` and `ErrorCode` pattern.

## Project Goals

AISniffer aims to provide:

1. **Context-aware static analysis**
   Compare pull request changes with patterns already present in the project.

2. **Explainable review signals**
   Show why a piece of code may require additional review.

3. **AI-smell risk scoring**
   Combine multiple weak signals into a review priority score.

4. **PR-friendly feedback**
   Produce concise comments that help reviewers focus on risky changes.

5. **LLM-free core analysis**
   Keep the core analysis deterministic and reproducible.
   LLMs may be used later only as an optional comment rewriting layer.

## Non-Goals

AISniffer does not aim to:

* prove that code was written by AI
* punish developers for using AI tools
* replace human code review
* replace tests
* replace Checkstyle, PMD, SpotBugs, Error Prone, or other static analyzers
* make final architectural decisions

AISniffer is a review assistant, not a judge.

## How It Differs from Traditional Static Analysis

Traditional static analyzers usually detect predefined style, bug, or quality issues.

AISniffer focuses on **context deviation**.

For example, a traditional rule may say:

> Do not throw `RuntimeException` directly.

AISniffer should explain:

> This project appears to use `BusinessException` with `ErrorCode` for service-layer failures.
> This pull request introduces a direct `RuntimeException`, which may bypass the project’s standard error response handling.

The goal is not only to detect a pattern, but to explain why it matters in this specific project.

## Planned Architecture

```text
aisniffer/
├── aisniffer-core
│   ├── parser
│   ├── rule-engine
│   ├── project-profiler
│   ├── scoring
│   └── report
│
├── aisniffer-cli
│   └── local static analysis
│
└── aisniffer-action
    ├── pull request diff analysis
    ├── review comment generation
    └── GitHub Actions integration
```

## Planned Rule Categories

### 1. Context Ignorance

Detects code that does not follow existing project patterns.

Examples:

* different exception model
* different DTO naming style
* different mapper usage
* inconsistent package structure

### 2. Shallow Exception Handling

Detects vague or overly generic error handling.

Examples:

* direct `RuntimeException`
* broad `catch (Exception e)`
* generic error messages
* missing project-specific error codes

### 3. Responsibility Blur

Detects methods that mix too many responsibilities.

Examples:

* validation, mapping, persistence, and response creation in one service method
* controller directly accessing repository
* domain logic placed in the wrong layer

### 4. Superficial Tests

Detects weak test signals.

Examples:

* only happy-path tests
* no failure-path tests for newly added behavior
* broad `assertThrows(Exception.class)`
* tests that verify existence rather than behavior

### 5. Maintainability Drift

Detects code that appears isolated from the surrounding codebase.

Examples:

* inconsistent naming
* generic helper methods
* redundant comments
* boilerplate-heavy implementation
* low alignment with nearby code style

## Example Review Comment

```markdown
### AISniffer: Generic Exception Handling

This change throws `RuntimeException` directly.

The existing service layer appears to use `BusinessException` with `ErrorCode`
for domain failures. This change may bypass the project’s standard error
response handling.

Please review whether this should follow the existing exception model.
```

## Example Report

```text
AISniffer Review Report

Review Risk: Medium

Detected Signals:
- Generic exception handling
- Project exception model deviation
- Missing failure-path test

Recommendation:
This pull request should receive focused review on error handling and test coverage.
```

## Philosophy

AISniffer is based on one principle:

> AI usage is fine. Unverified, contextless code is not.

Developers may use AI tools.
However, the final code should still be understandable, reviewable, testable, and aligned with the project.

AISniffer helps reviewers focus on code that may need additional attention before merge.

## Status

This project is currently in the idea and design phase.

Planned MVP:

* Java source analysis
* basic rule engine
* Markdown report output
* GitHub Actions integration
* pull request summary comments

## Roadmap

### Phase 1: Core Analyzer

* Parse Java source files
* Detect basic AI-smell signals
* Generate local Markdown reports

### Phase 2: CLI

* Provide `aisniffer scan`
* Support configurable rule severity
* Support project-level configuration

### Phase 3: GitHub Actions

* Analyze pull request diffs
* Post summary comments
* Report review risk score

### Phase 4: Context Profiling

* Learn existing project exception patterns
* Detect naming and package drift
* Compare new changes with nearby code conventions

### Phase 5: Optional LLM Rewriting

* Use LLMs only to rewrite comments
* Keep analysis deterministic
* Never invent findings beyond static analysis results

## License

TBD
