# cybersecurity-skill

**Expert cybersecurity skill for Claude  OWASP · NIST CSF · ISO 27001 · CIS Controls · LLM Security**

A modular, production-grade security skill for Claude that applies a 7-layer defensive analysis methodology across web applications, APIs, infrastructure, LLM systems, and enterprise compliance frameworks. Built to function as a universal defensive intelligence layer — not a checklist, but a reasoning engine that adapts to context.

---

## What this skill does

When active, Claude applies a structured 7-layer security analysis to every relevant task:

- Generates secure code by default (Spring Boot, Angular, Node.js, Python) with hardening applied during generation, not as an afterthought
- Audits code and architecture across 7 layers: static analysis, dependency scanning, secret detection, infrastructure hardening, authentication logic, business logic, and runtime observability
- Produces structured bug bounty reports formatted for HackerOne, Bugcrowd, and YesWeHack
- Performs red team analysis by reasoning from the attacker's perspective
- Maps findings to OWASP, CWE, CVE, CVSS, NIST CSF, ISO 27001, and CIS Controls
- Generates compliance reports (NIST CSF maturity assessment, ISO 27001 SoA, CIS IG1/IG2/IG3 gap analysis)
- Models threats using STRIDE for new architectures and services
- Covers LLM-specific vulnerabilities: prompt injection, insecure output handling, excessive agency, model theft, and more

Every analysis ends with a Security Scorecard and a mandatory "residual risks" section. The skill never concludes that an application is secure — only that specific risks have been identified and mitigated.

---

## Frameworks covered

| Framework | Scope | Coverage |
|---|---|---|
| OWASP Web Top 10 (2021) | Web vulnerabilities with payloads, fixes, CWE references | Full |
| OWASP LLM Top 10 (2025) | LLM-specific risks: prompt injection, excessive agency, RAG poisoning | Full |
| NIST CSF 2.0 | GOVERN / IDENTIFY / PROTECT / DETECT / RESPOND / RECOVER | 70% |
| ISO 27001:2022 | 93 controls — focus on A.8 (technological) | 60% |
| CIS Controls v8 | 18 controls, IG1/IG2/IG3 progression, remediation SLAs | 75% |

---

## The 7 analysis layers

```
Layer 1  Static analysis       Injection tracing, XSS, deserialization, IDOR, path traversal
Layer 2  Dependency analysis   CVE matching, SLA-based remediation, transitive vulnerabilities
Layer 3  Secrets detection     API keys, hardcoded credentials, entropy verification, vault usage
Layer 4  Infrastructure        Dockerfile hardening, Kubernetes RBAC, NetworkPolicy, securityContext
Layer 5  Auth and authz        Password hashing, JWT storage, IDOR simulation, MFA, rate limiting
Layer 6  Business logic        Payment bypass, race conditions, TOCTOU, idempotency gaps
Layer 7  Runtime observability Security logs, HTTP headers, CORS, anomaly detection
```

---

## File structure

```
cybersecurity/
    SKILL.md                   Orchestrator — 7 layers, 5 operational modes, output formats
    marketplace.json           Marketplace metadata for one-command installation
    references/
        owasp-web.md           OWASP Web Top 10 2021 — payloads, fixes, CWE
        owasp-llm.md           OWASP LLM Top 10 2025 — agents, RAG, prompt injection
        nist-csf.md            NIST CSF 2.0 — governance, incident response playbook
        iso27001.md            ISO 27001:2022 — 93 controls mapped to dev practices
        cis-v8.md              CIS Controls v8 — 18 controls, IG progression, benchmarks
```

The orchestrator (`SKILL.md`) loads reference files dynamically based on detected context. A conversation about a RAG pipeline loads only `owasp-llm.md`. A compliance audit loads `iso27001.md` or `nist-csf.md`. The active context never exceeds 600 lines.

---

## Installation

### Claude.ai (Pro, Max, Team, Enterprise)

1. Go to **Settings > Capabilities > Skills**
2. Click **Upload a skill**
3. Upload `cybersecurity-v2.skill` (download from [Releases](../../releases))

### Claude Code

```bash
mkdir -p ~/.claude/skills/cybersecurity
unzip cybersecurity-v2.skill -d ~/.claude/skills/cybersecurity
```

### Via marketplace (if registered)

```
/plugin install cybersecurity@cybersecurity-skill
```

---

## Usage examples

