### Topic 7: Defaults: defaults

With environment variables handled, let's discuss defaults. The GitHub docs cover `defaults` as a root-level or job-level key to set default settings for steps, like shell or working directory, to avoid repetition.

Key points:
- **defaults**: A map with subkeys `run` for step defaults.
- **Subkeys under defaults.run**:
  - `shell`: Default shell for `run:` steps (e.g., `bash`, `pwsh` for PowerShell, `python`). Defaults to `bash` on Linux/macOS, `cmd` on Windows.
  - `working-directory`: Default dir for steps (e.g., `./data` for data folders). Relative to repo root.
- **Scopes**:
  - Root-level `defaults`: Applies to all jobs.
  - Job-level `defaults` (under `jobs.<job_id>.defaults`): Only for that job's steps, overrides root.
- **Usage**: Steps can override with their own `shell` or `working-directory`. Useful for consistency.
- **Data Science Angle**: Set `shell: python` for direct Python commands in steps, or `working-directory: notebooks/` for Jupyter workflows.
- **Best Practices**: Use for workflows with many similar steps. Only affects `run:` steps, not `uses:` actions.

Examples:
```yaml
# Root-level defaults
defaults:
  run:
    shell: bash
    working-directory: src

on: push

jobs:
  build:
    defaults:  # Job-level override
      run:
        shell: python
    steps:
      - run: print("Hello from Python!")  # Runs in Python shell
```

Common Pitfalls: Invalid shells (must be installed on runner), or absolute paths (use relative).

#### Practice Questions for Topic 7
1. Add root-level `defaults` to this workflow for `shell: bash` and `working-directory: 'data/scripts'`.
   ```yaml
   name: Script Workflow
   on: push
   jobs:
     execute:
       runs-on: ubuntu-latest
       steps:
         - run: ls
   ```
2. Write a job snippet with job-level `defaults.run.shell: pwsh` and a step that runs a PowerShell command (e.g., `Write-Output "Test"`).

3. Fix this invalid defaults: The structure is wrong, and shell is invalid.
   ```yaml
   defaults: shell: sh  # Missing run map
   jobs:
     test:
       steps:
         - run: echo "Hi"
   ```

#### Solutions
1. Solution:
   ```yaml
   name: Script Workflow
   defaults:
     run:
       shell: bash
       working-directory: 'data/scripts'
   on: push
   jobs:
     execute:
       runs-on: ubuntu-latest
       steps:
         - run: ls
   ```

2. Solution:
   ```yaml
   jobs:
     execute:
       defaults:
         run:
           shell: pwsh
       steps:
         - run: Write-Output "Test"
   ```

3. Solution:
   ```yaml
   defaults:
     run:
       shell: sh
   jobs:
     test:
       steps:
         - run: echo "Hi"
   ```

