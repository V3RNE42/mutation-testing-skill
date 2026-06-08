# Mutation Testing — Key Papers & Resources

## Foundational Papers

| Year | Title | Authors | Venue | Why It Matters |
|---|---|---|---|---|
| 1978 | *Hints on Test Data Selection: Help for the Practicing Programmer* | DeMillo, Lipton, Sayward | IEEE Computer | Founding paper — introduces the concept of mutation testing and the competent programmer hypothesis |
| 1980 | *An Experimental Evaluation of Mutation Testing* | Acree et al. | Georgia Tech | First empirical validation: mutation testing finds bugs that coverage misses |
| 1996 | *An Analysis and Survey of the Development of Mutation Testing* | Jia, Harman | IEEE TSE | Comprehensive survey establishing the field's maturity |

## Modern Research

| Year | Title | Authors | Venue | Why It Matters |
|---|---|---|---|---|
| 2009 | *Higher Order Mutation Testing* | Harman, Jia, Langdon | Information & Software Technology | Introduces HOMs and subsuming mutants — complex bugs hiding in combined mutations |
| 2018 | *Mutation Testing Advances: An Analysis and Survey* | Papadakis et al. | Advances in Computers | Definitive modern survey — covers selective mutation, equivalent mutants, tool landscape |
| 2024 | *Static and Dynamic Comparison of Mutation Testing Tools for Python* | ACM | ACM | Benchmarks CosmicRay, MutPy, MutMut, Mutatest head-to-head. Essential for Python tool selection |
| 2024 | *LLMs for Equivalent Mutant Detection* | Zhao Tian et al. | ISSTA 2024 | ~85% accuracy using LLMs to classify equivalent mutants. Practical for CI pipelines |
| 2024 | *Mutation Testing in Practice: Insights From Open Source Software* | IEEE | IEEE | Empirical study of how OSS projects actually use mutation testing. Adoption patterns and ROI data |

## C# / .NET Resources

| Resource | Type | Link / Note |
|---|---|---|
| **Stryker.NET documentation** | Official docs | https://stryker-mutator.io/docs/stryker-net/ |
| **Microsoft .NET mutation testing docs** | Official MS docs | https://learn.microsoft.com/en-us/dotnet/core/testing/mutation-testing |
| **Stryker.NET GitHub** | Source code | https://github.com/stryker-mutator/stryker-net |
| **Mutation Testing for C#: A Practical Guide** | Community article | Search: "Stryker.NET mutation testing C#" |
| **Roslyn-based mutation analysis (research)** | Conference paper | Durelli et al. (2018) — technical underpinnings of .NET mutation tools |

## TypeScript / JavaScript Resources

| Resource | Type | Link / Note |
|---|---|---|
| **StrykerJS documentation** | Official docs | https://stryker-mutator.io/docs/stryker-js/ |
| **StrykerJS GitHub** | Source code | https://github.com/stryker-mutator/stryker-js |
| **Stryker Playground** | Online tool | https://stryker-mutator.io/stryker-playground/ |
| **awesome-mutation-testing** | GitHub curated list | https://github.com/theofidry/awesome-mutation-testing |

## Recommended Reading Order

```
New to mutation testing:
  1. DeMillo 1978 (original paper — short, readable)
  2. Papadakis 2018 (comprehensive survey)
  3. Tool docs (PIT / StrykerJS / Stryker.NET)

Implementing in CI:
  1. IEEE 2024 (real-world adoption patterns)
  2. Tool-specific docs (CI section)

Deep / research:
  1. Harman 2009 (HOM)
  2. Zhao Tian 2024 (LLM + equivalent mutants)
  3. ACM 2024 (Python tools comparison)
```

## Other Useful Resources

- **awesome-mutation-testing** (GitHub) — curated list: https://github.com/theofidry/awesome-mutation-testing
- **Stryker Playground** — try mutation testing online: https://stryker-mutator.io/stryker-playground/
- **PITest docs**: https://pitest.org
- **Stryker docs**: https://stryker-mutator.io
- **MutMut docs**: https://mutmut.readthedocs.io
