# A Governed Zero Trust AI Gateway

A governed, self-hosted AI access pattern, **aligned to modern AppSec and SDLC practices**, for private inference, strong identity, scoped knowledge access, and clean offboarding.

## Overview

This repository documents **A Governed Zero Trust AI Gateway**, a self-hosted AI reference architecture designed to expose private AI services to approved users without placing those services directly on the public internet.

The architecture combines:

- Tailscale WireGuard mesh access
- Nginx Proxy Manager with `auth_request`
- Authelia MFA
- OpenWebUI for AI chat and RAG access
- Ollama for local inference
- macOS `pf` firewall restriction of the Ollama API
- AdGuard Home with Quad9 DNS-over-HTTPS
- Role-scoped knowledge base access
- Admin-mediated provisioning and four-step offboarding

The goal is simple:

> Private AI. Strong identity. Scoped knowledge. Auditable access. Clean offboarding.

## Published Artifacts

| Document | Purpose |
|---|---|
| [A Governed Zero Trust AI Gateway Architecture](./A%20Governed%20Zero%20Trust%20AI%20Gateway_Architecture.pdf) | End-to-end architecture diagram showing the client path, Tailscale overlay, Umbrel services, Authelia MFA, OpenWebUI, Ollama inference, AdGuard DNS, and host firewall enforcement. |
| [A Governed Zero Trust AI Gateway Executive 1-Pager](./A%20Governed%20Zero%20Trust%20AI%20Gateway_Executive_1Pager.pdf) | Executive summary of the problem, what was built, Zero Trust controls, identity provisioning model, known gaps, and control outcome. |
| [RAG Pipeline STRIDE Data Flow Diagram](./DFD_RAG_Pipeline.pdf) | Level 1 data flow diagram showing the governed RAG path, trust boundary, role-scoped KB retrieval, conversation logging, Ollama inference, and STRIDE threat labels. |

## Problem

Most AI deployments trade governance for speed.

Common weaknesses include shared API keys, no per-user identity, weak auditability, no offboarding procedure, and unclear control over what data the model can access.

Self-hosted AI solves part of the sovereignty problem, but it can introduce new risks if inference endpoints, knowledge bases, and user sessions are not governed.

Capability without governance is not a security posture.

## Architecture Summary

Primary OpenWebUI access path:

```text
Client
→ Tailscale WireGuard overlay
→ Nginx Proxy Manager
→ Authelia MFA
→ OpenWebUI
→ Ollama local inference
```

DNS path:

```text
Device
→ Tailscale DNS policy
→ AdGuard Home
→ Quad9 DoH
```

Inference path:

```text
OpenWebUI on Umbrel
→ MacBook Air M4
→ Ollama API on port 11434
```

The Ollama inference API is restricted by host firewall so only localhost and the Umbrel server can reach it.

## Core Components

### Tailscale

Tailscale provides private network access using a WireGuard-based overlay. Devices must be authorized before they can reach services. Tailscale attempts peer-to-peer connectivity where possible, with encrypted DERP relay fallback when NAT traversal fails.

### Nginx Proxy Manager

Nginx Proxy Manager serves as the reverse proxy and TLS termination layer. The critical control is Nginx `auth_request`, which forces an authorization check with Authelia before traffic reaches OpenWebUI. This turns NPM from a basic reverse proxy into the gateway enforcement point for the protected AI interface.

### Authelia

Authelia provides MFA enforcement before AI access. The implementation supports WebAuthn, Touch ID, and TOTP.

### OpenWebUI

OpenWebUI provides the AI chat interface, user-scoped access, model routing, and role-scoped RAG knowledge base access.

### Ollama

Ollama runs locally and provides private inference. The API is restricted with macOS `pf` firewall rules so only localhost and Umbrel can connect.

### AdGuard Home

AdGuard Home provides DNS filtering for tailnet-connected devices. Allowed DNS queries are forwarded upstream to Quad9 using DNS-over-HTTPS.

## Security Controls

| Control Area | Implementation |
|---|---|
| Device trust | Tailscale device authorization |
| User identity | Authelia MFA |
| Network privacy | WireGuard overlay |
| TLS termination | Nginx Proxy Manager with Tailscale-issued certificate |
| AI access enforcement | NPM `auth_request` before OpenWebUI |
| RAG access control | OpenWebUI role-scoped KB collections |
| Inference API protection | macOS `pf` firewall restricting Ollama to localhost and Umbrel |
| DNS filtering | AdGuard Home with Quad9 DoH |
| Offboarding | Four-step revocation across Tailscale, Authelia, OpenWebUI, and KB access scope |

## Offboarding Model

Offboarding is a defined four-step process:

1. Revoke Tailscale access or node share.
2. Disable the Authelia account.
3. Remove OpenWebUI access.
4. Remove KB collection scope or access.

This prevents lingering access through device trust, identity, application access, or knowledge base permissions.

## AI Security Validation

This Zero Trust AI gateway includes AI-specific validation beyond traditional application testing.

Tested areas include:

- Prompt injection
- RAG poisoning
- Instruction hierarchy failure
- Unauthorized disclosure attempts
- KB scope validation
- AIBOM / knowledge base recovery validation

This is not traditional SAST or DAST. It is dynamic AI security validation focused on model behavior, retrieval boundaries, and governance of the AI reasoning layer.

This work forms part of the CloudSentinel Solutions reference methodology for governed AI access and AI security validation.

## Known Gaps

This is a working reference architecture, not a finished enterprise platform.

Known gaps include:

- Ollama inference depends on MacBook availability.
- Tailscale certificate lifecycle still requires manual renewal handling.
- Dynamic policy decisioning is not yet implemented.
- Formal STRIDE data flow diagrams and attack trees are still in progress.
- WireGuard uses Curve25519 for handshake/key exchange, which is not post-quantum safe.

These gaps are documented intentionally. The architecture is designed to be defensible, not overstated.

## Roadmap

Planned improvements include:

- Move Authelia to an always-on host.
- Add a break-glass administrative procedure.
- Automate Tailscale certificate renewal and NPM import.
- Formalize onboarding and offboarding workflows.
- Expand STRIDE modeling beyond the RAG pipeline into additional service paths.
- Improve access log retention and evidence capture.
- Evaluate Open Policy Agent or similar PDP capability.
- Productize the pattern into a reusable CloudSentinel reference blueprint.

## What This Is Not

This repository is not:

- A compliance certification package
- A production high-availability platform
- A replacement for enterprise IAM
- A complete threat model
- A claim of zero residual risk

It is a documented, working security architecture pattern for governed self-hosted AI access.

## Purpose

This architecture demonstrates that private AI can be exposed to approved users while preserving identity, segmentation, scoped knowledge, local inference, and controlled offboarding.

## Author

Adam McKinski  
CloudSentinel Solutions

## License / Use

© 2026 Adam McKinski / CloudSentinel Solutions. All rights reserved.

This repository is published for portfolio, reference, and educational review purposes. No license is granted to copy, modify, redistribute, or commercialize the documents or architecture materials without written permission.
