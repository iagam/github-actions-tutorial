### Topic 12: Steps in Jobs: steps (Basics: id, if, name, run, uses)
**Steps** are the individual actions within a job—they're the "what to do" inside each job (e.g., checking out code, installing dependencies, running a Python script for data analysis). A job must have at least one step under the `steps` key (a list of maps). Steps run sequentially in the job's runner environment.

From the official GitHub documentation (workflow syntax section on steps):
- **Structure**: Under `jobs.<job_id>.steps`, define a list where each step is a map. Steps can execute shell commands (`run`) or reuse existing actions (`uses`).
- **name**: A string label for the step, shown in the GitHub UI logs (e.g., "Install Python packages"). Optional but recommended for readability.
- **id**: A unique string identifier for the step (e.g., `install-deps`). Used to reference outputs later (e.g., `${{ steps.id.outputs.value }}`). Optional.
- **if**: Conditional expression to run/skip the step (e.g., based on previous step success). Uses `${{ }}` syntax, like `${{ success() }}`.
- **run**: Executes inline shell commands (e.g., `pip install numpy pandas`). Supports multi-line with `|`. The shell is inherited from `runs-on` or `defaults` (bash on Linux, PowerShell on Windows).
- **uses**: References a reusable action from a marketplace, repo, or local path (e.g., `actions/checkout@v4` to clone the repo). Can include version pins for stability.
- **Key Notes**:
  - Steps run in order; a failure stops the job unless `continue-on-error` is set (next topic).
  - Always start with `uses: actions/checkout@v4` to access your repo files (essential for data scripts).
  - Outputs from `run` or `uses` can be captured for later steps/jobs.
  - For data science: Use `run` for custom Python/R commands; `uses` for pre-built actions like testing or caching.

**Syntax Overview**:
```yaml
jobs:
  <job_id>:
    runs-on: ubuntu-latest
    steps:
      - name: Step name  # Optional
        id: step-id  # Optional
        if: ${{ condition }}  # Optional
        run: |  # Or 'uses: action@version'
          echo "Command 1"
          echo "Command 2"
```

**Examples Tailored to Data Science**:
1. **Basic Steps**: A job with steps to set up Python, install libs, and run a data script.
   ```yaml
   jobs:
     analyze:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4  # Clones repo
         - name: Set up Python
           uses: actions/setup-python@v5
           with:
             python-version: '3.11'
         - name: Install dependencies
           run: |
             python -m pip install --upgrade pip
             pip install pandas scikit-learn
         - name: Run analysis
           run: python analyze_data.py  # e.g., generates plots
   ```

2. **With id and if**: Capture output from a step and conditionally run another.
   ```yaml
   jobs:
     validate:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - id: check-size
           name: Check dataset size
           run: echo "size=$(wc -c < data.csv)" >> $GITHUB_OUTPUT
         - name: Validate if large
           if: steps.check-size.outputs.size > 1000000  # >1MB
           run: python validate_large.py
   ```

3. **Mixing run and uses**: Use a marketplace action for caching (speeds up repeated installs in data workflows).
   ```yaml
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - name: Cache Python deps
           uses: actions/cache@v4
           with:
             path: ~/.cache/pip
             key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
         - name: Install and build
           run: |
             pip install -r requirements.txt
             python build_model.py
   ```

**Common Pitfalls**:
- Forget `actions/checkout@v4`? Steps can't access repo files.
- `run` commands must be shell-compatible (e.g., no Windows paths on Linux).
- `uses` needs exact version (e.g., `@v4`) to avoid breaking changes.
- Indentation: Steps list uses `-` at the same level.

This makes your jobs actionable—next, we'll advance steps with customizations like env and timeouts.

#### Practice Questions for Topic 12
1. Write a YAML snippet for a job called `process-data` (runs on `ubuntu-latest`) with three steps: (a) Checkout code using `uses`, (b) A named step with `run` to install `numpy` via pip, (c) An `id`-ed step with `run` to execute `python process.py` (assume it processes a CSV).

2. Fix this broken steps list (issues: missing `-` for list items, invalid if without `${{ }}`, run multi-line not using `|`):
   ```
   jobs:
     test:
       runs-on: ubuntu-latest
       steps:
         name: Setup
         uses: actions/setup-node@v4
         if: success  # Wrong syntax
         run: npm install
           npm test
   ```

3. Create steps for a job `visualize` that includes: A conditional step (`if: github.ref == 'refs/heads/main'`) named "Generate Plot" that runs `python plot.py` only on main branch, and another step using `uses: actions/upload-artifact@v4` to upload results (include basic `with` for path, but keep it simple—no full with yet).

#### Solutions
1. Solution:
    ```yaml
    jobs:
      process-data:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v4
          - name: Install NumPy
            run: pip install numpy
          - id: process-csv
            name: Process a CSV
            run: python process.py
    ```

2. Solution:
    ```yaml
    jobs:
      test:
        runs-on: ubuntu-latest
        steps:
          - name: Setup Node
            uses: actions/setup-node@v4
          - name: Install and Test
            if: ${{ success() }}
            run: |
              npm install
              npm test
    ```

3. Solution:
    ```yaml
    jobs:
      visualize:
        runs-on: ubuntu-latest
        steps:
          - name: Generate Plot
            if: ${{ github.ref == 'refs/heads/main' }}
            run: python plot.py  # Generates the plot file
          - name: Upload Plot Artifact
            uses: actions/upload-artifact@v4
            with:
              name: plot-results
              path: ./output/  # Adjust to where plot.py saves files
    ```