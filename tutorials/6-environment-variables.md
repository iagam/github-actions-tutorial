### Topic 6: Environment Variables: env

#### Explanation
Now that triggers are covered, let's move to environment variables. The GitHub docs describe `env` as a root-level key (or at job/step levels) for defining variables available to your workflow. These are like shell env vars, useful for storing configs without hardcoding.

Key points:
- **env**: A map of key-value pairs. Keys are uppercase by convention (e.g., `API_KEY`), values are strings (can use expressions like `${{ secrets.MY_SECRET }}` for dynamic).
- **Scopes**:
  - Root-level `env`: Available to all jobs/steps.
  - Job-level `env` (under `jobs.<job_id>.env`): Only for that job.
  - Step-level `env` (under `steps.env`): Only for that step, overrides higher scopes.
- **Usage**: Reference as `$VAR_NAME` in shell commands (`run:`), or `${{ env.VAR_NAME }}` in YAML expressions.
- **Built-in Vars**: GitHub provides contexts like `${{ github.repository }}` or `${{ secrets.GITHUB_TOKEN }}`—`env` is for custom ones.
- **Secrets**: Use `env` with `${{ secrets.NAME }}` to inject secure values (stored in repo settings > Secrets and variables > Actions).
- **Data Science Angle**: Store API keys for data APIs (e.g., `KAGGLE_API`), paths to datasets, or Python versions. E.g., `env: { PYTHON_VERSION: '3.10' }` then use in steps.
- **Best Practices**: Don't store sensitive data directly in `env`—use secrets. Vars are strings; for multi-line, use `|` or quotes. Case-sensitive.

Examples:
```yaml
# Root-level env
env:
  GREETING: Hello
  DYNAMIC: ${{ github.actor }}  # Expression

on: push

jobs:
  greet:
    env:  # Job-level
      TARGET: World
    steps:
      - run: echo "$GREETING, $TARGET!"  # Outputs "Hello, World!"
      - env:  # Step-level override
          TARGET: Universe
        run: echo "$GREETING, $TARGET!"  # "Hello, Universe!"
```

Common Pitfalls: Forgetting `$` in `run:`, or using vars before definition (order matters in steps). Expressions evaluate at runtime.

#### Practice Questions
1. Add root-level `env` to this workflow with `DATA_PATH: 'data/input.csv'` and `MODEL_TYPE: ${{ inputs.model_type }}` (assuming workflow_dispatch inputs).
   ```yaml
   name: Data Workflow
   on: workflow_dispatch
   jobs:
     process:
       runs-on: ubuntu-latest
       steps:
         - run: echo "Processing..."
   ```

2. Write a job snippet with job-level `env: { DB_URL: ${{ secrets.DATABASE_URL }} }` and a step that uses it in a `run:` command (e.g., echo the var).

3. Fix this invalid env usage: The var reference is wrong, and secret is hardcoded.
   ```yaml
   env:
     API_KEY: my-secret-key  # Should use secret
   jobs:
     api:
       steps:
         - run: curl --header "Authorization: {{ env.API_KEY }}"  # Wrong syntax
   ```


