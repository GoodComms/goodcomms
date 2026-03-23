# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in GoodComms, **please do not open a public GitHub issue.**

Report it privately by emailing **security@goodcomms.app** with:

- A clear description of the vulnerability
- Steps to reproduce it
- The potential impact
- Your suggested fix (optional but appreciated)

We will acknowledge your report within **48 hours** and aim to provide a fix or mitigation within **14 days** for critical issues. We'll keep you updated throughout.

---

## Scope

### In Scope
- Authentication bypass or session hijacking (JWT, token_version, WebSocket auth)
- Server-side privilege escalation (RBAC bypass, accessing private channels without permission)
- Remote code execution or SQL injection on the server
- Sensitive data exposure (passwords, tokens, user content)
- SSRF or path traversal in file upload/drive endpoints
- Voice/video stream interception or spoofing (UDP auth bypass)
- Denial-of-service vulnerabilities affecting server stability

### Out of Scope
- Vulnerabilities requiring physical access to the server machine
- Social engineering attacks
- Issues in third-party dependencies that are already publicly disclosed and not yet patched upstream
- Self-hosted servers that have been misconfigured by the operator
- Client-side issues that require the attacker to already have local machine access

---

## Supported Versions

| Version | Supported |
|---------|-----------|
| Latest release | ✅ |
| Older releases | ❌ — please upgrade |

We only maintain the latest release. If a vulnerability affects an older version, the fix will be in the next release.

---

## Disclosure Policy

We follow **coordinated disclosure**. Once a fix is released, we'll credit the reporter in the release notes (unless you prefer to remain anonymous).

We do not currently offer a bug bounty program, but we genuinely appreciate responsible disclosure — it helps keep the community safe.
