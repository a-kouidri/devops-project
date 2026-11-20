# Lab 3 – Containerization and Docker Security

**Team Members:**
- Hamami Soundous Asma
- Kouidri Aicha
- Teachtine Safa

---

## Exercise 1 – Multi-Stage Builds and Docker Cache

### (a) Image Size Comparison

The final `scoreboard:local` image has a size of **292 MB**, while the base image `python:3.12-slim` is **205 MB**.

### (b) Build Stage vs. Runtime Stage

The build stage contains:
- Application source code (`/app/src`).
- Packaging files (`pyproject.toml`, `README.md`, and `requirements.txt`).
- Installed dependencies and build-related files.

The runtime stage contains only the prepared virtual environment (`/opt/venv`) and the application data directory (`data/`). This produces a smaller and cleaner production image.

### (c) Security Benefits

Multi-stage builds improve security by excluding source code, packaging files, and unnecessary build tools from the final image.

This reduces the attack surface and limits the information and tools available to an attacker who gains access to the running container.

### (d) Docker Build Cache

Docker reuses cached layers when their inputs have not changed.

By copying `requirements.txt` and installing dependencies before copying the application source code, dependency installation does not need to run again when only `app.py` changes.

In our experiment:
- First build: **27.4 seconds**.
- Second build after changing one byte in `app.py`: **5.1 seconds**.
- Steps 2–6, including dependency installation (approximately 20 seconds), were cached.

This demonstrates how Docker layer ordering improves build performance.

---

## Exercise 2 – Container Hardening and Persistent Storage

### (a) Writable Locations

The container uses:
- `--read-only` to make the root filesystem read-only.
- `--tmpfs /tmp` to provide temporary writable storage.
- `-v sbdata:/app/data` to provide persistent writable storage for the SQLite database.

### (b) Named Volume vs. Host Directory

A named volume is managed by Docker and provides persistent storage independently of the project directory.

In our configuration, the application running as UID 10001 can write to the named volume.

A host directory may have different ownership and permissions, potentially preventing the non-root container user from writing to it.

Therefore, named volumes offer convenient and portable persistent storage.

### (c) Additional Security Options

**1. `--cap-drop ALL`**

Removes all Linux capabilities from the container, reducing the privileges available to the application and limiting potentially dangerous administrative operations.

**2. `--security-opt no-new-privileges`**

Prevents processes inside the container from gaining additional privileges through mechanisms such as setuid or setgid executables.

These options help reduce the impact of a compromised application.

---

## Exercise 3 – Graceful Shutdown and PID 1

### (a) Shutdown Time Comparison

| Container | PID 1 | Shutdown Time |
|---|---|---:|
| `sb` | Uvicorn | 0.566 seconds |
| `sb2` | Shell | 3.256 seconds |

### (b) Why Are the Results Different?

In container `sb`, Uvicorn runs directly as PID 1. When Docker sends SIGTERM, Uvicorn receives the signal and performs a graceful shutdown.

The logs show:
- Shutting down.
- Finished server process [1].
- Exit code 0.

In container `sb2`, the shell runs as PID 1 and starts the application as a child process.

The shell does not forward SIGTERM to its child. Consequently, the application does not shut down gracefully. Docker eventually sends SIGKILL after the grace period.

The container exits with code 137.

### (c) Impact on Rolling Updates

The difference between the two shutdown times is approximately **2.69 seconds per container**.

For 20 replicas:

`20 × 2.69 = 53.8 seconds`

This can add approximately 54 seconds to a sequential rolling update.

Poor signal handling can also interrupt active requests and cause service errors.

---

## Exercise 4 – Container Vulnerability Scanning

### (a) Trivy Scan Results

The Trivy scan reported:

- **Total vulnerabilities:** 2
- **HIGH:** 2
- **CRITICAL:** 0
- **Python package vulnerabilities:** 0

Both findings were associated with the Debian operating-system package `zlib1g` in Debian 13.7:

- CVE-2026-76844
- CVE-2026-85091

No secrets were detected in the visible scan results.

### (b) Trivy vs. pip-audit

`pip-audit` checks Python dependencies against known Python security advisories.

Trivy performs a broader scan of the container image, including:
- Operating-system packages.
- Python dependencies.
- Other supported image components.
- Potentially exposed secrets.

The two `zlib1g` findings were detected by Trivy because this package belongs to the Debian base image rather than the Python requirements file.

### (c) Why Pin the Scanner Version?

Pinning the Trivy version makes the scanning environment more predictable and reproducible.

It prevents an unexpected scanner update from changing pipeline behavior without an intentional decision.

It also reduces the risks associated with using an uncontrolled, floating tool version in the CI/CD pipeline.

---

## Exercise 5 – Image Publishing and Version Traceability

### (a) How Does the Commit SHA Reach the Running Application?

The commit identifier travels through the following process:

1. GitHub identifies the commit using its SHA.
2. GitHub Actions provides the value through `github.sha`.
3. The workflow passes it as the `BUILD_SHA` build argument.
4. The Dockerfile receives it through `ARG BUILD_SHA`.
5. The value is stored in the image using `ENV BUILD_SHA`.
6. The running application reads the environment variable.
7. The `/version` endpoint displays the corresponding commit identifier.

The same SHA is also used as the published image tag in GitHub Container Registry (GHCR).

This provides traceability between the Git commit, Docker image, and running application.

### (b) Why Should We Avoid Using Only the `latest` Tag?

The `latest` tag is mutable, meaning it can point to different images over time.

This can create problems when multiple application versions are deployed:
- Different machines may run different versions of the application.
- It becomes difficult to identify which version is running.
- Rolling back to a specific previous version becomes less reliable.

Using a unique commit SHA as the image tag provides an immutable version identifier and makes deployments easier to track and reproduce.

---

## Conclusion

In this lab, we explored Docker containerization, multi-stage builds, build caching, container hardening, persistent storage, graceful shutdown, and vulnerability scanning.

We also examined how to automate Docker image publishing through GitHub Actions and GitHub Container Registry.

Finally, we learned how commit-based image tagging improves deployment traceability and helps maintain consistent application versions.

These practices contribute to building more secure, efficient, and maintainable containerized applications.