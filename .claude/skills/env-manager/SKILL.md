---
name: env-manager
description: Manage environment variables and configuration files. Validates .env files, detects missing variables, generates templates, and ensures consistency across environments.
---

# env-manager

Manage environment variables and configuration files. Validates .env files, detects missing variables, generates templates, and ensures consistency across environments.

## Usage

```
/env-manager <command>
```

Commands:
- `validate` - Check .env against .env.example and code usage
- `sync` - Update .env.example from current .env (strips values)
- `check` - Find env vars used in code but not defined
- `generate` - Create .env.example from code analysis
- `compare` - Compare env files across environments

## Instructions

### Step 1: Discover Project Context

Use the **Explore** agent to discover project context:

**Explore Prompt:**
> Discover project context for environment management. Find and read:
>
> 1. **Root CLAUDE.md** - Read `CLAUDE.md` at project root. All instructions are MANDATORY.
> 2. **Relevant CLAUDE.md Files** - Search `**/CLAUDE.md` for keywords: env, environment, config, secrets, variables
> 3. **Env Files** - Find .env, .env.example, .env.local, .env.development, .env.production, etc.
> 4. **Config Files** - Find config files that reference environment variables
>
> Return: Env file locations, env loading mechanism, required variables, environment-specific configs

From the Explore results, extract:
- All env file locations
- How env vars are loaded (dotenv, native, framework-specific)
- Required vs optional variables
- Environment-specific overrides

---

## Commands

### Validate

Check that all required environment variables are present and properly formatted.

**Steps:**
1. Read .env.example (source of truth for required vars)
2. Read current .env file
3. Compare and identify missing/extra variables
4. Validate format (no spaces around =, proper quoting)

**Validation Rules:**
```
✓ Variable exists in both .env and .env.example
✓ No spaces around = sign
✓ Values with spaces are quoted
✓ No trailing whitespace
✓ URLs are valid format
✓ Ports are numeric
✓ Boolean values are consistent (true/false, 1/0)
```

**Output:**
```
## Environment Validation

**File**: .env
**Template**: .env.example

### Status: ⚠️ Issues Found

### Missing Variables (defined in .env.example)

| Variable | Description | Required |
|----------|-------------|----------|
| DATABASE_URL | Database connection string | Yes |
| REDIS_HOST | Redis server hostname | No |

### Extra Variables (not in .env.example)

| Variable | Recommendation |
|----------|----------------|
| DEBUG_MODE | Add to .env.example if needed |
| TEMP_TOKEN | Remove if temporary |

### Format Issues

| Line | Issue | Fix |
|------|-------|-----|
| 5 | Space before = | Remove space |
| 12 | Unquoted value with spaces | Add quotes |

### Validation Passed

- ✓ 15 variables properly defined
- ✓ URL formats valid
- ✓ Port numbers valid
```

---

### Sync

Update .env.example from current .env, keeping descriptions and stripping sensitive values.

**Steps:**
1. Read current .env
2. Read existing .env.example (preserve comments/descriptions)
3. Add new variables from .env
4. Remove variables no longer in .env
5. Strip actual values, use placeholders

**Placeholder Rules:**
```
# String values
API_KEY=your_api_key_here
SECRET=your_secret_here

# URLs
DATABASE_URL=postgresql://user:password@host:5432/dbname
REDIS_URL=redis://localhost:6379

# Ports/Numbers
PORT=3000
MAX_CONNECTIONS=10

# Booleans
DEBUG=false
ENABLE_FEATURE=true

# Lists
ALLOWED_ORIGINS=http://localhost:3000,http://localhost:8080
```

**Output:**
```
## Sync Results

**Source**: .env (23 variables)
**Target**: .env.example

### Changes

| Action | Variable | Note |
|--------|----------|------|
| Added | NEW_API_ENDPOINT | New variable detected |
| Added | FEATURE_FLAG_X | New variable detected |
| Removed | OLD_SETTING | No longer in .env |
| Updated | PORT | Value stripped |

### Generated .env.example

[Show file content or diff]
```

---

### Check

Find environment variables used in code but not defined in env files.

**Steps:**
1. Scan code for env var access patterns
2. Compare against defined variables
3. Report undefined usages

**Patterns to Search:**

**JavaScript/TypeScript:**
```javascript
process.env.VARIABLE_NAME
process.env['VARIABLE_NAME']
import.meta.env.VARIABLE_NAME
```

**Python:**
```python
os.environ['VARIABLE_NAME']
os.environ.get('VARIABLE_NAME')
os.getenv('VARIABLE_NAME')
```

