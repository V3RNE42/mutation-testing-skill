# C# (.NET) Mutation Testing — Comprehensive Reference

## Tool Landscape Overview

| Tool | Maturity | Stars | Approach | Key Feature |
|------|----------|-------|----------|-------------|
| **Stryker.NET** (dotnet-stryker) | Production (v4.14+) | ~2K+ | Source-level mutant schemata | Full Roslyn-based mutation, CI-ready |
| **VisualMutant** | Dead/Archived | ~30 | N/A | Unmaintained, no .NET Core support |
| **CREMAS** | Academic/Dead | <10 | N/A | Research prototype only |
| **Microsoft/Stryker integration** | Documentation only | — | — | Microsoft docs recommend Stryker.NET |

**Bottom line:** Stryker.NET is the *only* actively maintained C# mutation testing tool. VisualMutant and CREMAS are defunct. Microsoft's official .NET documentation explicitly recommends [Stryker.NET](https://learn.microsoft.com/en-us/dotnet/core/testing/mutation-testing).

---

## 1. Stryker.NET (dotnet-stryker)

### 1.1 Installation

#### Global install (system-wide)
```bash
dotnet tool install -g dotnet-stryker
```

#### Local install (per-project, recommended for CI)
```bash
# Create tool manifest in your project root
dotnet new tool-manifest

# Install locally
dotnet tool install dotnet-stryker

# Team members restore
dotnet tool restore
```

#### Update
```bash
dotnet tool update -g dotnet-stryker          # global
dotnet tool update dotnet-stryker             # local
```

### 1.2 Version Compatibility

| Component | Supported Versions |
|-----------|-------------------|
| **Stryker.NET runtime** | Requires .NET 8.0+ runtime installed |
| **Project target frameworks** | .NET Core 1.1+, .NET Framework 4.5+, .NET Standard 1.3+ |
| **Tested against** | .NET Core 3.1, .NET Framework 4.8, .NET 6/7/8/9 |
| **C# language version** | C# 2 through C# 12 supported; `preview` and `latest` track embedded Roslyn |
| **Current package version** | 4.14.2 (as of mid-2025) |

> **Important:** Your application does NOT need to target .NET 8+. You only need the .NET 8+ runtime installed so Stryker can run. Your project can target any supported framework.

### 1.3 Architecture: Mutant Schemata (Mutation Switching)

Stryker.NET uses **mutant schemata** (also called mutation switching), NOT bytecode manipulation or per-mutant compilation:

```csharp
// Original code:
i++;

// Stryker injects:
if (Environment.GetEnvironmentVariable("ActiveMutation") == "1")
{
    i--; // mutated code
}
else
{
    i++; // original code
}
```

**Pros:** All mutants compiled at once (fast), exact location shown, assembly kept in memory.
**Cons:** Cannot mutate constant values, method names, or access modifiers.

### 1.4 Configuration

#### Config file: `stryker-config.json`
```json
{
    "stryker-config": {
        "solution": "../MySolution.sln",
        "project": "MyProject.csproj",
        "test-projects": ["../Tests/MyProject.UnitTests.csproj"],
        "test-case-filter": "(FullyQualifiedName~UnitTest1&TestCategory=CategoryA)",
        "mutate": ["**/*.cs", "!**/*.Generated.cs", "!**/Migrations/*"],
        "language-version": "latest",
        "configuration": "Debug",
        "target-framework": "net8.0",
        "test-runner": "vstest",
        "mutation-level": "Standard",
        "reporters": ["html", "progress", "json"],
        "report-file-name": "mutation-report",
        "additional-timeout": 5000,
        "concurrency": 4,
        "thresholds": { "high": 80, "low": 60, "break": 0 },
        "coverage-analysis": "perTest",
        "disable-bail": false,
        "disable-mix-mutants": false,
        "ignore-mutations": ["string", "linq.First"],
        "ignore-methods": ["*Log", "Console.Write*", "*Exception.ctor", "ConfigureAwait"],
        "project-info": {
            "name": "github.com/myorg/myrepo",
            "module": "my-module",
            "version": "feat/my-branch"
        },
        "since": { "target": "main", "enabled": true },
        "baseline": { "enabled": true, "provider": "Dashboard" }
    }
}
```

#### YAML format (`stryker-config.yaml`)
```yaml
stryker-config:
  solution: '../SolutionFile.sln'
  project: 'ExampleProject.csproj'
```

