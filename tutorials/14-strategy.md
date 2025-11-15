### Topic 14: Strategies: strategy (Matrix, fail-fast, max-parallel)
Now, onto **strategies**, which add parallelism and variation to *jobs* (not steps). The `strategy` key under a job lets you run multiple *instances* of that job with different configurations, like testing across environments. This is a game-changer for data science: e.g., validate a model on multiple Python versions/OSes without duplicating jobs. By default, jobs run once; `strategy` scales them out.

From the official GitHub documentation (workflow syntax section on job strategies):
- **Structure**: Under `jobs.<job_id>.strategy`, a map with keys like `matrix`, `fail-fast`, and `max-parallel`.
- **matrix**: The powerhouse—defines a grid of variations via a map (e.g., `{ os: [ubuntu-latest, windows-latest], python-version: [3.10, 3.11] }`). This creates a Cartesian product (e.g., 2 OS x 2 Python = 4 job instances). Access vars in steps via `${{ matrix.os }}`. Include/exclude subsets to customize (e.g., skip Windows+3.10).
- **fail-fast**: Boolean (`true`/`false`, default `true`). If `true`, one instance failing cancels the others—saves time on bad configs. Set `false` for full runs (e.g., compare all Python versions even if one fails).
- **max-parallel**: Number (default unlimited, max 256). Limits concurrent instances (e.g., `5` to avoid runner overload in heavy DS jobs like GPU training).
- **Key Notes**:
  - Only on jobs with steps; combines with `runs-on: ${{ matrix.os }}` for dynamic envs.
  - Outputs from matrix jobs: Use `strategy.job-matrix` context to aggregate.
  - For DS: Matrix over datasets, hyperparameters, or libs (e.g., test pandas vs. polars).
  - Limits: Too large matrices can hit GitHub's concurrency quotas (e.g., free tier: 20 parallel jobs).

**Syntax Overview**:
```yaml
jobs:
  <job_id>:  # e.g., test-matrix
    runs-on: ${{ matrix.os }}  # Dynamic from matrix
    strategy:
      fail-fast: false  # Run all even if some fail
      max-parallel: 3   # Limit to 3 at once
      matrix:
        os: [ubuntu-latest, windows-latest]  # List
        python-version: [3.10, 3.11]  # Creates 4 combos
        # include:  # Optional: Add extra rows
        #   - os: macos-latest
        #     python-version: 3.12
        # exclude:  # Optional: Remove combos
        #   - os: windows-latest
        #     python-version: 3.10
    steps:
      - run: echo "Running on ${{ matrix.os }} with Python ${{ matrix.python-version }}"
```

**Examples Tailored to Data Science**:
1. **Basic Matrix**: Test data processing on multiple OS/Python combos.
   ```yaml
   jobs:
     validate:
       runs-on: ${{ matrix.os }}
       strategy:
         matrix:
           os: [ubuntu-latest, macos-latest]
           python: [3.9, 3.11]
       steps:
         - uses: actions/checkout@v4
         - uses: actions/setup-python@v5
           with:
             python-version: ${{ matrix.python }}
         - name: Validate CSV
           run: python validate.py  # e.g., checks pandas compatibility
   ```
   (Runs 4 instances: Ubuntu/3.9, Ubuntu/3.11, macOS/3.9, macOS/3.11.)

2. **With fail-fast and max-parallel**: Run hyperparam sweeps, but cap concurrency for resource-heavy jobs.
   ```yaml
   jobs:
     tune-model:
       runs-on: ubuntu-latest
       strategy:
         fail-fast: false  # Get results from all params
         max-parallel: 2   # Don't overwhelm shared runners
         matrix:
           learning-rate: [0.01, 0.001, 0.0001]
           batch-size: [32, 64]
       steps:
         - uses: actions/checkout@v4
         - name: Train variant
           run: python train.py --lr ${{ matrix.learning-rate }} --bs ${{ matrix.batch-size }}
   ```
   (6 instances, but only 2 run at once; all complete even if some overfit.)

