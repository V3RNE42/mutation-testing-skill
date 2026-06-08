# Mutation Testing — Quick Reference

## Core Formula
```
Score = Killed / (Total - Equivalent) × 100
```

## Decision Tree: Which Tool?

```
Language?
├── Java/Kotlin → PIT (pitest.org)
├── C# (.NET)   → Stryker.NET (dotnet-stryker) ← only active tool
├── JS/TS       → StrykerJS (stryker-mutator.io) ← only production-grade tool
├── Scala       → Stryker4s
├── Python      → MutMut (modern) / MutPy (classic)
├── Rust        → cargo-mutants / mutagen
├── Ruby        → mutant
├── Go          → go-mutesting
└── Swift       → Muter (emerging)
```

## Quick Commands

| Language | First Run | Install |
|---|---|---|
| **C#** | `dotnet stryker` | `dotnet tool install -g dotnet-stryker` |
| **JS/TS** | `npx stryker run` | `npm i -D @stryker-mutator/core` |
| **Java** | `mvn pitest:mutationCoverage` | Add `pitest-maven` plugin to `pom.xml` |
| **Python** | `mutmut run` | `pip install mutmut` |

## CI Strategy Decision

```
Codebase size?
├── Small (<10K LOC) → Full suite nightly + incremental per PR
├── Medium (10-100K) → Critical modules only in PR, full nightly
└── Large (>100K)    → Diff-aware per PR, sampling in nightly
```

## Quick CI Examples

### C# — GitHub Actions (incremental + baseline)
```yaml
- name: Mutation testing
  run: |
    dotnet tool install --tool-path .stryker dotnet-stryker
    .stryker/dotnet-stryker --since:main --with-baseline:main --break-at 60
```

### C# — Azure DevOps
```yaml
- script: dotnet tool install --tool-path .stryker dotnet-stryker
  displayName: 'Install Stryker'
- script: .stryker/dotnet-stryker --break-at 60
  displayName: 'Mutation testing'
- task: PublishMutationReport@1
  inputs:
    reportPattern: '**/mutation-report.json'
```

### JS/TS — GitHub Actions (incremental)
```yaml
- name: Mutation testing
  run: |
    CHANGED=$(git diff --name-only origin/main...HEAD)
    if [ -n "$CHANGED" ]; then
      npx stryker run --mutate "$CHANGED"
    fi
```

## C# Stryker.NET Config Quick Reference

```json
{
  "stryker-config": {
    "project": "MyProject.csproj",
    "mutation-level": "Standard",
    "thresholds": { "high": 80, "low": 60, "break": 0 },
    "ignore-methods": ["*Log", "Console.Write*", "ConfigureAwait"],
    "ignore-mutations": ["string", "linq.First"]
  }
}
```

**Mutation levels:** Basic → Standard (default) → Advanced → Complete

## JS/TS StrykerJS Config Quick Reference

```json
{
  "$schema": "./node_modules/@stryker-mutator/core/schema/stryker-schema.json",
  "mutate": ["src/**/*.ts", "!src/**/*.test.ts"],
  "testRunner": "vitest",
  "thresholds": { "high": 80, "low": 70, "break": true },
  "concurrency": 4,
  "coverageAnalysis": "perTest",
  "checkers": ["typescript"],
  "incremental": true
}
```

## PIT Mutator Groups

| Group | Use Case |
|---|---|
| DEFAULTS | Everyday CI — stable, few equivalents |
| STRONGER | Deep audit — adds REMOVE_CONDITIONALS |
| ALL | Research — adds AOR, ROR, UOI (many trivial kills) |

## Typical Coverage → Score Mapping

| Line Coverage | Mutation Score (decent tests) | Mutation Score (weak tests) |
|---|---|---|
| 90%+ | 60-70% | 30-40% |
| 80% | 50-60% | 20-30% |
| 60% | 35-50% | 10-20% |

## Language-Specific Pitfalls (TL;DR)

| Language | #1 Pitfall | Mitigation |
|---|---|---|
| **C#** | IL weavers (PostSharp, Fody) inject post-compile | `ignore-methods` for weaved methods |
| **C#** | Async/await state machines produce unreachable mutants | `ignore-methods` with `*MoveNext*` |
| **JS/TS** | Mock-heavy tests kill few mutants | Prefer integration tests over mocks |
| **JS/TS** | Snapshot tests pass most mutations | Write explicit assertions |
| **Java** | Lombok generates code PIT can mutate | Exclude generated methods |
| **Python** | Dynamic typing → many trivial mutants survive | Use type hints, focus on logic-heavy code |

## Reference Docs

- C# (.NET) deep dive: `references/dotnet-mutation-testing.md`
- TypeScript/JS deep dive: `references/typescript-mutation-testing.md`
- Java/Kotlin deep dive: `references/java-mutation-testing.md`
- Python deep dive: `references/python-mutation-testing.md`
- Key papers: `references/key-papers.md`
