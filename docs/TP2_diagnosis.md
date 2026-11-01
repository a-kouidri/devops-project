\# TP2 Diagnosis



| Defect | Gate that failed | Name in log | One-line cause |

|---|---|---|---|

| Unknown team handling | Tests | `test\_an\_unknown\_team\_is\_not\_found` — `AttributeError` | The endpoint accesses `team.id` when the requested team does not exist, instead of returning HTTP 404. |

| Tie-breaking order | Tests | `test\_equal\_points\_are\_broken\_by\_who\_got\_there\_first` — `AssertionError` | The scoring logic orders equal-point teams incorrectly instead of respecting the expected first-to-arrive rule. |

| Exposed secrets | Secret scan | `gitleaks` — 2 leaks found | Gitleaks detected two potential secrets in the repository. |

| Vulnerable dependency | Dependency audit | `requests 2.31.0` — `PYSEC-2026-1873`, `PYSEC-2026-1872`, `PYSEC-2026-2275` | The dependency has known vulnerabilities, causing pip-audit to fail. |

