# Security Report — 2026-10-14
## Tools Run
| Tool | Status | Finding count |
|---|---|---|
| security-analysis agent (full-audit, Phase 0 scanner) | ok | 9 (0 critical, 0 high, 1 medium, 4 low, 4 info) |
| gitleaks | ok | 7 (history only, false positives) |
| trufflehog | ok | 20 unverified (history only, false positives) |
| semgrep (owasp-top-ten, secrets) | ok | 0 |
| trivy fs | ok | 0 |
| osv-scanner | n/a | no manifests |
| mcp-exfil-scan | ok | 0 |
| skillspector (--no-llm) | degraded | low signal |
| skill-audit | ok | 1 false positive (prose match) |
| bandit / mcp-scan / CodeQL / mcps-audit | skipped | no .py / opt-in off / n/a |
## Findings
- SEC-001 Medium: inspect.mjs local server serves whole cwd (CWE-200/668)
- SEC-002 Low: untrusted remote text printed to agent context (CWE-1427)
- SEC-003 Low: assets.mjs fetches arbitrary hosts, no size cap (CWE-918/400)
- SEC-004 Low: credits HTML hrefs not scheme-validated (CWE-79)
- SEC-005 Low: cdp.mjs remote-debugging-port on loopback (CWE-668)
- SEC-006 Info: allow rules read from direction.json
- SEC-007 Info: marketplace.json tracks mutable ref main
- SEC-008 Info: token-shaped strings in old history (taste-blocks/), not in tree
- SEC-009 Info: draw.mjs history path via env var
## Fixes Applied
- SEC-001: dotfile segments denied and realpath containment check in inspect.mjs
- SEC-003: 50 MB cap on image downloads in assets.mjs (host allowlist not added)
- SEC-004: credits links limited to http(s) URLs in assets.mjs
## Known Remaining Issues
SEC-002, SEC-005, SEC-007 (design-level, left as-is); SEC-003 SSRF host allowlist; SEC-006/008/009 need no action. Deep-audit skipped (unattended run).
