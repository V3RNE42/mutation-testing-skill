# Mutation Testing — Hermes Agent Skill 🧬

> A comprehensive Hermes Agent skill for mutation testing: theory, tools, operators, CI/CD integration, and adoption strategies.

## What is Mutation Testing?

Mutation testing is the **gold standard** of test quality metrics. Unlike code coverage (which only measures what lines are *executed*), mutation testing measures whether your tests can **actually detect bugs** — by introducing small changes (mutations) into the code and checking if the tests catch them.

```
Coverage:  "Did this line run?"         → quantity metric
Mutation:  "Would my test catch a bug   → quality metric
            on this line?"
```

## What This Skill Covers

- **Theory**: Strong vs Weak vs Firm mutation, mutation score calculation
- **Operators**: Full taxonomy (MOTHRA, PIT, StrykerJS, language-specific)
- **Tools per Language**: PIT (Java), Stryker (JS/TS/C#/Scala), MutMut (Python), cargo-mutants (Rust), and more
- **Higher Order Mutation (HOM)**: Subsuming and masked mutants
- **CI/CD Integration**: Incremental per PR, nightly, critical module enforcement
- **Problems & Solutions**: Equivalent mutants, mutant explosion, computational cost
- **Quick Start Guide**: Baseline → analyze → improve → integrate → scale

## Usage

Load the skill in any Hermes Agent session:

```python
from hermes_tools import skill_view
skill_view("mutation-testing")
```

Or just ask your agent about mutation testing — the skill auto-triggers on relevant queries.

## Key Metrics

| Metric | Impact |
|--------|--------|
| Production defect reduction | 25–30% |
| CI overhead | 15–20% |
| Technical debt reduction | 35% |

## Industry Adoption

Used by: **Google**, **Meta**, **Spotify**, **Microsoft**

## Installation as Local Skill

Clone into your Hermes skills directory:

```bash
git clone https://github.com/V3RNE42/mutation-testing-skill.git \
  ~/.hermes/skills/software-development/mutation-testing
```

## License

MIT — use freely, modify, share.
