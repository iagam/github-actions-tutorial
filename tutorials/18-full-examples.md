### Topic 18: Putting It All Together
This topic synthesizes everything into full, production-ready workflows tailored to data science scenarios (e.g., ETL pipelines, ML training/deploy, scheduled reports). We'll draw from prior topics: triggers/filters, jobs/steps/strategies, containers/services, reusables, etc. Then, tips for debugging when things go sideways (e.g., failed runs in GitHub UI).

From the official GitHub documentation (full workflow syntax + using workflows):
- **Holistic View**: A complete workflow YAML nests under root keys: `name`, `on` (triggers/filters), `env/defaults/permissions`, `jobs` (with strategies, steps, containers). Use expressions `${{ }}` everywhere for dynamics (e.g., `${{ github.sha }}` for commit hash).
- **Best Practices Recap**:
  - Start simple: Checkout → Setup → Run → Upload artifacts.
  - Modularize: Reusables for shared logic (e.g., validation).
  - Secure: Secrets/env for keys; permissions minimal.
  - Efficient: Filters/concurrency/matrix to run only what's needed.
  - Reproducible: Containers for envs; cache deps.
- **For DS**: Focus on idempotency (reruns safe), outputs for chaining (e.g., model path to deploy), and schedules for cron jobs (e.g., daily data refresh).

**Full Example 1: ETL Pipeline (Triggers, Jobs, Steps, Strategies)**
A scheduled/push-triggered workflow to extract, transform, load data—runs on multiple Python versions, with matrix for OS.
```yaml
name: DS ETL Pipeline
on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight UTC
  push:
    branches: [main]
    paths: ['data/**', 'etl/**']  # Only data/ETL changes
env:
  DATA_URL: https://example.com/api/data  # Workflow-level var
jobs:
  extract:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - name: Install deps
        run: pip install requests pandas
      - name: Extract data
        id: extract
        run: |
          python etl/extract.py --url $DATA_URL --output data/raw.csv
        env:
          API_KEY: ${{ secrets.API_KEY }}
        timeout-minutes: 10
    outputs:
      file-size: ${{ steps.extract.outputs.size }}  # Assume script sets via GITHUB_OUTPUT

  transform:
    runs-on: ${{ matrix.os }}
    needs: extract
    if: needs.extract.outputs.file-size > 1000  # Skip if tiny file
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        python-version: [3.10, 3.11]
      fail-fast: false
      max-parallel: 2
    container:
      image: python:${{ matrix.python-version }}-slim
    steps:
      - uses: actions/checkout@v4
      - name: Transform data
        working-directory: ./etl
        run: |
          pip install scikit-learn
          python transform.py --input ../data/raw.csv --output ../data/transformed.parquet
        continue-on-error: true  # Proceed if one matrix fails

  load:
    runs-on: ubuntu-latest
    needs: transform
    permissions:
      contents: read  # Minimal for checkout
    steps:
      - uses: actions/checkout@v4
      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: processed-data
          path: data/transformed.parquet
      - name: Load to DB
        run: python load.py --file data/transformed.parquet
        env:
          DB_URL: ${{ secrets.DB_CONNECTION }}
```

**Full Example 2: ML Training + Deploy (Reusables, Concurrency, Services)**
Calls a reusable for validation, uses concurrency to avoid overlapping trains, services for mock DB.
```yaml
name: ML Train & Deploy
on:
  workflow_dispatch:  # Manual trigger
  push:
    tags: ['v*.*.*']  # Filtered releases
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true  # Cancel old runs on new push
jobs:
  validate:
    uses: ./.github/workflows/validate-ml.yml@v1  # Reusable from Topic 16
    with:
      model-path: ./models/base.pkl
      threshold: 0.85
    secrets: inherit
    needs: none  # Standalone

  train:
    runs-on: ubuntu-latest
    needs: validate
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
    container:
      image: tensorflow/tensorflow:2.15.0-gpu
    steps:
      - uses: actions/checkout@v4
      - name: Train model
        run: |
          pip install torch
          python train.py --data data/train.csv --db postgres://postgres:test@postgres:5432/ml_db
        timeout-minutes: 30
        shell: bash
      - name: Save model
        uses: actions/upload-artifact@v4
        with:
          name: trained-model
          path: ./output/model.pkl

  deploy:
    runs-on: ubuntu-latest
    needs: [validate, train]
    if: ${{ success() }}  # Only if both pass
    environment: production  # Requires approval
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: trained-model
      - name: Deploy
        run: python deploy.py --model model.pkl --env prod
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
```

**Debugging Tips for GitHub Actions**:
- **UI Logs**: Click jobs/steps in Actions tab—expand for full output. Use `::debug::msg` in runs for custom logs (e.g., `echo "::debug::File size: $SIZE"`).
- **Common Errors**:
  - Indent/YAML: Use GitHub editor (`.github/workflows/` → Edit) for auto-lint; or yamllint CLI.
  - No Trigger: Check "All workflows" tab; test with `workflow_dispatch` or manual rerun.
  - Step Fails: Pin actions (`@v4`), check env/secrets (masked in logs), use `continue-on-error: true` temporarily.
  - Matrix/Concurrency: View per-instance logs; use `max-parallel: 1` to serialize.
  - Reusables: Ensure `workflow_call`; test standalone first (add dummy `on: push`).
- **Tools**: GitHub CLI (`gh run list/view`), VS Code Actions extension for local test, or `act` CLI for offline sim.
- **DS-Specific**: For long runs, add progress echoes; cache datasets (`actions/cache` with `key: ${{ runner.os }}-data-${{ hashFiles('data/**') }}`); monitor quotas (free: 2000 min/month).
