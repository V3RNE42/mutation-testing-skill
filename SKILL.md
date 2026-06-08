---
name: mutation-testing
description: "Comprehensive reference on mutation testing: theory, tools, operators, CI/CD integration, adoption strategies, and common pitfalls. Covers Java, C#, TypeScript/JavaScript, Python, and more."
---

# Mutation Testing

## Overview

Mutation testing is a **fault-based testing technique** that measures test *quality* — not just what lines execute (coverage), but whether tests can actually detect bugs. It is the gold standard of test metrics.

**The chain of reasoning:**

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

#### PIT (Java — bytecode level)

See full reference: `references/pit-mutation-testing.md`

| Group | Mutator | Effect | Default |
|---|---|---|---|
| **DEFAULTS** | CONDITIONALS_BOUNDARY | `<` ↔ `<=`, `>` ↔ `>=` | Yes |
| | INCREMENTS | `i++` ↔ `i--` (local vars only) | Yes |
| | INVERT_NEGS | `-x` → `x` | Yes |
| | MATH | `+` → `-`, `*` → `/`, `&` → `\|`, `<<` → `>>` | Yes |
| | NEGATE_CONDITIONALS | `==` ↔ `!=`, `<` ↔ `>=` | Yes |
| | VOID_METHOD_CALLS | Remove void method calls | Yes |
| | EMPTY/FALSE/TRUE/NULL/PRIMITIVE returns | Mutate return values per type | Yes |
| **STRONGER** | REMOVE_CONDITIONALS | Force if to always true/false | No |
| **ALL** | Full spectrum including AOR, ROR, UOI | | No |

#### StrykerJS (JS/TS — 15+ mutator categories)

See full reference: `references/typescript-mutation-testing.md`

| Category | Examples | Tests For |
|---|---|---|
| Arithmetic | `a + b` → `a - b`, `a * b` → `a / b` | Math logic |
| Equality | `===` → `!==`, `==` → `!=` | Type/equality |
| Logical | `&&` → `\|\|`, `\|\|` → `&&` | Boolean logic |
| String | `"abc"` → `""`, template literal mutations | String handling |
| Array | `arr.length` → `0`, `arr[0]` → `undefined` | Array access |
| Block | Remove entire code blocks | Dead/guard code |
| Optional chaining | `?.` → `.` | Null-safety assumptions |
| Object literal | `{a:1}` → `{}` | Object shape |
| Assignment | `+=` → `-=` | Update logic |
| Unary | `i++` → `i--` | Increment/decrement |
| RegExp | `/abc/` → `/(?:)/` | Regex patterns |

#### Stryker.NET (C# — 19 mutator categories via mutant schemata)

See full reference: `references/dotnet-mutation-testing.md`

