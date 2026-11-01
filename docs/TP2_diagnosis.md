# TP2 diagnosis

| Defect | Gate that failed | Name in log | One-line cause |
|---|---|---|---|
| Challenge default | Tests | `test_defaults_are_applied` — `AssertionError: assert 100 == 10` | The default `decay_solves` value is 100 instead of 10. |
| Missing challenge file | Tests | `test_a_missing_file_is_reported_clearly` — `FileNotFoundError` | The missing-file error is not caught and reported as the expected challenge-file error. |
| Credential | Secret scan | `gitleaks` — 1 leak found | The scan detected a credential in the repository. |
| Vulnerable dependency | Dependency audit | `urllib3 2.0.5` — `PYSEC-2023-212`, `PYSEC-2026-1999`, `PYSEC-2026-1998`, `PYSEC-2026-1994`, `PYSEC-2026-1996`, `PYSEC-2026-141`, `PYSEC-2026-1995`, `PYSEC-2026-4177`, `PYSEC-2026-4175` | The pinned urllib3 version has known vulnerabilities. |
