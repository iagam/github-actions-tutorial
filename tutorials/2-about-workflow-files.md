### Topic 2: About Workflow Files

#### Explanation
Workflow files are the YAML documents that define your automation. According to the GitHub docs (from the provided URL), a workflow is a configurable automated process that will run one or more jobs. Workflows are defined by a YAML file checked in to your repository and will run when triggered by an event in your repository, or they can be triggered manually, or at a defined schedule.

Key points:
- **Location**: Workflow files must be stored in the `.github/workflows` directory of your repository. This is a special path—GitHub looks here for YAML files to recognize as workflows. You can have multiple files in this directory for different workflows.
- **File Naming**: Files must have a `.yml` or `.yaml` extension (both are acceptable, but `.yml` is more common). The name can be anything descriptive, like `ci.yml` or `data-pipeline.yaml`. Avoid spaces in filenames; use hyphens or underscores.
- **Basic Setup**: A workflow file starts with root-level keys like `name:` (optional, for display name) and `on:` (required, for triggers). The file must be valid YAML. GitHub ignores files outside `.github/workflows` or without the right extension.
- **Limits and Best Practices**: You can have up to 20 workflows per repo (but usually fewer). Keep files modular—separate workflows for different tasks (e.g., one for testing data scripts, another for deploying models). Use version control to track changes.
- **Data Science Angle**: As a data scientist, you might create a workflow file like `ml-training.yml` in `.github/workflows` to automate running Jupyter notebooks or Python scripts on push events.

Example of a minimal workflow file (we'll build on this later):
```yaml
# .github/workflows/basic.yml
name: My First Workflow

on: push  # Trigger on push events

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Hello, world!"
```

This file would go in your repo's `.github/workflows/basic.yml`. When you push code, it runs a job that prints a message.

Common Pitfalls: Wrong directory (e.g., not in `.github/workflows`), invalid YAML (indentation errors), or missing required keys like `on:`.

#### Practice Questions
1. Write a basic YAML workflow file snippet (just the top-level structure) named "data-check.yml" that would be placed in the correct directory. Include a comment with the full path, a simple `name:` for "Data Validation Workflow", and an `on:` trigger (we'll detail triggers later, so just use `on: push` for now).
2. Explain in YAML comments why a file named "workflow.txt" in the root of the repo wouldn't work as a GitHub Actions workflow. Then, suggest a corrected filename and path in a comment.
3. Create a minimal workflow YAML for a data science scenario: Name it "report-generation.yml", trigger on push, and include a placeholder job (e.g., `jobs: generate: runs-on: ubuntu-latest` with no steps yet).

#### Solutions
1. Solution:
   ```yaml
   # .github/workflows/data-check.yml
   name: Data Validation Workflow
   on: push
   ```

2. Solution:
   ```yaml
   # github expects a file under .github/workflows/
   # .github/workflows/workflow.yml
   ```

3. Solution:
   ```yaml
   # .github/workflows/report-generation.yml
   name: Report Generation
   on: push
   jobs:
     generate:
       runs-on: ubuntu-latest
       steps:
   ```
