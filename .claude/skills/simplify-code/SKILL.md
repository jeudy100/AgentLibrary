# simplify-code

Analyze code for complexity and suggest simplifications. Detects verbose patterns, redundant logic, and opportunities for cleaner code.

## Usage

```
/simplify-code
```

Analyzes recent changes or discussed files by default. You can also specify:
- A file path to analyze a specific file
- A directory to analyze all source files in it
- `--apply` to apply safe, mechanical transformations (use with caution)

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for simplifying code. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: conventions, style, patterns, complexity, simplification
> 3. **Project Type** - Detect from package.json, pyproject.toml, go.mod, Cargo.toml, etc.
>
> Return: Project language, coding conventions, complexity guidelines

From the Explore results, extract:
- Project type and language
- Coding conventions and patterns
- Any simplification guidelines from CLAUDE.md

### Step 2: Identify Target Code

Determine what to analyze based on context:

**If specific files provided:**
- Analyze the provided files

**If current context has recent changes:**
```bash
# Get recently modified files
git diff --name-only HEAD~5
# Or unstaged changes
git diff --name-only
```

**If no specific target:**
- Analyze source files in the project
- Skip test files, generated files, node_modules, vendor, etc.

### Step 3: Analyze for Complexity Issues

Scan the code for these patterns:

#### Structural Issues
| Issue | Threshold | Example |
|-------|-----------|---------|
| Deep nesting | > 3 levels | Nested if/for/while |
| Long functions | > 30 lines | Functions doing too much |
| Many parameters | > 4 params | Functions with long signatures |
| Long files | > 300 lines | Files doing too much |
| Complex conditionals | > 3 conditions | `if (a && b \|\| c && d)` |

#### Redundant Code
| Pattern | Simplification |
|---------|---------------|
| `if (x) return true else return false` | `return x` |
| `if (x) return true; return false` | `return x` |
| `if (x == true)` | `if (x)` |
| `if (x == false)` | `if (!x)` |
| `if (x != null && x != undefined)` | `if (x != null)` or optional chaining |
| Dead code after return | Remove unreachable code |
| Unused variables | Remove or use |
| Duplicate code blocks | Extract to function |

#### Verbose Patterns
| Verbose | Simplified |
|---------|-----------|
| Manual loops for transformation | map/filter/reduce |
| Manual null checks | Optional chaining |
| String concatenation | Template literals |
| Callback nesting | async/await |
| Multiple assignments | Destructuring |

#### AI-Generated Code Smells
| Smell | Issue |
|-------|-------|
| Overly verbose variable names | `theResultOfTheCalculation` -> `result` |
| Excessive inline comments | Comments stating the obvious |
| Boilerplate wrapper functions | Single-use wrappers |
| Over-abstraction | Premature abstraction for one use case |
| Unnecessary type annotations | Types that can be inferred |

### Step 4: Language-Specific Simplifications

#### JavaScript/TypeScript
```javascript
// Verbose
if (value !== null && value !== undefined) {
  result = value.property;
}
// Simplified
result = value?.property;

// Verbose
const x = value !== null && value !== undefined ? value : defaultValue;
// Simplified
const x = value ?? defaultValue;

// Verbose
const obj = { name: name, age: age };
// Simplified
const obj = { name, age };

// Verbose
function getData() {
  return fetch(url).then(res => res.json()).then(data => process(data));
}
// Simplified
async function getData() {
  const res = await fetch(url);
  const data = await res.json();
  return process(data);
}
```

#### Python
```python
# Verbose
result = []
for item in items:
    if item.active:
        result.append(item.name)
# Simplified
result = [item.name for item in items if item.active]

# Verbose
name = "Hello " + first_name + " " + last_name
# Simplified
name = f"Hello {first_name} {last_name}"

# Verbose
if value is not None:
    result = value
else:
    result = default
# Simplified
result = value if value is not None else default
# Or with walrus operator (Python 3.8+)
if (result := get_value()) is not None:
    process(result)

# Verbose
file = open("data.txt")
try:
    data = file.read()
finally:
    file.close()
# Simplified
with open("data.txt") as file:
    data = file.read()
```

#### Go
```go
// Verbose
var result string
result = getValue()
// Simplified
result := getValue()

// Verbose
if err != nil {
    return fmt.Errorf("failed: %v", err)
}
// Simplified (Go 1.13+)
if err != nil {
    return fmt.Errorf("failed: %w", err)
}

// Verbose
func process() (result int, err error) {
    result = 0
    err = nil
    // ...
    return result, err
}
// Simplified with named returns
func process() (result int, err error) {
    // ...
    return
}
```

