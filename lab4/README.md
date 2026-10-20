# Lab 4

| Image | Base image | Image size | Rebuild time after one-line code change |
|---|---|---:|---:|
| naive | `python:latest` | 1.15 GB | 4.401 seconds |
| slim | `python:3.12-slim-bookworm` | 137 MB | 0.710 seconds |



** Base image impact: The slim image is approximately 88% smaller than the naive image. Most of the difference comes from the base image: the naive image uses the moving `python:latest` tag, which resolved to Python 3.15.0 in this build, while the slim image uses Python 3.12 slim Bookworm. The optimized image also uses `--no-cache-dir` to avoid retaining pip's download cache. The exact size contribution of the base image alone was not measured.
** Dependency layer caching: During the code-only rebuild, the slim image's `RUN pip install --no-cache-dir -r requirements.txt` layer showed `CACHED`. Docker reused the dependency layer because `requirements.txt` is copied and installed before `app.py`. In the naive image, changing the source code caused the pip-installation step to run again because the source was copied before dependencies were installed.
** .dockerignore context exclusion: The `.dockerignore` file excludes `.git`, `.github`, `__pycache__/`, `*.pyc`, `.pytest_cache/`, `evidence/`, `*.png`, `*.md`, and `Dockerfile*`. These exclusions prevent unnecessary files from being sent in the Docker build context.

