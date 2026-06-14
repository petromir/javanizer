# Dockerfile Best Practices

This document compiles the best practices for writing efficient, secure, and maintainable Dockerfiles, gathered from multiple authoritative sources.

## 1. General Principles

*   **Create Ephemeral Containers:** The image defined by your Dockerfile should generate containers that are as ephemeral as possible—meaning they can be stopped, destroyed, rebuilt, and replaced with minimal setup.
*   **Use Multi-stage Builds:** Split your Dockerfile instructions into distinct stages to ensure that the resulting output only contains the files needed to run the application. This reduces image size and can speed up builds by executing stages in parallel.
*   **Decouple Applications (Single Concern):** Each container should have only one concern. Limiting each container to one process is a good rule of thumb, though not a hard rule (e.g., init processes).
*   **Don't Install Unnecessary Packages:** Avoid "nice-to-have" packages to reduce complexity, file size, and build times.
*   **Minimize Layers:** Combine related commands (especially `RUN` instructions) to reduce the number of layers.
*   **Sort Multi-line Arguments:** Sort arguments (like package lists) alphanumerically to avoid duplicates and improve readability/PR reviews.

## 2. Base Image Strategy

### Choosing the Right Image
*   **Use Official Images:** Start with official, trusted, and verified images. They are optimized, tested, and regularly updated.
    *   *Bad:* Manually installing a heavy SDK on a generic OS image.
    *   *Good:* `FROM mcr.microsoft.com/dotnet/sdk:8.0`
*   **Minimize Size:** Smaller base images (Alpine, Distroless) offer portability, faster downloads, and a smaller attack surface.
*   **Alpine vs. Others:** While Alpine is recommended for its small size (< 6MB), be aware that it may have performance issues with certain technologies (e.g., Python's `musl` vs `glibc`) and some security scanners may struggle with its package format.
*   **Distroless Images:** Consider "distroless" images for production; they contain only your application and its runtime dependencies, excluding shell and package managers.

### Image Tagging
*   **Avoid `latest`:** Using `latest` makes builds non-reproducible and prone to breaking.
*   **Pin to `major.minor` Versions:** This ensures reproducibility while still allowing you to receive patch-level security updates when rebuilding.
*   **Pin to SHA Digest:** For maximum security and supply chain integrity, pin to the image digest (e.g., `image:tag@sha256:...`).
*   **Bridge the Gap:** Use tools like `docker-lock` to manage the trade-off between reproducibility (pinning SHAs) and automated security updates (using tags).

## 3. Instruction Best Practices

### FROM
*   Use current official images as a basis.
*   Prefer minimal distributions like Alpine or Debian Slim.

### LABEL
*   Add labels to organize images by project, record licensing, or aid automation.
```dockerfile
LABEL com.example.version="0.0.1-beta" \
      com.example.release-date="2024-04-18" \
      vendor="ACME Incorporated"
```

### RUN
*   **Chain Commands:** Use `&&` to combine commands to reduce layers.
*   **Clean Up Immediately:** Remove temporary files in the same `RUN` layer (e.g., `rm -rf /var/lib/apt/lists/*`).
*   **Pipes:** Use `set -o pipefail` to ensure that a failure in any part of a command pipe causes the build to fail.
```dockerfile
RUN set -o pipefail && wget -O - https://some.site | wc -l > /number
```
*   **Here Documents:** Use here-docs for cleaner multi-line scripts:
```dockerfile
RUN <<EOF
apt-get update
apt-get install -y --no-install-recommends \
    curl \
    git
rm -rf /var/lib/apt/lists/*
EOF
```
*   **apt-get:** Always combine `apt-get update` with `apt-get install` to prevent caching issues ("cache busting"). Use `--no-install-recommends`.

### CMD & ENTRYPOINT
*   **Preferred Form:** Always use the **exec form**: `["executable", "param1", "param2"]`.
*   **Roles:** Use `ENTRYPOINT` for the main command and `CMD` for default arguments.
```dockerfile
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```
*   **Init Processes:** For multi-process containers, consider using an init process like `tini`.

### EXPOSE
*   **Informational:** `EXPOSE` is for documentation and does not actually publish the port.
*   Use the traditional port for your application (e.g., `EXPOSE 80` for web, `EXPOSE 27017` for MongoDB).

### COPY & ADD
*   **Prefer COPY:** It is more explicit and predictable.
*   **Use ADD Sparingly:** Use `ADD` only for auto-extracting local tar files or fetching remote URLs with checksum validation.
*   **Bind Mounts:** For temporary build dependencies, use `--mount=type=bind` to avoid persisting them in the image:
```dockerfile
RUN --mount=type=bind,source=requirements.txt,target=/tmp/requirements.txt \
    pip install --requirement /tmp/requirements.txt
```

### USER
*   **Run as Non-Root:** Always change to a non-root user.
*   **Creation Example:**
```dockerfile
RUN groupadd -r myuser && useradd --no-log-init -r -g myuser myuser
USER myuser
```
*   **UID Selection:** Use a static UID/GID above 10,000 to avoid overlapping with privileged host users.
*   **Dynamic UIDs:** Be aware that some environments (like OpenShift) run containers with random UIDs. Ensure your application can handle this by making necessary resources world-readable and writing temporary data to `/tmp`.
*   **Permissions:** Ensure executables are owned by root and not writable by the application user.

### WORKDIR
*   **Absolute Paths:** Always use absolute paths (e.g., `WORKDIR /app`).
*   **Avoid `RUN cd`:** Use `WORKDIR` instead of chaining `cd` commands.

### ENV & ARG
*   **Versioning:** Use `ENV` to set version numbers for easier maintenance.
*   **Secrets:** **NEVER** store secrets in `ENV` or `ARG`. If you must use a secret during build, do it in a single `RUN` layer and unset it:
```dockerfile
RUN export MY_SECRET="password" \
    && ./build-script.sh --key=$MY_SECRET \
    && unset MY_SECRET
```

### VOLUME
*   **Mutable Data:** Use `VOLUME` for database storage, configuration, or any mutable parts.

### ONBUILD
*   **Child Images:** `ONBUILD` commands execute when another image is built `FROM` the current image. Useful for language stacks (e.g., `ruby:onbuild`).

## 4. Build Cache Optimization

*   **Order Matters:** Structure your Dockerfile to maximize cache hits:
    - `FROM`
    - `ENV`
    - `RUN` (system packages)
    - `WORKDIR`
    - `COPY` (dependency files like `package.json`)
    - `RUN` (install dependencies)
    - `COPY` (application source code)
    - `CMD` / `ENTRYPOINT`

## 5. Build Context

*   **Use `.dockerignore`:** Exclude irrelevant files (e.g., `.git`, `node_modules`, `*.md`, secrets) to reduce build context size and prevent accidental leaks.
*   **Small Context:** Only include the files necessary for the build in the context directory.
