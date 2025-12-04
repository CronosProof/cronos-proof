# CronosProof

**Cryptographic Authorization for AI Agents**

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-2.1.0--OSS-green.svg)]()

---

## Why CronosProof Exists

Modern AI agents are powerful, but they aren't flawless. Even highly capable models can misinterpret instructions, act autonomously beyond their mandate, or be manipulated through prompt injection—leading to unexpected or dangerous outcomes.

Traditional AI governance relies on **detection-time guardrails**: systems that identify policy violations *after* execution has begun. This creates an unacceptable window for catastrophic failure in high-stakes domains.

**CronosProof eliminates this gap.** It enforces authorization *before* any action executes, using cryptographic proofs that cannot be bypassed, fooled, or ignored.

---

## What CronosProof Is

CronosProof is an **execution-layer authorization framework** for AI agents. It acts as cryptographically-enforced middleware between AI systems and the real world.

Unlike prompt-layer guardrails that monitor what an AI *says*, CronosProof controls what an AI *does*. Every action requiring real-world effect must pass through the authorization gate—no exceptions.

**Key Insight:** The AI never holds credentials. It can only *request* actions. A separate, hardened component decides whether to execute them.

---

## Core Principles

- **Zero-Trust by Default** — No action proceeds without verified authorization
- **Cryptographic Enforcement** — Ed25519 signatures bind approvals to specific actions
- **Capability Decoupling** — AI agents cannot directly access credentials or execute effects
- **Human-in-Final-Control** — High-risk actions require explicit human approval
- **Fail-Secure Architecture** — Ambiguous or unverified requests are blocked, never guessed

---

## Architecture: The Zero-Trust Triangle

CronosProof implements a three-component architecture where no single component can act alone:

```
┌─────────────────┐
│  Untrusted      │  AI agent requests actions
│  Client (AI)    │  Has NO credentials or direct access
└────────┬────────┘
         │ Request
         ▼
┌─────────────────┐
│  Governor       │  Policy engine (OPA/Rego)
│                 │  Validates tokens, enforces constraints
│                 │  Triggers Suspension Protocol if needed
└────────┬────────┘
         │ Authorized Action
         ▼
┌─────────────────┐
│  Smart Proxy    │  Actuator with real credentials
│                 │  Executes ONLY Governor-approved actions
│                 │  Logs everything to audit trail
└─────────────────┘
```

**The AI never touches the Smart Proxy directly.** All requests flow through the Governor, which enforces policy before any effect occurs.

---

## How It Works

1. **Action Request** — AI agent submits a structured action request to the Governor
2. **Policy Evaluation** — Governor evaluates the request against OPA/Rego policies
3. **Risk Assessment** — High-risk actions trigger the **Suspension Protocol** (HTTP 202), pausing execution until human approval via authenticated channel (e.g., biometric confirmation on mobile device)
4. **Token Issuance** — Approved actions receive a signed authorization token (Ed25519)
5. **Execution** — Smart Proxy validates the token and executes the action
6. **Audit Logging** — Every decision and action is logged to a tamper-evident audit trail

---

## The Suspension Protocol

When an action exceeds policy thresholds (e.g., financial amount, risk tier, sensitive operation), the Governor doesn't approve or deny—it **suspends**:

```
[14:32:07] | finance-bot    | POST /stripe/charges $15,000  | SUSPENDED (202)
>>> HIGH RISK DETECTED. TRIGGERING SUSPENSION PROTOCOL.
>>> Sending Push Notification to Admin Device...
>>> Waiting for Biometric Signature...
>>> SIGNATURE VERIFIED (admin@company.com). RESUMING EXECUTION.
[14:32:45] | finance-bot    | POST /stripe/charges $15,000  | APPROVED
```

This is not a prompt asking "are you sure?"—it's a cryptographic gate that requires authenticated human intent before proceeding.

---

## Key Concepts

### The Fortress (Available Now)
The open-source, self-hosted authorization framework. Download the blueprints, deploy locally or in your cloud, and integrate with your AI agents. This is the production-ready Digital v2.1 release.

### The Warden (Waitlist)
A proposed cognitive integrity layer that verifies *how* the AI arrived at its decision, not just *what* it decided. Two variants are under development:

- **WARDEN-LITE** — Activation monitoring with trained classifiers (practical, near-term)
- **WARDEN-FULL** — Zero-knowledge proofs of neural state compliance (aspirational, research-stage)

The Warden addresses a gap CronosProof alone cannot fill: detecting when an AI has been compromised or manipulated at the cognitive level, even if its *requested action* appears legitimate.

