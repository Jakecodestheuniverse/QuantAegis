# Security Policy

## Our Commitment

QuantAegis is designed for use in financial institutions where security is non-negotiable. We take vulnerability reports seriously and commit to responding quickly and transparently.

---

## Supported Versions

| Version | Security Support |
|---|---|
| `main` branch | ✅ Active |
| Latest release | ✅ Active |
| Previous major release | ⚠️ Critical fixes only |
| Older releases | ❌ Not supported |

---

## Reporting a Vulnerability

**Please do NOT open a public GitHub Issue for security vulnerabilities.**

Report vulnerabilities via one of these channels:

1. **GitHub Private Security Advisory** (preferred): Use the "Report a Vulnerability" button on our GitHub Security tab
2. **Email**: security@quantaegis.dev — PGP key available at [link]

### What to Include

- Description of the vulnerability and its potential impact
- Steps to reproduce (minimal example preferred)
- Affected versions
- Any suggested mitigations you're aware of

### What to Expect

| Step | Timeline |
|---|---|
| Acknowledgment of report | Within 48 hours |
| Initial triage and severity assessment | Within 5 business days |
| Status update (fix timeline or decision) | Within 10 business days |
| Patch release (for confirmed critical issues) | Within 30 days |

We follow [coordinated vulnerability disclosure](https://cheatsheetseries.owasp.org/cheatsheets/Vulnerability_Disclosure_Cheat_Sheet.html). We will credit reporters in release notes unless they prefer to remain anonymous.

---

## Security Design Principles

### Secrets & Credentials
- Never hardcode credentials, API keys, or tokens — ever
- Production deployments integrate with Vault, AWS Secrets Manager, or Azure Key Vault
- `.env` files are for local development only and are gitignored
- CI/CD pipelines use ephemeral short-lived credentials

### Data Handling
- No real financial data should ever appear in the repository (tests use synthetic data only)
- The Data Masking Middleware must be enabled in any deployment handling PII, NPI, or MNPI
- Data at rest is encrypted (AES-256); data in transit uses TLS 1.3 minimum

### Agent Security
- Agents authenticate inter-service calls with short-lived JWTs (15-minute expiry)
- Agent outputs are never directly executed — they are analytical artifacts
- All external API calls from agents are logged and rate-limited
- Prompt injection mitigations are applied to all user-controlled inputs passed to LLMs

### Dependency Security
- `pip-audit` runs on every PR and nightly against the main branch
- `bandit` scans for common Python security anti-patterns in CI
- Dependencies are pinned in `requirements.txt` and reviewed on each release

### Network Security
- Default Docker Compose configuration binds only to localhost
- Kubernetes configuration uses NetworkPolicies to restrict inter-pod communication
- No agent makes outbound network calls except to explicitly allowlisted endpoints

---

## Financial-Specific Security Considerations

### MNPI (Material Non-Public Information)
This framework must not be used to process or act on MNPI in ways that violate securities law. Operators are responsible for configuring appropriate information barriers.

### Model Output Handling
All agent outputs must be treated as **analytical inputs to human decision-making**, not as executable trading signals. The framework is designed to make this boundary explicit, but operators must enforce it in their deployment.

### Audit Log Integrity
The audit log is append-only by design. Do not modify audit log files. Tampering with audit logs may violate regulatory requirements and is architecturally detectable.

### Regulatory Data Residency
If your regulatory environment requires data residency (e.g., GDPR, MAS TRM), configure the deployment to ensure all data processing and storage occurs in the required jurisdiction. See `docs/deployment/data-residency.md`.

---

## Known Limitations

- LLM outputs are probabilistic and may contain errors. Human review is required before acting on any output.
- The compliance rule engine covers common regulations but is not a substitute for qualified legal or compliance counsel.
- Third-party data connectors inherit the security posture of the data vendor.

---

## Security Changelog

Security fixes are documented in `CHANGELOG.md` with a `[SECURITY]` tag. Critical fixes trigger an immediate patch release.

---

*This policy is reviewed and updated at each major release.*
