### Topic 5: Triggers: on (Advanced: Activity Types, Filters, Schedules)

Building on basic events, the docs detail advanced configurations under `on` using maps. This allows specifying activity types (what sub-events trigger), filters (branches, tags, paths), and schedules (cron for timed runs).

Key points:
- **Advanced Syntax**: Use `on: <event>: { ... }` for config. For multiple events, `on: { event1: { ... }, event2: { ... } }`.
- **Activity Types (types)**: For events like `pull_request`, specify sub-events with `types: [opened, synchronize, reopened]`. Defaults vary by event (e.g., push defaults to all pushes). List common ones: for `pull_request`: opened, closed, edited, etc.; for `issues`: opened, deleted, etc. Use when you want granular control (e.g., only on PR opens, not edits).
- **Filters**: Limit triggers:
  - `branches`: List of branch names (e.g., `branches: [main, develop]`). Supports globs (`*`, `**` for recursive).
  - `branches-ignore`: Exclude branches (e.g., ignore feature/*).
  - `tags`: Similar for tags (e.g., `tags: [v*]` for version tags).
  - `tags-ignore`: Exclude tags.
  - `paths`: Trigger only if changes in specific paths (e.g., `paths: [data/**, scripts/*.py]` for data folders and Python scripts).
  - `paths-ignore`: Exclude paths.
  - Globs: Use `!` for negation in lists, but prefer -ignore keys. We'll cover advanced patterns in Topic 17.
- **Schedules**: Use `on: schedule: - cron: 'minute hour day month weekday'`. Cron syntax: e.g., '0 0 * * *' for daily at midnight UTC. Multiple crons in a list. Useful for data scientists: daily data refreshes or model retraining.
- **Combining**: You can mix types and filters per event. For `workflow_dispatch`, add `inputs` for parameters (e.g., user inputs in manual runs: `inputs: { logLevel: { description: 'Log level', required: true } }`).
- **Data Science Angle**: Schedule nightly data backups (`schedule`), filter to data/ dir changes (`paths`), or trigger on PRs to main only (`branches: [main]`).
- **Best Practices**: Use filters to avoid unnecessary runs (saves billing minutes). Test with `workflow_dispatch`. Cron is UTC-based; adjust for timezones.

Examples:
```yaml
# Advanced with types and filters
on:
  pull_request:
    types: [opened, synchronize]
    branches:
      - main
      - 'feature/**'
    paths-ignore:
      - 'docs/**'

# Schedule example
on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC
  workflow_dispatch:  # Also manual

# With inputs for dispatch
on:
  workflow_dispatch:
    inputs:
      dataset:
        description: 'Dataset to process'
        required: false
        default: 'default.csv'
```

Common Pitfalls: Invalid cron (test with crontab.guru), mismatched types for events, or globs not matching (case-sensitive).

#### Practice Questions for Topic 5
1. Add advanced config to this `on: pull_request` trigger: Include `types: [opened, closed]`, `branches: [main]`, and `paths: ['src/*.py', 'data/**']`.
   ```yaml
   name: PR Workflow
   on: pull_request
   jobs:
     check:
       runs-on: ubuntu-latest
   ```
2. Write a workflow top-level with `on` for a scheduled daily run at 5 AM UTC, plus manual dispatch with an input for 'model_type' (description: 'Type of ML model', required: true).
3. Fix this invalid advanced trigger: The cron is wrong, and branches should use a list.
   ```yaml
   on:
     schedule: cron: '0 5 * *'  # Missing weekday
     push:
       branches: main  # Not a list
   ```

#### Solutions
1. Solution:
   ```yaml
   name: PR Workflow
   on:
     pull_request:
       types: [opened, closed]
       branches:
         - main
       paths:
         - 'src/*.py'
         - 'data/**'
   jobs:
     check:
       runs-on: ubuntu-latest
   ```

2. Solution:
   ```yaml
   on:
     schedule:
       - cron: '0 5 * * *' # Daily at 5 AM UTC
     workflow_dispatch:
       inputs:
         model_type:
           description: 'Type of ML model'
           required: true
   ```

3. Solution:
   ```yaml
   on:
     schedule:
       - cron: '0 5 * * *'
     push:
       branches:
         - main
   ```
