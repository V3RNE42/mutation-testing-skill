# Java / Kotlin Mutation Testing — Comprehensive Reference

## Tool Landscape Overview

| Tool | Maturity | Approach | Key Feature |
|------|----------|----------|-------------|
| **PIT** (pitest.org) | Production (v1.18+) | Bytecode-level mutation | Maturest tool. Maven/Gradle. 20+ years. |
| **Arcmutate** | Commercial | AST-level (targeted) | Faster than PIT, integrates as plugin |
| **Major** | Research | Compiler-integrated | Schema-based mutation, academic |
| **µJava / javalanche** | Research | Source-level | Thesis projects, not maintained |

**Bottom line:** PIT is the standard for Java. Use PIT with Maven or Gradle. Arcmutate is a commercial alternative for large codebases.

---

## 1. Installation

### Maven

Add to `pom.xml`:

```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.18.2</version>
    <configuration>
        <targetClasses>
            <param>com.mycompany.*</param>
        </targetClasses>
        <targetTests>
            <param>com.mycompany.*</param>
        </targetTests>
        <mutationThreshold>80</mutationThreshold>
    </configuration>
</plugin>
```

### Gradle

```groovy
plugins {
    id 'info.solidsoft.pitest' version '1.15.0'
}

pitest {
    targetClasses = ['com.mycompany.*']
    targetTests  = ['com.mycompany.*']
    threads      = 4
    outputFormats = ['HTML', 'XML']
    mutationThreshold = 80
    coverageThreshold = 80
}
```

### Kotlin

Use `pitest-kind` for Kotlin support:

```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.18.2</version>
    <configuration>
        <targetClasses>
            <param>com.mycompany.*</param>
        </targetClasses>
        <outputFormats>
            <param>HTML</param>
        </outputFormats>
        <features>
            <feature>+KOTLIN</feature>
        </features>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>io.github.pitest-kind</groupId>
            <artifactId>pitest-kind</artifactId>
            <version>0.5.0</version>
        </dependency>
    </dependencies>
</plugin>
```

### Run

```bash
# Maven
mvn test pitest:mutationCoverage         # basic run
mvn org.pitest:pitest-maven:mutationCoverage  # full qualified

# Gradle
./gradlew pitest

# Quick local test (no full build)
mvn pitest:mutationCoverage -DskipTests=false
```

---

## 2. Mutator Groups

PIT provides three mutator groups:

| Group | Mutators | Use Case |
|---|---|---|
| **DEFAULTS** | INCREMENTS_MUTATOR, CONDITIONALS_BOUNDARY, RETURN_VALS, VOID_METHOD_CALLS, NEGATE_CONDITIONALS, MATH, INVERT_NEGS, EMPTY_RETURNS (and a few more) | Everyday CI — stable, few equivalent mutants |
| **STRONGER** | DEFAULTS + REMOVE_CONDITIONALS | Deep audit — catches all conditional logic errors |
| **ALL** | STRONGER + AOR, ROR, UOI, EXPERIMENTAL | Research / thorough audit — many trivial kills |

### Individual mutators

| Mutator Name | Example Original | Example Mutated | What It Tests |
|---|---|---|---|
| INCREMENTS_MUTATOR | `i++` | `i--` | Loop counters, increment logic |
| CONDITIONALS_BOUNDARY | `i < 10` | `i <= 10` | Off-by-one errors |
| RETURN_VALS | `return true` | `return false` | Boolean return assumptions |
| VOID_METHOD_CALLS | `obj.process()` | `// removed` | Side effect verification |
| NEGATE_CONDITIONALS | `if (a && b)` | `if (!(a && b))` | Condition coverage |
| MATH | `a + b` | `a - b` | Arithmetic correctness |
| INVERT_NEGS | `-x` | `x` | Negation logic |
| EMPTY_RETURNS | `return x` | `return null` or `return 0` | Return handling |
| AOR | `a * b` | `a / b` | Full arithmetic coverage |
| ROR | `a == b` | `a != b` | Full relational coverage |
| UOI | `-x` | `+x` | Unary operator coverage |

---

## 3. Configuration

### Maven — full example

