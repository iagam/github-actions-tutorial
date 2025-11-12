### Topic 3: Workflow Naming: name and run-name

#### Explanation
With workflow files covered, let's discuss naming. The GitHub docs explain that workflows can have a display name for easy identification in the UI, and individual runs can have dynamic names for better logging.

Key points:
- **name**: This is an optional root-level key that sets the workflow's name, shown in the GitHub Actions tab of your repo. If omitted, GitHub uses the filename (e.g., `data-check.yml` becomes "data-check"). Use it for clarity, especially with multiple workflows. It can be a simple string.
- **run-name**: This optional root-level key sets a name for each specific run of the workflow. It can be static (just a string) or dynamic using expressions like `${{ github.actor }}` (the user who triggered it) or `${{ github.ref }}` (branch/ref). Useful for distinguishing runs in logs, e.g., "Data Pipeline Run by @username on branch main".
- **Expressions**: GitHub Actions uses `${{ }}` for context variables (e.g., `${{ github.event_name }}` for the trigger event). These are evaluated at runtime. We'll dive deeper into expressions later, but for naming, they're great for personalization.
- **Best Practices**: Keep names concise and descriptive. For data scientists, something like "ML Model Training" or "Daily Data Ingestion Run ${{ github.triggering_actor }}". Names are strings, so quote if needed (e.g., for spaces), but unquoted is fine for simple ones.
- **Data Science Angle**: Use `run-name` for traceability in experiments, e.g., including commit SHA or branch to link back to specific data/model versions.

Examples:
```yaml
# Static name
name: Data Processing Pipeline

# Dynamic run-name
run-name: Processing Run on ${{ github.ref }} by ${{ github.actor }}

on: push  # (Trigger example, ignore for now)

jobs:  # (Placeholder)
  process:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Running..."
```

If omitted, the workflow name defaults to the file name, and run-name defaults to something like "Commit message / branch".

Common Pitfalls: Typos in expressions (they must be valid), or forgetting that `name` is workflow-wide while `run-name` is per-run.

#### Practice Questions
1. Add `name` and `run-name` to this basic workflow snippet. Set `name` to "Model Evaluation", and `run-name` dynamically to "Eval on branch ${{ github.ref }}".
   ```yaml
   on: push
   jobs:
     eval:
       runs-on: ubuntu-latest
       steps:
         - run: echo "Evaluating..."
   ```
2. Write a full workflow top-level structure (no jobs yet) with `name: "Daily Report Generator"` (quoted because of space), `run-name: "Report for ${{ github.event_name }}"`, and `on: schedule` (placeholder trigger).
3. Fix this invalid naming: The `run-name` expression has a syntax error.
   ```yaml
   name: Test Workflow
   run-name: Run by ${ github.actor }  # Missing ${{ }}
   on: push
   ```

#### Solutions
1. Solution:
   ```yaml
   name: Model Evaluation
   run-name: "Eval on branch ${{ github.ref }}"
   on: push
   jobs:
     eval:
       runs-on: ubuntu-latest
       steps:
         - run: echo "Evaluating..."
   ```

2. Solution:
   ```yaml
   name: "Daily Report Generator"
   run-name: "Report for ${{ github.event_name }}"
   on: schedule
   ```

3. Solution:
   ```yaml
   name: "Test Workflow"
   run-name: "Run by ${{ github.actor }}"
   on: push
   ```
