---
name: mutation-testing
description: "Actionable mutation testing workflow: measure test quality, interpret survivors, improve tests, lock in CI. Language-specific deep dives in ./references/."
---

# Mutation Testing

## What It Is & Why It Matters

Mutation testing is a **fault-based testing technique** that measures test *quality* — not just what lines execute (coverage), but whether tests can actually detect bugs. It is the gold standard of test metrics.

```csharp
// Line coverage says: "this line runs" ✓
// Mutation testing says: "can your tests tell the difference?"

int total = price + tax;      // Original

// Mutant: arithmetic swapped
int total = price - tax;      // ← Did your test catch this?

// If the mutant SURVIVES → your test assertion is too weak
// If the mutant is KILLED    → your test actually validates the behavior
```

**The coverage–quality gap:** 90% line coverage typically yields only 60–70% mutation score with decent tests, or 30–40% with weak assertions. This gap is *expected* — and closing it is the point.

### End Goal

You are not chasing a number. You are building a **test suite that reliably detects real bugs** — boundary conditions, logic inversions, missing edge cases, null-safety lapses. When you achieve:

| Score | What it means | You've achieved |
|---|---|---|
| > 80% | Core logic is guarded | Production defects from that module drop ~25–30% |
| > 90% | Excellent protection | Confidence to refactor fearlessly |
| Drift-down detected | CI catches regressions | Team can merge without quality surprise |

**Mutation testing is not a gate — it's a feedback loop.** Each survivor is a concrete hint: "write a test for this edge case."

---

## When to Use / When Not

| USE when you need to know | DO NOT use for |
|---|---|
| "Are my tests actually good, or just green?" | General testing strategy advice |
| "Where are my test gaps?" | Test automation framework selection |
| "Will my CI catch logic bugs?" | Performance / load testing |
| "How do I compare coverage vs actual test quality?" | Code review (mutation finds test gaps, not design flaws) |

---

## Theory — The Three Pillars (with C# examples)

### Pillar 1: Strong vs Weak vs Firm

| Type | Condition to Kill | Used By |
|---|---|---|
| **Strong** | Test assertion fails (output differs) | PIT, Stryker (default) |
| **Weak** | State *immediately after mutation* differs | Academic tools |
| **Firm** | State propagates partway | Hybrid approaches |

All production tools use **strong mutation** — the test must actually fail.

### Pillar 2: Mutation Score

```
Score = Killed / (Total - Equivalent) × 100
```

| Score | Meaning | Action |
|---|---|---|
| < 60% | Tests are weak | Write meaningful assertions, cover edge cases |
| 60–80% | Acceptable | Target survivors with highest risk |
| 80–90% | Good | Review remaining survivors for equivalent mutants |
| > 90% | Excellent | Verify no trivial tests inflate score |

### Pillar 3: Mutation Operators (C# — Stryker.NET)

```csharp
// Arithmetic operator
// Original         → Mutated
int a = x + y;     → int a = x - y;    // AOR

// Equality operator
if (x > y)         → if (x < y)        // ROR
if (x == y)        → if (x != y)       // ROR

// LINQ method swap
items.All(pred)    → items.Any(pred)   // LINQ
items.First()      → items.FirstOrDefault()

// Logical operator
a && b             → a || b            // LCR

// Boolean literal
bool flag = true;  → bool flag = false;

// Null-coalescing
a ?? b             → a                 // removal

// String mutation
"hello"            → ""                // empty

// Block removal
void F() { Age++; } → void F() {}      // statement deleted
```

For full mutator tables per language, see the corresponding language reference (`./references/`).

---

## Actionable Workflow

### Step 1: Measure Baseline

```bash
# C# (Stryker.NET)
dotnet stryker

# TS/JS (StrykerJS)
npx stryker run

# Java (PIT)
mvn pitest:mutationCoverage

# Python (MutMut)
mutmut run
```

**First run:** No config needed. Defaults will give you a baseline score. Record it — this is your starting point.

### Step 2: Interpret Survivors

| Survivor Pattern | Root Cause | Fix |
|---|---|---|
| Boundary survives (`<` vs `<=`) | No edge case tests | Add boundary: `value == min`, `value == max` |
| Logical operator survives (`&&` vs `\|\|`) | Branch not covered | Test each branch independently |
| Return value survives | Assertions too loose | Assert exact value, not just "not null" |
| LINQ swap survives (`First` vs `FirstOrDefault`) | Missing empty-sequence case | Test with empty collection |
| Method call removed survives | Side effects not verified | Assert state change, not just no exception |
| Null-coalescing removed survives | Null case never exercised | Test with `null` input |