### Digital Cage
Network isolation ensuring the AI agent can only communicate through the Governor. Implemented via eBPF/XDP traffic enforcement—the AI literally cannot reach external services directly.

### Audit Trail
Tamper-evident logging of every authorization decision, token issuance, and action execution. Supports compliance requirements and forensic analysis.

---

## Use Cases

| Domain | Problem Solved |
|--------|----------------|
| **Financial Systems** | Prevent unauthorized transactions, enforce approval thresholds, audit every trade |
| **Critical Infrastructure** | Block autonomous AI decisions on power grids, water systems, industrial controls |
| **Defense & Aerospace** | Require human authorization for high-consequence actions; prevent autonomous escalation |
| **Enterprise AI Governance** | Centralized policy enforcement across AI agents; complete audit trail for compliance |
| **Healthcare & Medical Devices** | Ensure human oversight for diagnostic or treatment recommendations |

---

## Why CronosProof Matters

**The problem with detection-time guardrails:**
```
AI decides → AI acts → Guardrail detects violation → Damage done
```

**The CronosProof approach:**
```
AI requests → Governor evaluates → Human approves (if needed) → Action executes
```

By the time traditional guardrails detect a problem, the irreversible action may have already occurred. CronosProof ensures unauthorized actions are **impossible by construction**—not merely unlikely or detectable.

---

## Repository Structure

```
cronosproof/
├── docs/
│   ├── architecture/        # System diagrams and protocol specifications
│   ├── api/                 # API reference and SDK documentation
│   ├── deployment/          # Installation and configuration guides
│   └── security/            # Threat model and cryptographic details
├── specs/
│   ├── digital-v2.1/        # Current open-source specification
│   └── research/            # Post-quantum and formal verification track
├── examples/
│   ├── quickstart/          # Minimal working example
│   ├── finance/             # Financial services integration
│   └── enterprise/          # Multi-agent governance patterns
├── policies/
│   └── rego/                # Example OPA/Rego policy bundles
└── tools/
    └── demo/                # Interactive demonstration console
```

---

## Technical Stack

- **Policy Engine:** Open Policy Agent (OPA) with Rego
- **Cryptography:** Ed25519 signatures (Digital track), Dilithium3/Kyber768 (Research track)
- **Transport:** mTLS with SPIFFE identity
- **Network Enforcement:** eBPF/XDP (Digital Cage)
- **Audit Storage:** PostgreSQL with row-level security
- **Deployment:** Kubernetes-native with Helm charts

---

## Roadmap

### Phase 1 — The Fortress (Available Now)
Open-source specification and reference architecture for cryptographic AI authorization. Engineers can download specs and integrate into existing systems.

### Phase 2 — The Warden (In Development)
Cognitive integrity verification layer. WARDEN-LITE (activation monitoring) targeted for 2026; WARDEN-FULL (ZK proofs) in research phase.

### Phase 3 — Enterprise & Automation
Multi-platform deployment, real-time dashboards, autonomous agent fleet management, and advanced compliance features.

---

## Contributing

CronosProof is open-source, and contributions are welcome from developers, researchers, and security professionals.

**Ways to contribute:**

- Develop SDKs and API integrations
- Write documentation, guides, and tutorials
- Perform security audits and protocol validation
- Submit feature requests and improvements
- Share deployment experiences and use cases

**Contribution Guidelines:**

1. Fork the repository and create feature branches
2. Submit pull requests with clear descriptions and test coverage
3. Follow coding and documentation standards in `/docs/contributing.md`

---

## Documentation

- **[Architecture Specification](docs/architecture/)** — System design, component interactions, protocol flows
- **[API Reference](docs/api/)** — Endpoints, SDK usage, integration examples
- **[Deployment Guide](docs/deployment/)** — Installation, configuration, Kubernetes setup
- **[Security Model](docs/security/)** — Threat model, cryptographic guarantees, operating conditions
- **[Policy Examples](policies/rego/)** — Sample OPA/Rego policies for common use cases

---

## Links

- **Website:** [cronosproof.org](https://cronosproof.org)
- **Blueprints & Waitlist:** [cronosproof.org/get-started](https://cronosproof.org/get-started)
- **Technical Specification (PDF):** [Digital v2.1 Release](docs/CronosProof_Digital_v2.1.pdf)

---

## License

Apache 2.0 — See [LICENSE](LICENSE) for details.

---

## Contact

- **Technical Inquiries:** tech@cronosproof.org
- **Security Issues:** security@cronosproof.org (PGP key available)

---

*The future of AI safety is cryptographic authorization. The time to build it is now.*