#### Rust
```rust
// Verbose
match result {
    Ok(value) => value,
    Err(e) => return Err(e),
}
// Simplified
result?

// Verbose
match optional {
    Some(value) => do_something(value),
    None => {},
}
// Simplified
if let Some(value) = optional {
    do_something(value);
}

// Verbose
let mut result = Vec::new();
for item in items {
    result.push(item.transform());
}
// Simplified
let result: Vec<_> = items.iter().map(|item| item.transform()).collect();
```

#### Java
```java
// Verbose
List<String> names = new ArrayList<>();
for (User user : users) {
    names.add(user.getName());
}
// Simplified
List<String> names = users.stream()
    .map(User::getName)
    .collect(Collectors.toList());

// Verbose
String result;
if (value != null) {
    result = value;
} else {
    result = "default";
}
// Simplified
String result = Optional.ofNullable(value).orElse("default");

// Verbose
public class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) { this.x = x; this.y = y; }
    // getters, equals, hashCode, toString...
}
// Simplified (Java 16+)
public record Point(int x, int y) {}
```

#### C#
```csharp
// Verbose
string result;
if (value != null) {
    result = value.ToUpper();
} else {
    result = null;
}
// Simplified
string result = value?.ToUpper();

// Verbose
List<string> names = new List<string>();
foreach (var user in users) {
    names.Add(user.Name);
}
// Simplified
var names = users.Select(u => u.Name).ToList();

// Verbose
if (obj is MyClass) {
    var typed = (MyClass)obj;
    typed.DoSomething();
}
// Simplified
if (obj is MyClass typed) {
    typed.DoSomething();
}
```

### Step 5: Prioritize Findings

Categorize issues by:

**Impact** (High to Low):
1. Bug risks (dead code, unreachable code)
2. Maintainability (complex functions, deep nesting)
3. Readability (verbose patterns)
4. Style (minor improvements)

**Risk** (for applying changes):
- **Safe**: Mechanical transformations (redundant boolean, obvious simplifications)
- **Moderate**: Restructuring (extracting functions, changing loops)
- **High**: Architectural changes (require understanding of business logic)

### Step 6: Generate Report

Output findings in this format:

```markdown
## Code Simplification Report

**Files Analyzed**: [count]
**Issues Found**: [count]
**Estimated Reduction**: [X lines / Y% reduction possible]

---

## Critical (Bug Risks)

### [Issue Title]
**File**: [path:line]
**Issue**: [description]
**Risk**: [Safe / Moderate / High]

**Before:**
```[language]
[original code]
```

**After:**
```[language]
[simplified code]
```

**Technique**: [name of simplification technique]

---

## Warnings (Maintainability)

### [Issue Title]
**File**: [path:line]
**Issue**: [description]
**Risk**: [Safe / Moderate / High]

**Before:**
```[language]
[original code]
```

**After:**
```[language]
[simplified code]
```

---

## Suggestions (Readability)

### [Issue Title]
**File**: [path:line]

**Before:**
```[language]
[original code]
```

**After:**
```[language]
[simplified code]
```

---

## Summary

| Metric | Before | After (if applied) |
|--------|--------|-------------------|
| Total Lines | X | Y |
| Avg Function Length | X | Y |
| Max Nesting Depth | X | Y |
| Complex Conditionals | X | Y |

## Safe to Apply

These changes are mechanical and safe to apply automatically:
- [ ] [Change 1] ([file:line])
- [ ] [Change 2] ([file:line])

Run `/simplify-code --apply` to apply safe changes.

## Requires Review

These changes need human review before applying:
- [ ] [Change 1] - [why it needs review]
- [ ] [Change 2] - [why it needs review]

## Recommendations

1. [High-impact recommendation]
2. [Medium-impact recommendation]
3. [Optional improvement]

**Next Steps:**
- Run tests after applying any changes
- Review moderate/high risk changes manually
- Consider refactoring [specific area] in a separate PR
```

---

## Safety Rules

### Default Mode (Suggest Only)
- Analyze and report findings
- Show before/after comparisons
- Do not modify any files

### Apply Mode (--apply)
Only apply changes that are:
- Mechanical transformations
- Safe (no behavior change possible)
- Examples: redundant boolean simplification, obvious null coalescing

**Never auto-apply:**
- Function extraction/reorganization
- Architectural changes
- Anything requiring business logic understanding
- Changes in complex conditional logic

### Always Recommend
- Run tests after any changes
- Review changes before committing
- Consider impact on team/codebase conventions

---

## Tips

- Focus on high-impact, low-risk improvements first
- Respect project conventions (from detect-context)
- Don't over-simplify at the cost of readability
- Consider team familiarity with advanced language features
- One simplification per issue for clarity
