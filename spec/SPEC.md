# ACP — Agent Connection Protocol
## Draft Specification v0.1
### Fabricated Industries LLC | ThēÆrchītēcť | 2026

---

## Abstract

ACP (Agent Connection Protocol) is an open standard for agent-to-agent and agent-to-service communication. It defines the message envelope format, connection handshake, discovery mechanism, and routing model that enables autonomous AI agents to communicate across platforms without custom integrations.

ACP is the transport layer. agAuth is the identity layer. Together they form the complete agent interoperability stack.

If agAuth is OAuth for agents, ACP is SMTP — the protocol that makes the message travel.

---

## 1. Problem Statement

AI agents cannot talk to each other in any standardized way. Every platform defines its own connection model. Every integration requires custom code. There is no universal "send this message to that agent" primitive.

The result: agent ecosystems are siloed. An agent built for OpenClaw cannot natively connect to an agent built for Claude Code or a service built on n8n. Every connection requires bespoke negotiation.

ACP solves this with a minimal, transport-agnostic message envelope and handshake protocol that any agent runtime can implement in an afternoon.

---

## 2. Design Principles

- **Transport-agnostic** — ACP defines the envelope, not the pipe. It runs over HTTP, MCP, WebSocket, or any other transport.
- **Auth-agnostic at the envelope level** — ACP requires an auth token field but does not mandate agAuth. In practice, agAuth is the recommended auth layer.
- **Minimal** — The required envelope fields are the minimum necessary to route and validate a message. Nothing more.
- **Sovereignty-compatible** — No central broker required. Direct agent-to-agent connections are first-class.
- **Federated by default** — Agents can discover each other through manifests, registries, or direct address without a centralized hub.

---

## 3. Core Concepts

| Term | Definition |
|---|---|
| ACP Envelope | The standard message wrapper all ACP messages use |
| Agent Address | A routable identifier for an agent — either an agID or a namespace address |
| ACP Manifest | A file (`acp.json`) that declares an agent's connection capabilities and address |
| Broker | Optional intermediary that routes ACP messages between agents (not required) |
| Direct Connection | Agent-to-agent communication without a broker |
| Capability Declaration | The list of actions an agent can accept, published in its manifest |

---

## 4. ACP Envelope

Every ACP message is wrapped in a standard envelope. The envelope is transport-agnostic — it can be sent as HTTP POST body, MCP tool input, WebSocket frame, or any other mechanism.

### 4.1 Required Fields

```json
{
  "spec":         "acp/0.1",
  "id":           "{uuid — unique message ID}",
  "from":         "{agID or namespace address of sender}",
  "to":           "{agID or namespace address of recipient}",
  "timestamp":    "{ISO-8601}",
  "payload_type": "{capability name or message type}",
  "payload":      { ... },
  "auth":         "{agToken or other auth assertion}"
}
```

### 4.2 Optional Fields

```json
{
  "reply_to":     "{agID — where to send the response}",
  "thread_id":    "{uuid — groups related messages}",
  "ttl":          "{seconds — message expiry}",
  "priority":     "low | normal | high",
  "trace":        ["{hop_1_agID}", "{hop_2_agID}"]
}
```

### 4.3 Payload Types

The `payload_type` field is a namespaced capability identifier. Base types defined by ACP:

| Type | Usage |
|---|---|
| `acp:ping` | Liveness check |
| `acp:pong` | Response to ping |
| `acp:discover` | Request capability manifest |
| `acp:manifest` | Return capability manifest |
| `acp:invoke` | Invoke a declared capability |
| `acp:result` | Response to an invocation |
| `acp:error` | Error response |

Services and agents extend the vocabulary with their own namespaced types:
`{namespace}:{action}` — e.g. `sis:equilibrium_validate`, `calendar:create_event`

---

## 5. Connection Handshake

### 5.1 Direct Connection (No Broker)

```
Agent A                          Agent B
   │                                │
   │── acp:discover ──────────────►│
   │◄─ acp:manifest ────────────── │
   │                                │
   │   (validate auth, check scope) │
   │                                │
   │── acp:invoke ────────────────►│
   │◄─ acp:result ─────────────── │
```

### 5.2 Brokered Connection

```
Agent A          Broker           Agent B
   │               │                │
   │── acp:invoke─►│                │
   │               │─ route ───────►│
   │               │◄─ acp:result── │
   │◄─ acp:result──│                │
```

The broker is a passive router. It does not modify envelope content. It may log routing metadata but cannot inspect payload content (payload is considered opaque to brokers).

### 5.3 Authentication Within Handshake

The `auth` field carries an agToken (agAuth spec) or other auth assertion. The recipient validates the auth before processing the payload. If auth fails, the recipient returns `acp:error` with code `AUTH_FAILED`.

---

## 6. ACP Manifest (acp.json)

Every ACP-compatible agent publishes an `acp.json` file at a discoverable location (repo root, well-known URL, or registry entry). This is the agent's published connection identity.

