### Topic 12: Steps in Jobs Basics: jobs.<job_id> (Structure, needs, if, runs-on)
Now that we've covered earlier topics like triggers, variables, and steps, we're diving into **jobs**—the core units of work in a GitHub Actions workflow. A workflow consists of one or more jobs, each of which runs in parallel by default (unless specified otherwise). Jobs are defined under the `jobs` key at the root level of your YAML file. This allows you to break down complex tasks, like a data science pipeline, into modular pieces: e.g., one job for data validation, another for model training, and a third for deployment.

From the official GitHub documentation (workflow syntax section on jobs):
- **Structure**: Each job is a map under `jobs.<job_id>`, where `<job_id>` is a unique string (e.g., `build`, `test`, `deploy`). Inside the job map, you define steps (which we'll cover in the next topic), but at a high level, jobs can include keys like `runs-on`, `needs`, `if`, `outputs`, and more. Jobs run on a runner (virtual machine or container) and can depend on each other.
- **runs-on**: Specifies the runner environment (e.g., `ubuntu-latest` for Linux, `windows-latest` for Windows, or self-hosted). For data science, use `ubuntu-latest` for compatibility with tools like Python, R, or Docker. You can also specify labels for self-hosted runners.
- **needs**: Defines job dependencies. A job only runs after the jobs listed in `needs` complete successfully. This creates a linear or DAG-like execution flow—useful for sequencing data tasks (e.g., `validate` needs to finish before `train`).
- **if**: A conditional expression (using GitHub's `${{ }}` syntax) to skip or run the job based on context (e.g., branch name, event type). Expressions can reference variables, secrets, or GitHub context like `${{ github.ref }}`.
- **Key Notes**:
  - Jobs run in parallel unless `needs` is used.
  - Failed jobs can halt the workflow (configurable later via `strategy`).
  - Outputs from one job can be passed to another via `outputs` (we'll touch on this briefly here; deeper in Topic 11).
  - All jobs inherit workflow-level `env` and `defaults`, but you can override at the job level.

**Syntax Overview**:
```yaml
jobs:
  <job_id>:  # e.g., validate-data
    runs-on: ubuntu-latest  # Required: the runner
    needs: []  # Optional: array of job_ids this depends on (empty = no deps)
    if: ${{ github.ref == 'refs/heads/main' }}  # Optional: conditional
    # outputs:  # Optional map for sharing data (e.g., { result: ${{ steps.step1.outputs.value }} })
    # env:  # Job-level environment variables
    # steps: ...  # Defined here (next topic)
```

**Examples Tailored to Data Science**:
1. **Basic Job Structure**: A single job to run a Python script for data cleaning.
   ```yaml
   jobs:
     clean-data:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Install Python deps
           run: pip install pandas
         - name: Clean dataset
           run: python clean_data.py
   ```

2. **With Dependencies (needs)**: Two jobs where `validate` must succeed before `train-model`.
   ```yaml
   jobs:
     validate:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Check data quality
           run: python validate.py  # e.g., checks for missing values

     train-model:
       runs-on: ubuntu-latest
       needs: validate  # Runs only after 'validate' succeeds
       steps:
         - uses: actions/checkout@v4
         - name: Train ML model
           run: python train.py  # e.g., fits a scikit-learn model
   ```

3. **With Conditional (if)**: Run a job only on the `main` branch, useful for production data pipelines.
   ```yaml
   jobs:
     deploy-model:
       runs-on: ubuntu-latest
       if: github.ref == 'refs/heads/main'  # Skips on feature branches
       needs: [validate, test]
       steps:
         - uses: actions/checkout@v4
         - name: Deploy to cloud
           run: python deploy.py  # e.g., uploads model to S3
   ```

**Common Pitfalls**:
- `runs-on` is required; omitting it causes errors.
- `needs` creates a dependency graph—circular references are invalid.
- `if` expressions must be valid GitHub context syntax; test with simple conditions first.
- Job IDs must be lowercase alphanumeric with hyphens (no spaces).

This sets the stage for orchestrating multi-step data workflows. Once comfortable, we'll build on this with job outputs and then steps.

#### Practice Questions for Topic 12
1. Write a YAML snippet for a workflow with two jobs: `fetch-data` (runs on `ubuntu-latest`, no dependencies) and `analyze-data` (runs on `macos-latest`, needs `fetch-data` to succeed). Include placeholder steps (e.g., `run: echo "Fetching..."`) for each.

2. Fix this broken job structure (issues: missing runs-on, invalid if syntax, needs references non-existent job):
   ```
   jobs:
     build:
     if: github.ref = 'main'  # Wrong
       needs: test  # 'test' not defined
     steps:
       - run: python build.py

     deploy:
       runs-on: ubuntu-latest
       needs: build
       steps:
         - run: python deploy.py
   ```

3. Create a job called `report` that only runs if the event is a `push` to the `develop` branch (use `if: github.event_name == 'push' && github.ref == 'refs/heads/develop'`). It should depend on a job called `test`, run on `windows-latest`, and have a simple step to generate a report.