| Mutator Group | C# Examples | Tests For |
|---|---|---|
| Arithmetic | `+` → `-`, `*` → `/`, `%` → `*` | Math logic |
| Equality | `==` → `!=`, `>` → `<`, `>=` → `>` | Comparison logic |
| Boolean | `true` → `false`, `&&` → `\|\|` | Boolean flags |
| Assignment | `+=` → `-=`, `\|\|=` → `&&=` | Compound assignments |
| LINQ (30+ pairs) | `.First()` → `.FirstOrDefault()`, `.All()` → `.Any()` | Collection queries |
| String methods | `.Contains()` → `.IndexOf()`, `.Trim()` → empty | String logic |
| Math methods | `.Abs()` → empty, `.Floor()` → `.Ceiling()` | Math calls |
| Checked | `checked` block removal | Overflow detection |
| Null-coalescing | `??` → `?? throw`, `?? throw` → `??` | Null handling |
| String literal | `"abc"` → `""` | Empty string handling |
| Conditional | ternary `? :` removal | Branch logic |
| Regex | `new Regex("pattern")` → match-all | Regex patterns |
| Update | `i++` → `i--` | Increment/decrement |
| Unary | `-x` → `x` | Sign handling |
| Collection expressions (C#12) | `[1,2,3]` → `[]` | Collection initialization |

**Mutation Levels (Stryker.NET):**
| Level | Included |
|---|---|
| **Basic** | Arithmetic, Block, Logical, Bitwise |
| **Standard** (default) | Basic + Equality, Boolean, Assignment, Initializer, String, LINQ, Checked, Unary, Update |
| **Advanced** | Standard + Regex, Math methods, String methods |
| **Complete** | Everything |

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

### III-A: PIT (Java/Kotlin)

See full reference: `references/pit-mutation-testing.md` *(forthcoming)*

### III-B: Stryker.NET (C#)

**Status:** ✅ Active — the only production-grade mutation testing tool for C#. Microsoft officially recommends it in .NET docs.

**Architecture:** Uses **mutant schemata** (mutation switching) — all mutants compiled at once via Roslyn instrumentation, not bytecode manipulation. No per-mutant recompilation.

```bash
# Install (global)
dotnet tool install -g dotnet-stryker

# Or local (recommended for CI)
dotnet new tool-manifest
dotnet tool install dotnet-stryker
```

**Config (`stryker-config.json`):**
```json
{
  "stryker-config": {
    "solution": "../MySolution.sln",
    "project": "MyProject.csproj",
    "mutate": ["**/*.cs", "!**/*.Generated.cs", "!**/Migrations/*"],
    "mutation-level": "Standard",
    "test-runner": "vstest",
    "reporters": ["html", "progress", "json"],
    "concurrency": 4,
    "coverage-analysis": "perTest",
    "thresholds": { "high": 80, "low": 60, "break": 0 },
    "ignore-mutations": ["string", "linq.First"],
    "since": { "target": "main", "enabled": true },
    "baseline": { "enabled": true, "provider": "Dashboard" }
  }
}
```

**Key C#-specific features:**
- **LINQ mutators:** 30+ pairs (`.First()` ↔ `.FirstOrDefault()`, `.All()` ↔ `.Any()`, `.Single()` ↔ `.SingleOrDefault()`)
- **Checked mutator:** Removes/inserts `checked` blocks to test overflow handling
- **String method mutators:** `.Contains(str)` ↔ `.IndexOf(str)`, `.Trim()` → empty
- **Math method mutators:** `.Abs(-5)` ↔ empty, `.Floor(x)` ↔ `.Ceiling(x)`
- **Collection expressions (C#12):** `[1,2,3]` → empty collection
- **Regex mutator:** `new Regex("pattern")` → matches everything

**CI integration (GitHub Actions):**
```yaml
- name: Mutation testing
  run: |
    dotnet tool install --tool-path .stryker dotnet-stryker
    .stryker/dotnet-stryker --break-at 60
```

**CI integration (Azure DevOps):**
```yaml
- task: PublishMutationReport@1
  inputs:
    reportPattern: '**/mutation-report.json'
```

**C#-specific pitfalls:**
| Pitfall | Why | Mitigation |
|---|---|---|
| IL weaving (PostSharp, Fody) | Aspects injected post-compile; Stryker sees pre-weave code | Exclude woven methods from mutation |
| Async/await state machines | Mutants inside state machine may be unreachable | Use `ignore-methods` for compiler-generated code |
| LINQ expression trees | Mutations inside expression trees can't be applied | Exclude heavy LINQ chains |
| Nullable reference types | `T?` → `T` mutations may be equivalent | Review survivors carefully |
| Source generators (Razor, Regex source gen) | Generated code not available at mutation time | Exclude `**/*.g.cs` |
| F# | Stryker.NET only mutates C# projects | Use separate F# strategy |
| Assembly binding redirects | .NET Framework test projects may fail to load | Ensure bindingRedirects in app.config |

See `references/dotnet-mutation-testing.md` for full reference.

### III-C: StrykerJS (JavaScript / TypeScript)

**Status:** ✅ Active — the only production-grade JS/TS mutation testing tool. ~6K GitHub stars, 150K+ weekly npm downloads.

**Architecture:** AST-level instrumentation via Babel/TypeScript compiler. Mutants are generated as syntactic variants of the source AST.

```bash
# Quick initialize
npm init stryker@latest

# Manual install
npm install --save-dev @stryker-mutator/core
npm install --save-dev @stryker-mutator/vitest-runner  # or jest-runner
npm install --save-dev @stryker-mutator/typescript-checker
```

**Config (`stryker.config.json`):**
```json
{
  "$schema": "./node_modules/@stryker-mutator/core/schema/stryker-schema.json",
  "mutate": ["src/**/*.ts", "!src/**/*.spec.ts", "!src/**/*.test.ts"],
  "testRunner": "vitest",
  "reporters": ["html", "clear-text", "progress", "dashboard"],
  "thresholds": { "high": 80, "low": 70, "break": true },
  "concurrency": 4,
  "coverageAnalysis": "perTest",
  "tsconfigFile": "tsconfig.json",
  "checkers": ["typescript"]
}
```

**Key TypeScript-specific features:**
- **TypeScript checker plugin:** Validates all generated mutants compile under `tsc` — eliminates type-invalid mutants before running tests
- **`disableTypeChecks`:** Disables type checking on mutated files to avoid false-positive compilation errors (default: enabled)
- **Incremental mode:** `"incremental": true` + `incrementalFile` — only re-tests mutants affected by changed code
- **ESM/CJS support:** Config in `.mjs` for ESM projects, `.js`/`.cjs` for CommonJS

**CI integration (GitHub Actions):**
```yaml
- name: Mutation testing (incremental)
  run: |
    CHANGED=$(git diff --name-only origin/main...HEAD \
      | grep '^src/' | tr '\n' ',' | sed 's/,$//')
    [ -n "$CHANGED" ] && npx stryker run --mutate "$CHANGED"
```

**TS-specific pitfalls:**
| Pitfall | Why | Mitigation |
|---|---|---|
| Type erasure | Interfaces, types, enums erased at runtime — no mutants generated | Normal — not a problem |
| Decorators / MobX / Vue reactivity | Proxy-based frameworks may not detect mutations inside reactive code | Exclude decorator-heavy files or review survivors manually |
| Async/await control flow | Mutations inside async functions may not propagate | Add assertions that await async results |
| Optional chaining `?.` → `.` | May cause runtime errors if parent is null | Use TS checker to validate |
| JSX/TSX components | Mutation may produce invalid JSX | Use TS checker plugin |
| Path aliases | `@/utils` → resolved paths may break mutants | Configure `resolve.alias` in Stryker config |
| `ts-jest` / `tsx` transformations | Test runner may fail on mutant code | Use `--buildCommand "tsc"` before testing |

**React component considerations:**
- StrykerJS supports JSX/TSX mutation — mutates conditional renders, event handlers, state setters
- Key survivor patterns in React: missing `key` prop not caught, empty dependency arrays not tested, memoized components not exercised
- Use Testing Library / React Testing Library for meaningful assertions (not just snapshot tests, which kill fewer mutants)

See `references/typescript-mutation-testing.md` for full reference.

## IV — Configuration Patterns by Goal

| Goal | PIT (Java) | StrykerJS (JS/TS) | Stryker.NET (C#) | MutMut (Python) |
|---|---|---|---|---|
| **First run (measure)** | No config needed | `npx stryker run` | `dotnet stryker` | `mutmut run` |
| **Focused (module)** | `targetClasses: ["core.*"]` | `"mutate": ["src/core/**"]` | `"mutate": ["src/core/**/*.cs"]` | `paths_to_mutate = src/core/` |
| **CI gate** | `mutationThreshold: 80` | `thresholds: {high:80, low:70, break:true}` | `thresholds: {high:80, low:60, break:0}` via `--break-at` | N/A (post-process) |
| **Full audit** | mutators: `STRONGER` | `"mutators": ["all"]` | `mutation-level: "Advanced"` | Default is full |
| **Incremental** | N/A | `"incremental": true` | `"since": {"enabled": true, "target": "main"}` | N/A |
| **Ignore patterns** | `excludedMethods` | `mutate: ["!**/*.spec.ts"]` | `ignore-mutations`, `ignore-methods` | `paths_to_mutate` exclusion |

### Stryker.NET (C#)

```json
{
  "stryker-config": {
    "solution": "MyApp.sln",
    "project": "MyApp.Core.csproj",
    "mutate": ["**/*.cs", "!**/*.Generated.cs"],
    "mutation-level": "Standard",
    "test-runner": "vstest",
    "reporters": ["html", "progress"],
    "thresholds": { "high": 80, "low": 60, "break": 0 },
    "coverage-analysis": "perTest",
    "concurrency": 4
  }
}
```

### StrykerJS (TypeScript)

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

### PIT (Java)

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
| **C#:** LINQ chain survivors (`.First()` → `.FirstOrDefault()`) | Collections not tested for emptiness | Add empty/enumerable tests |
| **C#:** Null-coalescing survivors (`??` → `?? throw`) | Null paths not tested | Test with null inputs |
| **TS:** Optional chaining survivors (`?.` → `.`) | Null-safety assumptions unchecked | Test with undefined/missing props |
| **TS:** Array method survivors (`.map()` → `.flatMap()`) | Collection shape assumptions | Assert result length and shape |

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

### GitHub Actions — StrykerJS (incremental)

```yaml
- name: Mutation testing (incremental)
  run: |
    CHANGED=$(git diff --name-only origin/main...HEAD \
      | grep '^src/' | tr '\n' ',' | sed 's/,$//')
    [ -n "$CHANGED" ] && npx stryker run --mutate "$CHANGED"
```

### GitHub Actions — Stryker.NET (basic)

```yaml
- name: Install Stryker.NET
  run: dotnet tool install --tool-path .stryker dotnet-stryker

- name: Run mutation tests
  run: .stryker/dotnet-stryker --break-at 60

- name: Upload mutation report
  uses: actions/upload-artifact@v4
  with:
    name: mutation-report
    path: Reports/mutation-report.json
```

### GitHub Actions — Stryker.NET (incremental + baseline)

```yaml
- name: Mutation testing (incremental)
  run: |
    dotnet tool install --tool-path .stryker dotnet-stryker
    .stryker/dotnet-stryker --since:main --with-baseline:main \
      --break-at 60 --open-report:dashboard
  env:
    STRYKER_DASHBOARD_API_KEY: ${{ secrets.STRYKER_DASHBOARD_API_KEY }}
```

### Azure DevOps — Stryker.NET

```yaml
- script: |
    dotnet tool install --tool-path $(Agent.TempDirectory)/.stryker dotnet-stryker
    $(Agent.TempDirectory)/.stryker/dotnet-stryker --break-at 60
  displayName: 'Mutation testing'

- task: PublishMutationReport@1
  inputs:
    reportPattern: '**/mutation-report.json'
```

### Thresholds by Module Risk

| Risk Level | Min | Target | Break CI? |
|---|---|---|---|
| Critical (finance, auth, health) | 80% | 90%+ | Yes |
| Core (business logic) | 60% | 80% | Gradual |
| Infra / glue | 40% | 60% | Report only |
| Legacy | Baseline | — | No |

## VII — Hard Problems

### 1. Equivalent Mutants

```java
int x = a + 0;  →  int x = a;   // Equivalent — same semantics
```

```csharp
if (x != null && x.HasValue)  →  if (x.HasValue)  // Equivalent — redundant check
```

```typescript
const x = a ?? b;  →  const x = a;  // NOT equivalent if a is null
const x = a?.b;    →  const x = a.b; // NOT equivalent if a is null/undefined
```

**Detection (cheapest first):**
1. Compiler equivalence — compile both with `-O3`. Same bytecode? Skip.
2. Automated filtering — PIT/Stryker detect common patterns
3. LLM (ISSTA 2024) — ~85% accuracy. Pass survivors to classify.
4. Human review — last resort.

### 2. Mutant Explosion

10K LOC → thousands of mutants.

| Strategy | How | Correlation | When |
|---|---|---|---|
| Sampling | Random 10% | ~0.98 | Exploratory |
| Selective | Only ABS, UOI, LCR, AOR, ROR | ~90% | CI |
| Diff-aware | Changed files only | Exact | Every PR |
| Test selection | Coverage-based filter | Depends | Large projects |

**C#:** Use mutation levels (Basic/Standard/Advanced/Complete) to control explosion
**TS:** Use `incremental: true` + `coverageAnalysis: "perTest"`

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

**Adopters:**
- **Google** (internal mutation testing framework)
- **Meta** (core modules)
- **Spotify** (StrykerJS)
- **Microsoft** (Stryker.NET — official .NET documentation recommends it)
- **ING, Philips, Adyen** (StrykerJS)
- **Angular, NestJS** (StrykerJS in CI)
- **Polly** (Stryker.NET)
- **ABP Framework** (Stryker.NET)
- **The Ladders, BSkyB** (PIT)

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

### C#-Specific

11. **IL weavers (PostSharp, Fody)** — Aspects injected post-compile. Stryker mutates pre-weave code. Use `ignore-methods` for weaved methods.
12. **Async/await state machines** — Compiler-generated state machine code may contain unreachable mutants. Use `ignore-methods` with `*MoveNext*`.
13. **LINQ expression trees** — Mutations inside expression trees can't be applied. Exclude heavy LINQ-heavy files.
14. **Source generators (Razor, Regex source gen)** — Generated code not available at mutation time. Exclude `**/*.g.cs` and `**/Migrations/*`.
15. **Assembly binding redirects** — .NET Framework test projects may fail to load. Ensure `bindingRedirect` in `app.config`.
16. **F# projects** — Stryker.NET does not support F# mutation. Use separate strategy for F# code.
17. **.NET Framework vs .NET Core** — Stryker.NET requires .NET 8+ runtime installed, but can test projects targeting .NET Framework. Install on CI separately.

### TypeScript-Specific

11. **Type erasure** — Interfaces, types, enums are erased at runtime. No mutants for pure type code — normal.
12. **Decorator/MobX/Vue proxy frameworks** — Mutants inside reactive wrappers may not propagate. Exclude decorator-heavy files.
13. **Mock-heavy test suites** — Tests that heavily mock dependencies kill fewer mutants. Prefer integration-style tests.
14. **Snapshot tests** — Automatically pass most mutations; they assert serialized output not logic. Write explicit assertions.
15. **`ts-jest` / `tsx` transformations** — May fail on mutant code. Use `--buildCommand "tsc"` or switch to Vitest.
16. **Path aliases** — `@/utils` may not resolve in sandbox. Configure `resolve.alias` in test runner config.
17. **StrykerJS v9 requires Node 22+** — Verify Node version in CI before running.
18. **Type checker can be slow** — Enabling `typescript` checker doubles runtime. Use it selectively or only on critical modules.

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
