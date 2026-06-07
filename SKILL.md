---
name: mutation-testing
description: "Use when asked about mutation testing, test quality measurement, mutation score, mutation operators, equivalent mutants, or integrating mutation testing into CI/CD pipelines."
version: 2.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [mutation-testing, testing, software-quality, code-coverage, test-automation]
    related_skills: [code-quality-workflow, systematic-debugging, better-prompt]
---

# Mutation Testing

## Overview — What This Skill Covers

Mutation testing is a **fault-based testing technique** that measures test *quality* — not just what lines execute (coverage), but whether tests can actually detect bugs. It is the gold standard of test metrics.

**The chain of reasoning this skill builds:**

```
Code Coverage (quantity)
     ↓  is insufficient because
Coverage = "lines executed" ≠ "faults detected"
     ↓  enter
Mutation Testing (quality)
     ↓  measures
Mutation Score = Killed / (Total - Equivalent) × 100
     ↓  which drives
Test improvement → CI thresholds → Production defect reduction
```

## When to Use / When Not

| USE when user asks about | DO NOT USE for |
|---|---|
| "How do I know if my tests are actually good?" | General testing strategy advice |
| Mutation testing tools (PIT, Stryker, MutMut) | Test automation framework selection |
| "What is mutation score / mutation coverage" | Performance/load testing |
| Comparing coverage vs mutation metrics | Manual test case design |
| CI/CD integration of mutation testing | Code review (mutation finds test gaps, not design flaws) |
| Equivalent mutants, HOM, selective mutation | |
| Practical "how to start" guide | |

## I — Theory: The Three Pillars

### Pillar 1: Strong vs Weak vs Firm

The category determines **what condition** qualifies as "killed". This affects tool choice and cost:

| Type | Condition to Kill | Cost | Used By |
|---|---|---|---|
| **Strong** | Test assertion fails (output differs) | Highest — full test run per mutant | PIT, Stryker (default) |
| **Weak** | State *immediately after mutation* differs | Lower — check intermediate state | Academic tools |
| **Firm** | State propagates partway | Intermediate | Hybrid approaches |

**Relationship to tool choice:** Strong is the gold standard but expensive. Weak is cheaper but less faithful. PIT/Stryker optimize strong with bytecode/AST-level efficiency.

### Pillar 2: Mutation Score

```
Mutation Score = Killed / (Total - Equivalent) × 100
```

| Score | Meaning | Action |
|---|---|---|
| < 60% | Tests are weak | Write meaningful assertions, cover edge cases |
| 60–80% | Acceptable | Target survivors with highest risk |
| 80–90% | Good | Review remaining survivors for equivalent mutants |
| > 90% | Excellent | Verify no trivial tests inflating score |

**The coverage-mutation gap:** 90% line coverage typically yields only 60-70% mutation score with decent tests, or 30-40% with weak tests. This gap is *expected* — don't panic, improve systematically.

### Pillar 3: Mutation Operators

#### Classic Taxonomy (MOTHRA, FORTRAN — 22 operators)

| Category | Operator | Example | What It Tests |
|---|---|---|---|
| Arithmetic | AOR | `+` → `-`, `*` → `/` | Math correctness |
| Relational | ROR | `<` → `<=`, `>` → `==` | Boundary conditions |
| Logical | LCR | `&&` → `\|\|` | Boolean logic |
| Unary | UOI | `x` → `-x` | Sign/bit handling |
| Absolute | ABS | `x` → `abs(x)`, `0` | Sign assumptions |
| Statement | SDL | Delete a line | Dead/unreachable code |
| Constant | CRP | `10` → `0`, `1`, `-1` | Magic number assumptions |

#### PIT (Java — bytecode level, grouped by stability)

| Group | Mutator | Effect | Default |
|---|---|---|---|
| **DEFAULTS** | CONDITIONALS_BOUNDARY | `<` ↔ `<=`, `>` ↔ `>=` | ✅ |
| | INCREMENTS | `i++` ↔ `i--` (local vars only) | ✅ |
| | INVERT_NEGS | `-x` → `x` | ✅ |
| | MATH | `+` → `-`, `*` → `/`, `&` → `\|`, `<<` → `>>` | ✅ |
| | NEGATE_CONDITIONALS | `==` ↔ `!=`, `<` ↔ `>=` | ✅ |
| | VOID_METHOD_CALLS | Remove void method calls | ✅ |
| | EMPTY/FALSE/TRUE/NULL/PRIMITIVE returns | Mutate return values per type | ✅ |
| **STRONGER** | REMOVE_CONDITIONALS | Force if to always true/false | ❌ |
| | EXPERIMENTAL_SWITCH | Modify switch cases | ❌ |
| **ALL** | INLINE_CONSTS, CONSTRUCTOR_CALLS, NON_VOID_METHOD_CALLS, AOR, ROR, UOI, etc. | Full spectrum | ❌ |

