# test-generator

Generate unit test stubs and boilerplate from source code analysis. Analyzes functions, classes, and modules to create comprehensive test scaffolding.

## Usage

```
/test-generator <file-or-directory>
```

Options:
- `--framework <name>` - Specify test framework (jest, pytest, go, etc.)
- `--style <unit|integration>` - Test style (default: unit)
- `--coverage` - Generate tests targeting uncovered code paths
- `--dry-run` - Preview tests without creating files

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for test generation. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: test, testing, unit test, mock, fixture
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, Cargo.toml, etc.
> 4. **Existing Tests** - Find existing test files to match patterns and conventions
>
> Return: Project type, test framework, test file naming convention, test directory structure, mocking libraries

From the Explore results, extract:
- Test framework and assertion library
- Test file naming convention (*.test.ts, *_test.go, test_*.py)
- Test directory structure (co-located, separate `tests/` folder)
- Mocking/stubbing libraries used
- Any testing conventions from CLAUDE.md

### Step 2: Analyze Source Code

For the target file(s), analyze:

1. **Functions/Methods**:
   - Function name and signature
   - Parameters and their types
   - Return type
   - Side effects (I/O, mutations, external calls)

2. **Classes**:
   - Constructor parameters
   - Public methods
   - Dependencies (injected or imported)

3. **Edge Cases**:
   - Null/undefined inputs
   - Empty collections
   - Boundary values
   - Error conditions

4. **Dependencies**:
   - External services to mock
   - Database calls
   - File system operations
   - Network requests

### Step 3: Generate Test Structure

Create test files following project conventions:

**JavaScript/TypeScript (Jest/Vitest):**
```typescript
import { describe, it, expect, vi } from 'vitest'; // or jest
import { functionName } from './source-file';

describe('functionName', () => {
  describe('when given valid input', () => {
    it('should return expected result', () => {
      // Arrange
      const input = /* */;

      // Act
      const result = functionName(input);

      // Assert
      expect(result).toBe(/* expected */);
    });
  });

  describe('when given invalid input', () => {
    it('should throw an error', () => {
      expect(() => functionName(null)).toThrow();
    });
  });

  describe('edge cases', () => {
    it('should handle empty input', () => {
      // ...
    });
  });
});
```

**Python (pytest):**
```python
import pytest
from source_file import function_name

class TestFunctionName:
    """Tests for function_name."""

    def test_returns_expected_result_for_valid_input(self):
        # Arrange
        input_value = ...

        # Act
        result = function_name(input_value)

        # Assert
        assert result == expected

    def test_raises_error_for_invalid_input(self):
        with pytest.raises(ValueError):
            function_name(None)

    @pytest.mark.parametrize("input,expected", [
        (case1_input, case1_expected),
        (case2_input, case2_expected),
    ])
    def test_handles_various_inputs(self, input, expected):
        assert function_name(input) == expected
```

**Go:**
```go
package mypackage

import (
    "testing"
)

func TestFunctionName(t *testing.T) {
    t.Run("returns expected result for valid input", func(t *testing.T) {
        // Arrange
        input := ...

        // Act
        result := FunctionName(input)

        // Assert
        if result != expected {
            t.Errorf("got %v, want %v", result, expected)
        }
    })

    t.Run("returns error for invalid input", func(t *testing.T) {
        _, err := FunctionName(nil)
        if err == nil {
            t.Error("expected error, got nil")
        }
    })
}
```

**Rust:**
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_function_name_with_valid_input() {
        // Arrange
        let input = ...;

        // Act
        let result = function_name(input);

        // Assert
        assert_eq!(result, expected);
    }

    #[test]
    #[should_panic(expected = "error message")]
    fn test_function_name_panics_on_invalid_input() {
        function_name(invalid_input);
    }
}
```

### Step 4: Generate Mocks/Stubs

For external dependencies:

**JavaScript/TypeScript:**
```typescript
// Mock external service
vi.mock('./external-service', () => ({
  fetchData: vi.fn().mockResolvedValue({ data: 'mocked' }),
}));

// Mock module
const mockDb = {
  query: vi.fn(),
  connect: vi.fn(),
};
```

**Python:**
```python
from unittest.mock import Mock, patch

@patch('module.external_service')
def test_with_mocked_service(mock_service):
    mock_service.fetch_data.return_value = {'data': 'mocked'}
    # ...

# Using pytest fixtures
@pytest.fixture
def mock_database():
    return Mock(spec=DatabaseClient)
```

**Go:**
```go
// Interface for mocking
type MockService struct {
    FetchDataFunc func() (Data, error)
}

func (m *MockService) FetchData() (Data, error) {
    return m.FetchDataFunc()
}
```

### Step 5: Add Test Cases

Generate tests for:

| Category | Test Cases |
|----------|------------|
| **Happy Path** | Normal input → expected output |
| **Validation** | Invalid input → proper error |
| **Boundaries** | Min/max values, empty arrays, zero |
| **Null Safety** | Null/undefined/nil handling |
| **Error Paths** | Service failures, timeouts, exceptions |
| **State** | Before/after mutations |

### Step 6: Output Generated Tests

Present tests with:
1. File path where test should be saved
2. Complete test code
3. Required imports/dependencies
4. Setup instructions if new dependencies needed

## Output Format

```
## Generated Tests

**Source**: [source file path]
**Test File**: [generated test file path]
**Framework**: [detected framework]

### Dependencies Needed

[List any new dev dependencies to install]

### Test Code

[Complete test file content]

### Test Cases Generated

| Function | Test Cases | Coverage |
|----------|------------|----------|
| funcA    | 5 cases    | happy, error, edge, null, boundary |
| funcB    | 3 cases    | happy, error, edge |

### Running Tests

[Command to run the generated tests]
```

## Test Naming Conventions

Follow framework conventions:

| Framework | File Pattern | Test Pattern |
|-----------|--------------|--------------|
| Jest/Vitest | `*.test.ts`, `*.spec.ts` | `it('should...')` |
| pytest | `test_*.py`, `*_test.py` | `def test_*():` |
| Go | `*_test.go` | `func Test*(t *testing.T)` |
| Rust | Same file, `#[cfg(test)]` | `#[test] fn test_*()` |
| RSpec | `*_spec.rb` | `it 'should...' do` |
| JUnit | `*Test.java` | `@Test void test*()` |

## Error Handling

### Cannot Parse Source File

```
Question: "Could not parse the source file. What should I do?"
Options:
  - Try parsing as a different language
  - Generate generic test template
  - Cancel
```

### No Testable Functions Found

```
Question: "No public functions/methods found in the file. What should I test?"
Options:
  - Generate integration tests for the module
  - Test internal functions (may need export changes)
  - Cancel
```

### Unknown Test Framework

```
Question: "Could not detect test framework. Which should I use?"
Options:
  - Jest (JavaScript/TypeScript)
  - Vitest (JavaScript/TypeScript)
  - pytest (Python)
  - unittest (Python)
  - Go testing (Go)
  - Specify other
```

### Existing Test File

```
Question: "Test file already exists at [path]. What should I do?"
Options:
  - Append new tests to existing file
  - Create with different name
  - Show diff of what would be added
  - Cancel
```

## Important Notes

- Match existing test style and conventions in the project
- Use descriptive test names that explain the scenario
- Prefer Arrange-Act-Assert (AAA) pattern
- Generate focused, single-assertion tests where practical
- Include TODO comments for tests needing manual implementation
- Don't generate tests for trivial getters/setters unless requested
- Consider data-driven tests for multiple similar cases
- Respect project's mocking library preferences
