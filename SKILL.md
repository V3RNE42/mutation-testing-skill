---
name: mutation-testing
description: "Comprehensive reference on mutation testing: theory, tools, operators, CI/CD integration, adoption strategies, and common pitfalls. Covers Java, C#, TypeScript/JavaScript, Python, and more. Deep-dive references in ./references/."
---

# Mutation Testing

## Overview

Mutation testing is a **fault-based testing technique** that measures test *quality* — not just what lines execute (coverage), but whether tests can actually detect bugs. It is the gold standard of test metrics.

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

| USE when you need to know | DO NOT USE for |
|---|---|
| "How do I know if my tests are actually good?" | General testing strategy advice |
| Mutation testing tools (PIT, Stryker, MutMut) | Test automation framework selection |
| "What is mutation score / mutation coverage" | Performance/load testing |
| Comparing coverage vs mutation metrics | Manual test case design |
| CI/CD integration of mutation testing | Code review (mutation finds test gaps, not design flaws) |
| Equivalent mutants, HOM, selective mutation | |
| Practical "how to start" guide | |

## Reference Files (load on demand)

| File | Load with | Contents |
|---|---|---|
| `references/dotnet-mutation-testing.md` | `skill_view(name='mutation-testing', file_path='references/dotnet-mutation-testing.md')` | Stryker.NET: install, config, 19 mutator categories with C# examples, mutation levels, CI/CD, performance, .NET-specific pitfalls, equivalent mutant patterns, Stryker.NET vs StrykerJS comparison |
| `references/typescript-mutation-testing.md` | `skill_view(name='mutation-testing', file_path='references/typescript-mutation-testing.md')` | StrykerJS: install, config, 15+ mutator categories with TS examples, test runners, TS checker, CI/CD, incremental mode, TS-specific pitfalls, React component testing, tools comparison |
| `references/quick-reference.md` | `skill_view(name='mutation-testing', file_path='references/quick-reference.md')` | Cheat sheet: commands, config snippets, CI examples, coverage-score mapping |
| `references/key-papers.md` | `skill_view(name='mutation-testing', file_path='references/key-papers.md')` | Academic papers, resources, reading order |

## I — Theory: The Three Pillars

### Pillar 1: Strong vs Weak vs Firm

| Type | Condition to Kill | Cost | Used By |
|---|---|---|---|
| **Strong** | Test assertion fails (output differs) | Highest — full test run per mutant | PIT, Stryker (default) |
| **Weak** | State *immediately after mutation* differs | Lower — check intermediate state | Academic tools |
| **Firm** | State propagates partway | Intermediate | Hybrid approaches |

### Pillar 2: Mutation Score

```
Mutation Score = Killed / (Total - Equivalent) × 100
```

| Score | Meaning | Action |
|---|---|---|
| < 60% | Tests are weak | Write meaningful assertions, cover edge cases |
| 60-80% | Acceptable | Target survivors with highest risk |
| 80-90% | Good | Review remaining survivors for equivalent mutants |
| > 90% | Excellent | Verify no trivial tests inflating score |

**The coverage-mutation gap:** 90% line coverage typically yields only 60-70% mutation score with decent tests, or 30-40% with weak tests. This gap is *expected*.

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

#### Per-Language Mutator Tables

See the corresponding reference file for language-specific mutators:

| Language | Tool | Reference | Mutator Highlights |
|---|---|---|---|
| Java | PIT | `references/pit-mutation-testing.md` *(forthcoming)* | DEFAULTS / STRONGER / ALL groups, bytecode-level |
| C# | Stryker.NET | `references/dotnet-mutation-testing.md` | 19 categories: LINQ (30+ pairs), Checked, String methods, Math methods, Null-coalescing, Regex, Collection expressions (C#12) |
| JS/TS | StrykerJS | `references/typescript-mutation-testing.md` | 15+ categories: Arithmetic, Equality, Optional chaining, Array, Object literal, RegExp, Block, Assignment |

## II — Higher Order Mutation Testing (HOM)

Introduced by Harman et al. (2009): FOM = 1 mutation. HOM = 2+ combined.

- **Subsuming HOM** — harder to kill than any component FOM. Represents subtle real-world bugs.
- **Masked HOM** — one mutation masks another; survives despite weaker individual mutations.

