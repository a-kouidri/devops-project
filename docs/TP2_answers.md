# TP2 answers

Team members: Hamami Soundous Asma, Kouidri Aicha, Teachtine Safa

## Exercise 1

(a) The `pull_request` event runs the pipeline before the changes are merged into `main`. It detects problems and prevents incorrect changes from being merged. A `push` event runs only after the changes are already pushed.

(b) Lint is a cheap and fast check. It detects style and formatting problems early, before the more expensive tests run.

(c) When a test suite takes 20 minutes, running the cheap gate first saves a lot of time and resources. If lint fails, we do not wait 20 minutes before learning about a simple formatting problem.

## Exercise 2

(a) `git commit --no-verify` skips Git hooks, including a pre-commit hook.

(b) A repository administrator, or any user allowed to bypass the branch protection rules, can still push a secret directly to `main`. The pipeline detects it, but it cannot block a push that bypasses the protection.

## Exercise 3

(a) Error line from the log: `ModuleNotFoundError: No module named 'scoreboard'`

(b) The project uses a src layout: the package is located in `src/scoreboard` and not in the project root. The `src` directory is not on Python's import path, so without installing the project, the tests cannot import `scoreboard`.

(c) The command is `pip install -e .`. On our computer, `make install` ran it for us. In the pipeline, the install step of the workflow runs it.

## Exercise 4

Test 1 failed by exception (`AttributeError`). Test 2 failed by assertion (`AssertionError`).

| Defect | Gate that failed | Name in the log | One-line cause |
|---|---|---|---|
| Test 1 | Tests | `test_an_unknown_team_is_not_found` (exception: `AttributeError`) | The code accesses `team.id` when the team does not exist, instead of returning HTTP 404. |
| Test 2 | Tests | `test_equal_points_are_broken_by_who_got_there_first` (assertion: `AssertionError`) | Teams with equal points are ordered incorrectly; the first-to-arrive rule is not respected. |
| Credential | Secret scan | gitleaks: 2 leaks found | gitleaks detected secrets committed in the repository. |
| Dependency | Dependency audit | `requests 2.31.0`: PYSEC-2026-1873, PYSEC-2026-1872, PYSEC-2026-2275 | The pinned version of `requests` has known vulnerabilities, so pip-audit fails. |

## Exercise 5

We fixed the application code (not the tests), removed the secret, and changed the dependency version in `requirements.txt` from `requests==2.31.0` to `requests==2.33.0`. The pull request became green.

(a) `requests` is a direct dependency.

(b) We found `requests==2.31.0` written explicitly in `requirements.txt`. Command used: `grep requests requirements.txt` (on Windows PowerShell: `Select-String "requests" requirements.txt`).
