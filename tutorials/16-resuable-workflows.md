### Topic 16: Reusable Workflows: uses, with, secrets
Now, **reusable workflows** let you call one workflow from another like a function, promoting DRY (Don't Repeat Yourself) principles. This is ideal for data science: e.g., a shared "data-validation" workflow called from multiple pipelines, passing inputs like dataset paths or secrets for cloud creds. Reusables are defined in `.github/workflows/` and invoked via `uses` in another workflow.

From the official GitHub documentation (workflow syntax section on reusable workflows):
- **Structure**: In the calling workflow, under `jobs.<job_id>.steps`, use a step with `uses: owner/repo/.github/workflows/workflow.yml@version` (or local path). Pass inputs via `with:` (like action inputs) and secrets via `secrets:`. The reusable workflow declares `inputs` and `secrets` at its root for what it expects.
- **uses**: References the reusable file (e.g., `@v1` for tags, `@main` for branch). Can be in same repo ( `./validate.yml` ) or remote.
- **with**: Map of inputs to pass (strings/numbers/booleans; use `${{ }}` for dynamics like `${{ github.sha }}`).
- **secrets**: Map of secrets to inherit/pass (e.g., `DB_PASS: ${{ secrets.PROD_DB }}`); must match the reusable's declared `secrets`.
- **Key Notes**:
  - Reusables run in their own context (e.g., triggers ignored); outputs via `outputs` from the reusable job.
  - Visibility: Public repos can call public reusables; private needs access.
  - For DS: Pass dataset paths as inputs, API keys as secrets for validation/deploy reusables.
  - Limits: Up to 4 levels of nesting; no matrix/strategy in called workflow (caller handles).

**Reusable Workflow Example** (save as `.github/workflows/validate-data.yml`):
```yaml
# Reusable: Data Validation
name: Validate Data
on:
  workflow_call:
    inputs:
      dataset-path:
        description: 'Path to dataset'
        required: true
        type: string
      threshold:
        description: 'Min data quality score'
        default: 0.8
        type: number
    secrets:
      api-key:
        required: true
on: {}  # No direct triggers
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Check quality
        run: |
          python validate.py --path ${{ inputs.dataset-path }} --threshold ${{ inputs.threshold }} --api ${{ secrets.api-key }}
```

**Calling Workflow Syntax** (in another `.github/workflows/main.yml`):
```yaml
jobs:
  call-validate:
    uses: ./.github/workflows/validate-data.yml@v1  # Local ref
    with:
      dataset-path: ./data/sales.csv
      threshold: 0.9
    secrets:
      api-key: ${{ secrets.ML_API_KEY }}
    # needs: ...  # Can depend on this job
```

**Examples Tailored to Data Science**:
1. **Basic Call**: Invoke a reusable for model testing, passing version as input.
   ```yaml
   # In main.yml
   jobs:
     test-model:
       uses: myorg/ds-utils/.github/workflows/test-model.yml@main
       with:
         model-version: 'v2.1'
         test-dataset: './test_data.parquet'
       secrets: inherit  # Passes all caller's secrets
   ```

2. **With Outputs**: Reusable outputs a quality score; caller uses it (declare `outputs` in reusable job).
   ```yaml
   # Reusable outputs example
   jobs:
     validate:
       outputs:
         score: ${{ steps.check.outputs.score }}
       steps:
         - id: check
           run: echo "score=0.95" >> $GITHUB_OUTPUT
   # Caller
   jobs:
     deploy:
       needs: test-model
       if: needs.test-model.outputs.score > 0.8  # From reusable
       run: echo "Deploying high-quality model"
   ```

3. **Remote Reusable**: Call a marketplace-like reusable for data linting, passing secrets.
   ```yaml
   jobs:
     lint:
       uses: actions/data-linter@v1  # Hypothetical remote
       with:
         files: 'data/*.csv'
       secrets:
         s3-key: ${{ secrets.AWS_ACCESS_KEY }}
   ```

**Common Pitfalls**:
- No `on: workflow_call` in reusable? Can't be called (errors with "not callable").
- Inputs/secrets mismatch: Caller passes undeclared key → failure; use defaults in reusable.
- Version pinning: `@main` drifts; use `@v1` tags for stability in DS pipelines.
- Secrets: `inherit` passes all, but explicit is safer (avoids leaking unrelated ones).
- Outputs: Must use `steps.id.outputs.key` in reusable, `${{ needs.job.outputs.key }}` in caller.
- Nesting: Deep calls can hit limits; flatten for complex DS chains.

Reusables modularize your workflows—next: Filter patterns for precise triggers (Topic 17).

#### Practice Questions for Topic 16
1. Write a snippet for a calling workflow job `run-validation` that uses a local reusable `./data-check.yml@v1`. Pass `with: file: 'reports.csv'` and `threshold: 0.85`. Include `secrets: db-creds: ${{ secrets.DATABASE_PASS }}`. Add a placeholder step after to use an output `quality: ${{ needs.run-validation.outputs.score }}`.

2. Fix this broken call (issues: missing workflow_call in reusable, wrong uses path, inputs not matching, no secrets pass):
   ```
   # Reusable: check.yml (missing on: workflow_call)
   jobs:
     check:
       steps:
         - run: echo "OK"

   # Caller: main.yml
   jobs:
     validate:
       uses: ./check.yml  # No @version
       with:
         unknown: value  # Not declared
   ```

3. Create a simple reusable workflow snippet (`deploy-ml.yml`) with `on: workflow_call`, inputs `model-path` (string, required) and `env-name` (string, default 'dev'), secrets `deploy-key` (required). Have a job `deploy` with a run step using them: `python deploy.py --model ${{ inputs.model-path }} --env ${{ inputs.env-name }} --key ${{ secrets.deploy-key }}`.

#### Solutions
1. Solution:
    ```yaml
    jobs:
      run-validation:
        uses: ./.github/workflows/data-check.yml@v1  # Full path for clarity
        with:
          file: 'reports.csv'
          threshold: 0.85
        secrets:
          db-creds: ${{ secrets.DATABASE_PASS }}
        outputs:
          score: ${{ jobs.validate.outputs.score }}  # Map reusable's output (assume job_id 'validate')

      post-validate:
        needs: run-validation
        runs-on: ubuntu-latest
        steps:
          - name: Use validation score
            if: needs.run-validation.outputs.score > 0.8  # Placeholder using output (renamed 'quality' for match)
            run: echo "Quality high: ${{ needs.run-validation.outputs.score }} - Proceed to deploy!"
    ```

2. Solution:
    ```yaml
    # Reusable: .github/workflows/check.yml
    name: Data Check
      on:
        workflow_call:
          inputs:
            model-version:
              description: 'Version to check'
              required: true
              type: string
          secrets:
            api-key:  # Example; optional
              required: false
    jobs:
      check:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - name: Echo check
            run: echo "OK for model ${{ inputs.model-version }}"

    # Caller: .github/workflows/main.yml
    jobs:
      validate:
        uses: ./.github/workflows/check.yml@main
        with:
          model-version: 'v2.1'
        secrets: inherit  # Passes any matching secrets
    ```

3. Solution:
    ```yaml
    name: Deploy ML
      on:
        workflow_call:
          inputs:
            model-path:
              description: 'Path to Model'
              required: true
              type: string
            env-name:
              description: 'Deployment environment'
              type: string
              default: 'dev'
          secrets:
            deploy-key:
            required: true
    jobs:
      deploy:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - uses: actions/setup-python@v5
            with:
              python-version: '3.11'
          - name: Deploy model
            run: python deploy.py --model ${{ inputs.model-path }} --env ${{ inputs.env-name }} --key ${{ secrets.deploy-key }}
    ```


