### Topic 11: Job Outputs and Environment: outputs, env

Now that we have job basics, let's discuss job-level outputs and env. These build on root-level versions but are scoped to individual jobs. Outputs allow passing data between jobs (e.g., from build to test), while env sets variables for the job's steps.

Key points from the docs:
- **env (Job-level)**: A map of key-value pairs, similar to root `env`, but only available in this job's steps (overrides root if same key). Use for job-specific configs (e.g., `env: { DATASET: 'train.csv' }`).
- **outputs**: A map of key-value pairs where values are expressions capturing data from steps (e.g., `outputs: { result: ${{ steps.compute.outputs.value }} }`). Outputs are available to dependent jobs via `${{ needs.<job_id>.outputs.<key> }}`.
- **Setting Outputs in Steps**: In a step, use `echo "key=value" >> $GITHUB_OUTPUT` in `run:` to set (multi-line with delimiters). Or, actions (uses:) can define outputs.
- **Usage**: In later jobs, reference like `if: ${{ needs.prep.outputs.success == 'true' }}` or in steps `run: echo "${{ needs.prep.outputs.path }}"`.
- **Data Science Angle**: Output model accuracy from a training job to decide if to deploy in a follow-up job. Use job env for specific Python env vars like `PYTHONPATH`.
- **Best Practices**: Outputs are strings; use for simple data. Env overrides root/job > step. Outputs only flow to jobs that `needs` this one.

Examples:
```yaml
jobs:
  prep:
    runs-on: ubuntu-latest
    env:  # Job env
      FILE: data.csv
    outputs:
      path: ${{ steps.find.outputs.file_path }}
    steps:
      - id: find
        run: echo "file_path=/path/to/${{ env.FILE }}" >> $GITHUB_OUTPUT

  analyze:
    needs: prep
    runs-on: ubuntu-latest
    steps:
      - run: echo "Using ${{ needs.prep.outputs.path }}"
```

Common Pitfalls: Forgetting `id:` on steps for referencing, wrong output syntax (must use >> $GITHUB_OUTPUT), outputs not available without `needs`.

#### Practice Questions for Topic 11
1. Add job-level `env` and `outputs` to the `validate` job: `env: { FORMAT: 'csv' }`, output `valid: ${{ steps.check.outputs.is_valid }}` from a step that sets it (use a placeholder run: to echo "is_valid=true" >> $GITHUB_OUTPUT).
   ```yaml
   jobs:
     validate:
       runs-on: ubuntu-latest
       steps:
         - id: check
           run: # Placeholder
   ```
2. Write a **two-job snippet**:
- `compute` with output `sum: ${{ steps.calc.outputs.result }}` (step `id: calc`, `run: echo "result=42" >> $GITHUB_OUTPUT`)
- and `use` that `needs: compute`, with a step echoing `${{ needs.compute.outputs.sum }}`.


3. Fix this invalid outputs/env: Env not a map, output reference wrong.
   ```yaml
   jobs:
     build:
       env: PYTHON=3.8  # Wrong
       outputs: version=${{ steps.get.outputs.ver }}  # Wrong syntax
       steps:
         - id: get
           run: echo "ver=1.0" >> $GITHUB_OUTPUT
   ```

#### Solutions
1. Solution:
    ```yaml
    jobs:
      validate:
        env:
          FORMAT: 'csv'
        outputs:
          valid: ${{ steps.check.outputs.is_valid }}
        runs-on: ubuntu-latest
        steps:
          - id: check
            run: echo "is_valid=true" >> $GITHUB_OUTPUT
    ```

2. Solution:
    ```yaml
    jobs:
      compute:
        runs-on: ubuntu-latest
        outputs:
          sum: ${{ steps.calc.outputs.result }}
        steps:
          - id: calc
            run: echo "result=42" >> $GITHUB_OUTPUT

      use:
        needs: compute
        runs-on: ubuntu-latest
        steps:
          - run: echo "${{ needs.compute.outputs.sum }}"
    ```

3. Solution:
    ```yaml
    jobs:
      build:
        env:
          PYTHON: '3.8'
        outputs:
          version: ${{ steps.get.outputs.ver }}
        steps:
          - id: get
            run: echo "ver=1.0" >> $GITHUB_OUTPUT
    ```