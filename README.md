# Mutation Testing

> Actionable workflows and language-specific references for mutation testing: measure test quality, interpret survivors, improve tests, and lock in CI.

## What is Mutation Testing?

Code coverage tells you what lines *execute*. Mutation testing tells you if your tests can **actually catch bugs** — by introducing small changes (mutations) into the code and checking if the tests fail.

```
Coverage:  "Did this line run?"         → quantity
Mutation:  "Would my test catch a bug   → quality
            on this line?"
```

**The goal is not a number.** It's a test suite that reliably detects real bugs — boundary conditions, logic inversions, missing edge cases. Each survivor is a concrete hint: "write a test for this edge case."

## Repository Structure

```
SKILL.md                              # Actionable workflow + pointers per language
references/
├── dotnet-mutation-testing.md        # C# / .NET (Stryker.NET)
├── typescript-mutation-testing.md    # TypeScript / JavaScript (StrykerJS)
├── java-mutation-testing.md          # Java / Kotlin (PIT)
├── python-mutation-testing.md        # Python (MutMut)
├── quick-reference.md                # Cheat sheet (commands, config, CI)
└── key-papers.md                     # Academic papers and resources
```

Each language reference is **self-contained**: install → config → mutator tables → CI/CD → performance → pitfalls → equivalent mutants.

## What the Main SKILL.md Covers

- **End goal**: What mutation testing achieves and why it matters
- **Theory**: Strong vs Weak vs Firm mutation, mutation score, operators (with C# examples)
- **Actionable workflow**: Measure baseline → interpret survivors → improve tests → lock in CI
- **CI/CD strategy**: Incremental per PR, thresholds by module risk
- **Hard problems**: Equivalent mutants, mutant explosion, performance budget
- **Language pointers**: Quick tool selection table + links to per-language references

## Languages Covered

| Language | Tool | Reference |
|---|---|---|
| C# (.NET) | Stryker.NET | `references/dotnet-mutation-testing.md` |
| TypeScript / JavaScript | StrykerJS | `references/typescript-mutation-testing.md` |
| Java / Kotlin | PIT (pitest) | `references/java-mutation-testing.md` |
| Python | MutMut | `references/python-mutation-testing.md` |
| Scala | Stryker4s | SKILL.md (tool selection) |
| Rust | cargo-mutants / mutagen | SKILL.md (tool selection) |
| Ruby | mutant | SKILL.md (tool selection) |
| Go | go-mutesting | SKILL.md (tool selection) |
| Swift | Muter | SKILL.md (tool selection) |

## License

MIT
