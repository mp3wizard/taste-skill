# Security Report — 2026-10-19
## Tools Run
| Tool | Status | Finding count |
|---|---|---|
| Gitleaks | run | 7 (history only, false positives) |
| TruffleHog | run | 20 unverified, 0 verified (history, false positives) |
| Semgrep (OWASP + secrets) | run | 0 |
| Trivy | run | 0 |
| OSV-Scanner | run (no manifests) | 0 |
| mcps-audit | run | 80 heuristic, mostly false positives |
| mcp-exfil-scan | run | 0 |
| skill-audit | run | 1 HIGH false positive (SEC-006) |
| skillspector (--no-llm) | run | 117 pattern hits, noise |
| Manual agent review | run | 4 code findings |
Skipped: bandit (no .py), semgrep TS (no .ts), CodeQL, mcp-scan (opt-in network).
## Findings
No CVEs. No live secrets.
- SEC-001 Low: `scenes.mjs` dev server lacked dotfile/symlink guard.
- SEC-002 Info: `inspect.mjs` serves whole cwd (guards present).
- SEC-003 Low: `assets.mjs` follows redirects without host allowlist; body buffered before size cap.
- SEC-004 Low: `charts.js` innerHTML with unescaped labels (XSS, CWE-79).
- SEC-005 Info: gitleaks/trufflehog history hits, false positives.
- SEC-006 Info: skill-audit "Netcat" false positive.
## Fixes Applied
- SEC-001: ported dotfile + realpath guards into `scenes.mjs`.
- SEC-004: added `esc()` and applied to label/sub in `charts.js`.
## Known Remaining Issues
- SEC-002: root remains cwd (narrowing could break pages using root-relative assets).
- SEC-003: no SSRF host allowlist / streaming cap in `assets.mjs`.
- SEC-005/006: false positives, not suppressed.