#### Init command
```bash
dotnet stryker init                              # generates config
dotnet stryker init --config-file "custom.json"  # custom name
dotnet stryker init --mutation-level "advanced"  # with overrides
```

#### Key CLI arguments

| CLI Flag | Description |
|----------|-------------|
| `-s` / `--solution` | Solution file path (required for .NET Framework) |
| `-p` / `--project` | Project file name under test |
| `-tp` / `--test-project` | Test project path (repeatable) |
| `-m` / `--mutate` | Glob patterns for files to mutate (repeatable) |
| `-l` / `--mutation-level` | `Basic`, `Standard`, `Advanced`, `Complete` |
| `-t` / `--test-runner` | `vstest` or `mtp` (preview) |
| `-r` / `--reporter` | Reporter type (repeatable) |
| `-c` / `--concurrency` | Number of concurrent workers |
| `-b` / `--break-at` | Fail CI if score below threshold |
| `-v` / `--version` | Version for dashboard report |
| `-f` / `--config-file` | Custom config file path |
| `-O` / `--output` | Output path for logs and reports |
| `--configuration` | Build configuration (Debug/Release) |
| `--target-framework` | Target framework moniker |
| `--threshold-high` | High threshold (default: 80) |
| `--threshold-low` | Low threshold (default: 60) |
| `--since:main` | Diff-aware testing since commit |
| `--with-baseline:main` | Baseline comparison with fallback |
| `--disable-bail` | Run all tests per mutant |
| `--open-report:dashboard` | Auto-open report in browser |
| `--verbosity trace --log-to-file` | Debug logging |
| `--dashboard-api-key` | API key for dashboard reporter |

#### Three operating modes

1. **Solution context:** Run from solution directory
   ```bash
   cd /my-solution && dotnet stryker
   ```
2. **Test project context:** Run from test project directory
   ```bash
   cd /my-test-project && dotnet stryker --project MyProject.csproj
   ```
3. **Source project context:** Run from source project, specify test projects
   ```bash
   cd /my-source-project && dotnet stryker --test-project "../tests/Tests.csproj"
   ```

### 1.5 Mutation Levels

| Level | Included Mutators |
|-------|-------------------|
| **Basic** | Arithmetic, Block statements, Logical, Bitwise |
| **Standard** (default) | Basic + Equality, Boolean, Assignment, Collection initializer, Unary, Update, String, LINQ, Checked |
| **Advanced** | Standard + Regex, Math methods, String methods |
| **Complete** | Everything |

---

## 2. Supported Mutators (with C# Examples)

### 2.1 Arithmetic Operators (`arithmetic`)
```csharp
// Original        → Mutated
int a = x + y;    → int a = x - y;
int a = x - y;    → int a = x + y;
int a = x * y;    → int a = x / y;
int a = x / y;    → int a = x * y;
int a = x % y;    → int a = x * y;
```

### 2.2 Equality Operators (`equality`)
```csharp
// Original            → Mutated
if (x > y)            → if (x < y)
if (x > y)            → if (x >= y)
if (x >= y)           → if (x > y)
if (x == y)           → if (x != y)
if (x != y)           → if (x == y)
if (x is string)      → if (x is not string)
```

### 2.3 Logical Operators (`logical`)
```csharp
// Original        → Mutated
a && b             → a || b
a || b             → a && b
a ^ b              → a == b
a and b            → a or b          // C# 9+ pattern logical
```

### 2.4 Boolean Literals (`boolean`)
```csharp
// Original                       → Mutated
bool flag = true;                 → bool flag = false;
bool flag = false;                → bool flag = true;
if (!person.IsAdult())            → if (person.IsAdult())
if (person.IsAdult())             → if (!person.IsAdult())
while (running)                   → while (!running)
```

### 2.5 Assignment Statements (`assignment`)
```csharp
// Original   → Mutated
x += y;      → x -= y;
x -= y;      → x += y;
x *= y;      → x /= y;
x /= y;      → x *= y;
x %= y;      → x *= y;
x <<= y;     → x >>= y;
x >>= y;     → x <<= y;
x &= y;      → x |= y;
x |= y;      → x &= y;
x ??= y;     → x = y;            // null-coalescing assignment
```

