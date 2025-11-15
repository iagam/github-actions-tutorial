### Topic 15: Containers and Services: container, services
Now, we're tackling **containers and services**, which let you run jobs (or parts) inside Docker containers for isolation and reproducibility. This is huge for data science: e.g., spin up a container with pre-installed ML libs (TensorFlow, Jupyter) or attach a service like PostgreSQL for testing data pipelines without local setup.

From the official GitHub documentation (workflow syntax section on container and services):
- **container**: Applies to the *entire job*—runs the job's steps inside a specified Docker image (e.g., `python:3.11-slim`). Overrides the base runner OS. Specify image, credentials (for private registries), ports, env, volumes, etc.
- **services**: Defines *companion containers* (e.g., a Redis or MySQL instance) that run alongside the job container. Useful for testing integrations (e.g., query a temp DB while processing data). Services start before steps, accessible via aliases (e.g., `localhost:5432` for Postgres).
- **Key Notes**:
  - Requires `runs-on: ubuntu-latest` (Linux only; Windows/macOS runners don't support containers yet).
  - Pull images from public (Docker Hub) or private (with secrets).
  - Env vars and mounts persist; services share the network.
  - For DS: Use `container` for envs with GPU/ML deps; `services` for mock APIs/DBs.
  - Limits: Up to 10 services; images must be <5GB.

**Syntax Overview**:
```yaml
jobs:
  <job_id>:
    runs-on: ubuntu-latest  # Required for containers
    container:
      image: python:3.11  # Docker image
      credentials:  # For private
        username: ${{ secrets.DOCKER_USER }}
        password: ${{ secrets.DOCKER_PASS }}
      env:
        VAR: value
      ports:
        - 8080
      volumes:
        - /host/path:/container/path
    services:
      db:  # Service alias
        image: postgres:15
        env:
          POSTGRES_PASSWORD: pw
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    steps:
      - run: python script.py  # Runs inside container; access db via 'db:5432'
```

**Examples Tailored to Data Science**:
1. **Basic Container**: Run a Python data job in a slim image with Jupyter-like deps.
   ```yaml
   jobs:
     analyze:
       runs-on: ubuntu-latest
       container:
         image: python:3.11-slim
         env:
           DATA_PATH: /workspace/data
       steps:
         - uses: actions/checkout@v4
         - name: Install and analyze
           run: |
             pip install pandas jupyter
             python analyze.py  # e.g., loads CSV, runs EDA
   ```

2. **With Services**: Test a pipeline against a Postgres service for DB queries.
   ```yaml
   jobs:
     test-db:
       runs-on: ubuntu-latest
       services:
         postgres:
           image: postgres:15-alpine
           env:
             POSTGRES_PASSWORD: secret
             POSTGRES_DB: testdb
           options: --health-cmd pg_isready --health-interval 10s
       container:
         image: python:3.11
       steps:
         - uses: actions/checkout@v4
         - name: Install psycopg2
           run: pip install psycopg2-binary
         - name: Query DB
           run: |
             python -c "
             import psycopg2
             conn = psycopg2.connect('postgresql://postgres:secret@postgres:5432/testdb')
             cur = conn.cursor()
             cur.execute('CREATE TABLE IF NOT EXISTS data (id SERIAL PRIMARY KEY, value NUMERIC);')
             conn.commit()
             print('DB ready for data insert!')
             "
   ```

3. **Advanced: Private Image + Volumes**: Use a custom ML image, mount volumes for large datasets.
   ```yaml
   jobs:
     train:
       runs-on: ubuntu-latest
       container:
         image: myorg/ml-env:v1  # Private
         credentials:
           username: ${{ secrets.DOCKERHUB_USERNAME }}
           password: ${{ secrets.DOCKERHUB_TOKEN }}
         volumes:
           - /tmp/datasets:/data  # Mount host dir to container
       steps:
         - uses: actions/checkout@v4
         - name: Train model
           run: python train.py --data /data/large.csv  # Access mounted data
   ```

**Common Pitfalls**:
- No `runs-on: ubuntu-latest`? Container fails (docs enforce Linux).
- Services not healthy? Steps wait indefinitely—add `options` with health checks.
- Private images without creds: Pull errors; use GitHub secrets.
- Volumes/ports: Paths must exist on runner; ports for internal access only.
- Matrix + Containers: Works, but each instance pulls images (use `max-parallel` low).
- No Windows support: Stick to Linux for DS containers.

Containers/services ensure "it works on my machine" becomes "it works in CI." Next: Reusable workflows for templating (Topic 16).

#### Practice Questions for Topic 15
1. Write a YAML snippet for a job `etl-pipeline` (runs on `ubuntu-latest`) using `container: image: node:20` (for JS-based ETL). Include a step to run `npm install` and `node etl.js` (assume it processes JSON data).

2. Fix this broken container/services setup (issues: wrong runs-on, missing health for service, env under wrong key, no image pull creds if private):
   ```
   jobs:
     app-test:
       runs-on: macos-latest  # Wrong for containers
       container:
         image: myapp:v1  # Assume private
       services:
         redis:
           image: redis
           env:  # Should be under services item
             REDIS_PASS: secret
       steps:
         - run: node test.js  # Connects to redis:6379
   ```

3. Create a job `ml-test` with `container: image: tensorflow/tensorflow:2.15.0-gpu` (for GPU ML). Add a `services` for `redis: image: redis:alpine` with basic health options. Include steps: Checkout, and a run step `python test_model.py` using env `REDIS_URL: redis://redis:6379`.

#### Solutions
1. Solution:
    ```yaml
    jobs:
      etl-pipeline:
        runs-on: ubuntu-latest
        container:
          image: node:20
        steps:
          - uses: actions/checkout@v4
          - name: Run ETL Pipeline
            run: |
              npm install
              node etl.js
    ```

2. Solution:
    ```yaml
    jobs:
      app-test:
        runs-on: ubuntu-latest
        container:
          image: myorg/myapp:v1
          credentials:
            username: ${{ secrets.DOCKERHUB_USERNAME }}
            password: ${{ secrets.DOCKERHUB_TOKEN }}
        services:
          redis:
            image: redis
            env:
              REDIS_PASS: secret
            options: >-
              --health-cmd "redis-cli ping"
              --health-interval 10s
              --health-timeout 5s
              --health-retries 5
        steps:
          - uses: actions/checkout@v4
          - name: Run Node Test
            run: node test.js  # e.g., connects with redis://default:secret@redis:6379
    ```

3. Solution:
    ```yaml
    jobs:
      ml-test:
        runs-on: ubuntu-latest
        container:
          image: tensorflow/tensorflow:2.15.0-gpu
        services:
          redis:
            image: redis:alpine
            env:
              REDIS_PASS: secret
            options: >-
              --health-cmd "redis-cli ping"
              --health-interval 10s
              --health-timeout 5s
              --health-retries 5
        steps:
          - uses: actions/checkout@v4
          - uses: actions/setup-python@v5
            with:
              python-version: 3.11
          - name: Install Redis Client
            run: pip install redis
          - name: Test Model
            run: python test_model.py
            env:
              REDIS_URL: redis://redis:6379
    ```