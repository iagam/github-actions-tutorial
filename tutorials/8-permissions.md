### Topic 8: Permissions: permissions

Following defaults, we come to permissions. The GitHub docs explain `permissions` as a root-level or job-level key to control what the workflow can access, for security. By default, workflows have read access to the repo, but you can grant more or restrict.

Key points:
- **permissions**: A map of permission keys to values like `read`, `write`, or `none`. Keys include `actions`, `checks`, `contents`, `deployments`, `id-token`, `issues`, `packages`, `pages`, `pull-requests`, `repository-projects`, `security-events`, `statuses`.
- **Common Permissions**:
  - `contents: read` (default; for checking out code).
  - `issues: write` (to create/comment on issues).
  - `pull-requests: write` (for PR comments/labels).
  - `id-token: write` (for OIDC auth with clouds like AWS).
- **Scopes**:
  - Root-level `permissions`: Applies to all jobs.
  - Job-level `permissions` (under `jobs.<job_id>.permissions`): Only for that job, overrides root (can be more restrictive or additive).
- **All/None Shortcuts**: `permissions: read-all` (read for everything), `write-all`, or `{}` for minimal (only what's needed).
- **Data Science Angle**: For workflows deploying models to clouds, use `id-token: write`. For reading data from packages, `packages: read`. Restrict to `contents: read` for simple CI to minimize risks.
- **Best Practices**: Principle of least privilege—grant only what's needed. Use for third-party actions to limit damage if compromised.

Examples:
```yaml
# Root-level: Read contents, write issues
permissions:
  contents: read
  issues: write

on: issues

jobs:
  respond:
    permissions:  # Job-level: Add PR write
      pull-requests: write
    steps:
      - run: echo "Responding..."
```

Common Pitfalls: Forgetting permissions for actions (e.g., needs `contents: write` for pushes), or over-granting (security risk).

#### Practice Questions for Topic 8
1. Add root-level `permissions` to this workflow for `contents: read` and `pull-requests: write`.
   ```yaml
   name: PR Comment Workflow
   on: pull_request
   jobs:
     comment:
       runs-on: ubuntu-latest
       steps:
         - run: echo "Commenting..."
   ```
2. Write a job snippet with job-level `permissions: { id-token: write, packages: read }` for a deployment job.

3. Fix this invalid permissions: The map is wrong, and uses invalid value.
   ```yaml
   permissions: contents=read  # Wrong syntax
   jobs:
     build:
       steps:
         - run: build
   ```

#### Solutions
1. Solution:
   ```yaml
   name: PR Comment Workflow
   permissions:
     contents: read
     pull-requests: write
   on: pull_request
   jobs:
     comment:
       runs-on: ubuntu-latest
       steps:
         - run: echo "Commenting..."
   ```

2. Solution:
   ```yaml
   jobs:
     deployment:
       permissions:
         id-token: write
         packages: read
       steps:
   ```

3. Solution:
   ```yaml
   permissions:
     contents: read
   jobs:
     build:
       steps:
         - run: build
   ```