**Practical caveat:** HOM is research-advanced. FOM with good defaults catches 90%+. Only explore HOM for deep auditing of critical modules.

## III — Tool Selection

```
Language?
├── Java/Kotlin → PIT (pitest.org), pro: arcmutate
├── C# → Stryker.NET (dotnet-stryker) ← only active tool
├── JavaScript/TS → StrykerJS ← only production-grade tool
├── Scala → Stryker4s
├── Python → MutMut (modern) or MutPy (classic)
├── Rust → cargo-mutants or mutagen
├── Ruby → mutant
├── Go → go-mutesting
└── Swift → Muter (emerging, ⚠️ low activity)
```

### Quick Install

```bash
npm i -D @stryker-mutator/core                 # JS/TS
pip install mutmut                              # Python
dotnet tool install -g dotnet-stryker           # C# (.NET)
# Java: add pitest-maven plugin to pom.xml
```

### Per-Language Deep Dives

| Language | Status | Install | Config file | Key feature |
|---|---|---|---|---|
| **C#** | ✅ Active — Microsoft-recommended | `dotnet tool install -g dotnet-stryker` | `stryker-config.json` | Mutant schemata (no per-mutant recompilation). Mutation levels (Basic/Standard/Advanced/Complete). 19 mutator categories including LINQ, Checked, String methods. |
| **TypeScript/JS** | ✅ Active — 6K★, 150K+ npm/week | `npm i -D @stryker-mutator/core` | `stryker.config.json` | AST-level instrumentation. TS checker plugin. Incremental mode. Vitest/Jest/Mocha support. JSX/TSX mutation. |
| **Java** | ✅ Active — most mature | PIT Maven plugin | `pom.xml` | Bytecode-level. DEFAULTS/STRONGER/ALL. Longest track record. |

See the corresponding reference in `references/` for full details (install, config, mutators, CI/CD, performance, pitfalls).

## IV — Configuration Patterns by Goal

