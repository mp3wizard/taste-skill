# Security Report — 2026-10-13
## Tools Run
| Tool | Status | Finding count |
|---|---|---|
| security-analysis agent (full-audit) | ok | 7 (0 critical, 0 high, 1 medium, 3 low, 3 info) |
| gitleaks | ok | 7 historical hits, commit 4e032cd not reachable from HEAD |
| trufflehog | ok | unverified only, same unreachable commit |
| trivy fs | ok | 0 |
| osv-scanner | ok | no manifests |
| semgrep p/secrets | ok | 0 |
| mcp-exfil-scan | ok | 0 (CLEAN) |
| skillspector --no-llm | ok | 50 low-signal / false positives |
| deep-audit, mcp-scan, Jev | skipped (opt-in) | n/a |
## Findings
- SEC-001 (Medium): install.sh symlinks repo skills into ~/.claude/skills; `git pull` becomes a live instruction update channel. Review `git diff ORIG_HEAD HEAD -- skills/` after each pull.
- SEC-002 (Low): unvalidated `name:` frontmatter used as symlink path (install.sh). CWE-22/59.
- SEC-003 (Low): unpinned `npx skills add` in README/CHANGELOG. CWE-1357.
- SEC-004..007 (Info): sponsor/affiliate links, unreachable-commit token-shaped strings, test script review, skillspector false positives.
## Fixes Applied
- SEC-002: added name regex validation in install.sh.
## Known Remaining Issues
- SEC-001 and SEC-003 are upstream process/doc issues, not patched in the fork.