### 2.6 Initialization/Collection (`initializer`)
```csharp
// Original                            → Mutated
new int[] { 1, 2 };                   → new int[] { };
int[] numbers = { 1, 2 };             → int[] numbers = { };
new List<int> { 1, 2 };               → new List<int> { };
new Dictionary<int,int> { {1,1} };    → new Dictionary<int,int> { };
new SomeClass { Foo = "Bar" };        → new SomeClass { };
```

### 2.7 Removal Mutators (`statement`, `block`)
```csharp
// Original                                    → Mutated
void Function() { Age++; }                     → void Function() {}  // block emptied
return value;                                  → removed
return;                                        → removed
break;                                         → removed
continue;                                      → removed
throw exception;                               → removed
yield return value;                            → removed
MyMethodCall();                                → removed
```

### 2.8 Unary Operators (`unary`)
```csharp
// Original   → Mutated
-x           → +x
+x           → -x
~x           → x
```

### 2.9 Update Operators (`update`)
```csharp
// Original  → Mutated
x++;        → x--;
x--;        → x++;
++x;        → --x;
--x;        → ++x;
```

### 2.10 Checked Statements (`checked`) — Stryker.NET specific
```csharp
// Original                → Mutated
checked(2 + 4)             → 2 + 4
checked { sum += value; }  → sum += value;
```

### 2.11 LINQ Methods (`linq`)
```csharp
// Original                → Mutated
.All(predicate)            → .Any(predicate)
.Any(predicate)            → .All(predicate)
.Count()                   → .Sum()
.Sum()                     → .Count()
.First()                   → .FirstOrDefault()
.FirstOrDefault()          → .First()
.Last()                    → .First()
.Min()                     → .Max()
.Max()                     → .Min()
.Skip(n)                   → .Take(n)
.Take(n)                   → .Skip(n)
.OrderBy(k)                → .OrderByDescending(k)
.OrderByDescending(k)      → .OrderBy(k)
.Single()                  → .SingleOrDefault()
.SingleOrDefault()         → .Single()
.Union(seq)                → .Intersect(seq)
.Intersect(seq)            → .Union(seq)
.Concat(seq)               → .Except(seq)
.Except(seq)               → .Concat(seq)
.SkipWhile(p)              → .TakeWhile(p)
.TakeWhile(p)              → .SkipWhile(p)
.Append(e)                 → .Prepend(e)
.Prepend(e)                → .Append(e)
.Reverse()                 → .AsEnumerable()
.AsEnumerable()            → .Reverse()
.Order()                   → .OrderDescending()
.OrderDescending()         → .Order()
.MaxBy(k)                  → .MinBy(k)
.MinBy(k)                  → .MaxBy(k)
.IntersectBy(k, seq)       → .UnionBy(k, seq)
.UnionBy(k, seq)           → .IntersectBy(k, seq)
```

### 2.12 String Literals (`string`)
```csharp
// Original                          → Mutated
"foo"                               → ""
""                                  → "Stryker was here!"
$"foo {bar}"                        → $""
@"foo"                              → @""
string.Empty                        → "Stryker was here!"
string.IsNullOrEmpty(x)             → (x != null)
string.IsNullOrEmpty(x)             → (x != "")
string.IsNullOrWhiteSpace(x)        → (x != null)
```

### 2.13 String Methods (`stringmethod`)
```csharp
// Original        → Mutated
.StartsWith(x)    → .EndsWith(x)
.EndsWith(x)      → .StartsWith(x)
.ToLower()        → .ToUpper()
.ToUpper()        → .ToLower()
.Trim()           → ""
.TrimEnd()        → .TrimStart()
.TrimStart()      → .TrimEnd()
.IndexOf(c)       → .LastIndexOf(c)
.LastIndexOf(c)   → .IndexOf(c)
.PadLeft(n)       → .PadRight(n)
.PadRight(n)      → .PadLeft(n)
.Substring(n)     → ""
```

### 2.14 Bitwise Operators (`bitwise`)
```csharp
// Original   → Mutated
x << y       → x >> y
x >> y       → x << y
x & y        → x | y
x | y        → x & y
x ^ y        → ~(x ^ y)
```