**Design rationale:** PIT's defaults are "stable" — hard to detect trivially, few equivalent mutants. Classic AOR/ROR/UOI inflate scores with easy-to-kill mutants. Start with DEFAULTS, escalate to STRONGER for critical modules, ALL only for audits.

#### StrykerJS (JS/TS — 30+ mutators, source→AST)

| Category | Examples | Tests For |
|---|---|---|
| Arithmetic | `a + b` → `a - b`, `a * b` → `a / b` | Math logic |
| Equality | `===` → `!==`, `==` → `!=` | Type/equality |
| Logical | `&&` → `\|\|`, `\|\|` → `&&` | Boolean logic |
| String | `"abc"` → `""`, template literal mutations | String handling |
| Array | `arr.length` → `0` | Array access |
| Block | Remove entire code blocks | Dead/guard code |
| Optional chaining | `?.` → `.` | Null-safety assumptions |

## II — Higher Order Mutation Testing (HOM)

Introduced by Harman et al. (2009): FOM = 1 mutation. HOM = 2+ combined.

- **Subsuming HOM** — harder to kill than any component FOM. Represents subtle real-world bugs.
- **Masked HOM** — one mutation masks another; survives despite weaker individual mutations.

**Practical caveat:** HOM is research-advanced. FOM with good defaults catches 90%+. Only explore HOM for deep auditing of critical modules.

## III — Tool Selection

```
Language?
├── Java/Kotlin → PIT (pitest.org), pro: arcmutate
├── JavaScript/TS → StrykerJS
├── C# → Stryker.NET
├── Scala → Stryker4s
├── Python → MutMut (modern) or MutPy (classic)
├── Rust → cargo-mutants or mutagen
├── Ruby → mutant
├── Go → go-mutesting
└── Swift → Muter (emerging)
```

### Quick Install

```bash
npm i -D @stryker-mutator/core                 # JS/TS
pip install mutmut                              # Python
dotnet tool install -g dotnet-stryker           # C#
# Java: add pitest-maven plugin to pom.xml
```

## IV — Configuration Patterns by Goal

| Goal | PIT (Java) | StrykerJS (JS/TS) | MutMut (Python) |
|---|---|---|---|
| **First run (measure)** | No config needed | `npx stryker run` | `mutmut run` |
| **Focused (module)** | `targetClasses: ["core.*"]` | `"mutate": ["src/core/**"]` | `paths_to_mutate = src/core/` |
| **CI gate** | `mutationThreshold: 80` | `"thresholds": {"high":80,"low":70,"break":true}` | N/A (post-process) |
| **Full audit** | mutators: `STRONGER` | `"mutators": ["all"]` | Default is full |

### PIT — Maven

```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <configuration>
        <targetClasses><param>com.app.core.*</param></targetClasses>
        <targetTests><param>com.app.core.*Test</param></targetTests>
        <mutationThreshold>80</mutationThreshold>
        <coverageThreshold>85</coverageThreshold>
        <threads>4</threads>
    </configuration>
</plugin>
```

### StrykerJS

```json
{
  "$schema": "./node_modules/@stryker-mutator/core/schema/stryker-schema.json",
  "mutate": ["src/**/*.ts", "!src/**/*.test.ts"],
  "testRunner": "vitest",
  "reporters": ["html", "clear-text", "progress"],
  "thresholds": { "high": 80, "low": 70, "break": true },
  "concurrency": 4
}
```

### MutMut (Python)

```ini
[mutmut]
paths_to_mutate = src/
tests_dir = tests/
runner = pytest
backup = false
```

## V — Execution Workflow

### Step 1: Baseline

```bash
npx stryker run         # JS/TS
mvn pitest:mutationCoverage  # Java
mutmut run              # Python
```

Record score. This is your starting point.

### Step 2: Interpret Survivors — Pattern Recognition

| Survivor Pattern | Root Cause | Fix |
|---|---|---|
| Boundary conditions survive (`<` vs `<=`) | No edge case tests | Add boundary value tests |
| Logical operators survive (`&&` vs `\|\|`) | Branch not covered | Test each branch |
| Return values survive | Assertions too loose | Assert exact values |
| Method calls removed survive | Side effects not verified | Assert state changes |
| Conditionals removed survive | Dead code or guard not tested | Test or remove |

### Step 3: Improve Tests

1. Write test targeting the mutated behavior → passes on original code
2. Re-run mutation testing → mutant should be killed
3. If it survives → check for equivalent mutant

### Step 4: Validate & Lock in CI

```
Post-run feedback loop:
  Score improved?   → Continue strategy
  Score regressed?  → Check new code without tests
  Unexpected EMs?   → Add to exclusion list
  Repeated pattern? → Fix root cause in test design
```

