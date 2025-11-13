### Topic 10: Jobs Basics: jobs.<job_id> (Structure, needs, if, runs-on)

With concurrency covered, we're finally to jobs—the core of workflows where work happens. The docs define `jobs` as a root-level map of job IDs to job configs. Each job runs in parallel by default (up to limits), on separate virtual machines.

Key points:
- **Structure**: `jobs: <job_id>: { ... }`. Job ID is a string (e.g., `build`, `test-data`); must be unique, no spaces (use hyphens/underscores).
- **needs**: List of job IDs this job depends on (e.g., `needs: [build]`—runs after build succeeds). Use for sequencing (e.g., test after build). If a needed job fails, this skips.
- **if**: Conditional expression to run the job (e.g., `if: ${{ github.event_name == 'push' }}`). Uses `${{ }}`; true/false. Skips if false.
- **runs-on**: Required; specifies the runner (virtual machine). Common: `ubuntu-latest`, `windows-latest`, `macos-latest`. Can be labels or self-hosted. For matrix (later), expressions ok.
- **Other Job Keys**: We'll cover env, outputs, steps, etc., in later topics.
- **Data Science Angle**: Jobs for stages like `data-prep` (needs nothing), `train-model` (needs data-prep), `evaluate` (if success, needs train-model). Use `runs-on: ubuntu-latest` for Linux-based ML tools.
- **Best Practices**: Keep jobs focused (one responsibility). Use needs for pipelines. Default parallel unless needed.

Examples:
```yaml
jobs:
  prep:  # Job ID
    runs-on: ubuntu-latest
    steps: [...]  # Placeholder

  train:
    needs: prep  # Depends on prep
    if: ${{ success() }}  # Run if previous succeeded (default anyway)
    runs-on: ubuntu-latest
    steps: [...]
```

Common Pitfalls: Missing `runs-on` (required), invalid job IDs (must start with letter/underscore), circular needs.

#### Practice Questions for Topic 10
1. Add basic `jobs` to this workflow: Two jobs—`validate-data` (runs-on ubuntu-latest) and `analyze` (needs validate-data, runs-on ubuntu-latest).
   ```yaml
   name: Data Pipeline
   on: push
   ```
2. Write a job snippet for `model-train` with `runs-on: macos-latest`, `if: ${{ github.ref == 'refs/heads/main' }}`.

3. Fix this invalid jobs: Missing runs-on, and needs not a list.
   ```yaml
   jobs:
     build:
       needs: test  # Wrong format
     test:  # No runs-on
       steps: [...]
   ```

