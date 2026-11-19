# TP3 answers

Team members: Hamami Soundous Asma, Kouidri Aicha, Teachtine Safa

## Exercise 1

(a) `scoreboard:local` is 292MB and `python:3.12-slim` is 205MB.

(b) The build stage contains the source tree (`/app/src`) and the packaging files (`pyproject.toml`, `README.md`, `requirements.txt`), and the layers created while installing the packages. The runtime image only contains the finished virtualenv `/opt/venv` and the `data/` folder.

(c) Without the source tree and the packaging files, an attacker who gets code execution in the container cannot read or modify the source code, or rebuild or reinstall the project from it. The image is also smaller, so there are fewer files to scan and to attack. Without the build layers, no leftover build files, caches or temporary data are shipped.

(d) Docker caches each layer, and a layer is rebuilt only when its input changes. `COPY requirements.txt` and `pip install -r requirements.txt` come first, so they do not depend on the source. In the second build (after one byte changed in `app.py`), steps 2 to 6 of the build stage said CACHED, including the 20 s package installation. Only `COPY src/` and `pip install .` ran again, and the build took 5.1 s instead of 27.4 s.

## Exercise 2

(a) The named volume `-v sbdata:/app/data` is a writable mount on `/app/data`, where the SQLite database is. `--tmpfs /tmp` also gives a writable place for temporary files.

(b) A named volume is created and managed by Docker, so the process running as UID 10001 can write to it. A host folder like `$PWD/data` belongs to the host user, and UID 10001 usually cannot write there (permission denied). A named volume also does not depend on a host path, and it keeps the data away from the project folder.

(c) `--cap-drop ALL`: the attacker no longer has any Linux capability, so he cannot do privileged actions such as raw network access, changing file ownership or mounting. `--security-opt no-new-privileges`: the attacker cannot gain more privileges through setuid programs.

## Exercise 3

(a) `docker stop sb` (uvicorn is PID 1): 0.566 s. `docker stop sb2` (a shell is PID 1): 3.256 s.

(b) In `sb`, uvicorn is PID 1, so it receives SIGTERM directly from `docker stop` and shuts down cleanly. Its log shows "Shutting down" and "Finished server process [1]", and the exit code is 0. In `sb2`, PID 1 is `sh -c "python -m scoreboard; echo exited"`. The shell does not pass SIGTERM to its child, so the application (PID 7) never received it. Docker waited, then killed the container with SIGKILL: exit code 137 and no shutdown message in the log.

(c) During a rolling update each replica is stopped one after the other. With our times, each stop is about 2.7 s slower, so twenty replicas add about 54 s. The longer the grace period, the worse it is. The requests that are still running are cut brutally, with no clean shutdown, so users can see errors during the update.

## Exercise 4

(a) There are 2 findings (HIGH: 2, CRITICAL: 0). Both are in the same package, `zlib1g` (an operating system package of Debian 13.7): CVE-2026-76844 and CVE-2026-85091. All the Python packages show 0 findings.

(b) `pip-audit` only checks the Python packages listed in `requirements.txt`, against a database of Python advisories. Trivy scans the whole image, including the operating system packages of the base image. `zlib1g` comes from the Debian base image and is not in `requirements.txt`, so `pip-audit` cannot see it.

(c) With a fixed version, the scan is reproducible: the same image gives the same result, and a new scanner version cannot change the result or break the pipeline without a decision of ours. It is also a supply chain protection: a floating tag could silently pull a different or compromised tool.

## Exercise 5

(a) The commit identifier goes through these steps: the Git commit (its SHA) -> `github.sha` in the GitHub Actions workflow -> the build argument `BUILD_SHA` passed by `docker/build-push-action` -> `ARG BUILD_SHA` in the Dockerfile -> `ENV BUILD_SHA` in the image -> the environment variable read by the running process -> the `/version` endpoint. The same SHA is also the tag of the image in ghcr.io.

(b) A tag like `latest` is mutable: each push moves it to the new image. If the two versions both use `latest`, we cannot tell which version a container runs, and we cannot run both on purpose. A node can keep the old cached `latest` while another node pulls the new one, so replicas of the same deployment run different code, and a rollback to the previous version is impossible. With a SHA tag, each version has its own identity.