3. **With include/exclude**: Selective testing, e.g., exclude unstable combos for ML libs.
   ```yaml
   jobs:
     test-libs:
       runs-on: ${{ matrix.os }}
       strategy:
         matrix:
           os: [ubuntu-latest, windows-latest]
           lib: [pandas, polars]
         include:
           - os: ubuntu-latest
             lib: dask  # Extra: Test distributed
         exclude:
           - os: windows-latest
             lib: polars  # Skip known issue
       steps:
         - run: python test_${{ matrix.lib }}.py
   ```

**Common Pitfalls**:
- Forget `${{ matrix.key }}` in `runs-on` or steps? All instances use the first value—wrong!
- Large matrices: GitHub limits (e.g., 256 max-parallel total); use `exclude` to trim.
- `fail-fast: true` hides issues—use `false` for debugging, but it slows workflows.
- Nested lists in matrix must be simple (no maps inside lists); use `include` for complex.
- Outputs: Matrix jobs don't auto-aggregate—use a follow-up job with `needs` and `fromJSON(needs.job.outputs.matrix)`.

Strategies unlock efficient CI/CD for DS experimentation. Next: Containers/services for Dockerized envs (Topic 15).

#### Practice Questions for Topic 14
1. Write a YAML snippet for a job `lint-code` (runs on `${{ matrix.os }}`) using a matrix with `os: [ubuntu-latest, macos-latest]` and `python-version: [3.10, 3.12]`. Include `fail-fast: false` and a simple step to run `pylint my_script.py`.

2. Fix this broken strategy (issues: missing dynamic runs-on, invalid matrix key, max-parallel as string, exclude syntax wrong):
   ```
   jobs:
     test:
       strategy:
         matrix:
           os: [ubuntu, windows]
           version = 3.11  # Wrong
         max-parallel: '4'
         fail-fast: true
         exclude:
           os: windows  # Incomplete
       steps:
         - run: echo "Testing"
   ```

3. Create a job `benchmark` with a matrix over `dataset: [small, medium, large]` and `framework: [sklearn, torch]`. Set `max-parallel: 3`, use `include` to add a `framework: xgboost` for `dataset: large` only, and have a step that runs `python benchmark.py --ds ${{ matrix.dataset }} --fw ${{ matrix.framework }}`.

#### Solutions
1. Solution:
    ```yaml
    jobs:
      lint-code:
        runs-on: ${{ matrix.os }}
        strategy:
          fail-fast: false
          matrix:
            os: [ubuntu-latest, macos-latest]
            python-version: [3.10, 3.12]
        steps:
          - uses: actions/checkout@v4
          - uses: actions/setup-python@v5
            with:
              python-version: ${{ matrix.python-version }}
          - name: Lint code
            run: |
              pip install pylint
              pylint my_script.py
    ```

2. Solution:
    ```yaml
    jobs:
      test:
        runs-on: ${{ matrix.os }}
        strategy:
          matrix:
            os: [ubuntu-latest, windows-latest]
            version: [3.11]
          max-parallel: 4
          fail-fast: true
          exclude:
            - os: windows-latest
            version: 3.11
        steps:
          - run: echo "Testing on ${{ matrix.os }} with version ${{ matrix.version }}"
    ```

3. Solution:
    ```yaml
    jobs:
      benchmark:
        runs-on: ubuntu-latest
        strategy:
          max-parallel: 3
          matrix:
            dataset: [small, medium, large]
            framework: [sklearn, torch]
          include:
            - dataset: large
              framework: xgboost
        steps:
          - uses: actions/checkout@v4
          - uses: actions/setup-python@v5
            with:
              python-version: 3.11
          - name: Install frameworks
            run: |
              pip install scikit-learn torch xgboost
          - name: Run Benchmark
            run: python benchmark.py --ds ${{ matrix.dataset }} --fw ${{ matrix.framework }}
    ```