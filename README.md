# Roadmap for Learning GitHub Actions Workflows with YAML
As a data scientist, you'll likely use `GitHub Actions` to automate tasks like,
- data processing pipelines
- model training
- testing datasets
- deploying ML models
- running scheduled reports

This roadmap is based on the official GitHub documentation for workflow syntax, starting from the fundamentals and building up to more advanced concepts. We'll focus on practical YAML writing skills, with examples tailored where possible to data science scenarios (e.g., `running Python scripts` for data analysis or `using containers` for reproducible environments).
The roadmap is divided into following logical topics.

## Roadmap Topics:
1. **YAML Basics:** Quick intro to YAML syntax (since all workflows are YAML files). We'll cover structure, data types, indentation, and common pitfalls.

2. **About Workflow Files:** Where to store them, file extensions, and basic file setup.

3. **Workflow Naming:** `name` and `run-name`: How to name your workflow and individual runs.

4. **Triggers:** `on` (Basic Events): Simple event triggers like `push` or `pull_request`.

5. **Triggers:** `on` (Advanced: Activity Types, Filters, Schedules): Filtering by branches/tags/paths, activity types, and scheduling (useful for daily data jobs).

6. **Environment Variables:** `env`: Setting and using variables across workflows (e.g., for API keys in data scripts).

7. **Defaults:** `defaults`: Setting default shells and working directories.

8. **Permissions:** `permissions`: Controlling access for security (e.g., reading repo contents for data files).

9. **Concurrency:** `concurrency`: Managing simultaneous runs (e.g., avoiding race conditions in data processing).

10. **Jobs Basics:** `jobs.<job_id>` (Structure, needs, if, runs-on): Defining jobs, dependencies, conditions, and runners.

11. **Job Outputs and Environment:** `outputs`, `env`: Sharing data between jobs and job-level vars.

12. **Steps in Jobs:** `steps` (Basics: id, if, name, run, uses): Core building blocks for executing commands or actions.

13. **Steps Advanced:** `with`, `env`, `continue-on-error`, `timeout-minutes`, `shell`, `working-directory`: Customizing step behavior.

14. **Strategies:** `strategy` (Matrix, fail-fast, max-parallel): Running variations (e.g., testing on multiple Python versions or OS).

15. **Containers and Services:** `container`, `services`: Using Docker for reproducible data environments (e.g., with Jupyter or databases).

16. **Reusable Workflows:** `uses`, `with`, `secrets`: Calling pre-built workflows (e.g., for standard data validation).

17. **Filter Patterns:** Advanced glob patterns for branches, paths, etc.

