# Security Report — 2026-10-25

## Tools Run
| Tool | Status | Finding count |
|---|---|---|
| gitleaks | OK | 7 (history only, sourcegraph-token pattern, likely false positives) |
| semgrep (owasp-top-ten, secrets) | OK | 0 |
| trivy fs | OK | 0 |
| trufflehog | OK | 0 verified, 20 unverified (history only) |
| osv-scanner | N/A (no manifests) | 0 |
| skill-audit | OK | 4 benign flags |
| mcp-exfil-scan | OK | 0 (CLEAN) |
| skillspector (--no-llm) | OK, degraded coverage | 150 (reviewed by hand, none real) |
| security-analysis agent (manual) | OK | 2 low, 4 informational |
| bandit / CodeQL / mcps-audit | skipped (not applicable) | - |

## Findings
- SEC-001 (Low, CWE-918): `scripts/assets.mjs:243-256,749-795` literal-host SSRF filter only; redirects followed without re-check.
- SEC-002 (Low, CWE-770): `scripts/assets.mjs:~757` image body buffered before 50 MB check.
- SEC-003 (Info, CWE-1357): `.github/workflows/check-skills.yml:19` `yaml@2.6.1` installed without lockfile.
- SEC-004 (Info, CWE-798): historical sourcegraph/Circle token-pattern hits, not present at HEAD, unverified.
- SEC-005 (Info): inspect.mjs loopback server reviewed, no escape path.
- SEC-006 (Info): charts.js / stage.js innerHTML sinks all escaped.
- No CVEs, no live secrets.

## Fixes Applied
- Merge of upstream v2.1.5 initially took upstream's `inspect.mjs`, dropping fork hardening (Host check against DNS rebinding, dotfile block, realpath symlink check, `TASTE_SERVE_CWD` root). Restored.
- `charts.js` `esc` kept null-safe (`String(v ?? '')`) while adopting upstream's added escaping.

## Known Remaining Issues
- SEC-001, SEC-002, SEC-003: upstream-owned code, not patched in the fork to avoid recurring merge conflicts; fix before the asset downloader handles untrusted references.
- SEC-004: needs manual triage of historical blobs.
- skillspector could not parse probe.mjs, rules.mjs, lint.mjs, scenes.mjs.
