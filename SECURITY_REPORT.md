# Security Report — 2026-10-20
## Tools Run
| Tool | Status | Finding count |
|---|---|---|
| Gitleaks | run | 7 (history only, false positives) |
| TruffleHog | run | 20 unverified, 0 verified (false positives) |
| Bandit | run | 0 (no .py files) |
| Semgrep OWASP / secrets | run | 0 / 0 |
| Trivy | run | 0 (no manifests) |
| OSV-Scanner | run | 0 (no lockfiles) |
| mcps-audit | run | 80 (heuristic; one real: charts.js tooltip, fixed) |
| skill-security-auditor | run | 0 |
| mcp-exfil-scan | run | 0 |
| skillspector (--no-llm) | run | 74 (noise) |
| config-audit | run | host-scope only |
| mcp-scan, Jev, CodeQL | skipped | opt-in off / not applicable |
## Findings
No CVEs, no live secrets. 3 Low, 6 Informational.
- SEC-001 Low: assets.mjs follows redirects with no host allowlist; body buffered before 50 MB cap.
- SEC-002 Low: charts.js:303 tooltip inserted labels/series names unescaped (DOM XSS).
- SEC-003 Low: inspect.mjs/scenes.mjs dev server serves cwd, no Host header check (DNS rebinding, narrow).
- SEC-004/005 Info: gitleaks/trufflehog history-only false positives.
- SEC-006 Info: skillspector/mcps-audit heuristic noise.
- SEC-007 Info: headless Chrome launch flags (cdp.mjs).
- SEC-008 Info: CI actions on mutable tags, no permissions block.
- SEC-009 Info: unpinned `npx skills` in install docs.
- SEC-010 Info: history file at ~/.taste-skill/history.json.
## Fixes Applied
- SEC-002: charts.js tooltip now uses esc() for labels and series names.
- SEC-008 (partial): workflow now sets `permissions: contents: read`.
## Known Remaining Issues
SEC-001 (URL/host allowlist, streamed size cap), SEC-003 (Host header check), SEC-008 (SHA pinning), SEC-009, SEC-007, SEC-010. Deep-audit not run (unattended run).
