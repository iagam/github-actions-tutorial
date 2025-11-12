### Topic 4: Triggers: on (Basic Events)

Triggers define when your workflow runs. The docs cover the `on` key as a root-level required element that specifies events. For basics, we'll focus on simple event triggers without filters.

Key points:
- **on**: This key can be a single event (string), a list of events, or a map for more config. Events are GitHub webhook events like code pushes or PRs.
- **Basic Events**: Common ones include:
  - `push`: Runs on code pushed to the repo (default: any branch).
  - `pull_request`: Runs on PR opened, synchronized, reopened, etc.
  - `workflow_dispatch`: Allows manual triggering via GitHub UI/API (useful for testing data pipelines).
  - `schedule`: For cron-based timing (e.g., daily data jobs—we'll detail in next topic).
  - Others: `issues` (on issue events), `release` (on releases), etc.
- **Syntax**:
  - Single: `on: push`
  - List: `on: [push, pull_request]`
  - Map: `on: { push: {...}, pull_request: {...} }` (for advanced, covered next).
- **Data Science Angle**: Use `push` for CI on data scripts, `workflow_dispatch` for ad-hoc model runs, `schedule` for nightly ETL.
- **Best Practices**: Start simple; avoid over-triggering to save resources (GitHub has limits on minutes).

Examples:
```yaml
# Single event
on: push

# Multiple events
on:
  - push
  - pull_request

# Map for basic (no filters yet)
on:
  push: null  # Equivalent to simple push
```

If no `on`, the workflow won't run. Events are case-sensitive.

Common Pitfalls: Misspelling events (e.g., "Push" instead of "push"), or using invalid events.

#### Practice Questions for Topic 4
1. Add a basic `on` trigger to this workflow snippet for push events only.
   ```yaml
   name: Basic Trigger Workflow
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - run: echo "Triggered!"
   ```
2. Write a top-level `on` as a list for both push and workflow_dispatch events. Include `name: "Manual Data Run"`.

3. Fix this invalid trigger: It uses a wrong event name.
   ```yaml
   name: Error Workflow
   on: Push
   jobs:
     error:
       runs-on: ubuntu-latest
   ```

#### Solutions
1. Solution:
   ```yaml
   name: Basic Trigger Workflow
   on: push
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - run: echo "Triggered!"
   ```

2. Solution:
   ```yaml
   name: "Manual Data Run"
   on:
     - push
     - workflow_dispatch
   ```

3. Solution:
   ```yaml
   name: Error Workflow
   on: push # Should be lowercase
   jobs:
     error:
       runs-on: ubuntu-latest
   ```

