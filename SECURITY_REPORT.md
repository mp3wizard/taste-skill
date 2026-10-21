# Security Report — 2026-10-21
## Tools Run
| Tool | Status | Finding count |
|---|---|---|
| gitleaks | OK (history) | 7 (false positives) |
| trufflehog | OK | 20 unverified, 0 verified (false positives) |
| trivy | OK | 0 |
| osv-scanner | OK (no lockfiles) | 0 |
| semgrep | OK | 2 (CI) |
| skillspector (no LLM) | OK, degraded | 0 |
| skill-audit | OK | 0 |
| mcp-exfil-scan | OK | 0 |
| config-audit | OK | 0 |
| manual agent review | OK | 13 |
| mcps-audit, bandit, mcp-scan, CodeQL | N/A or skipped | - |
## Findings
No CVEs, no live secrets. 8 Low, 5 Informational:
- L01 mutable action tags (check-skills.yml:15-16)
- L02 npm install without --ignore-scripts (check-skills.yml:21)
- L03 preview servers serve whole cwd (inspect.mjs, scenes.mjs)
- L04 realpath/read race in static servers
- L05 drop() may SIGKILL recycled PID (cdp.mjs:84)
- L06 assets.mjs follows redirects to any host (SSRF)
- L07 assets.mjs poster fetch has no size cap
- L08 innerHTML re-parse in stage.js:106
- I01 history-only secret-scanner false positives; I02-I05 tmp sweep, URL navigation, labels/charts.js interpolation
## Fixes Applied
- L01 CI actions pinned to commit SHAs; L02 npm install uses --ignore-scripts
- L03 preview servers serve only the target file's folder (TASTE_SERVE_CWD=1 widens) and reject wrong Host header
- L05 Chrome pid cleared on exit, so drop() never signals a recycled pid
- L06 assets.mjs rejects non-http(s), localhost and private/link-local literal hosts (no DNS resolution, redirects not re-checked)
- L07 poster download capped at 50 MB
- L08 stage.js builds spans with DOM APIs, not innerHTML
## Known Remaining Issues
- L04 check/use race in static servers (needs local write access; accepted)
- L06 DNS-based and redirect-based SSRF not covered
- I01-I05 informational, no action
