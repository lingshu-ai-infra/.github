# Security Policy

## Supported Versions

| Version | Supported          |
|---------|--------------------|
| 0.1.x (alpha) | ✅ active development |
| < 0.1.0 | ❌ no longer supported |

## Reporting a Vulnerability

**Please do not open a public issue for security problems.**

Send your report by email to **security@lingshu.ai** with:

1. A clear description of the vulnerability
2. Reproduction steps (PoC if possible)
3. Impact assessment (what an attacker could achieve)
4. Affected component + version

You should receive an initial acknowledgement within **48 hours**. We aim to
provide a timeline for fix or mitigation within **7 days**, and a coordinated
disclosure after the fix is shipped.

## Severity Classification

We follow the [CVSS v3.1](https://www.first.org/cvss/calculator/3.1) rubric:

| Severity | CVSS | SLA to fix |
|---|---|---|
| Critical | 9.0–10.0 | 7 days |
| High | 7.0–8.9 | 30 days |
| Medium | 4.0–6.9 | 90 days |
| Low | 0.1–3.9 | next release |

## What to Expect

1. **Acknowledgement** within 48h.
2. **Triage** — we confirm severity, affected scope, and assign owner.
3. **Patch** developed on a private branch.
4. **Coordinated disclosure** — once patched, we publish a CVE + advisory
   with credit to the reporter (if desired).
5. **Backport** to supported branches.

## Scope

In scope:

- Authentication / authorization bypass
- Code execution / RCE in any daemon (Gateway / Scheduler / Worker)
- Information disclosure (secrets, PII, internal cluster topology)
- Protocol-level attacks (Protobuf parsing, Netty framing)
- GPU memory leaks / cross-tenant data exposure
- Supply chain (compromised dependencies)

Out of scope:

- Theoretical vulnerabilities without a concrete attack path
- Issues requiring physical / root access already
- Social engineering
- DoS via legitimately-issued API quota (not bugs)

## Hall of Fame

We acknowledge reporters (with permission) in our release notes. 🙏

## Contact

- **Security email**: security@lingshu.ai
- **PGP key**: published at <https://lingshu.ai/.well-known/pgp-key.txt>
  (placeholder; rotate once stable)