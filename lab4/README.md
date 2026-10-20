# Lab 4

| Image | Base image | Image size | Rebuild time after one-line code change |
|---|---|---:|---:|
| naive | `python:latest` | 1.15 GB | 4.401 seconds |
| slim | `python:3.12-slim-bookworm` | 137 MB | 0.710 seconds |



** Base image impact: The slim image is approximately 88% smaller than the naive image. Most of the difference comes from the base image: the naive image uses the moving `python:latest` tag, which resolved to Python 3.15.0 in this build, while the slim image uses Python 3.12 slim Bookworm. The optimized image also uses `--no-cache-dir` to avoid retaining pip's download cache. The exact size contribution of the base image alone was not measured.
** Dependency layer caching: During the code-only rebuild, the slim image's `RUN pip install --no-cache-dir -r requirements.txt` layer showed `CACHED`. Docker reused the dependency layer because `requirements.txt` is copied and installed before `app.py`. In the naive image, changing the source code caused the pip-installation step to run again because the source was copied before dependencies were installed.
** .dockerignore context exclusion: The `.dockerignore` file excludes `.git`, `.github`, `__pycache__/`, `*.pyc`, `.pytest_cache/`, `evidence/`, `*.png`, `*.md`, and `Dockerfile*`. These exclusions prevent unnecessary files from being sent in the Docker build context.




# CI timings table

| Measurement | Baseline | Cache miss | Cache hit | Worth it? |
|---|---:|---:|---:|---|
| Pip install step | 2 sec | 3 sec | 3 sec | No measurable improvement |
| Docker build step | 13 sec | 21 sec | 8 sec | Yes, compared with cache miss |
| Whole workflow (wall clock) | 56 sec | 2 min 27 sec | 1 min 17 sec | Mixed; hit was faster than miss but slower than baseline |

** Pip cache: The logs confirmed successful cache restoration but the measured install step did not become faster, so the timing evidence does not show a performance benefit in these runs.

** Docker layer cache: The build action decreased from 21 seconds on the cache-miss run to 8 seconds on the cache-hit run, saving 13 seconds and supporting the use of Docker layer caching.




* The dependency setup is three steps that must run on the same runner as the tests, so it is a composite action.

* The image build is a whole job needing `packages: write` that the test job must not have. Hence, it is a reusable workflow.


*Build and Run Command

# Build the Docker image
docker build -t acs730-lab4:local lab4

# Start the container and map port 8000
docker run -d --name lab4-app -p 8000:8000 acs730-lab4:local

# Check the application health endpoint
curl http://localhost:8000/healthz


The /healthz endpoint returns a healthy status response such as:

{"status":"ok","version":"dev"}

# Experiments

Invert the Dockerfile Layer Order: Moving COPY app.py ./ above the requirements and dependency-installation steps will make a code-only rebuild slower than the original 0.710-second rebuild.

Remove .dockerignore: Temporarily renaming .dockerignore will increase the build context because more repository files will be sent to Docker
Because the slim Dockerfile uses specific COPY instructions, removing .dockerignore will increase the context without significantly increasing the final image size.