### Secure code generation

```
Generate a Spring Boot login endpoint with JWT
```

Claude will produce code with bcrypt (cost 12), RS256 JWT in httpOnly cookie, rate limiting, auth logging, and a Security Scorecard at the end.

### Security audit

```
Audit this Angular service that stores the auth token in localStorage
```

Claude will produce a finding with CWE-922, OWASP A07:2021, an XSS proof of concept, and a corrected implementation using httpOnly cookies.

### LLM security

```
Is my RAG pipeline with Claude vulnerable to prompt injection?
```

Claude loads `owasp-llm.md` and covers direct injection, indirect injection via poisoned documents, mitigation patterns, and context isolation code.

### Bug bounty report

```
I found an IDOR on /api/invoices/{id}. Help me write the HackerOne report.
```

Claude produces a structured report with CVSS vector, steps to reproduce, curl proof of concept, and business impact.

### Compliance

```
Evaluate our application against ISO 27001:2022 controls A.8
```

Claude loads `iso27001.md` and produces a compliance report with per-control status, non-conformities, and a prioritized remediation plan.

---

## Operational modes

| Mode | Trigger phrases | Behavior |
|---|---|---|
| Secure generation | "generate an API", "create an endpoint" | Applies all 7 layers during code generation |
| Full audit | "audit my code", "security review" | Systematic 7-layer analysis with scorecard |
| PR review | "review this diff", "check this PR" | Focuses on regressions introduced by changes |
| Red team | "how would an attacker...", "bug bounty" | Attacker-perspective analysis with PoC |
| Compliance | "NIST", "ISO 27001", "CIS", "certification" | Loads relevant reference, produces compliance report |

---

## Security Scorecard format

Every audit and generation ends with a scorecard:

```
+------------------------------------------------------------------+
|                       SECURITY SCORECARD                         |
+------------------------------------------------------------------+
|  Layer 1 - Static analysis            [ X/10 ]                   |
|  Layer 2 - Dependencies               [ X/10 ]                   |
|  Layer 3 - Secrets                    [ X/10 ]                   |
|  Layer 4 - Infrastructure             [ X/10 ]                   |
|  Layer 5 - Auth / Authz               [ X/10 ]                   |
|  Layer 6 - Business logic             [ X/10 ]                   |
|  Layer 7 - Observability              [ X/10 ]                   |
+------------------------------------------------------------------+
|  GLOBAL SCORE: XX/100 - [RISK LEVEL]                             |
+------------------------------------------------------------------+
|  CRITICAL  - block production deployment                         |
|  HIGH      - fix within 30 days                                  |
|  MEDIUM    - fix within 60 days                                  |
+------------------------------------------------------------------+
|  RESIDUAL RISKS:                                                  |
|  [Always ends here - never concludes "application is secure"]    |
+------------------------------------------------------------------+
```

---

## Design principles

**The skill never says "this is secure."** It says: "Here are the identified residual risks, their probability, and their impact. The decision is yours."

Security is a continuous process, not a final state. Every output reflects this by ending with an explicit list of what remains exposed after identified issues are addressed.

---

## Limitations

- No dynamic execution: cannot test a running application, timing attacks, or live race conditions
- No real infrastructure access: cannot scan ports, verify production certificates, or test network configurations
- Knowledge cutoff applies to CVE references: use a live scanner (Snyk, Dependabot) for current vulnerability data
- No physical security coverage: ISO 27001 theme A.7 (physical controls) is out of scope
- Business logic coverage depends on context provided: the more you describe your domain rules, the more accurate the analysis

---

## Contributing

Pull requests are welcome. Priority areas for contribution:

- PCI-DSS reference file (`references/pci-dss.md`)
- HIPAA reference file (`references/hipaa.md`)
- GraphQL-specific security patterns (extend `owasp-web.md`)
- Mobile security patterns (OWASP Mobile Top 10)
- Additional language-specific examples (Go, Rust, PHP)

To add a new framework: create a file in `references/`, add detection signals to the context table in `SKILL.md`, and update `marketplace.json` keywords.

---

## Author

Built by **Ousmane Diene** — Fullstack Engineer at Jasmine Conseil, Dakar, Senegal.  
Specialization: Java/Spring Boot, Angular, DevSecOps, customs administration systems (Douanes Senegalaises — GAINDE Modulaire platform).

---

## License

MIT — see [LICENSE](LICENSE)