### 2.15 Math Methods (`math`)
```csharp
// Original                 → Mutated
Math.Cos(x)                → Math.Sin(x)    // (and cyclic permutations)
Math.Sin(x)                → Math.Cos(x)
Math.Floor(x)              → Math.Ceiling(x)
Math.Ceiling(x)            → Math.Floor(x)
Math.Exp(x)                → Math.Log(x)
Math.Log(x)                → Math.Exp(x)
Math.Pow(x, y)             → Math.Log(x)
Math.Min(a, b)             → Math.Max(a, b)
Math.Max(a, b)             → Math.Min(a, b)
Math.BitDecrement(x)       → Math.BitIncrement(x)
// Full set includes cyclic permutations of:
// Acos/Acosh/Asin/Asinh/Atan/Atanh, Cos/Cosh/Sin/Sinh/Tan/Tanh
```

### 2.16 Null-coalescing Operators (`nullcoalescing`)
```csharp
// Original   → Mutated
a ?? b       → b ?? a     // swap operands
a ?? b       → a          // remove right
a ?? b       → b          // remove left
```

### 2.17 Conditional Operators (`conditional`)
```csharp
// Original          → Mutated
x ? a : b           → true ? a : b    // force true
x ? a : b           → false ? a : b   // force false
```

### 2.18 Collection Expressions (`collectionexpression`) — C# 12+
```csharp
// Original         → Mutated
[]                 → [default]      // empty to single default
[1, 2, 3]          → []             // filled to empty
```

### 2.19 Regex Mutations
Applied to `new Regex("...")` and regex literals. Uses weapon-regex library.
```csharp
// Original   → Mutated
^abc         → abc
abc$         → abc
[abc]        → [^abc]
\d           → \D
\s           → \S
\w           → \W
a?           → a      // quantifier removal
a*           → a
a+           → a
(?=abc)      → (?!abc)  // lookahead negation
(?!abc)      → (?=abc)
```

---

## 3. Test Runners Supported

| Runner | CLI Value | Status | Notes |
|--------|-----------|--------|-------|
| **VsTest** (default) | `vstest` | Production | Works with xUnit, NUnit, MSTest, SpecFlow via adapters |
| **Microsoft Test Platform** | `mtp` | Preview | Better perf, supports TUnit, newer frameworks |

```bash
# Use MTP runner (preview)
dotnet stryker -t mtp

# In config
"test-runner": "mtp"
```

**Test framework compatibility:**
- **xUnit** — Full support. Handles theories (InlineData, MemberData, ClassData).
- **NUnit** — Full support. Handles TestCase, TestCaseSource.
- **MSTest** — Full support. DataRow, DynamicData.
- **TUnit** — Supported via MTP runner (preview).
- **SpecFlow** — Works as test project reference.

**Key nuance:** xUnit theories with run-time data (MemberData, ClassData) are discovered as fewer VsTest cases than compile-time (InlineData) theories. This can affect coverage analysis accuracy.

---

## 4. CI/CD Integration

### 4.1 GitHub Actions

```yaml
name: Mutation Testing

on:
  pull_request:
    branches: [main]

jobs:
  mutation-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup .NET
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'

      - name: Restore tools
        run: dotnet tool restore

      - name: Build
        run: dotnet build --configuration Release

      - name: Run Stryker.NET mutation tests
        run: |
          dotnet stryker --break-at 60 --reporter "json" --reporter "html"
        working-directory: ./tests/MyProject.UnitTests

      - name: Upload mutation report
        uses: actions/upload-artifact@v4
        with:
          name: mutation-report
          path: '**/StrykerOutput/**/mutation-report.html'
```

#### Incremental (diff-aware) on PRs
```yaml
- name: Run mutation tests (diff-aware)
  run: |
    dotnet stryker --since:${{ github.base_ref }} --break-at 60
  working-directory: ./tests/MyProject.UnitTests
```

#### With baseline (full report from partial run)
```yaml
- name: Run mutation tests with baseline
  run: |
    dotnet stryker \
      --with-baseline:${{ github.base_ref }} \
      --version ${{ github.head_ref }} \
      --break-at 60
  working-directory: ./tests/MyProject.UnitTests
  env:
    STRYKER_DASHBOARD_API_KEY: ${{ secrets.STRYKER_DASHBOARD_API_KEY }}
```

### 4.2 Azure DevOps