**Go:**
```go
os.Getenv("VARIABLE_NAME")
os.LookupEnv("VARIABLE_NAME")
```

**Rust:**
```rust
std::env::var("VARIABLE_NAME")
env::var("VARIABLE_NAME")
```

**Ruby:**
```ruby
ENV['VARIABLE_NAME']
ENV.fetch('VARIABLE_NAME')
```

**Output:**
```
## Environment Variable Check

### Undefined Variables (used in code, not in .env)

| Variable | File | Line | Context |
|----------|------|------|---------|
| API_SECRET | src/auth.ts | 23 | process.env.API_SECRET |
| CACHE_TTL | src/cache.ts | 15 | process.env.CACHE_TTL |

### Defined but Unused

| Variable | Recommendation |
|----------|----------------|
| OLD_API_KEY | Consider removing |
| DEBUG_LEGACY | Consider removing |

### Summary

- **Total in .env**: 20
- **Used in code**: 18
- **Missing**: 2
- **Unused**: 2
```

---

### Generate

Create .env.example by analyzing code for environment variable usage.

**Steps:**
1. Scan all source files for env var access
2. Extract variable names
3. Infer types from usage context
4. Generate categorized template

**Output:**
```
## Generated .env.example

### Analysis Summary

Scanned 45 files, found 23 environment variables

### Generated Template

# ================================
# Application Configuration
# ================================

# Server settings
PORT=3000
HOST=localhost
NODE_ENV=development

# ================================
# Database
# ================================

# Primary database connection
DATABASE_URL=postgresql://user:password@localhost:5432/myapp

# ================================
# Authentication
# ================================

# JWT configuration
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRES_IN=7d

# OAuth providers
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret

# ================================
# External Services
# ================================

# Email service
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=your_smtp_user
SMTP_PASSWORD=your_smtp_password

# ================================
# Feature Flags
# ================================

ENABLE_NEW_DASHBOARD=false
ENABLE_ANALYTICS=true
```

---

### Compare

Compare environment files across different environments.

**Usage:**
```
/env-manager compare .env.development .env.production
```

**Output:**
```
## Environment Comparison

**Files**: .env.development vs .env.production

### Variables by Status

#### ✓ Present in Both (matching structure)
| Variable | Dev Value | Prod Value |
|----------|-----------|------------|
| PORT | 3000 | 8080 |
| LOG_LEVEL | debug | error |

#### ⚠️ Only in Development
| Variable | Recommendation |
|----------|----------------|
| DEBUG_SQL | Remove or add to prod |
| MOCK_EXTERNAL_API | Dev-only, OK |

#### ⚠️ Only in Production
| Variable | Recommendation |
|----------|----------------|
| CDN_URL | Add to dev with local value |
| SENTRY_DSN | Add to dev or document |

#### ❌ Potential Issues
| Variable | Issue |
|----------|-------|
| API_URL | Different domains - verify intentional |
| DATABASE_URL | Ensure dev doesn't point to prod |
```

---

## Final Step: Evaluate Reusable Patterns

Run the **pattern-evaluator** agent to assess whether any reusable patterns (rules, skills, or agents) were discovered during this session and should be persisted.

## Error Handling

### No .env File Found

```
Question: "No .env file found. What should I do?"
Options:
  - Create .env from .env.example
  - Generate .env.example from code analysis
  - Specify env file location
  - Cancel
```

### No .env.example Exists

```
Question: "No .env.example found. Should I create one?"
Options:
  - Generate from current .env (strip values)
  - Generate from code analysis
  - Skip template generation
  - Cancel
```

### Sensitive Value Detected

```
Warning: Sensitive value detected in .env.example

Variable: API_SECRET
Value: actual_secret_value_123

Question: "Should I replace with placeholder?"
Options:
  - Yes, use placeholder
  - Skip this variable
  - Cancel
```

### Invalid Syntax

```
## Syntax Errors in .env

Line 15: Missing value after =
Line 23: Invalid character in variable name
Line 31: Unclosed quote

Fix these issues before proceeding.
```

## Important Notes

- NEVER commit actual .env files to git
- Always add .env to .gitignore
- .env.example should contain placeholders, not real values
- Document required vs optional variables
- Use consistent naming (SCREAMING_SNAKE_CASE)
- Group related variables with comments
- Consider using validation libraries (envalid, pydantic, etc.)
- For sensitive values, consider secret managers over env files
- Different frameworks load env files differently - know your stack