## VI — CI/CD Integration

### Priority (cost vs value)

```
[Best]   Incremental per PR   → changed files only. Seconds-minutes.
[Better] Critical module gate → full mutation on core. Minutes.
[Good]   Nightly full suite   → trend tracking. Hours. No blocking.
[Worst]  Full suite every PR  → only for tiny codebases.
```

### GitHub Actions (StrykerJS, incremental)

```yaml
- name: Mutation testing (incremental)
  run: |
    CHANGED=$(git diff --name-only origin/main...HEAD \
      | grep '^src/' | tr '\n' ',' | sed 's/,$//')
    [ -n "$CHANGED" ] && npx stryker run --mutate "$CHANGED"
```

### Thresholds by Module Risk

| Risk | Min | Target | Break CI? |
|---|---|---|---|
| 🔴 Critical (finance, auth, health) | 80% | 90%+ | Yes |
| 🟡 Core (business logic) | 60% | 80% | Gradual |
| 🟢 Infra / glue | 40% | 60% | Report only |
| ⚪ Legacy | Baseline | — | No |

## VII — Hard Problems

### 1. Equivalent Mutants

```java
int x = a + 0;  →  int x = a;   // Equivalent — same semantics
```

**Detection (cheapest first):**
1. Compiler equivalence — compile both with `-O3`. Same bytecode? Skip.
2. Automated filtering — PIT/Stryker detect common patterns
3. LLM (ISSTA 2024) — ~85% accuracy. Pass survivors to classify.
4. Human review — last resort.

**Score correction:** Track `excluded = killed_equivalent + filtered_equivalent`. Score uses adjusted total.

### 2. Mutant Explosion

10K LOC → thousands of mutants.

| Strategy | How | Correlation | When |
|---|---|---|---|
| Sampling | Random 10% | ~0.98 | Exploratory |
| Selective | Only ABS, UOI, LCR, AOR, ROR | ~90% | CI |
| Diff-aware | Changed files only | Exact | Every PR |
| Test selection | Coverage-based filter | Depends | Large projects |

### 3. Performance Budget

| Scale | Mutants | Defaults | ALL | CI impact |
|---|---|---|---|---|
| 10K LOC | ~1K | ~3 min | ~15 min | Negligible |
| 100K LOC | ~8K | ~20 min | ~2 h | 15-20% |
| 1M LOC | ~50K | Hours | Days | Use incremental |

## VIII — Industry Evidence

| Metric | Value | Source |
|---|---|---|
| Production defect reduction | 25–30% | Multi-study surveys |
| CI overhead (large projects) | 15–20% | Practice reports |
| Technical debt reduction | 35% | Adopter surveys |
| Initial velocity impact | 10–15% (temporary) | Rollout studies |

**Adopters:** Google (internal), Meta (core modules), Spotify (StrykerJS), Microsoft (Stryker.NET), The Ladders (PIT — "90% score made coverage irrelevant"), BSkyB (PIT — "find redundant code").

## IX — Key Papers

| Year | Title | Lead | Contribution |
|---|---|---|---|
| 1978 | *Hints on Test Data Selection* | DeMillo et al. | Founding paper |
| 2009 | *Higher Order Mutation Testing* | Harman et al. | HOMs, subsuming mutants |
| 2018 | *Mutation Testing Advances* | Papadakis et al. | Definitive survey |
| 2024 | *Comparison of Python MT Tools* | ACM | CosmicRay, MutPy, MutMut, Mutatest |
| 2024 | *LLMs for Equivalent Mutant Detection* | Zhao Tian et al. (ISSTA) | LLM classification |
| 2024 | *Mutation Testing in Practice* | IEEE | Empirical OSS study |

## X — Pitfalls

1. **Full suite every PR** — use incremental. Full runs = nightly.
2. **Ignoring equivalent mutants** — penalizes score unfairly.
3. **Chasing 100%** — impossible (EMs). 85% solid > 100% trivial.
4. **No baseline** — can't track improvement/regression.
5. **No CI threshold** — scores drift down silently.
6. **ALL mutators on every build** — DEFAULTS are enough.
7. **Confusing coverage with mutation** — 90% coverage != 90% score.
8. **Thinking mutation replaces code review** — different concerns.
9. **No survivor review cadence** — weekly recommended.
10. **Starting too late** — day 1 > month 1 > never.

## Verification Checklist

- [ ] Tool selected and installed
- [ ] Baseline score measured and documented
- [ ] Thresholds configured (break/high/low)
- [ ] Scope limited to relevant modules
- [ ] CI: incremental per PR + nightly full run
- [ ] Equivalent mutant filtering active
- [ ] Survivor review cadence established
- [ ] Team trained on interpretation
- [ ] Dashboard/reporting active
- [ ] Coverage-mutation gap communicated to stakeholders
