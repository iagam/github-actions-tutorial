### Topic 1: YAML Basics

#### Explanation
GitHub Actions workflows are defined in YAML files, so understanding YAML is the foundation. YAML (YAML Ain't Markup Language) is a human-readable data serialization format that's stricter about indentation than Python but flexible for configs. It's case-sensitive, uses spaces (not tabs) for indentation (usually 2 spaces), and doesn't require quotes for most strings unless they contain special chars.

Key concepts from the docs (and general YAML knowledge):
- **Structure**: YAML uses key-value pairs. Keys end with a colon (`:`), and values can be scalars (strings, numbers), lists (arrays), or maps (dictionaries/objects).
- **Indentation**: Must be consistent; nested elements are indented further.
- **Data Types**:
  - Strings: Can be unquoted (`hello`), single-quoted (`'hello'`), or double-quoted (`"hello"`). Use quotes for strings with spaces or special chars like `:`, `{`, `}`.
  - Numbers: Integers (e.g., `5`) or floats (e.g., `3.14`).
  - Booleans: `true` or `false` (unquoted).
  - Lists: Start with `-` (hyphen), each on a new line, indented.
  - Maps: Nested key-value pairs.
  - Multi-line strings: Use `|` (literal, preserves newlines) or `>` (folded, collapses newlines).
- **Comments**: Start with `#`.
- **Expressions in GitHub Actions**: You'll see `${{ expression }}` for dynamic values (e.g., variables), but that's GitHub-specific—we'll cover it later.
- **Common Pitfalls**: No tabs for indent; colons need a space after; lists/maps must align properly.

Examples:
```yaml
# Simple key-value
key: value  # String value

# Nested map
person:
  name: Alice  # Unquoted string
  age: 30      # Number
  is_data_scientist: true  # Boolean

# List
fruits:
  - apple
  - banana
  - cherry

# Multi-line string
description: |
  This is a
  multi-line
  string.

# Quoted string with special chars
special: "Hello: World!"
```

In workflows, everything nests under root keys like `name:` or `on:`.

#### Practice Questions
1. Write a simple YAML snippet for a data scientist's profile: Include a map with keys `name` (string), `skills` (list of 3 strings like "Python", "SQL", "ML"), and `bio` (multi-line string describing yourself in 2 lines).
2. Fix this broken YAML (it has indentation and quoting issues):
   ```
   user:
   name: Bob
     role: 'Data Analyst'
   tools:
   -Pandas
   - NumPy
   ```
3. Create a YAML list of datasets, where each item is a map with `name` (string) and `size_mb` (number).

#### Solutions
1. Solution:
   ```yaml
   data_scientist:
     name: Bob
     skills:
       - Python
       - SQL
       - ML
     bio: |
       I love data and
       I like numbers.
   ```

2. Solution:
   ```yaml
   user:
     name: Bob
     role: 'Data Analyst'
     tools:
       - Pandas
       - NumPy
   ```

3. Solution:
   ```yaml
   datasets:
     - name: Traffic Dataset
       size_mb: 1024
     - name: Sensor Data
       size_mb: 256
   ```
