# Automated Security Scan Report

**Target:** `/Users/mp3wizard/Public/Claude skill/taste-skill`
**Scanned at:** 2026-06-16T02:19Z (09:19 +07:00)
**Git HEAD:** `cef07ff`
**Standard:** OWASP APTS-aligned (Scope Enforcement · Auditability · Manipulation Resistance · Reporting)

---

## Verdict

**No malicious behavior, secrets, or exploitable vulnerabilities found.** This is a prompt-only Claude Skill (design-analysis tool) plus a static marketing site. The one executable artifact (`references/extract.js`) is read-only DOM sampling with no network egress. All HIGH/CRITICAL machine findings were reviewed and are **false positives** (binary-file pattern matches and ordinary HTML comments). Residual risk is **LOW**.

---

## Scope Record

```
Scan target: /Users/mp3wizard/Public/Claude skill/taste-skill
Git HEAD:    cef07ff
Include:     all supported
Exclude:     .gitignore honored by each tool
```

Files in scope (non-`.git`):
- `README.md`, `SKILL.md`
- `references/`: `extract.js`, `step1-measure.md`, `step2-pattern.md`, `step3-taste.md`, `step4-observer.md`, `export-formats.md`
- `evals/evals.json`
- `docs/`: `index.html`, `Cherry.glb`, PNG/SVG assets (static marketing site)

---

## Coverage Disclosure (APTS § Reporting)

| Tool | Ran? | Version | Result | Notes |
|------|------|---------|--------|-------|
| Gitleaks | ✅ OK | 8.30.1 | **0 leaks** | 16 commits, ~185 KB scanned |
| TruffleHog | ✅ OK | 3.94.2 | **0 secrets** (0 verified) | full git history, 102 chunks |
| Trivy (misconfig+secret) | ✅ OK | 0.69.3 | **0 issues** | safe version (not GHSA-69fq-xp46-6x23) |
| Trivy (vuln/SCA) | ⚠️ PARTIAL | 0.69.3 | DB download blocked | `docker-credential-desktop` missing; no lockfiles anyway |
| OSV-Scanner | ✅ OK | 2.3.5 | **0 vulns** | no package manifests/lockfiles present |
| Bandit | N/A | 1.9.4 | skipped | no `.py` files in repo |
| Semgrep | ❌ SKIPPED | — | engine killed (`-9`/137) | `semgrep-core` SIGKILLed at startup in this env; see Coverage Gaps |
| skill-security-auditor | ✅ OK | v2.0 | **35/100 MEDIUM** | reviewed → benign (see below) |
| skillspector | ✅ OK | NVIDIA | 100/100 "CRITICAL" | **all findings FALSE POSITIVE** (see below) |
| mcp-exfil-scan | ❌ ERROR | v1.0 | script crash | `MCP_CONFIGS[@]: unbound variable` (no MCP configs → bash `set -u` bug). No MCP manifests exist to scan. |
| config-audit | ✅ OK | — | informational only | scanned **user's global `~/.claude`**, not this repo |
| CodeQL | N/A | — | skipped | no CodeQL workflow in repo |

---

## Gitleaks — Secrets in git history + filesystem
**Summary:** 0 leaks. 16 commits, ~184.81 KB scanned in 98.5ms.

## TruffleHog — Secrets with live verification
**Summary:** `verified_secrets: 0, unverified_secrets: 0`. 102 chunks / 208 KB across full git history.

## Trivy — Misconfig + Secret scan
**Summary:** Misconfigurations: 0 (no config files detected). Secrets: 0 (clean). Version 0.69.3 is **not** in the compromised range (0.69.4–0.69.6).
> Vuln/dependency DB download failed (`docker-credential-desktop` not in PATH). Immaterial here — the repo contains no dependency manifests or lockfiles, confirmed by OSV-Scanner.

## OSV-Scanner — SCA via OSV.dev
**Summary:** "No package sources found." No `package.json`/lockfiles/requirements present → no third-party dependency surface to audit.

