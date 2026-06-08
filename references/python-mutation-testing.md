# Python Mutation Testing — Comprehensive Reference

## Tool Landscape Overview

| Tool | Maturity | Approach | Key Feature |
|------|----------|----------|-------------|
| **MutMut** | Active (v5.x) | AST-level | Modern, fast, CI-ready, auto-excludes dunder methods |
| **MutPy** | Maintenance | AST-level | Classic tool, last release 2020 |
| **CosmicRay** | Dormant | AST-level | STOCHASTIC mode — samples mutations rather than enumerating |
| **Mutatest** | Dormant | AST-level | Sorted mutation trial sets, 2019 release |

**Bottom line:** MutMut is the recommended choice. It's actively maintained, has the best developer experience, and integrates cleanly into CI. MutPy is the fallback if you need Python 2 support or specific academic mutators.

---

## 1. Installation

### MutMut

```bash
pip install mutmut

# Or with specific Python version
pipx install mutmut             # isolated from project deps
pip install mutmut==5.5.0       # pin version

# With pytest extras
pip install mutmut pytest

# Verify
mutmut --version
```

### MutPy

```bash
pip install mutpy
```

### CosmicRay

```bash
pip install cosmic-ray
```

---

## 2. Quick Start

```bash
# MutMut — first run
mutmut run --paths-to-mutate src/myproject/

# MutMut — basic run with tests
mutmut run --paths-to-mutate src/ --tests-dir tests/

# MutPy
mut.py --target src/myproject/ --unit-test tests/test_myproject.py

# CosmicRay (stochastic)
cosmic-ray init my_config.toml my_session.sqlite
cosmic-ray exec my_config.toml my_session.sqlite
cr-report my_session.sqlite
```

---

## 3. MutMut Configuration

### `setup.cfg` or `pyproject.toml`

```ini
# setup.cfg
[mutmut]
paths_to_mutate = src/myproject/
tests_dir = tests/
test_timeout = 30
backup = false
runner = pytest
dict_sync = true
exclude_lines = logger\.(info|debug|warning|error)
exclude_lines = @abstractmethod
```

```toml
# pyproject.toml
[tool.mutmut]
paths_to_mutate = "src/myproject/"
tests_dir = "tests/"
test_timeout = 30
backup = false
runner = "pytest"
dict_sync = true
exclude_lines = [
    "logger\\.(info|debug|warning|error)",
    "@abstractmethod",
]
```

### CLI arguments

```bash
mutmut run [options]

# Key options:
--paths-to-mutate       # Files/dirs to mutate (required)
--tests-dir             # Test directory (required for discovery)
--runner                # Test runner: pytest (default), unittest
--test-timeout          # Per-test timeout in seconds (default: 30)
--backup                # Backup mutated files (default: false)
--use-coverage          # Only run tests covering mutated code (faster)
--dict-sync             # Ensure dict iteration order stability
--exclude-lines         # Regex for lines to skip mutation
--paths-to-exclude      # Glob patterns to exclude
--override-existing-settings  # Force override from CLI
```

### Run with coverage optimization

```bash
mutmut run --paths-to-mutate src/ --tests-dir tests/ --use-coverage
```

Uses coverage data to only run tests that actually cover the mutated code. Much faster than running all tests for every mutant.

---

## 4. Mutators

### MutMut Mutators

| Mutator | Original | Mutated | What It Tests |
|---|---|---|---|
| **Arithmetic** | `a + b` | `a - b` | Math correctness |
| **Arithmetic** | `a * b` | `a / b` | |
| **Comparison** | `a == b` | `a != b` | Boundary conditions |
| **Comparison** | `a < b` | `a <= b` | |
| **Comparison** | `a in b` | `a not in b` | |
| **Comparison** | `a is b` | `a is not b` | |
| **Boolean** | `True` | `False` | Boolean logic |
| **Boolean** | `a and b` | `a or b` | |
| **Boolean** | `not a` | `a` | |
| **Call removal** | `func()` | `# removed` | Side effects |
| **Number literal** | `42` | `0` | Magic numbers |
| **String literal** | `"hello"` | `""` | String handling |
| **Slice** | `a[start:end]` | `a[:end]` | Slice logic |
| **Attribute** | `obj.attr` | `obj` (removed) | Attribute access |
| **Keyword removal** | `func(x=y)` | `func(x)` | Default params |

