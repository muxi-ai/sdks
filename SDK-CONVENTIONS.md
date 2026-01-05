# MUXI SDK Cross-Language Conventions

**Version:** 1.0  
**Last Updated:** 2025-01-05  
**Status:** Design Document

This document defines conventions that ALL MUXI SDKs must follow to ensure consistency across languages while remaining idiomatic.

---

## Client Names

| Component | Canonical Name |
|-----------|----------------|
| Server management client | `ServerClient` |
| Formation usage client | `FormationClient` |

**Language Adaptations:**
- Python: `ServerClient`, `FormationClient` (classes)
- TypeScript: `ServerClient`, `FormationClient` (classes)
- Go: `ServerClient`, `FormationClient` (structs)
- Rust: `ServerClient`, `FormationClient` (structs)

---

## Method Names

All methods follow this canonical naming. Adapt casing to language conventions.

### ServerClient Methods

| Canonical | Go | Python | TypeScript |
|-----------|-----|--------|------------|
| `DeployFormation` | `DeployFormation` | `deploy_formation` | `deployFormation` |
| `DeployFormationStreaming` | `DeployFormationStreaming` | `deploy_formation_streaming` | `deployFormationStreaming` |
| `ListFormations` | `ListFormations` | `list_formations` | `listFormations` |
| `GetFormation` | `GetFormation` | `get_formation` | `getFormation` |
| `UpdateFormation` | `UpdateFormation` | `update_formation` | `updateFormation` |
| `StopFormation` | `StopFormation` | `stop_formation` | `stopFormation` |
| `StartFormation` | `StartFormation` | `start_formation` | `startFormation` |
| `RestartFormation` | `RestartFormation` | `restart_formation` | `restartFormation` |
| `RollbackFormation` | `RollbackFormation` | `rollback_formation` | `rollbackFormation` |
| `DeleteFormation` | `DeleteFormation` | `delete_formation` | `deleteFormation` |
| `CancelUpdate` | `CancelUpdate` | `cancel_update` | `cancelUpdate` |
| `GetFormationLogs` | `GetFormationLogs` | `get_formation_logs` | `getFormationLogs` |
| `StreamFormationLogs` | `StreamFormationLogs` | `stream_formation_logs` | `streamFormationLogs` |
| `Status` | `Status` | `status` | `status` |
| `Health` | `Health` | `health` | `health` |
| `Ping` | `Ping` | `ping` | `ping` |
| `GetServerLogs` | `GetServerLogs` | `get_server_logs` | `getServerLogs` |

### FormationClient Methods