```csharp
// Example: LINQ First() survived
var result = items.First();   // Mutant: → items.FirstOrDefault()

// Fix: add test for empty collection
[Fact]
public void Throws_when_empty()
{
    var items = new List<int>();
    Assert.Throws<InvalidOperationException>(() => Sut.Process(items));
}
```

### Step 3: Improve Tests

1. Pick the **most actionable survivor** (high-risk logic, not string literals)
2. Write a test targeting the mutated behavior — passes on original code
3. Re-run mutation testing on that file
4. If the mutant survives → check for equivalent mutant. If not equivalent, strengthen the assertion
5. Repeat. Each killed survivor is a genuine improvement

### Step 4: Validate & Lock in CI

```bash
dotnet stryker --break-at 60    # C# — fail CI under 60%
npx stryker run --thresholds.break true  # TS/JS
mvn pitest:mutationCoverage -DmutationThreshold=60  # Java
```

**Feedback loop:**
- Score improved? → Continue strategy
- Score regressed? → Check new code without tests
- Repeated survivor pattern? → Fix root cause in test design
- Unexpected equivalent mutants? → Add to exclusion list

---

## CI/CD Strategy

### Priority

```
[Best]   Incremental per PR   → changed files only. Seconds–minutes.
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

---

## Hard Problems

### 1. Equivalent Mutants

Mutants that behave identically to the original — can't be killed by any test.

```csharp
int x = a + 0;  →  int x = a;   // Equivalent — same semantics
```

**Detection (cheapest first):**
1. Compiler equivalence — same bytecode? Skip.
2. Automated filtering — PIT/Stryker detect common patterns
3. LLM (ISSTA 2024) — ~85% accuracy
4. Human review — last resort

Language-specific patterns → see language reference.

### 2. Mutant Explosion

| Strategy | How | When |
|---|---|---|
| Sampling | Random 10% (correlation ~0.98) | Exploratory |
| Selective | Only ABS, UOI, LCR, AOR, ROR (~90% coverage) | CI |
| Diff-aware | Changed files only | Every PR |
| Test selection | Coverage-based filter | Large projects |

### 3. Performance Budget

| Scale | Defaults | ALL |
|---|---|---|
| 10K LOC | ~3 min | ~15 min |
| 100K LOC | ~20 min | ~2 h |
| 1M LOC | Hours | Use incremental |

---

## Tool Selection by Language

```
Language?
├── C# (.NET)         → Stryker.NET  (dotnet-stryker)   ← only active tool
├── Java / Kotlin     → PIT          (pitest.org)       ← most mature
├── TypeScript / JS   → StrykerJS    (stryker-mutator.io) ← only production-grade
├── Python            → MutMut       (mutmut)           ← modern choice
├── Scala             → Stryker4s
├── Rust              → cargo-mutants / mutagen
├── Ruby              → mutant
├── Go                → go-mutesting
└── Swift             → Muter (emerging, low activity)
```

---

## Language Reference Pages

Each reference is self-contained: install → config → mutator tables → CI/CD → performance → pitfalls → equivalent mutants.

| Language | Reference |
|---|---|---|
| **C# (.NET)** | `/references/dotnet-mutation-testing.md` | 
| **TypeScript / JavaScript** | `/references/typescript-mutation-testing.md` | 
| **Java / Kotlin** | `/references/java-mutation-testing.md` | 
| **Python** | `/references/python-mutation-testing.md` | 

### Quick Reference

A cheat sheet with commands, config snippets, and CI examples: `/references/quick-reference.md`

### Key Papers

Foundational and modern research: `/references/key-papers.md`

---

## Verification Checklist

- [ ] Tool selected and installed (see language reference)
- [ ] Baseline score measured and documented
- [ ] Thresholds configured (break/high/low)
- [ ] Scope limited to relevant modules
- [ ] CI: incremental per PR + nightly full run
- [ ] Equivalent mutant filtering active
- [ ] Survivor review cadence established
- [ ] Team trained on interpretation
- [ ] Coverage–mutation gap communicated to stakeholders
- [ ] Dashboard / reporting active