### MutPy Mutators

MutPy uses a similar set with slightly different naming:

- **AOR** — Arithmetic Operator Replacement (`+` → `-`)
- **ROR** — Relational Operator Replacement (`<` → `<=`)
- **COR** — Conditional Operator Replacement (`and` → `or`)
- **COD** — Conditional Operator Deletion (remove `not`)
- **COI** — Conditional Operator Insertion (insert `not`)
- **SIR** — Slice Index Remove
- **CRP** — Constant Replacement (`42` → `0`)
- **LCR** — Logical Connector Replacement (`and` → `or`)

---

## 5. CI/CD Integration

### GitHub Actions (MutMut)

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
      - uses: actions/setup-python@v5
        with:
          python-version: '3.12'

      - name: Install dependencies
        run: |
          pip install mutmut pytest pytest-cov
          pip install -r requirements.txt

      - name: Run mutation tests
        run: mutmut run --paths-to-mutate src/ --tests-dir tests/ --use-coverage

      - name: Check mutation score
        run: |
          SCORE=$(mutmut result --json | python3 -c "import json,sys; d=json.load(sys.stdin); print(d.get('score', 0))")
          echo "Mutation score: $SCORE%"
          if (( $(echo "$SCORE < 60" | bc -l) )); then
            echo "FAIL: Mutation score below 60%"
            exit 1
          fi
```

### GitLab CI

```yaml
mutation-testing:
  stage: test
  image: python:3.12
  script:
    - pip install mutmut pytest pytest-cov
    - pip install -r requirements.txt
    - mutmut run --paths-to-mutate src/ --tests-dir tests/
    - mutmut result
  artifacts:
    paths:
      - html/
    expire_in: 30 days
```

### Post-process score for thresholds

```bash
# Get JSON results
mutmut result --json > mutation_results.json

# Extract score with jq
jq '.score' mutation_results.json

# Fail if below threshold
THRESHOLD=60
SCORE=$(jq '.score' mutation_results.json)
if (( $(echo "$SCORE < $THRESHOLD" | bc -l) )); then
    echo "FAIL: Score $SCORE% < $THRESHOLD%"
    exit 1
fi
```

---

## 6. Equivalent Mutant Patterns in Python

```python
# 1. Augmented assignment identity
x += 0          # → x -= 0   (equivalent for most numeric types)
x *= 1          # → x /= 1   (not equivalent: division by zero)

# 2. In / not in with empty containers
if x in []:     # → if x not in []  (both always false)
    pass

# 3. Dunder method mutations
__repr__ -> return str()   # mutated — rarely matters for test assertions
# Mitigation: exclude __repr__, __str__, __hash__, __eq__ from mutation

# 4. Assert statements
assert x == 5   # mutations inside assertions are caught by assertion
                # but mutmut skips assert lines by default

# 5. Logging calls removed
logger.info("done")  # → removed
# If tests don't mock logs, always survives

# 6. Type annotations (not mutated — they're metadata, not runtime)
def process(x: int) -> str: ...  # types not mutated
```

---

## 7. Known Pitfalls

### 7.1 Dynamic Typing

Python's dynamic typing means more trivial survivors:

```python
def process(items):
    result = []
    for item in items:          # mutation: for item in [] (block removal)
        result.append(item * 2) # mutation: item + 2, item / 2
    return result
```

If the test only checks `len(result)` or `result is not None`, arithmetic and comparison mutations survive.

**Mitigation:** Assert exact values, not just existence.

### 7.2 Magic Mock Survivors

```python
from unittest.mock import Mock

