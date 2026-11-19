# syntax=docker/dockerfile:1

# ---- build stage: compile wheels into a virtualenv --------------------------
FROM python:3.15-slim AS build
ENV PIP_NO_CACHE_DIR=1 PIP_DISABLE_PIP_VERSION_CHECK=1
RUN python -m venv /venv
# Copy only the requirements first so code changes don't invalidate this layer.
COPY requirements.txt .
RUN /venv/bin/pip install -r requirements.txt

# ---- runtime stage: only the venv and the code, running as non-root ---------
FROM python:3.15-slim AS runtime
ARG APP_VERSION=dev
ARG GIT_SHA=unknown
ENV PATH=/venv/bin:$PATH \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    APP_VERSION=$APP_VERSION \
    GIT_SHA=$GIT_SHA
RUN useradd --uid 10001 --no-create-home --shell /usr/sbin/nologin app
COPY --from=build /venv /venv
WORKDIR /srv
COPY app ./app
USER 10001
EXPOSE 8000

HEALTHCHECK --interval=10s --timeout=3s --start-period=5s --retries=3 \
    CMD ["python", "-c", "import urllib.request,sys; sys.exit(0 if urllib.request.urlopen('http://127.0.0.1:8000/health/live', timeout=2).status == 200 else 1)"]

# Exec form: uvicorn is PID 1 and receives SIGTERM directly, so shutdown is
# graceful. One worker per container: scale with replicas, not processes.
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "1", "--timeout-graceful-shutdown", "10"]