| Canonical | Go | Python | TypeScript |
|-----------|-----|--------|------------|
| `Health` | `Health` | `health` | `health` |
| `Chat` | `Chat` | `chat` | `chat` |
| `ChatStream` | `ChatStream` | `chat_stream` | `chatStream` |
| `AudioChat` | `AudioChat` | `audio_chat` | `audioChat` |
| `AudioChatStream` | `AudioChatStream` | `audio_chat_stream` | `audioChatStream` |
| `GetStatus` | `GetStatus` | `get_status` | `getStatus` |
| `GetConfig` | `GetConfig` | `get_config` | `getConfig` |
| `GetFormationInfo` | `GetFormationInfo` | `get_formation_info` | `getFormationInfo` |
| `GetAgents` | `GetAgents` | `get_agents` | `getAgents` |
| `GetAgent` | `GetAgent` | `get_agent` | `getAgent` |
| `GetSecrets` | `GetSecrets` | `get_secrets` | `getSecrets` |
| `GetSecret` | `GetSecret` | `get_secret` | `getSecret` |
| `SetSecret` | `SetSecret` | `set_secret` | `setSecret` |
| `DeleteSecret` | `DeleteSecret` | `delete_secret` | `deleteSecret` |
| `GetMCPServers` | `GetMCPServers` | `get_mcp_servers` | `getMcpServers` |
| `GetMCPServer` | `GetMCPServer` | `get_mcp_server` | `getMcpServer` |
| `GetMCPTools` | `GetMCPTools` | `get_mcp_tools` | `getMcpTools` |
| `GetMemoryConfig` | `GetMemoryConfig` | `get_memory_config` | `getMemoryConfig` |
| `GetMemories` | `GetMemories` | `get_memories` | `getMemories` |
| `AddMemory` | `AddMemory` | `add_memory` | `addMemory` |
| `DeleteMemory` | `DeleteMemory` | `delete_memory` | `deleteMemory` |
| `GetUserBuffer` | `GetUserBuffer` | `get_user_buffer` | `getUserBuffer` |
| `ClearUserBuffer` | `ClearUserBuffer` | `clear_user_buffer` | `clearUserBuffer` |
| `ClearSessionBuffer` | `ClearSessionBuffer` | `clear_session_buffer` | `clearSessionBuffer` |
| `GetSchedulerConfig` | `GetSchedulerConfig` | `get_scheduler_config` | `getSchedulerConfig` |
| `GetSchedulerJobs` | `GetSchedulerJobs` | `get_scheduler_jobs` | `getSchedulerJobs` |
| `GetSchedulerJob` | `GetSchedulerJob` | `get_scheduler_job` | `getSchedulerJob` |
| `CreateSchedulerJob` | `CreateSchedulerJob` | `create_scheduler_job` | `createSchedulerJob` |
| `DeleteSchedulerJob` | `DeleteSchedulerJob` | `delete_scheduler_job` | `deleteSchedulerJob` |
| `GetSessions` | `GetSessions` | `get_sessions` | `getSessions` |
| `GetSession` | `GetSession` | `get_session` | `getSession` |
| `GetSessionMessages` | `GetSessionMessages` | `get_session_messages` | `getSessionMessages` |
| `RestoreSession` | `RestoreSession` | `restore_session` | `restoreSession` |
| `GetRequests` | `GetRequests` | `get_requests` | `getRequests` |
| `GetRequestStatus` | `GetRequestStatus` | `get_request_status` | `getRequestStatus` |
| `CancelRequest` | `CancelRequest` | `cancel_request` | `cancelRequest` |
| `StreamRequest` | `StreamRequest` | `stream_request` | `streamRequest` |
| `GetUserIdentifiers` | `GetUserIdentifiers` | `get_user_identifiers` | `getUserIdentifiers` |
| `ResolveUser` | `ResolveUser` | `resolve_user` | `resolveUser` |
| `LinkUserIdentifier` | `LinkUserIdentifier` | `link_user_identifier` | `linkUserIdentifier` |
| `UnlinkUserIdentifier` | `UnlinkUserIdentifier` | `unlink_user_identifier` | `unlinkUserIdentifier` |
| `ListCredentials` | `ListCredentials` | `list_credentials` | `listCredentials` |
| `GetCredential` | `GetCredential` | `get_credential` | `getCredential` |
| `CreateCredential` | `CreateCredential` | `create_credential` | `createCredential` |
| `DeleteCredential` | `DeleteCredential` | `delete_credential` | `deleteCredential` |
| `GetTriggers` | `GetTriggers` | `get_triggers` | `getTriggers` |
| `GetTrigger` | `GetTrigger` | `get_trigger` | `getTrigger` |
| `FireTrigger` | `FireTrigger` | `fire_trigger` | `fireTrigger` |
| `GetSOPs` | `GetSOPs` | `get_sops` | `getSops` |
| `GetSOP` | `GetSOP` | `get_sop` | `getSop` |
| `StreamEvents` | `StreamEvents` | `stream_events` | `streamEvents` |
| `StreamLogs` | `StreamLogs` | `stream_logs` | `streamLogs` |
| `GetOverlordConfig` | `GetOverlordConfig` | `get_overlord_config` | `getOverlordConfig` |
| `GetOverlordPersona` | `GetOverlordPersona` | `get_overlord_persona` | `getOverlordPersona` |
| `GetLLMSettings` | `GetLLMSettings` | `get_llm_settings` | `getLlmSettings` |
| `GetAuditLog` | `GetAuditLog` | `get_audit_log` | `getAuditLog` |
| `ClearAuditLog` | `ClearAuditLog` | `clear_audit_log` | `clearAuditLog` |

---

## Error Types

All SDKs must implement these error types with consistent names and codes.

### Error Hierarchy

```
MuxiError (base)
├── AuthenticationError   (401)
├── AuthorizationError    (403)
├── NotFoundError         (404)
├── ConflictError         (409)
├── ValidationError       (422)
├── RateLimitError        (429)
├── ServerError           (5xx)
└── ConnectionError       (network)
```

### Error Codes (String Constants)

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `UNAUTHORIZED` | 401 | Invalid/missing credentials |
| `FORBIDDEN` | 403 | Valid creds, insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `CONFLICT` | 409 | Resource conflict (already exists) |
| `VALIDATION_ERROR` | 422 | Invalid request parameters |
| `RATE_LIMITED` | 429 | Rate limit exceeded |
| `SERVER_ERROR` | 5xx | Server-side error |
| `CONNECTION_ERROR` | - | Network/connection issue |

