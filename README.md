# Mutation Testing

> A comprehensive reference on mutation testing: theory, tools, operators, CI/CD integration, and adoption strategies.

## What is Mutation Testing?

Mutation testing is the **gold standard** of test quality metrics. Unlike code coverage (which only measures what lines are *executed*), mutation testing measures whether your tests can **actually detect bugs** — by introducing small changes (mutations) into the code and checking if the tests catch them.

```
Coverage:  "Did this line run?"         → quantity metric
Mutation:  "Would my test catch a bug   → quality metric
            on this line?"
```

## What This Covers

- **Theory**: Strong vs Weak vs Firm mutation, mutation score calculation
- **Operators**: Full taxonomy (MOTHRA, PIT, StrykerJS, language-specific)
- **Tools per Language**: PIT (Java), Stryker (JS/TS/C#/Scala), MutMut (Python), cargo-mutants (Rust), and more
- **Higher Order Mutation (HOM)**: Subsuming and masked mutants
- **CI/CD Integration**: Incremental per PR, nightly, critical module enforcement
- **Problems & Solutions**: Equivalent mutants, mutant explosion, computational cost
- **Quick Start Guide**: Baseline → analyze → improve → integrate → scale

## Key Metrics

| Metric | Impact |
|--------|--------|
| Production defect reduction | 25-30% |
| CI overhead | 15-20% |
| Technical debt reduction | 35% |

## Industry Adoption

Used by: **Google**, **Meta**, **Spotify**, **Microsoft**, **The Ladders**, **BSkyB**

## License

MIT