```yaml
# Install Stryker
- task: DotNetCoreCLI@2
  displayName: 'Install dotnet-stryker'
  inputs:
    command: custom
    custom: tool
    arguments: install dotnet-stryker --tool-path $(Agent.BuildDirectory)/tools

# Run Stryker
- task: PowerShell@2
  displayName: 'Run dotnet-stryker'
  inputs:
    workingDirectory: '$(System.DefaultWorkingDirectory)/tests/MyProject.UnitTests'
    targetType: 'inline'
    pwsh: true
    script: |
      $(Agent.BuildDirectory)/tools/dotnet-stryker `
        --break-at 60 `
        --reporter "html"

# Publish mutation report
- task: PublishMutationReport@1
  displayName: 'Publish Mutation Test Report'
  inputs:
    reportPattern: '**/mutation-report.html'
```

> The **Mutation Report Publisher** Azure DevOps extension is available at [marketplace.visualstudio.com](https://marketplace.visualstudio.com/items?itemName=stryker-mutator.mutation-report-publisher). It adds a "Mutation Testing" tab to build results.

### 4.3 Azure DevOps with dashboard compare (PR baseline)
```yaml
- task: PowerShell@2
  displayName: 'Run dotnet-stryker with baseline'
  inputs:
    workingDirectory: 'tests/MyProject.UnitTests'
    targetType: 'inline'
    pwsh: true
    script: |
      $(Agent.BuildDirectory)/tools/dotnet-stryker `
        --with-baseline:$(System.PullRequest.TargetBranch) `
        --dashboard-api-key $(Stryker.Dashboard.Api.Key) `
        --version $(System.PullRequest.SourceBranch) `
        --break-at 60
```

---

## 5. Performance Tuning

### 5.1 Concurrency
```json
"concurrency": 4
```
Default: `number of logical processors / 2`. For a quad-core i7 with hyperthreading (8 logical cores), Stryker uses 4 workers by default.

**Tuning tips:**
- Leave some cores free for OS/test runner overhead.
- For CI runners with limited cores, set explicitly to avoid over-subscription.
- For very large projects, increasing concurrency can help but watch for memory pressure.

### 5.2 Coverage Analysis
```json
"coverage-analysis": "perTest"   // default — fastest
```
Options:
- **`perTest`** — Captures mutants covered by each test. Only relevant tests run per mutant. Fastest option.
- **`perTestInIsolation`** — Each test runs in isolation. More accurate for static constructor scenarios at cost of longer startup.
- **`all`** — Captures which mutants any test covers. Non-covered mutants assumed survivors. Fast.
- **`off`** — All tests run against all mutants. Slowest but safest.

### 5.3 Project Filtering
```json
"mutate": [
    "**/*Services.cs",
    "**/*Controllers.cs",
    "!**/*.Generated.cs",
    "!**/Migrations/*"
]
```
Only mutate business-critical code. Exclude auto-generated code, migrations, DTOs.

### 5.4 Diff-Aware Testing (`--since`)
```bash
dotnet stryker --since:main
```
Uses git diff to test only code changed since the target branch. Dramatically reduces CI runtime on PRs.

### 5.5 Baseline / Incremental Analysis (`--with-baseline`)
```bash
dotnet stryker --with-baseline:main --version $BRANCH_NAME
```
Saves mutation report to storage (Dashboard, Azure File Share, or S3). On subsequent runs, only changed mutants are retested; previous results are reused. Provides a full report from a partial run.

### 5.6 Bail Optimization
```json
"disable-bail": false   // default
```
Stryker aborts test run for a mutant as soon as one test fails. Set `true` to run all tests per mutant (useful for identifying useless tests).

### 5.7 Mutant Mixing
```json
"disable-mix-mutants": false   // default
```
Stryker combines multiple non-overlapping mutants in one test run. Disable if side effects cause issues.

### 5.8 Additional Timeout
```json
"additional-timeout": 5000   // default (ms)
```
Formula: `timeout = initialTestTime + additionalTimeout`. Increase if mutations cause timeouts. Decrease if you have infinite loops slowing runs.

### 5.9 Test Case Filter
```json
"test-case-filter": "(Category=UnitTest)"
```
Use `dotnet test --filter` syntax to run only a subset of tests during mutation testing.

---

## 6. Threshold Configuration

```json
"thresholds": { "high": 80, "low": 60, "break": 0 }
```

| Threshold | Default | Behavior |
|-----------|---------|----------|
| **high** | 80 | Score >= high → green (pass) |
| **low** | 60 | Score < high && >= low → yellow (warning) |
| **break** | 0 | Score < break → exit code 1 (CI failure) |

Using `--break-at`:
```bash
dotnet stryker --break-at 60    # fail CI if score < 60
```

Setting `break-at` to 0 means CI never fails on mutation score alone. This is useful during initial adoption.

---

## 7. Ignoring Mutations

### 7.1 By mutation type
```json
"ignore-mutations": [
    "string",
    "logical",
    "linq.First",
    "linq.Sum"
]
```

### 7.2 By method signature
```json
"ignore-methods": [
    "*Log",
    "Console.Write*",
    "*Exception.ctor",
    "ConfigureAwait"
]
```

### 7.3 By file glob
```json
"mutate": ["!**/*.Generated.cs", "!**/Migrations/*"]
```

### 7.4 Inline comments (most granular)
```csharp
// Stryker disable all : reason
i++; // won't be mutated

// Stryker restore all
i--; // will be mutated

// Stryker disable once Arithmetic
y++; // only Arithmetic mutant disabled

// Stryker disable once Arithmetic,Update
i--; // multiple types disabled
```

**Note:** `once` applies to the next SYNTAX CONSTRUCT (statement, block, method, or class depending on placement).

---

## 8. Equivalent Mutant Patterns in C#

Equivalent mutants produce the same observable behavior despite code changes. Common C# patterns:

### 8.1 Symmetric conditional with identity operation
```csharp
// Original
if (a >= b) { a *= 10 ** (max - min); }
else        { b *= 10 ** (max - min); }

// Mutant: >= changed to <=
// When max == min (i.e., a == b), both branches do 10^0 = 1 — identical result
```
**Pattern:** Any `if` where both branches converge to the same value when condition reaches boundary.

### 8.2 Zero/non-significant operations
```csharp
// Original
var result = value + 0;
// Mutant: + changed to -

// Original
var result = value * 1;
// Mutant: * changed to /
```

### 8.3 BigInt zero-equivalence
```csharp
// Original
a = 0n;
a = a <= 0n ? -a : a;

// Mutant of <= to <, >, >= produces same result for 0n
// because -0n == 0n in BigInt
```

### 8.4 Null-coalescing with null-check
```csharp
// Original
var result = obj ?? Fallback();

// Mutant: ?? swapped to just obj
// If obj is non-null in all test scenarios, this survives
```

### 8.5 LINQ with single-element sequences
```csharp
// Original
var first = items.First();

// Mutant: First() changed to FirstOrDefault()
// If items always contains at least one element in tests, both return same
```

### 8.6 Double negation
```csharp
// Original
if (!!isValid) { ... }

// Mutant removal of !
// Both !!x and x are equivalent in boolean context
```

---

## 9. Known Pitfalls Specific to .NET

### 9.1 Assembly Binding Issues
Stryker.NET compiles mutated code at runtime. If your project has complex assembly binding redirects (especially .NET Framework projects with `app.config`), the mutated assembly may fail to load or resolve dependencies.

**Mitigation:** Ensure all NuGet packages are resolved during build. Use solution context when possible. For .NET Framework, the solution path is required.

### 9.2 IL Weaving (PostSharp, Fody, etc.)
Tools like PostSharp, Fody (PropertyChanged, Costura), or other IL weavers modify assemblies post-compilation. Stryker operates on source code via Roslyn, so it sees the *original* source before weaving.

**Effect:** Mutants may be applied to code that IL weavers will transform, potentially producing inaccurate results or compilation errors.

**Mitigation:** Exclude weaver-generated code patterns from mutation. Use `ignore-methods` for weaver-injected methods.

### 9.3 Async/Await Mutants
```csharp
// Stryker may mutate inside async methods
var result = await GetDataAsync();
```
**Issue:** Mutating expressions inside async state machines can produce:
- Deadlock potential if synchronization context is captured
- Compiler-generated state machine confusion
- False survivors from `ConfigureAwait` mutations

**Mitigation:** Use `ignore-methods: ["ConfigureAwait"]` to skip ConfigureAwait mutations. Review async method survivors carefully.

### 9.4 LINQ Expression Trees
```csharp
// Stryker cannot mutate inside expression trees
Expression<Func<int, bool>> expr = x => x > 5;
```
**Issue:** Expression trees are compiled as data (not IL), so Stryker's source mutation doesn't apply. Survivors inside expression trees are expected.

**Mitigation:** Test expression tree logic separately by compiling and executing with varied inputs.

### 9.5 Nullable Reference Types (NRT)
```csharp
// C# 8+ nullable contexts
string? maybeNull = GetValue();
if (maybeNull != null) { ... }
```
**Issue:** Mutating `!= null` to `== null` can interact with NRT warnings. The compiler's nullable flow analysis may suppress warnings that would otherwise surface.

**Mitigation:** Review nullable-related survivors carefully. They may or may not be equivalent depending on NRT annotations.

### 9.6 Static Constructors and Coverage Analysis
Static constructors (and static initializers) run only once per test run, making coverage tracking unreliable. Mutants touched by static constructors are tested against ALL tests in `perTest` mode.

**Mitigation:** Use `perTestInIsolation` to isolate tests and get accurate coverage for static constructor-adjacent code.

### 9.7 Partial Classes and Generated Code
ASP.NET Razor Pages, Blazor components, and Entity Framework migrations generate partial classes. Stryker may mutate both your code and generated portions.

**Mitigation:**
```json
"mutate": [
    "!**/*.g.cs",
    "!**/*.generated.cs",
    "!**/*.Designer.cs",
    "!**/Migrations/*",
    "!**/Templates/*"
]
```

### 9.8 F# Support
Stryker.NET has experimental F# support but it is less mature. The class library for F# mutation exists but has known gaps in coverage and accuracy.

### 9.9 Source Generators (C# 9+)
Code generated by source generators (e.g., System.Text.Json source gen) may not be visible to Stryker's file scanning. Generated files should be excluded via `mutate` patterns.

### 9.10 .NET Framework NuGet Resolution
For .NET Framework projects, NuGet.exe must be installed and on PATH. Stryker uses NuGet for dependency resolution.

---

## 10. Reporters

| Reporter | Description |
|----------|-------------|
| `html` (default) | Interactive HTML report with file-by-file breakdown |
| `progress` (default) | Console progress bar with ETA |
| `json` | Machine-readable JSON report |
| `cleartext` | Text summary in console |
| `cleartexttree` | Tree-structured console output |
| `dots` | Minimal dot-per-mutant progress |
| `dashboard` | Uploads to Stryker Dashboard (stryker-mutator.io) |
| `all` | Enables all reporters |

```bash
dotnet stryker --reporter "html" --reporter "json"
```

---

## 11. Stryker.NET vs StrykerJS: Key Differences

| Dimension | Stryker.NET | StrykerJS |
|-----------|-------------|-----------|
| **Language** | C# (via Roslyn) | JavaScript/TypeScript |
| **Installation** | `dotnet tool install dotnet-stryker` | `npm install @stryker-mutator/core` |
| **Config format** | `stryker-config.json` (JSON) or `.yaml` | `stryker.conf.json` (JSON) or `.js`/`.mjs`/`.ts` |
| **Config schema** | Single `"stryker-config"` section | Top-level fields (`$schema`, `mutate`, etc.) |
| **CLI name** | `dotnet stryker` | `npx stryker run` |
| **Mutation approach** | Mutant schemata (compile-time switching) | Source code mutation (per-mutant file copy + test) |
| **Config key** | `"stryker-config": { ... }` | Flat JSON with `"$schema"` |
| **Test runner** | VsTest or MTP | Built-in runners (Jasmine, Mocha, Jest, etc.) |
| **Coverage** | `"coverage-analysis": "perTest"` | `"coverageAnalysis": "perTest"` (different casing) |
| **Dashboard** | `"project-info": { "name": "..." }` | `"dashboard": { "project": "..." }` |
| **Mutators specific** | `checked`, `linq`, `math`, `stringmethod`, `collectionexpression`, `nullcoalescing` | `optional chaining`, `object literal`, `array declaration` |
| **Config file name** | `stryker-config.json` | `stryker.conf.json` |
| **Report file** | `StrykerOutput/*/mutation-report.html` | `reports/mutation-report.html` |
| **Reporter names** | `html`, `progress`, `cleartext` | `html`, `progress`, `clear-text`, `dots`, `dashboard` |

### Config schema comparison

**Stryker.NET:**
```json
{
    "stryker-config": {
        "project": "MyProject.csproj",
        "mutation-level": "Standard",
        "reporters": ["html"]
    }
}
```

**StrykerJS:**
```json
{
    "$schema": "./node_modules/@stryker-mutator/core/schema/stryker-schema.json",
    "packageManager": "npm",
    "mutate": ["src/**/*.ts", "!src/**/*.spec.ts"],
    "testRunner": "jest",
    "reporters": ["html", "progress"],
    "coverageAnalysis": "perTest"
}
```

### Mutator differences (Stryker.NET only)
- **Checked statements** `checked { ... }` removal
- **LINQ method swaps** (30+ LINQ pairs, e.g., `Count()` ↔ `Sum()`)
- **Math method swaps** (cyclic permutations of trig/hyperbolic functions)
- **String method swaps** (`StartsWith` ↔ `EndsWith`, `ToLower` ↔ `ToUpper`)
- **Null-coalescing** operators (`??` swap, removal)
- **Collection expressions** (C# 12 `[]` to `[default]`)
- **Bitwise assignment** (`<<=` ↔ `>>=`, `&=` ↔ `|=`)

### Mutator differences (StrykerJS only)
- **Optional chaining** (`foo?.bar` → `foo.bar`)
- **Object literal** (`{ foo: 'bar' }` → `{}`)
- **Array declaration** (`new Array(1,2)` → `new Array()`)
- **Strict equality** (`===` ↔ `!==`) — C# uses `==`/`!=` only

---

## 12. Real-World Adoption Evidence

### Official Microsoft endorsement
Microsoft's official .NET documentation includes a dedicated [mutation testing guide](https://learn.microsoft.com/en-us/dotnet/core/testing/mutation-testing) that exclusively recommends Stryker.NET. The guide covers installation, interpretation of results, and CI/CD integration.

### Projects known to use Stryker.NET
- **Polly** (App-vNext/Polly) — The popular .NET resilience library uses Stryker.NET for mutation testing (referenced in CHANGELOG and CI config).
- **ABP Framework** (abpframework/abp) — The open-source web application framework for ASP.NET Core cites Stryker in v8.1 preview documentation.
- **Stryker.NET itself** — Dogfoods its own mutation testing (dashboard shows score at stryker-mutator.io).
- **Various .NET open-source projects** on GitHub reference `dotnet-stryker` in their CI workflows.

### NuGet download metrics
- `dotnet-stryker` package has hundreds of thousands of downloads on NuGet.org.
- Active maintenance with regular releases (current: v4.14.2).

### Community
- Active Slack community (#stryker-net channel)
- GitHub issues and discussions with responsive maintainers
- Sponsored by [Info Support](https://infosupport.com)

---

## 13. Quick Start — Cheat Sheet

```bash
# Install
dotnet tool install -g dotnet-stryker

# Basic run (from test project directory)
cd MyProject.Tests
dotnet stryker

# With config
dotnet stryker init
# Edit stryker-config.json
dotnet stryker

# CI-ready
dotnet stryker --break-at 60 --reporter "html" --reporter "json"

# Incremental (PR)
dotnet stryker --since:main --break-at 60

# Debug
dotnet stryker --verbosity trace --log-to-file

# Specify project (multi-reference test projects)
dotnet stryker -p MyProject.csproj

# Specify solution (.NET Framework)
dotnet stryker -s ../MySolution.sln

# Change mutation level
dotnet stryker -l Advanced

# Custom concurrency
dotnet stryker -c 8
```

---

## 14. Further Reading

- [Stryker.NET Documentation](https://stryker-mutator.io/docs/stryker-net/introduction)
- [Stryker.NET GitHub](https://github.com/stryker-mutator/stryker-net)
- [Stryker.NET NuGet](https://www.nuget.org/packages/dotnet-stryker)
- [Microsoft .NET Mutation Testing Docs](https://learn.microsoft.com/en-us/dotnet/core/testing/mutation-testing)
- [Stryker Dashboard](https://dashboard.stryker-mutator.io)
- [Mutation Testing Elements (report format)](https://github.com/stryker-mutator/mutation-testing-elements)
- [Mutation Report Publisher (Azure DevOps)](https://marketplace.visualstudio.com/items?itemName=stryker-mutator.mutation-report-publisher)
- [Stryker.NET Slack Community](https://join.slack.com/t/stryker-mutator/shared_invite/enQtOTUyMTYyNTg1NDQ0LTU4ODNmZDlmN2I3MmEyMTVhYjZlYmJkOThlNTY3NTM1M2QxYmM5YTM3ODQxYmJjY2YyYzllM2RkMmM1NjNjZjM)
