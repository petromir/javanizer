# Docker Runtime and Security

While Dockerfiles define how an image is built, these best practices cover the security and management of containers at runtime and during the development lifecycle.

## 1. Runtime Security

*   **Protect the Docker Socket:** `/var/run/docker.sock` is a highly privileged gateway. Ensure it has restricted permissions (root-only). **Never** expose it over unprotected TCP; use TLS if remote access is required.
*   **Drop Capabilities:** Use `--cap-drop=ALL` and then `--cap-add` only the specific kernel capabilities required. This minimizes the impact of a container breakout.
*   **Use Security Profiles:**
    *   **AppArmor:** Use or create AppArmor profiles to restrict container capabilities (e.g., `--security-opt apparmor=docker-default`).
    *   **Seccomp:** Use seccomp profiles to filter the syscalls a container can make (e.g., `--security-opt seccomp=unconfined` is dangerous; use the default or a custom profile).
*   **Read-Only Filesystem:** Run containers with `--read-only` to prevent attackers from writing malicious files or modifying the application at runtime. Use temporary volumes for specific writable paths if needed.
*   **Limit Resources:** Always set `--memory` and `--cpus` limits to prevent a single container from causing a Denial of Service (DoS) on the host.

## 2. Image Management

*   **Sign and Verify Images:** Use **Docker Content Trust (DCT)** by setting `export DOCKER_CONTENT_TRUST=1`. This ensures only signed images can be pulled and run.
*   **Scan for Vulnerabilities:**
    *   **Shift Left:** Integrate scanning into CI/CD (e.g., using `trivy`, `clair`, or `snyk`).
    *   **Docker Scout:** Use Docker Scout's "Up-to-Date Base Images" policy to automatically identify when base images have newer versions or security patches.
*   **Tag Mutability and docker-lock:**
    *   Be aware that tags like `v1.0` can be updated.
    *   Use `docker-lock` to generate a lockfile that tracks the exact SHA256 digests of your images while still using human-readable tags in your Dockerfile.

## 3. Health and Monitoring

*   **Health Checks:** Use the `HEALTHCHECK` instruction to allow Docker/Swarm to monitor service status. In Kubernetes, prefer `livenessProbes` and `readinessProbes`.
*   **Logging:** Follow the 12-factor app rule: log everything to `stdout` and `stderr`. Use logging drivers (e.g., `fluentd`, `json-file`) to aggregate logs centrally.

## 4. Host Integration

*   **Sensitive Mounts:** Avoid bind-mounting host directories like `/etc`, `/root`, or the Docker socket. If you must mount the socket (e.g., for a CI agent), ensure the agent is highly trusted.
*   **Rootless Docker:** Consider using "Rootless Mode" for the Docker daemon itself to run the entire Docker stack without root privileges on the host.
*   **Sticky Bit on /tmp:** Use `/tmp` for temporary data as it usually has the sticky bit set, preventing users from deleting each other's files.

## 5. Development Workflow

*   **Optimization Tools:**
    *   **Hadolint:** A linter for Dockerfiles to catch best practice violations.
    *   **Dive:** A tool for exploring a docker image, layer contents, and discovering ways to shrink your image size.
    *   **DockerSlim:** Automatically minifies your containers by analyzing the runtime requirements.
*   **Local Scanning:** Scan images locally during development (`docker scout quickview`, `trivy image ...`) before pushing to any registry.
*   **Minimize Context:** Always use a `.dockerignore` file to ensure secrets, `.git`, and unnecessary large files aren't sent to the Docker daemon.
