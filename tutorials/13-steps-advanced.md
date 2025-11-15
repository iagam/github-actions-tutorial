
### Topic 13: Steps Advanced: with, env, continue-on-error, timeout-minutes, shell, working-directory
Now we're advancing **steps** with customizations that make them more robust and flexible. These keys let you pass inputs to actions (`with`), set isolated variables (`env`), handle failures gracefully (`continue-on-error`), enforce time limits (`timeout-minutes`), choose execution shells (`shell`), or change directories (`working-directory`). They're all optional but crucial for real-world data workflows, like ensuring a long-running ML training doesn't hang or isolating env vars for sensitive API calls.

From the official GitHub documentation (workflow syntax section on steps):
- **with**: Passes inputs to a `uses` action (e.g., version or paths). It's a map of key-value pairs specific to the action's schema—check action docs for required/optional inputs.
- **env**: Defines environment variables for *that step only* (overrides job/workflow-level `env`). Useful for secrets or dynamic paths (e.g., `DATA_PATH=/tmp/data`).
- **continue-on-error**: Boolean (`true`/`false`). If `true`, the step failure won't fail the job—logs the error but proceeds. Great for optional tasks like notifications.
- **timeout-minutes**: Number (e.g., `10`). Kills the step after X minutes to prevent infinite loops (e.g., in data scraping).
- **shell**: Overrides the default shell (e.g., `bash` on Linux). Options: `bash`, `pwsh` (PowerShell), `python`, or custom (e.g., `python {0}` for inline scripts).
- **working-directory**: String path for the step's current working directory (e.g., `./src/data`). Changes `pwd` for that step—handy for organized repos.
- **Key Notes**:
  - These apply per-step; inherit from job/workflow if omitted.
  - `with` only for `uses`; `env` works with both `run` and `uses`.
  - Use `${{ }}` expressions in values (e.g., `${{ env.VAR }}` or `${{ github.sha }}`).
  - For data science: `env` for model paths, `timeout` for ETL jobs, `working-directory` for subdirs.

**Syntax Overview**:
```yaml
jobs:
  <job_id>:
    runs-on: ubuntu-latest
    steps:
      - name: Advanced Step
        uses: some/action@v1  # Or run: command
        with:  # Map of inputs (action-specific)
          input1: value1
          input2: ${{ env.DYNAMIC }}
        env:  # Step-level vars
          VAR1: value1
          SECRET: ${{ secrets.API_KEY }}
        continue-on-error: true
        timeout-minutes: 5
        shell: bash  # Or 'python' for Python scripts
        working-directory: ./subdir
        run: echo "Hello from $VAR1"
```

**Examples Tailored to Data Science**:
1. **with and env**: Setup Python with version input and env for a package.
   ```yaml
   jobs:
     setup:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Set up Python
           uses: actions/setup-python@v5
           with:  # Inputs to the action
             python-version: '3.11'
             cache: 'pip'  # Caches deps for speed
           env:  # Step-specific
             PIP_CACHE_DIR: /tmp/pip-cache
         - name: Install ML libs
           run: pip install torch pandas
           working-directory: ./ml-project  # Run in subdir
   ```

2. **continue-on-error and timeout-minutes**: Run a potentially flaky data fetch, but don't halt on timeout.
   ```yaml
   jobs:
     fetch:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Fetch large dataset
           run: python fetch_data.py  # e.g., downloads from API
           timeout-minutes: 15  # Kill if >15 min
           continue-on-error: true  # Proceed even if fails (e.g., network issue)
           env:
             API_URL: https://data.example.com
         - name: Proceed anyway
           run: echo "Using cached data if fetch failed"
   ```

3. **shell and working-directory**: Execute a Python script in a custom shell from a subdir.
   ```yaml
   jobs:
     analyze:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Run analysis in Python shell
           shell: python  # Uses Python as shell (runs code directly)
           run: |
             import pandas as pd
             df = pd.read_csv('data.csv')
             print(df.head())
           working-directory: ./notebooks  # Assumes data.csv is there
           env:
             PYTHONPATH: ./src
   ```

**Common Pitfalls**:
- `with` keys must match the action's inputs exactly (case-sensitive)—mismatch = error.
- `env` vars aren't persisted across steps; use job outputs for that.
- `continue-on-error: true` can mask real issues—use sparingly, and check logs.
- `timeout-minutes: 0` means no timeout (default); set low for tests, high for training.
- `shell` changes affect `run` commands—e.g., `pwsh` uses PowerShell syntax.
- Paths in `working-directory` are relative to runner's workspace; use absolute if needed.

These tweaks turn basic steps into production-ready ones. Next up: Strategies for parallel testing (Topic 14).

#### Practice Questions for Topic 13
1. Write a YAML snippet for a job `train-model` (runs on `ubuntu-latest`) with a step using `uses: actions/setup-python@v5` that passes `with: python-version: '3.12'` and `cache: true`. Add a `run` step to install `scikit-learn` with `env: MODEL_DIR: /tmp/models` and `working-directory: ./training`.

2. Fix this broken advanced step (issues: invalid with key, env var not quoted if needed, timeout as string, shell misspelled):
   ```
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Setup
           uses: actions/setup-python@v5
           with:
             python-version: 3.11
             cach: true  # Typo
           env:
             CACHE_PATH = /tmp/cache  # Wrong syntax
         - name: Install
           run: pip install -r requirements.txt
           timeout-minutes: '10'  # Should be number
           shell: bashh  # Misspelled
   ```

3. Create a step in a job `notify` that runs `python notify.py` with `continue-on-error: true`, `timeout-minutes: 2`, and `shell: pwsh` (for cross-platform). Include `env: SLACK_WEBHOOK: ${{ secrets.SLACK_URL }}` to send a data pipeline status update.


#### Solutions
1. Solution:
    ```yaml
    jobs:
      train-model:
        runs-on:  ubuntu-latest
        steps:
          -  uses:  actions/setup-python@v5
          -  with:
               python-version:  '3.12'
               cache:  true
          -  name:  Install scikit-learn
             run:  pip install scikit-learn
             env:
               MODEL_DIR: /tmp/models
             working-directory:  ./training
    ```

2. Solution:
    ```yaml
    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - name: Setup
            uses: actions/setup-python@v5
            with:
              python-version: '3.11'
              cache: true
            env:
              CACHE_PATH: /tmp/cache
          - name: Install
            run: pip install -r requirements.txt
            timeout-minutes: 10
            shell: bash
    ```

3. Solution:
    ```yaml
    jobs:
      notify:
      runs-on: windows-latest
      steps:
        - uses: actions/checkout@v4
        - name: Send status update
          run: python notify.py
          continue-on-error: true
          timeout-minutes: 2
          shell: pwsh
          env:
            SLACK_WEBHOOK: ${{ secrets.SLACK_URL }}
    ```