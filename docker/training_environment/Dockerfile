# Use a Python 3.12 base image. 'slim-bookworm' is a good choice for smaller image size.
FROM python:3.12-slim-bookworm

# Set environment variables for non-interactive installs and to ensure Python output is unbuffered
ENV DEBIAN_FRONTEND=noninteractive \
    PYTHONUNBUFFERED=1 \
    POETRY_HOME="/opt/poetry" \
    PATH="/opt/poetry/bin:$PATH"

# Install system dependencies
# These include essential build tools, git, and potentially other libs for ML frameworks
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential \
    git \
    curl \
    gnupg \
    procps \
    locales \
    # Add any other core system libraries needed by your Python packages
    # For example, if you use anything with image processing: libsm6 libxext6 libxrender1
    && rm -rf /var/lib/apt/lists/*

# Set up locale (important for some tools/libraries)
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Install Google Cloud SDK (gcloud CLI tools)
# This is crucial for gcloud auth, gsutil, bq etc.
# Using 'gcloud' version for 'bookworm' (Debian 12)
RUN echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk-bookworm main" | tee -a /etc/apt/sources.list.d/google-cloud-sdk.list && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key --keyring /usr/share/keyrings/cloud.google.gpg add - && \
    apt-get update && \
    apt-get install -y google-cloud-sdk google-cloud-sdk-gke-gcloud-auth-plugin && \
    rm -rf /var/lib/apt/lists/*

# Install Poetry (Python dependency manager)
# Use the official installer script for robustness
RUN curl -sSL https://install.python-poetry.org | python -

# Set the working directory for the project
WORKDIR /usr/src/app

# Copy pyproject.toml and poetry.lock first to leverage Docker layer caching
# This means if only your source code changes, but dependencies don't,
# Docker won't re-run the poetry install step.
COPY pyproject.toml poetry.lock ./

# The devcontainer.json's postCreateCommand will handle 'poetry install'
# because it needs the full project context to find all required files.

# Copy the rest of your project code
# This copies everything from the build context (which is the project root due to devcontainer.json's "context": "..")
COPY . .

# Optional: Create a non-root user for security (recommended for production, but root is fine for dev container learning)
# For a learning project, 'root' is generally acceptable in a dev container.
# If you decide to add a user, uncomment these lines and set "remoteUser": "vscode" in your .devcontainer.json
# RUN useradd -m vscode && chown -R vscode /usr/src/app
# USER vscode

# Expose any ports your services might use (e.g., MLflow UI, Airflow UI, FastAPI)
# These are then forwarded by devcontainer.json
EXPOSE 5000 8080 8000

# Default command when the container runs (can be overridden by devcontainer or exec)
CMD ["bash"]