```xml
<plugin>
    <groupId>org.pitest</groupId>
    <artifactId>pitest-maven</artifactId>
    <version>1.18.2</version>
    <configuration>
        <targetClasses>
            <param>com.myapp.service.*</param>
            <param>com.myapp.domain.*</param>
        </targetClasses>
        <targetTests>
            <param>com.myapp.*</param>
        </targetTests>
        <excludedClasses>
            <param>com.myapp.dto.*</param>
        </excludedClasses>
        <excludedMethods>
            <param>hashCode</param>
            <param>equals</param>
            <param>toString</param>
        </excludedMethods>
        <mutators>
            <mutator>DEFAULTS</mutator>
        </mutators>
        <outputFormats>
            <param>HTML</param>
            <param>XML</param>
        </outputFormats>
        <threads>4</threads>
        <timeoutConstant>5000</timeoutConstant>
        <timeoutFactor>1.25</timeoutFactor>
        <maxMutationsPerClass>0</maxMutationsPerClass>
        <coverageThreshold>80</coverageThreshold>
        <mutationThreshold>80</mutationThreshold>
        <exportLineCoverage>true</exportLineCoverage>
        <failWhenNoMutations>false</failWhenNoMutations>
        <avoidCallsTo>
            <avoidCallsTo>java.util.logging</avoidCallsTo>
            <avoidCallsTo>org.slf4j</avoidCallsTo>
        </avoidCallsTo>
        <excludedTestClasses>
            <param>*IT</param>
            <param>*IntegrationTest</param>
        </excludedTestClasses>
        <verbose>true</verbose>
    </configuration>
</plugin>
```

### Gradle — full example

```groovy
pitest {
    targetClasses = ['com.myapp.service.*', 'com.myapp.domain.*']
    targetTests   = ['com.myapp.*']
    excludedClasses = ['com.myapp.dto.*']
    excludedMethods = ['hashCode', 'equals', 'toString']
    mutators = ['DEFAULTS']
    outputFormats = ['HTML', 'XML']
    threads = 4
    timeoutConstant = 5000
    timeoutFactor = 1.25
    mutationThreshold = 80
    coverageThreshold = 80
    exportLineCoverage = true
    failWhenNoMutations = false
    avoidCallsTo = ['java.util.logging', 'org.slf4j']
    excludedTestClasses = ['*IT', '*IntegrationTest']
    verbose = true
}
```

---

## 4. CI/CD Integration

### GitHub Actions (Maven)

```yaml
name: Mutation Testing

on:
  pull_request:
    branches: [main]

jobs:
  mutation:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '21'

      - name: Run PIT mutation testing
        run: mvn pitest:mutationCoverage

      - name: Upload mutation report
        uses: actions/upload-artifact@v4
        with:
          name: pit-report
          path: target/pit-reports/**/
```

### GitHub Actions (Gradle)

```yaml
- name: Run PIT mutation testing
  run: ./gradlew pitest

- name: Upload mutation report
  uses: actions/upload-artifact@v4
  with:
    name: pit-report
    path: build/reports/pitest/**/
```

### Jenkins

```groovy
stage('Mutation Testing') {
    steps {
        sh 'mvn pitest:mutationCoverage'
    }
    post {
        always {
            junit 'target/pit-reports/**/*.xml'
            publishHTML(target: [
                reportName: 'PIT Mutation Report',
                reportDir: 'target/pit-reports',
                reportFiles: 'index.html'
            ])
        }
    }
}
```

### Incremental / Diff-Aware

PIT doesn't have built-in incremental mode. Alternatives:

```bash
# Option 1: Target only changed files (via git diff)
CHANGED=$(git diff --name-only origin/main...HEAD | grep '^src/' | sed 's/src\\///' | tr '/' '.')
if [ -n "$CHANGED" ]; then
    mvn pitest:mutationCoverage -DtargetClasses="$CHANGED"
fi

# Option 2: Use history file (PIT tracks per-class results)
mvn pitest:mutationCoverage -DwithHistory
```

---

## 5. Equivalent Mutant Patterns in Java

```java
// 1. Symmetric identity operations
int x = a + 0;       → int x = a;         // Equivalent
int x = a - 0;       → int x = a;         // Equivalent
double x = a * 1.0;  → double x = a / 1.0; // Not equivalent (1.0/0 = Infinity)

// 2. Double negation in conditionals
if (!!flag)          → if (flag)          // Equivalent in boolean context

// 3. Chain of equals in sorted data
if (score >= 100)    → if (score > 100)
// When score is never exactly 100 in tests → survives

// 4. Buffer/accumulator patterns
sb.append('\n');     → // removed         // Only matters if read
// If sb is only built and never inspected, mutant survives

// 5. Logging removal
logger.info("done"); → // removed
// If tests don't verify logs → always survives
```

---

## 6. Known Pitfalls (Java-Specific)

### 6.1 Lombok

Lombok generates constructors, getters, setters, builders at compile time. PIT operates on bytecode, so it can mutate Lombok-generated code.

```xml
<excludedMethods>
    <param>builder</param>
    <param>hashCode</param>
    <param>equals</param>
    <param>toString</param>
    <param>canEqual</param>
</excludedMethods>
```

