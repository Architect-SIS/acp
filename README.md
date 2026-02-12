# ACP — Agent Connection Protocol

**The transport layer for autonomous AI agents.**

[![Spec Version](https://img.shields.io/badge/spec-acp%2F0.1-1A1A2E?style=flat)](spec/SPEC.md)
[![Status](https://img.shields.io/badge/status-draft-yellow?style=flat)](spec/SPEC.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## What is ACP?

ACP (Agent Connection Protocol) is an open standard for agent-to-agent and agent-to-service communication.

AI agents cannot talk to each other in any standardized way. Every platform defines its own connection model. Every integration requires custom code. There is no universal "send this message to that agent" primitive.

**ACP solves this** with a minimal, transport-agnostic message envelope and handshake protocol that any agent runtime can implement in an afternoon.

```
No ACP:   Agent A ──(custom)──► Service X
          Agent A ──(custom)──► Agent B
          Agent A ──(custom)──► Service Y

With ACP: Agent A ──[acp envelope]──► anyone
```

---

## The Stack

ACP is one layer of a three-layer agent trust and interoperability stack:

```
AgentAudit  →  Is this agent safe?           (security)
agAuth      →  Is this agent who it claims?  (identity)
ACP         →  How do agents talk?           (transport)  ← this repo
```

| Layer | Repo |
|---|---|
| Security | [AgentAudit](https://agentaudit.dev) |
| Identity | [agauth](https://github.com/Architect-SIS/agauth) |
| Transport | **ACP** (this repo) |

---

## Core Concepts

| Concept | Description |
|---|---|
| **ACP Envelope** | Standard message wrapper all ACP messages use |
| **Agent Address** | Routable identifier — agID or namespace address |
| **ACP Manifest** | `acp.json` file declaring an agent's connection capabilities |
| **Capability** | A declared action an agent can accept and respond to |
| **Direct Connection** | Agent-to-agent without a broker |
| **Broker** | Optional intermediary router (not required) |

---

## Minimal Envelope

```json
{
  "spec":         "acp/0.1",
  "id":           "msg-uuid-here",
  "from":         "ag-4f8a2c1b-000001-9e3d",
  "to":           "ag-7b3c9d2e-000042-ff1a",
  "timestamp":    "2026-02-12T00:00:00Z",
  "payload_type": "acp:invoke",
  "payload":      { },
  "auth":         "agtoken_here"
}
```

---

## Specification

- [Full Specification →](spec/SPEC.md)
- [ACP Envelope Schema →](spec/envelope.schema.json)
- [ACP Manifest Schema →](spec/manifest.schema.json)
- [Payload Types →](spec/SPEC.md#43-payload-types)

---

## Reference Implementation

The canonical reference implementation is the **DeltaZero MCP stack** by Fabricated Industries LLC.

First ACP manifest in production: [`Architect-SIS/sis-skill`](https://github.com/Architect-SIS/sis-skill)
- agID: `ag-4f8a2c1b-000001-9e3d`
- Framework: BCOL / DeltaZero — ΣΔ = 0

---

## Implementations

| Package | agID | Transport | Status |
|---|---|---|---|
| [sis-skill](https://github.com/Architect-SIS/sis-skill) | `ag-4f8a2c1b-000001-9e3d` | MCP, HTTP | ✅ Genesis |

*Building an ACP implementation? Open a PR to add it here.*

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## License

MIT License — Copyright 2026 Fabricated Industries LLC (ThēÆrchītēcť)

---

*ACP — Agent Connection Protocol — Draft v0.1*
*https://github.com/Architect-SIS/acp*
