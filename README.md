# Mutation Testing

> A comprehensive reference on mutation testing: theory, tools, operators, CI/CD integration, and adoption strategies.
> Covers **Java (PIT)**, **C# (Stryker.NET)**, **TypeScript/JavaScript (StrykerJS)**, **Python (MutMut)**, and more.

## What is Mutation Testing?

Mutation testing is the **gold standard** of test quality metrics. Unlike code coverage (which only measures what lines are *executed*), mutation testing measures whether your tests can **actually detect bugs** — by introducing small changes (mutations) into the code and checking if the tests catch them.

```
Coverage:  "Did this line run?"         → quantity metric
Mutation:  "Would my test catch a bug   → quality metric
            on this line?"
```

## Repository Structure

```
SKILL.md                          # Main reference (~400 lines)
references/
├── dotnet-mutation-testing.md    # C# / .NET deep dive (985 lines)
├── typescript-mutation-testing.md # TypeScript/JS deep dive (1,707 lines)
├── quick-reference.md            # Cheat sheet
└── key-papers.md                 # Academic papers and resources
```

## What This Covers

- **Theory**: Strong vs Weak vs Firm mutation, mutation score calculation
- **Operators**: Full taxonomy (MOTHRA, PIT, StrykerJS, Stryker.NET — language-specific)
- **Tools per Language**: PIT (Java), Stryker.NET (C#), StrykerJS (JS/TS), MutMut (Python), cargo-mutants (Rust), and more
- **Higher Order Mutation (HOM)**: Subsuming and masked mutants
- **CI/CD Integration**: Incremental per PR, nightly, critical module enforcement
- **Problems & Solutions**: Equivalent mutants, mutant explosion, computational cost
- **Language-Specific Pitfalls**: IL weaving, async/await (C#); type erasure, decorators, JSX (TS)

## Languages Covered in Depth

| Language | Tool | Reference File | Key Info |
|---|---|---|---|
| **C# (.NET)** | Stryker.NET | `references/dotnet-mutation-testing.md` | Only active C# tool. 19 mutator categories. LINQ, Checked, String method mutators. Microsoft-recommended. |
| **TypeScript / JavaScript** | StrykerJS | `references/typescript-mutation-testing.md` | Only production-grade JS/TS tool. 15+ mutator categories. TS checker plugin. Vitest/Jest/Mocha support. |
| **Java / Kotlin** | PIT (pitest) | SKILL.md (main section) | Bytecode-level. DEFAULTS/STRONGER/ALL groups. |
| **Python** | MutMut | SKILL.md (main section) | Modern Python tool. |
| **Rust** | cargo-mutants | SKILL.md (tool selection) | |
| **Go** | go-mutesting | SKILL.md (tool selection) | |

## Key Metrics

| Metric | Impact |
|--------|--------|
| Production defect reduction | 25-30% |
| CI overhead | 15-20% |
| Technical debt reduction | 35% |

## Industry Adoption

Used by: **Google**, **Meta**, **Spotify**, **Microsoft** (official .NET docs recommend Stryker.NET), **ING**, **Philips**, **Adyen**, **Angular**, **NestJS**, **Polly**, **ABP Framework**, **The Ladders**, **BSkyB**

## License

MIT