### 6.2 Mockito / Mock-Heavy Tests

```java
// If tests mock every dependency, mutations to real logic are never exercised
@Mock
private UserRepository repo;

// Mutation: repo.findById(id) → return 0 (different return) still passes
// because mock always returns the stubbed value
```

**Mitigation:** Use integration tests for core logic. Prefer real objects over mocks in mutation-tested modules.

### 6.3 Java Records

```java
public record User(String name, int age) {}
// Mutations to equals/hashCode/toString of records are often irrelevant
```

**Mitigation:** Exclude record auto-generated methods.

### 6.4 Spring / DI Proxies

Aspects and AOP proxies can mask mutations. If a mutation changes a method's return value but the Spring proxy logs and passes through, the test may pass even with the mutation active.

**Mitigation:** Test through the public API (controller → service → repository). Integration tests > unit tests for mutation.

### 6.5 Stream API

```java
items.stream()
    .filter(x -> x > 5)     // > mutated to >=
    .map(String::toLowerCase) // mapped to toUpperCase
    .collect(toList());
```

Stream operations are lambda-based and well-mutated by PIT. The challenge is that lambda bodies often have no meaningful assertions downstream.

### 6.6 JUnit 5 Parameterized Tests

```java
@ParameterizedTest
@CsvSource({"1, 2, 3", "3, 5, 8"})
void sum(int a, int b, int expected) {
    assertEquals(expected, calculator.add(a, b));
}
```

PIT handles parameterized tests correctly — each parameter set is a separate test case. A mutant that changes `add` to `subtract` will fail on some inputs and pass on others (e.g., `"1, 2, -1"` would pass for `a - b`).

### 6.7 Kotlin-Specific

- **Null safety:** Kotlin's null-safe types (`String?`) produce fewer null-related mutants. PIT may still generate `null` return values.
- **Companion objects / static members:** Mutated like Java statics.
- **Default parameter values:** Mutated as constant values.
- **Coroutines:** Suspend functions may under-report coverage due to continuation-passing style.

```xml
<!-- Enable Kotlin support -->
<features>
    <feature>+KOTLIN</feature>
</features>
```

---

## 7. Performance Tuning

### Threads

```xml
<threads>4</threads>   <!-- Gradle -->
```

Default: number of CPU cores. Tune based on available CI resources.

### History Mode

```xml
<withHistory>true</withHistory>   <!-- Maven only -->
```

PIT stores per-class results. On subsequent runs, unchanged classes reuse previous results.

### Avoid Calls

```xml
<avoidCallsTo>
    <avoidCallsTo>java.util.logging</avoidCallsTo>
    <avoidCallsTo>org.slf4j</avoidCallsTo>
</avoidCallsTo>
```

Skips mutation of specified package calls. Reduces trivial survivors from logging/builder methods.

### Exclude Generated Code

```xml
<excludedClasses>
    <param>**/*MapperImpl</param>   <!-- MapStruct -->
    <param>**/Q*</param>            <!-- QueryDSL -->
    <param>**/model/*</param>      <!-- DTOs -->
</excludedClasses>
```

---

## 8. Reporters and Output

```xml
<outputFormats>
    <param>HTML</param>   <!-- Interactive report with class-level drill-down -->
    <param>XML</param>    <!-- Machine-readable for CI integration -->
    <param>CSV</param>    <!-- Tabular for dashboards -->
</outputFormats>
```

HTML report location: `target/pit-reports/YYYYMMDDHHMMSS/index.html`
XML report: `target/pit-reports/YYYYMMDDHHMMSS/mutations.xml`

---

## 9. Quick Start — Cheat Sheet

```bash
# Maven — add plugin, run
mvn test pitest:mutationCoverage

# Gradle — add plugin, run
./gradlew pitest

# First run (no config)
mvn pitest:mutationCoverage

# CI gate
mvn pitest:mutationCoverage -DmutationThreshold=80

# Focused (single module)
mvn pitest:mutationCoverage -DtargetClasses="com.myapp.core.*"

# Full audit
mvn pitest:mutationCoverage -Dmutators=STRONGER

# Export coverage for comparison
mvn pitest:mutationCoverage -DexportLineCoverage=true
```

---

## 10. Further Reading

- [PITest documentation](https://pitest.org)
- [PITest GitHub](https://github.com/hcoles/pitest)
- [Arcmutate](https://www.arcmutate.com)
- [PITest Kind (Kotlin support)](https://github.com/pitest-kind/pitest-kind)
- [Gradle PITest plugin](https://github.com/szpak/gradle-pitest-plugin)
- [Awesome Mutation Testing](https://github.com/theofidry/awesome-mutation-testing)