| Goal | PIT (Java) | StrykerJS (JS/TS) | Stryker.NET (C#) | MutMut (Python) |
|---|---|---|---|---|
| **First run (measure)** | No config needed | `npx stryker run` | `dotnet stryker` | `mutmut run` |
| **Focused (module)** | `targetClasses: ["core.*"]` | `"mutate": ["src/core/**"]` | `"mutate": ["src/core/**/*.cs"]` | `paths_to_mutate = src/core/` |
| **CI gate** | `mutationThreshold: 80` | `thresholds: {high:80, low:70, break:true}` | `thresholds: {high:80, low:60, break:0}` via `--break-at` | N/A (post-process) |
| **Full audit** | mutators: `STRONGER` | `"mutators": ["all"]` | `mutation-level: "Advanced"` | Default is full |
| **Incremental** | N/A | `"incremental": true` | `"since": {"enabled": true, "target": "main"}` | N/A |
| **Ignore patterns** | `excludedMethods` | `mutate: ["!**/*.spec.ts"]` | `ignore-mutations`, `ignore-methods` | `paths_to_mutate` exclusion |

Config examples for each tool: see `references/quick-reference.md`.

## V — Execution Workflow

### Step 1: Baseline

```bash
npx stryker run         # JS/TS
dotnet stryker          # C#
mvn pitest:mutationCoverage  # Java
mutmut run              # Python
```

### Step 2: Interpret Survivors

| Survivor Pattern | Root Cause | Fix |
|---|---|---|
| Boundary conditions survive (`<` vs `<=`) | No edge case tests | Add boundary value tests |
| Logical operators survive (`&&` vs `\|\|`) | Branch not covered | Test each branch |
| Return values survive | Assertions too loose | Assert exact values |
| Method calls removed survive | Side effects not verified | Assert state changes |
| Conditionals removed survive | Dead code or guard not tested | Test or remove |

Language-specific survivor patterns (LINQ, null-coalescing, optional chaining, etc.): see corresponding `references/` file.

### Step 3: Improve Tests

1. Write test targeting the mutated behavior — passes on original code
2. Re-run mutation testing — mutant should be killed
3. If it survives — check for equivalent mutant

### Step 4: Validate & Lock in CI

```
Post-run feedback loop:
  Score improved?   → Continue strategy
  Score regressed?  → Check new code without tests
  Unexpected EMs?   → Add to exclusion list
  Repeated pattern? → Fix root cause in test design
```

## VI — CI/CD Integration

### Priority

```
[Best]   Incremental per PR   → changed files only. Seconds-minutes.
[Better] Critical module gate → full mutation on core. Minutes.
[Good]   Nightly full suite   → trend tracking. Hours. No blocking.
[Worst]  Full suite every PR  → only for tiny codebases.
```

### Thresholds by Module Risk

| Risk Level | Min | Target | Break CI? |
|---|---|---|---|
| Critical (finance, auth, health) | 80% | 90%+ | Yes |
| Core (business logic) | 60% | 80% | Gradual |
| Infra / glue | 40% | 60% | Report only |
| Legacy | Baseline | — | No |

### CI Examples by Language

See `references/quick-reference.md` or the per-language reference:

- **C#:** `references/dotnet-mutation-testing.md` — GitHub Actions (basic + incremental + baseline), Azure DevOps
- **JS/TS:** `references/typescript-mutation-testing.md` — GitHub Actions (incremental), GitLab CI, CircleCI, dashboard + badges
- **Java:** PIT docs

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

Language-specific equivalent mutant patterns: see `references/dotnet-mutation-testing.md` (C#) and `references/typescript-mutation-testing.md` (TS).

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
| Production defect reduction | 25-30% | Multi-study surveys |
| CI overhead (large projects) | 15-20% | Practice reports |
| Technical debt reduction | 35% | Adopter surveys |
| Initial velocity impact | 10-15% (temporary) | Rollout studies |

**Adopters:** Google (internal), Meta (core modules), Spotify (StrykerJS), Microsoft (Stryker.NET — official docs), ING, Philips, Adyen, Angular, NestJS, Polly, ABP Framework, The Ladders, BSkyB.

## IX — Key Papers

See `references/key-papers.md` for full details and reading order.

| Year | Title | Lead | Contribution |
|---|---|---|---|
| 1978 | *Hints on Test Data Selection* | DeMillo et al. | Founding paper |
| 2009 | *Higher Order Mutation Testing* | Harman et al. | HOMs, subsuming mutants |
| 2018 | *Mutation Testing Advances* | Papadakis et al. | Definitive survey |
| 2024 | *Comparison of Python MT Tools* | ACM | CosmicRay, MutPy, MutMut, Mutatest |
| 2024 | *LLMs for Equivalent Mutant Detection* | Zhao Tian et al. (ISSTA) | LLM classification |
| 2024 | *Mutation Testing in Practice* | IEEE | Empirical OSS study |

## X — Pitfalls

### General (all languages)

1. **Full suite every PR** — use incremental. Full runs = nightly.
2. **Ignoring equivalent mutants** — penalizes score unfairly.
3. **Chasing 100%** — impossible (EMs). 85% solid > 100% trivial.
4. **No baseline** — can't track improvement/regression.
5. **No CI threshold** — scores drift down silently.
6. **ALL mutators on every build** — DEFAULTS/Standard are enough.
7. **Confusing coverage with mutation** — 90% coverage != 90% score.
8. **Thinking mutation replaces code review** — different concerns.
9. **No survivor review cadence** — weekly recommended.
10. **Starting too late** — day 1 > month 1 > never.

### Per-Language Pitfalls

| Language | #1 Pitfall | Mitigation |
|---|---|---|
| **C#** | IL weavers (PostSharp, Fody) — aspects injected post-compile | `ignore-methods` for weaved methods |
| **C#** | Async/await state machines — unreachable mutants | `ignore-methods` with `*MoveNext*` |
| **C#** | Source generators — generated code not available | Exclude `**/*.g.cs`, `**/Migrations/*` |
| **C#** | LINQ expression trees — mutations can't apply | Exclude heavy LINQ chains |
| **TS/JS** | Mock-heavy tests kill few mutants | Prefer integration tests |
| **TS/JS** | Snapshot tests pass most mutations | Write explicit assertions |
| **TS/JS** | StrykerJS v9 requires Node 22+ | Verify Node version in CI |
| **TS/JS** | Type checker is slow | Use selectively on critical modules |
| **Java** | Lombok generated code | Exclude generated methods |

Full per-language pitfalls with examples: see `references/dotnet-mutation-testing.md` and `references/typescript-mutation-testing.md`.

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
