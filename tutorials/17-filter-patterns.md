### Topic 17: Filter Patterns
Now, **filter patterns** refine your triggers (`on` events) using glob-like syntax for precision. Instead of triggering on *every* push, filter by branches, tags, paths (files changed), or activity types. This is clutch for data science: e.g., run validation only on data file changes (`paths: ['data/**']`) or deploy on tagged releases (`tags: ['v*']`), avoiding unnecessary runs on README tweaks.

From the official GitHub documentation (workflow syntax section on event activity types and filters):
- **Structure**: Under `on.<event>` (e.g., `push`, `pull_request`), add filters like `branches`, `tags`, `paths`, `paths-ignore` as lists of glob patterns. Globs use `*` (any chars), `**` (recursive dirs), `!` (negate, in some contexts), and `?` (single char). For advanced: `types` for activity (e.g., `opened`, `synchronize`).
- **branches/tags**: Filter by name (e.g., `['main', 'develop']` or `['feature/*']`). Use `branches-ignore` to skip.
- **paths/paths-ignore**: File/dir paths from root (e.g., `['src/**.py', 'data/*.csv']`)—triggers only if matching changes. Case-insensitive on Linux, sensitive on Windows.
- **types**: For PRs/issues (e.g., `['opened', 'reopened']`); for workflows, combine with `activity_types`.
- **Key Notes**:
  - Filters are ANDed within a list (e.g., branch *and* path must match).
  - Multiple filters: Use arrays or separate events (e.g., `on: push: branches: [...]` or `on: [push, pull_request]`).
  - For DS: `paths: ['notebooks/**']` for Jupyter changes; `tags: ['release/v*']` for model versioning.
  - Caching: Patterns evaluated post-checkout, so paths see full repo.

**Syntax Overview**:
```yaml
on:
  push:
    branches: ['main', 'feature/*']  # Include these branches
    branches-ignore: ['dev']  # Skip dev pushes
    paths: ['data/**', 'src/*.py']  # Only if these change
    paths-ignore: ['docs/**']  # Ignore doc changes
  pull_request:
    types: ['opened', 'synchronize']  # PR events
    branches: ['main']
  release:
    types: [created]
    tags: ['v*']  # Semantic versioning
```

**Examples Tailored to Data Science**:
1. **Path Filters for Data Changes**: Trigger analysis only on dataset updates.
   ```yaml
   on:
     push:
       paths:
         - 'data/*.csv'
         - 'data/*.json'
       branches: ['main']
   jobs:
     analyze:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Run EDA
           run: python eda.py  # Only if data files changed
   ```

2. **Branch + Tag Filters**: Deploy on main pushes or release tags.
   ```yaml
   on:
     push:
       branches: ['main']
     release:
       types: [published]
       tags: ['v[0-9]+.*']  # Matches v1.0, v2.1, etc.
   jobs:
     deploy:
       needs: test
       runs-on: ubuntu-latest
       steps:
         - name: Deploy model
           run: python deploy.py  # Tags for prod releases
   ```

3. **Advanced: PR Types + Paths**: Lint code on PR opens/reopens if Python files touched.
   ```yaml
   on:
     pull_request:
       types: ['opened', 'reopened', 'synchronize']
       paths:
         - '**/*.py'
         - 'requirements.txt'
       branches: ['develop', 'main']
   jobs:
     lint:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/setup-python@v5
           with:
             python-version: '3.11'
         - name: Lint
           run: pylint src/
   ```

**Common Pitfalls**:
- Globs too broad? `*` matches everything—use `**/*.csv` for specifics.
- Paths from root: `'data/*.csv'` misses `Data/*.csv` (case-sensitive on some runners).
- No match = no trigger: Test with small pushes; use `workflow_run` for downstream if needed.
- Ignore lists: `paths-ignore` skips if *any* ignored path changes—combine carefully.
- Tags/branches: Refs like `refs/heads/main` in expressions, but lists use short names.
- Performance: Many paths slow evaluation—limit to <100 patterns.

Filters keep workflows lean and targeted. Final topic next: Full examples + debugging (Topic 18).

#### Practice Questions for Topic 17
1. Write a YAML snippet for the `on:` trigger that runs on pushes to `main` or `develop` branches, but only if paths in `data/` or `models/` change (use `paths: ['data/**', 'models/**']`).

2. Fix this broken filter (issues: invalid glob in branches, paths as string not list, types for wrong event, missing branches for PR):
   ```
   on:
     push:
       branches: [main, feature/??]  # ?? invalid for multi-char
       paths: 'src/*.py'  # Not list
     pull_request:
       types: [push]  # Wrong types
       paths: ['data/**']
   jobs:
     test: ...
   ```

3. Create an `on:` section for a release workflow that triggers on `release: types: [published]` with tags matching `v*.*.*` (e.g., v1.2.3), and also on PRs to `main` only if `notebooks/` changes (`pull_request: branches: [main], paths: ['notebooks/**']`).

