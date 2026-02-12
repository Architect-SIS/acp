# ACP Examples

## Minimal Envelope (ping)

```json
{
  "spec":         "acp/0.1",
  "id":           "550e8400-e29b-41d4-a716-446655440000",
  "from":         "ag-4f8a2c1b-000001-9e3d",
  "to":           "ag-7b3c9d2e-000042-ff1a",
  "timestamp":    "2026-02-12T00:00:00Z",
  "payload_type": "acp:ping",
  "payload":      {},
  "auth":         "agtoken_here"
}
```

## Capability Invocation

```json
{
  "spec":         "acp/0.1",
  "id":           "msg-uuid",
  "from":         "ag-4f8a2c1b-000001-9e3d",
  "to":           "fabricated.industries/sis",
  "timestamp":    "2026-02-12T00:00:00Z",
  "payload_type": "sis:equilibrium_validate",
  "payload":      { "input": "Analyze this decision...", "context": {} },
  "auth":         "agtoken_here",
  "reply_to":     "ag-4f8a2c1b-000001-9e3d",
  "thread_id":    "thread-uuid",
  "ttl":          30
}
```

## ACP Manifest (acp.json)

```json
{
  "spec":             "acp/0.1",
  "agent_name":       "S.I.S. - Sovereign Intelligence System",
  "agID":             "ag-4f8a2c1b-000001-9e3d",
  "transport":        ["mcp", "http"],
  "message_format":   "acp-envelope/0.1",
  "discovery": {
    "mode":           "manifest",
    "manifest_path":  "/acp.json"
  },
  "connection_modes": ["direct", "brokered", "federated"],
  "auth_layer":       "agauth/0.1",
  "capabilities":     [
    "acp:ping",
    "sis:equilibrium_validate",
    "sis:symbol_execute",
    "sis:state_validate"
  ],
  "routing": {
    "address":   "ag-4f8a2c1b-000001-9e3d",
    "namespace": "fabricated.industries/sis"
  }
}
```

## Error Response

```json
{
  "spec":         "acp/0.1",
  "id":           "response-uuid",
  "from":         "ag-7b3c9d2e-000042-ff1a",
  "to":           "ag-4f8a2c1b-000001-9e3d",
  "timestamp":    "2026-02-12T00:00:00Z",
  "payload_type": "acp:error",
  "payload": {
    "code":    "SCOPE_INSUFFICIENT",
    "message": "Token scope does not include sis:equilibrium_validate",
    "ref_id":  "msg-uuid"
  },
  "auth":    "agtoken_here"
}
```

## Base Payload Types

| Type | Usage |
|---|---|
| `acp:ping` | Liveness check |
| `acp:pong` | Response to ping |
| `acp:discover` | Request capability manifest |
| `acp:manifest` | Return capability manifest |
| `acp:invoke` | Invoke a declared capability |
| `acp:result` | Response to invocation |
| `acp:error` | Error response |

Custom types: `{namespace}:{action}` — e.g. `sis:equilibrium_validate`, `calendar:create_event`