### Error Properties

All error types should expose:

| Property | Type | Description |
|----------|------|-------------|
| `code` | string | Error code from table above |
| `message` | string | Human-readable message |
| `status_code` | int | HTTP status code (if applicable) |
| `details` | map/dict | Additional error data |
| `retry_after` | int | Seconds to wait (RateLimitError only) |

---

## Streaming Patterns

### Chunk Types

All streaming responses use these chunk type strings:

| Type | Description |
|------|-------------|
| `text` | Text content chunk |
| `tool_call` | Tool invocation |
| `tool_result` | Tool execution result |
| `agent_handoff` | Agent delegation |
| `thinking` | Reasoning/thinking content |
| `error` | Error during stream |
| `done` | Stream complete |

### Language-Specific Patterns

| Language | Pattern | Example |
|----------|---------|---------|
| Go | `<-chan Chunk` | `for chunk := range stream.Events()` |
| Python (async) | `AsyncIterator` | `async for chunk in client.chat_stream()` |
| Python (sync) | `Iterator` | `for chunk in client.chat_stream()` |
| TypeScript | `AsyncIterable` | `for await (const chunk of client.chatStream())` |
| Rust | `Stream` | `while let Some(chunk) = stream.next().await` |
| Swift | `AsyncSequence` | `for await chunk in client.chatStream()` |
| Kotlin | `Flow` | `client.chatStream().collect { chunk -> }` |

---

## HTTP Headers

### Required Headers

| Header | Value | Description |
|--------|-------|-------------|
| `X-Muxi-SDK` | `{lang}/{version}` | SDK identifier |
| `Content-Type` | `application/json` | Request body type |

### Optional Headers

| Header | Value | Description |
|--------|-------|-------------|
| `X-Muxi-Client` | `{os}-{arch}/{runtime}` | Client environment info |
| `X-Muxi-Idempotency-Key` | UUID string | Auto-generated per request |

### Idempotency Key Behavior

- **Auto-generated**: SDKs generate a UUID for every request
- **Header**: `X-Muxi-Idempotency-Key`
- **No configuration**: Always on, no override/disable options

Server ignores until idempotency support is implemented. Zero client changes needed when server adds support.

### Header Format Examples

```
X-Muxi-SDK: go/1.0.0
X-Muxi-SDK: python/1.0.0
X-Muxi-SDK: typescript/1.0.0

X-Muxi-Client: darwin-arm64/go1.21.5
X-Muxi-Client: linux-amd64/python3.11
X-Muxi-Client: darwin-x64/node20.10.0
```

---

## Pagination

The API uses simple limit-based pagination.

### Request Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | int | varies | Maximum items to return |

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `has_more` | bool | Whether more results exist |
| `count` | int | Number of items returned (optional) |

### SDK Pattern

```go
// Go
sessions, err := client.GetSessions(ctx, userID, &muxi.ListOptions{Limit: 50})
if sessions.HasMore {
    // Fetch more using last item as reference
}
```

```python
# Python
sessions = client.get_sessions(user_id, limit=50)
if sessions.has_more:
    # Fetch more
```

**Note:** The API does not use cursor/offset pagination. For large result sets, filter by time range or use streaming endpoints.

---

## Timeouts

### Default Values

| Operation | Default | Description |
|-----------|---------|-------------|
| Regular requests | 30s | Standard HTTP timeout |
| Streaming | None | No timeout (infinite) |

### Configuration

```go
// Go - client-level
client := muxi.NewFormationClient(&muxi.FormationConfig{
    Timeout: 60 * time.Second,  // Override default
})

// Go - per-request via context
ctx, cancel := context.WithTimeout(ctx, 10*time.Second)
defer cancel()
client.GetAgents(ctx)
```

Streaming operations should not have a timeout - they run until complete or context is cancelled.

---

## Response Metadata

All responses include metadata that SDKs should expose:

### Metadata Fields

| Field | Type | Description |
|-------|------|-------------|
| `request_id` | string | Unique request identifier (for support/debugging) |
| `timestamp` | int64 | Server timestamp (Unix ms) |

### SDK Pattern