def test_process():
    mock = Mock()
    result = sut.process(mock)
    assert result is not None   # ← passes for almost any mutation
```

Mocks return `Mock()` for any attribute access and method call. This masks nearly all mutations.

**Mitigation:** Use real objects, or stub only what's needed. Assert specific return values, states, or side effects.

### 7.3 Property / Descriptor Survivors

```python
class Temperature:
    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        self._celsius = value

    @property
    def fahrenheit(self):
        return self._celsius * 9/5 + 32  # mutated: - to +, * to /
```

Property-based tests with Hypothesis can help kill survivors by generating random inputs and checking invariants.

### 7.4 Pytest Fixtures

```python
@pytest.fixture
def user():
    return User(name="Alice", age=30)  # mutation: age to 0

def test_user_age(user):
    assert user.age == 30  # ← passes even if mutation changes age
```

A fixture that always returns the same value makes the test assertion trivially correct. The mutation changes the fixture value but the assertion follows it.

**Mitigation:** Vary fixture data across tests. Use `pytest.mark.parametrize` for different inputs.

### 7.5 NumPy / Pandas Floating Point

```python
import numpy as np

result = np.mean(data)   # mutation: np.sum(data) / len(data)?
# Floating-point comparisons may mask arithmetic mutations
```

**Mitigation:** Use `np.testing.assert_almost_equal` with tight tolerance, or `assert_array_equal`.

### 7.6 Django ORM

```python
# Model query mutation
users = User.objects.filter(age__gte=18)  # mutation: > to <, >= to <=
# If test creates users of varying ages and counts results, mutation may still pass
```

**Mitigation:** Test with specific, verifiable query results. Assert exact sets, not counts.

### 7.7 Async Code (asyncio)

Python async code presents similar challenges to C# async — state machine coverage may be incomplete.

```python
async def fetch_data(url):
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as response:
            return await response.json()  # mutation inside async method
```

**Mitigation:** Use `pytest-asyncio` and assert exact return values.

---

## 8. Reading Results

```bash
# Text summary
mutmut result

# JSON output
mutmut result --json

# Show surviving mutants
mutmut show 1  # show mutant #1 details

# HTML report (requires mutmut-html)
pip install mutmut-html
mutmut html
# Opens html/index.html

# List untested code
mutmut junk
```

### Result interpretation

| Status | Meaning |
|---|---|
| ⚪ KILLED | Test caught the mutation |
| 🔵 SURVIVED | Test passed — mutation not detected |
| 🟡 TIMEOUT | Test timed out on mutant (may indicate infinite loop) |
| ⚫ SKIPPED | Mutant was skipped (config or exclusion) |

---

## 9. Quick Start — Cheat Sheet

```bash
# Install
pip install mutmut

# First run
mutmut run --paths-to-mutate src/ --tests-dir tests/

# With coverage optimization (faster)
mutmut run --paths-to-mutate src/ --tests-dir tests/ --use-coverage

# View results
mutmut result
mutmut result --json

# View specific survivor
mutmut show 42

# Exclude patterns
mutmut run --paths-to-mutate src/ --exclude-lines "logger\.(info|debug)"

# HTML report
pip install mutmut-html
mutmut html

# CI gate
SCORE=$(mutmut result --json | python3 -c "import json,sys; print(json.load(sys.stdin)['score'])")
if [ "$SCORE" -lt 60 ]; then exit 1; fi
```

---

## 10. Further Reading

- [MutMut documentation](https://mutmut.readthedocs.io)
- [MutMut GitHub](https://github.com/boxed/mutmut)
- [MutPy GitHub](https://github.com/mutpy/mutpy)
- [CosmicRay GitHub](https://github.com/sixty-north/cosmic-ray)
- [ACM 2024: Python Mutation Testing Tools Comparison](https://doi.org/10.1145/3629526.3645042)
- [Awesome Mutation Testing](https://github.com/theofidry/awesome-mutation-testing)