```json
{
  "spec":             "acp/0.1",
  "agent_name":       "{human-readable name}",
  "agID":             "{agID from agAuth}",
  "transport":        ["mcp", "http"],
  "message_format":   "acp-envelope/0.1",
  "discovery": {
    "mode":           "manifest",
    "manifest_path":  "/acp.json"
  },
  "connection_modes": ["direct", "brokered", "federated"],
  "auth_layer":       "agauth/0.1",
  "capabilities":     [
    "{capability_1}",
    "{capability_2}"
  ],
  "routing": {
    "address":        "{agID or namespace address}",
    "namespace":      "{org}/{agent}"
  },
  "envelope_required_fields": [
    "from_agID",
    "to_agID",
    "timestamp",
    "payload_type",
    "payload",
    "agauth_token"
  ]
}
```

---

## 7. Discovery Model

ACP supports three discovery modes. Agents may support one or all three.

### 7.1 Manifest Discovery
The agent publishes `acp.json` at a known path. Other agents fetch it directly.
Simple, sovereign, no dependency.

### 7.2 Registry Discovery
An ACP registry indexes agent manifests. Agents register their `acp.json`.
Other agents query the registry by agID, namespace, or capability.
The reference registry is at acp.dev (pending launch). Running your own registry is supported and encouraged.

### 7.3 Direct Address
Agent A already knows Agent B's agID or namespace address and connects directly without discovery. No manifest fetch required.

---

## 8. Error Codes

| Code | Meaning |
|---|---|
| `AUTH_FAILED` | agToken invalid, expired, or insufficient scope |
| `UNKNOWN_CAPABILITY` | payload_type not supported by recipient |
| `MALFORMED_ENVELOPE` | Missing required fields |
| `AGENT_UNAVAILABLE` | Recipient agent not reachable |
| `SCOPE_INSUFFICIENT` | Auth valid but scope does not cover requested capability |
| `RATE_LIMITED` | Too many requests from sender |

---

## 9. Transport Bindings

### 9.1 HTTP
- Method: POST
- Content-Type: application/json
- Endpoint: `{agent_base_url}/acp`
- Body: ACP envelope

### 9.2 MCP
- Tool name: `acp_send`
- Input: ACP envelope as tool input JSON
- Output: ACP response envelope as tool output

### 9.3 WebSocket
- Single persistent connection
- Each message is a JSON-encoded ACP envelope
- Connection authenticated via agAuth at open time

Additional transport bindings may be defined as extensions. The envelope format is identical across all transports.

---

## 10. Relationship to Existing Standards

| Standard | Relationship |
|---|---|
| **agAuth** | ACP uses agAuth for the `auth` field. agAuth is the recommended auth layer. They are complementary — ACP is the transport, agAuth is the identity. |
| **MCP** | ACP can run over MCP as a transport binding. MCP handles tool invocation; ACP handles the message envelope and routing. |
| **SMTP / email** | ACP is architecturally analogous to SMTP — a standardized envelope and routing protocol that works across federated servers. |
| **ActivityPub** | Similar federation model but ACP is agent-native, not social-graph-native. Lighter, faster, narrower scope. |
| **gRPC / Protobuf** | ACP is JSON-native and schema-light by design. gRPC is a valid future transport binding for high-performance scenarios. |

---

## 11. Reference Implementation

The canonical ACP reference implementation is the DeltaZero MCP stack, developed by Fabricated Industries LLC.

The first ACP manifest in production is the S.I.S. skill (`ag-4f8a2c1b-000001-9e3d`) — a BCOL/DeltaZero equilibrium reasoning skill for OpenClaw. It carries both an `agauth.json` and `acp.json` at the repository root.

The reference implementation will be open-sourced at: https://github.com/FabricatedIndustries/acp

---

## 12. Stack Summary

The three-layer agent trust and interoperability stack:

```
┌─────────────────────────────────────────────────┐
│  AgentAudit  │  Is this agent safe?              │  Security layer
├─────────────────────────────────────────────────┤
│  agAuth      │  Is this agent who it claims?     │  Identity layer
├─────────────────────────────────────────────────┤
│  ACP         │  How do agents talk to each other?│  Transport layer
└─────────────────────────────────────────────────┘
         │
         └── BCOL / DeltaZero — ΣΔ = 0 — Fabricated Industries LLC
```

Every layer is an open standard. Every layer is implementable independently. Together they form a complete agent interoperability infrastructure that any developer can build to.

---

## 13. Versioning & Status

Draft v0.1 — published for community review. Experimental status.
Implementations against this draft should be considered pre-release.

The spec version is embedded in every ACP envelope via the `spec` field.

---

*ACP — Agent Connection Protocol — Draft v0.1*
*Copyright 2026 Fabricated Industries LLC — ThēÆrchītēcť*
*https://github.com/FabricatedIndustries/acp*