```go
// Go - metadata on response
resp, err := client.Chat(ctx, req)
fmt.Printf("Request ID: %s\n", resp.RequestID)
fmt.Printf("Timestamp: %d\n", resp.Timestamp)
```

```python
# Python
resp = client.chat(req)
print(f"Request ID: {resp.request_id}")
```

**Use case:** When reporting issues, users can provide the `request_id` for server-side log correlation.

---

## Configuration

All SDKs should support these environment variables:

| Variable | Description |
|----------|-------------|
| `MUXI_SERVER_URL` | Server URL for ServerClient |
| `MUXI_KEY_ID` | HMAC key ID for server auth |
| `MUXI_SECRET_KEY` | HMAC secret key for server auth |
| `MUXI_FORMATION_URL` | Direct formation URL |
| `MUXI_ADMIN_KEY` | Formation admin key |
| `MUXI_CLIENT_KEY` | Formation client key |
| `MUXI_DEBUG` | Enable debug logging (1/true) |

### Config Struct Fields

| Field | Type | Description |
|-------|------|-------------|
| `url` | string | Direct URL (FormationClient) |
| `server_url` | string | Server URL for proxy mode |
| `base_url` | string | Explicit base URL override |
| `key_id` | string | HMAC key ID |
| `secret_key` | string | HMAC secret |
| `admin_key` | string | Formation admin key |
| `client_key` | string | Formation client key |
| `formation_id` | string | Formation identifier |
| `timeout` | duration | Request timeout |
| `retry` | RetryConfig | Retry configuration |
| `debug` | bool | Enable debug logging |

---

## Request/Response Types

### Naming Convention

Request/response types use flat naming in package namespace:

```
ChatRequest       (not FormationClientChatRequest)
ChatResponse
DeployRequest
DeployResult
FormationList
Agent
Session
...
```

### File Attachment Structure

```json
{
  "filename": "document.pdf",
  "content": "base64-encoded-content",
  "content_type": "application/pdf",
  "size": 12345
}
```

---

## Retry Configuration

### v1: Keep It Simple

| Setting | Default | Description |
|---------|---------|-------------|
| `max_retries` | 0 | Max retry attempts (0 = disabled) |

```go
// Go
client := muxi.NewFormationClient(&muxi.FormationConfig{
    MaxRetries: 3,
})
```

### Retry Behavior (Hardcoded in v1)

- **Delay**: 500ms initial, doubles each retry, max 30s
- **Jitter**: ±10% to prevent thundering herd
- **Retryable codes**: 408, 429, 500, 502, 503, 504
- **429 handling**: Respect `Retry-After` header
- **Methods**: GET/DELETE always, POST never (too risky without idempotency)

Future versions may expose `RetryConfig` for fine-tuning if needed.

---

## Testing

### Build Tags/Markers

| Language | Unit Tests | Integration Tests |
|----------|------------|-------------------|
| Go | default | `//go:build integration` |
| Python | default | `@pytest.mark.integration` |
| TypeScript | `*.test.ts` | `*.integration.test.ts` |

### Test Server Configuration

Integration tests should read from environment:

```
MUXI_TEST_SERVER_URL
MUXI_TEST_KEY_ID
MUXI_TEST_SECRET_KEY
MUXI_TEST_FORMATION_URL
MUXI_TEST_CLIENT_KEY
```

---

## Package Structure

### Recommended Layout

```
muxi-{lang}/
├── client.{ext}           # Constructors, common logic
├── server_client.{ext}    # ServerClient
├── formation_client.{ext} # FormationClient
├── streaming.{ext}        # SSE parsing
├── auth.{ext}             # HMAC signing
├── models.{ext}           # Types/structs
├── errors.{ext}           # Error types
├── retry.{ext}            # Retry logic
├── examples/
│   ├── deploy/
│   ├── chat/
│   ├── streaming/
│   └── monitoring/
└── tests/
```

---

## Version Format

SDK versions follow semver: `MAJOR.MINOR.PATCH`

- `MAJOR`: Breaking API changes
- `MINOR`: New features, backward compatible
- `PATCH`: Bug fixes, backward compatible

---

## References

- [SDK-DESIGN.md](./SDK-DESIGN.md) - Overall philosophy
- [RELEASE_PLAN.md](./RELEASE_PLAN.md) - Launch schedule
- [go/DESIGN.md](./go/DESIGN.md) - Go-specific decisions
- [Server API Spec](../schemas/api/server-api-v1-final.yaml)
- [Formation API Spec](../schemas/api/formation-api-v1-final.yaml)