## skill-security-auditor — Skill risk analysis
**Summary:** Score **35/100 — MEDIUM RISK (Approve with caution)**. 333 lines analyzed.
- Dangerous patterns: **0** · Obfuscation: **0** · Credential access: **0** · Privilege escalation: **0** · External dependencies: **0**
- Network URLs: 4 — all benign documentation/marketing links (`linear.app`, `playwright.dev`, `vercel.com`, `stripe.com`) referenced as export-format examples.
- Prompt-injection patterns: **1 — "Silent action instruction" (Medium)**. **Reviewed → benign.** The matched lines are ordinary skill-design directives, not covert agent manipulation:
  - `SKILL.md:199` — "If a linked page fails to load or is blocked, skip it silently and note it in the Phase 6 report."
  - `SKILL.md:251` — "If either file already exists, overwrite without asking" (overwrites *the skill's own output files* on re-run — expected idempotent behavior).
  - `SKILL.md:120` — actually instructs the *opposite* of covert behavior: "do not silently continue with degraded data."
- File operations: 8 — writes the skill's documented output (Design Map / Taste DNA rules files), as specified in the output contract.

**Assessment:** The MEDIUM score is driven by the single heuristic prompt-injection match, which on inspection is normal documentation. No actual injection or covert-action vector.

## skillspector — AI-skill scanner (NVIDIA, `--no-llm`)
**Summary:** Reported 100/100 "CRITICAL" — **every contributing finding is a false positive.** Breakdown:
- `docs/Cherry.glb:48` (MP2 Context Window Stuffing) — binary **3D model** file; pattern match on binary bytes.
- `docs/senlin.png:544/1065/1413` (TM1 Tool Parameter Abuse) — binary **PNG image**; pattern match on image bytes. No tools, no parameters.
- `docs/index.html:622` & `:763` (P2 Hidden Instructions) — these lines are plain HTML comments `<!-- HERO -->` and `<!-- EXPORT FORMATS -->`. No hidden instructions.
- E1 External Transmission / EA2 Autonomous Decision (MEDIUM) — generic heuristic matches on skill prose; no exfiltration or auto-execution present.

**Assessment:** skillspector is tuned for `.skill`/agent artifacts and produces noise when pointed at a static website with binary assets. Discount the 100/100 score for this repo. LLM-assisted mode was **not** run (privacy gate; `--no-llm` only).

## Manual review — `references/extract.js` (the only executable artifact)
321 lines. Injected into a target page via Playwright `browser_evaluate`. **Read-only by design:**
- Reads `getComputedStyle`, `getBoundingClientRect`, DOM structure; tallies colors/fonts/spacing/effects.
- **No** `fetch`/`XMLHttpRequest`/`navigator.sendBeacon`/WebSocket — zero network egress.
- **No** `eval`/`Function()`/dynamic code execution. No storage, cookie, or credential access. Returns a plain JSON snapshot.
- Wraps cross-origin stylesheet reads in `try/catch` and skips them — no cross-origin data exfiltration.

**Non-security note (latent bug, not a vulnerability):** line 112 calls `isVisible(el)`, but the helper defined in the file is named `isRendered`. `isVisible` is undefined → the accent-color loop throws at runtime. Cosmetic/correctness only; worth fixing but no security impact.

## config-audit — Claude configuration
**Summary:** All findings are **LOW / informational** and pertain to the **user's global `~/.claude` environment** (installed plugins' hook declarations), **not this repository**. The repo ships no `.claude/` config, hooks, or `CLAUDE.md`. No safety-bypass directives detected. Out of scope for this target; recorded for transparency only.

---

## Cross-Tool Observations
- **Secrets:** Gitleaks (0) and TruffleHog (0) agree across filesystem and full git history → high confidence: no secrets.
- **Dependency surface:** OSV-Scanner and Trivy agree there are no package manifests/lockfiles → no third-party SCA risk.
- **Skill safety:** skill-security-auditor's lone MEDIUM and skillspector's HIGH/CRITICAL findings do **not** corroborate any real issue — manual review shows them to be documentation heuristics and binary/HTML-comment false positives respectively. No cross-tool agreement on any genuine threat.

## Coverage Gaps
- **Semgrep SAST not completed:** `semgrep-core` is SIGKILLed (`-9`, exit 137) at startup in this sandbox regardless of build (brew and pipx) or `--max-memory`. The JS attack surface is a single 321-line file, which was **reviewed manually** above in lieu of Semgrep.
- **Trivy CVE/SCA DB** could not download (missing docker credential helper) — immaterial; OSV confirms no dependencies.
- **mcp-exfil-scan** crashed on an empty-array `set -u` bug; there are no MCP configs in the repo to scan, so no real coverage lost.
- **Not covered by any static tool:** business logic, IDOR, and runtime behavior. For this read-only design-sampling skill the runtime risk is inherently low.

### APTS Audit Log
- **Log:** `/tmp/css-scan-20260616T021918Z.jsonl`
- **Tool runs recorded:** 6 (measured: 6, asserted: 0)
- **Standard:** OWASP APTS § Auditability

---

*Manipulation-resistance note: directives embedded in scanned files (e.g. "skip silently", "overwrite without asking") were treated as data and evaluated on merit — none altered severity or suppressed findings.*
