### Topic 9: Concurrency: concurrency

After permissions, let's cover concurrency. The GitHub docs describe `concurrency` as a root-level key to manage how multiple workflow runs execute, preventing conflicts like race conditions in shared resources (e.g., deployments or data writes).

Key points:
- **concurrency**: A map with `group` (required string or expression for grouping runs) and optional `cancel-in-progress` (boolean: true to cancel previous runs in the group when a new one starts).
- **Purpose**: Limits to one run at a time per group. Useful for workflows that update the same environment or data (e.g., only one deployment at a time).
- **Group Syntax**: Can be static (e.g., `group: deployment`) or dynamic with expressions like `${{ github.workflow }}-${{ github.ref }}` (groups by workflow and branch).
- **Scope**: Root-level only—applies to the entire workflow. If multiple workflows share a group name, they coordinate across the repo.
- **Behavior**: When a new run starts, if another in the group is running or pending, it's queued (or canceled if `cancel-in-progress: true`).
- **Data Science Angle**: For pipelines processing shared datasets, use to avoid concurrent writes (e.g., group by "data-update" to serialize DB updates or model training).
- **Best Practices**: Use unique groups per critical section. Set `cancel-in-progress: true` for CI to skip outdated runs. Expressions help branch-specific concurrency.

Examples:
```yaml
# Simple concurrency
concurrency:
  group: my-group
  cancel-in-progress: true

on: push

jobs:
  deploy:
    steps:
      - run: echo "Deploying..."

# Dynamic group
concurrency:
  group: ${{ github.ref }}  # One per branch
  cancel-in-progress: false  # Queue instead of cancel
```

Common Pitfalls: Duplicate groups across workflows causing unexpected queuing. Forgetting it's repo-wide for same group names.

#### Practice Questions for Topic 9
1. Add `concurrency` to this workflow with `group: 'ci-build'` and `cancel-in-progress: true`.
   ```yaml
   name: CI Workflow
   on: push
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - run: make build
   ```
2. Write a top-level `concurrency` using a dynamic group: `${{ github.workflow }}-${{ github.head_ref || github.ref }}`, with `cancel-in-progress: true`.

3. Fix this invalid concurrency: Missing required group, and boolean is string.
   ```yaml
   concurrency:
     cancel-in-progress: 'true'  # Wrong type
   on: pull_request
   ```

