---
name: wopr-p2p
version: 1.0.0
description: Comprehensive P2P networking for WOPR using Hyperswarm, identity management, topic-based discovery, and secure peer communication
category: network
tags:
  - p2p
  - hyperswarm
  - networking
  - identity
  - encryption
  - discovery
  - security
  - a2a
triggers:
  - p2p
  - peer
  - hyperswarm
  - discovery
  - invite
  - peer-to-peer
  - network
  - swarm
  - topic
  - identity
requires:
  - wopr
  - wopr-security
---

# WOPR P2P Plugin

The P2P plugin provides decentralized peer-to-peer networking for WOPR using Hyperswarm DHT. It enables secure communication between WOPR nodes without central servers, using Ed25519/X25519 cryptography for identity and encryption.

---

## Installation

```bash
# Install the P2P plugin
wopr plugin install wopr-plugin-p2p

# Enable the plugin
wopr plugin enable p2p

# Verify installation
wopr p2p status
```

The plugin automatically creates a P2P identity on first use, stored at `~/.wopr/p2p/identity.json`.

---

## Core Concepts

### Identity

Every WOPR node has a unique cryptographic identity:

| Key Type | Algorithm | Purpose |
|----------|-----------|---------|
| Signing Key | Ed25519 | Authenticating messages |
| Encryption Key | X25519 | Key exchange for E2E encryption |
| Ephemeral Key | X25519 | Forward secrecy per session |

Identity files are stored with `0600` permissions in `~/.wopr/p2p/`.

### Hyperswarm

The plugin uses Hyperswarm DHT for peer discovery and connection:

- **Topics**: SHA-256 hashes used for peer discovery
- **DHT**: Distributed hash table for finding peers
- **Connections**: Direct encrypted streams between peers

### Trust Model

Peers are categorized by trust level:

| Level | Description | Capabilities |
|-------|-------------|--------------|
| Untrusted | Discovered via DHT | Gateway access only |
| Semi-trusted | Claimed invite | Limited sessions |
| Trusted | Explicit grant | Full access to specified sessions |

---

## CLI Commands

### Identity Management

```bash
# Show your P2P identity
wopr p2p status

# Rotate keys (scheduled, compromise, or upgrade)
wopr p2p rotate --reason scheduled

# Export public key for sharing
wopr p2p identity --export
```

### Peer Management

```bash
# List all known peers
wopr p2p peers

# Give a peer a friendly name
wopr p2p name <peer-id> "alice"

# Find peer by name or ID
wopr p2p find alice

# Revoke peer access
wopr p2p revoke alice
```

### Invites and Claims

```bash
# Create an invite for a specific peer
wopr p2p invite create <their-pubkey> --sessions "session1,session2"

# Create invite with expiration
wopr p2p invite create <their-pubkey> --sessions "*" --expire 24h

# Claim an invite token
wopr p2p invite claim "wop1://..."
```

### Access Grants

```bash
# Grant access to specific sessions
wopr p2p grant <peer-pubkey> session1 session2

# Grant with trust level
wopr p2p grant <peer-pubkey> session1 --trust trusted

# Grant with specific capabilities
wopr p2p grant <peer-pubkey> session1 --capabilities "inject,inject.tools"

# List all grants
wopr p2p grants

# List grants for specific peer
wopr p2p grants --peer alice
```

### Discovery

```bash
# Join a discovery topic
wopr p2p topic join myproject

# Leave a topic
wopr p2p topic leave myproject

# List active topics
wopr p2p topics

# View discovered peers
wopr p2p discover

# View discovered peers in a specific topic
wopr p2p discover --topic myproject

# Request connection with discovered peer
wopr p2p connect <peer-id>
```

### Messaging

Two modes of P2P messaging:

```bash
# LOG MODE: Fire-and-forget (message stored in peer's session history)
wopr p2p log alice --session main --message "FYI: deployment complete"

# INJECT MODE: Invoke peer's AI and get response
wopr p2p inject alice --session main --message "What's the status of the build?"

# With custom timeout (inject can take longer due to AI processing)
wopr p2p inject alice --session main --message "Analyze this code" --timeout 300s
```

**When to use which:**
- `log` - Notifications, status updates, fire-and-forget messages
- `inject` - Questions, tasks, anything needing an AI response

---

## A2A Tools

The plugin registers tools for agent-to-agent communication:

### Identity Tools

| Tool | Description |
|------|-------------|
| `p2p_get_identity` | Get or create P2P identity |
| `p2p_rotate_keys` | Rotate identity keys |

### Peer Tools

| Tool | Description |
|------|-------------|
| `p2p_list_peers` | List all known peers |
| `p2p_name_peer` | Assign friendly name |
| `p2p_revoke_peer` | Revoke peer access |

### Invite Tools

| Tool | Description |
|------|-------------|
| `p2p_create_invite` | Create invite token |
| `p2p_claim_invite` | Claim invite from peer |

### Access Tools

| Tool | Description |
|------|-------------|
| `p2p_grant_access` | Manually grant access |
| `p2p_list_grants` | List all access grants |

### Discovery Tools

| Tool | Description |
|------|-------------|
| `p2p_join_topic` | Join discovery topic |
| `p2p_leave_topic` | Leave discovery topic |
| `p2p_list_topics` | List active topics |
| `p2p_discover_peers` | List discovered peers |
| `p2p_connect_peer` | Request connection |
| `p2p_get_profile` | Get discovery profile |
| `p2p_set_profile` | Update discovery profile |

### Messaging Tools

| Tool | Description |
|------|-------------|
| `p2p_log_message` | Log message to peer's session (fire-and-forget, no AI response) |
| `p2p_inject_message` | Inject message and invoke peer's AI (get response back) |
| `p2p_status` | Get P2P network status |

### Session History (Mirror Access)

| Tool | Description |
|------|-------------|
| `sessions_history` | Read session history with pagination. Use `full=true` for complete untruncated mirror |

---

## Security Scenarios

### Scenario 1: Trusted Peer Setup

For peers you fully trust (e.g., your own devices):

```bash
# On Node A: Get your public key
wopr p2p status
# Note: publicKey: MCowBQYDK2VwAy...

# On Node B: Get their public key
wopr p2p status
# Note: publicKey: MCowBQYDK2VwAy...

# On Node A: Create invite for Node B
wopr p2p invite create <nodeB-pubkey> --sessions "*"
# Returns: wop1://eyJ2Ijo...

# On Node B: Claim the invite (Node A must be online)
wopr p2p invite claim "wop1://eyJ2Ijo..."
# Result: Connected with access to all sessions

# On Node A: Give friendly name
wopr p2p name <nodeB-shortId> "my-laptop"
```

**Security Properties:**
- Wildcard session access (`*`)
- Full `inject` capability
- Mutual trust established

### Scenario 2: Semi-Trusted Peer (Limited Access)

For collaborators who should only access specific sessions:

```bash
# Create invite with limited sessions
wopr p2p invite create <peer-pubkey> --sessions "shared-project,docs" --expire 72h

# After they claim, verify grants
wopr p2p grants --peer <peer-id>
# Confirm: sessions=["shared-project", "docs"]
```

**Security Properties:**
- Access limited to specified sessions
- Token expires after 72 hours
- No access to other sessions

### Scenario 3: Untrusted Discovery (Gateway Pattern)

For unknown peers discovered via topics:

```bash
# Join a public topic
wopr p2p topic join public-agents

# Discovered peers are NOT auto-accepted
# They must go through a gateway session

# Configure gateway session (in WOPR config)
# gateway:
#   enabled: true
#   session: "public-gateway"
#   capabilities: ["inject.limited"]

# Grant discovered peer gateway-only access
wopr p2p grant <discovered-peer-key> public-gateway --capabilities "inject.limited"
```

**Security Properties:**
- Auto-accept DISABLED by default
- Discovered peers require explicit grants
- Gateway session filters/validates requests
- Untrusted peers cannot access privileged sessions

### Scenario 4: Emergency Key Rotation

If you suspect key compromise:

```bash
# Immediately rotate keys with compromise reason
wopr p2p rotate --reason compromise --notify-all

# This:
# 1. Generates new Ed25519 + X25519 keypairs
# 2. Signs rotation message with OLD key (proves continuity)
# 3. Notifies all known peers of the rotation
# 4. Old key valid for 24-hour grace period only
```

---

## Key Rotation

### Automatic Rotation

The plugin supports key rotation to limit exposure from compromised keys.

```bash
# Scheduled rotation (recommended quarterly)
wopr p2p rotate --reason scheduled

# After security incident
wopr p2p rotate --reason compromise

# After software upgrade
wopr p2p rotate --reason upgrade
```

### Grace Period

When keys are rotated, old keys remain valid for a grace period:

| Reason | Grace Period |
|--------|--------------|
| scheduled | 24 hours |
| upgrade | 24 hours |
| compromise | 24 hours (reduced from 7 days) |

The grace period allows in-flight messages to complete while minimizing exposure window.

### Key History

Peers maintain key history for continuity:

```json
{
  "peerKey": "MCowBQ...",
  "keyHistory": [
    {
      "publicKey": "MC0wBQ...",
      "encryptPub": "MCwwBQ...",
      "validFrom": 1704067200000,
      "validUntil": 1704153600000,
      "rotationReason": "scheduled"
    }
  ]
}
```

---

## Rate Limiting

The plugin enforces per-peer rate limits to prevent abuse:

| Operation | Limit | Ban Duration |
|-----------|-------|--------------|
| Injects | 10/minute, 100/hour | 1 hour |
| Claims | 5/minute, 20/hour | 1 hour |
| Invalid Messages | 3/minute, 10/hour | 2 hours |

```bash
# Check if a peer is rate limited
wopr p2p status --peer <peer-id>
```

When rate limited, peers receive a `reject` message with reason `"rate limited"`.

---

## Replay Protection

Messages include nonces and timestamps to prevent replay attacks:

```typescript
{
  nonce: "a1b2c3d4e5f6...",  // Random 16-byte hex
  ts: 1704067200000,         // Timestamp
  sig: "MEUCIQDn..."         // Ed25519 signature
}
```

**Protection Window:** 5 minutes
**Max Tracked Nonces:** 10,000

Messages older than 5 minutes or with previously-seen nonces are rejected.

---

## Payload Limits

To prevent memory exhaustion attacks:

| Limit | Value | Description |
|-------|-------|-------------|
| `MAX_PAYLOAD_SIZE` | 1 MB | Maximum encrypted payload |
| `MAX_MESSAGE_SIZE` | ~1 MB + 4 KB | Payload + protocol overhead |

Oversized messages are rejected before parsing.

---

## Forward Secrecy

Protocol v2 uses ephemeral X25519 keypairs for each session:

```
Session Start:
  1. Each peer generates ephemeral X25519 keypair
  2. Ephemeral public keys exchanged in handshake
  3. Shared secret derived via ECDH
  4. Messages encrypted with ephemeral-derived key

Security Benefit:
  If long-term key compromised later,
  past messages remain secure
```

Ephemeral keys have configurable TTL (default: 1 hour).

---

## Encryption

### Message Encryption (AES-256-GCM)

```
+------------------------+
|  IV (12 bytes)         |
+------------------------+
|  Auth Tag (16 bytes)   |
+------------------------+
|  Encrypted Data        |
+------------------------+
```

### Key Derivation

```
Static Keys (Legacy):
  SharedSecret = X25519(myPriv, theirPub)
  Key = SHA256(SharedSecret)

Ephemeral Keys (Forward Secrecy):
  SharedSecret = X25519(ephemeralPriv, theirEphemeralPub)
  Key = SHA256(SharedSecret)
```

---

## Monitoring Peer Connections

### View Connection Status

```bash
# Overall P2P status
wopr p2p status

# Detailed peer info
wopr p2p peers --verbose

# Watch for events
wopr p2p events --follow
```

### Security Events

The plugin emits events for monitoring:

| Event | Description |
|-------|-------------|
| `p2p.peer.connected` | Peer connection established |
| `p2p.peer.rejected` | Peer connection rejected |
| `p2p.message.rejected` | Message rejected (rate limit, size, auth) |
| `p2p.grant.created` | Access grant created |
| `p2p.grant.revoked` | Access grant revoked |
| `p2p.key.rotated` | Peer key rotation processed |

### Log Monitoring

```bash
# View P2P logs
wopr logs --service p2p

# Filter security events
wopr logs --service p2p --filter "rejected\|revoked\|rotated"
```

---

## Revoking Access

### Revoke by Peer

```bash
# Revoke all access for a peer
wopr p2p revoke alice

# Revoke by public key
wopr p2p revoke <peer-pubkey>
```

### Revoke by Grant

```bash
# List grants to find the grant ID
wopr p2p grants

# Revoke specific grant
wopr p2p grant revoke <grant-id>
```

### Revoke Session Access

```bash
# Remove access to specific session
wopr p2p revoke alice --session sensitive-session

# Peer keeps access to other sessions
```

After revocation, the peer receives `reject` messages with reason `"unauthorized"`.

---

## Configuration

### Plugin Configuration

```json
{
  "plugins": {
    "data": {
      "p2p": {
        "uiPort": 7334,
        "discoveryTrust": "untrusted",
        "autoAccept": false,
        "keyRotationGraceHours": 24,
        "maxPayloadSize": 1048576,
        "rateLimit": {
          "injects": { "maxPerMinute": 10, "maxPerHour": 100 },
          "claims": { "maxPerMinute": 5, "maxPerHour": 20 }
        }
      }
    }
  }
}
```

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `WOPR_P2P_AUTO_ACCEPT` | Auto-accept discovered peers | `false` |
| `WOPR_P2P_DISCOVERY_TRUST` | Trust level for discovery | `untrusted` |
| `WOPR_P2P_MAX_PAYLOAD` | Max payload size (bytes) | `1048576` |
| `WOPR_P2P_KEY_GRACE_HOURS` | Key rotation grace period | `24` |

---

## Best Practices

### Identity Security

1. **Protect identity files**: Stored at `~/.wopr/p2p/identity.json` with `0600` permissions
2. **Regular key rotation**: Rotate keys quarterly or after personnel changes
3. **Immediate rotation on compromise**: Use `--reason compromise` if keys may be exposed

### Access Control

1. **Never auto-accept discovery peers**: Keep `autoAccept: false` (default)
2. **Use specific session grants**: Avoid wildcard (`*`) for untrusted peers
3. **Implement gateway sessions**: Route untrusted traffic through validation layer
4. **Audit grants regularly**: `wopr p2p grants` to review active access

### Network Security

1. **Use topic namespacing**: Include unique identifiers in topic names
2. **Monitor rate limit events**: Watch for abuse patterns
3. **Set appropriate timeouts**: Shorter timeouts for untrusted peers

### Invite Token Security

1. **Short expiration times**: Use `--expire 24h` for sensitive access
2. **Single-use invites**: Create new tokens for each peer
3. **Secure token transmission**: Share tokens via encrypted channels

### Monitoring

1. **Enable event logging**: Monitor `p2p.*.rejected` events
2. **Watch for patterns**: Multiple rejections may indicate attack
3. **Regular status checks**: Verify expected peers are connected

---

## Troubleshooting

### Peer Not Found

```bash
# Verify peer is in your peer list
wopr p2p peers

# Search by any identifier
wopr p2p find alice
wopr p2p find <short-id>
wopr p2p find <public-key>
```

### Claim Failed

```bash
# Common causes:
# 1. Issuer offline - they must be running WOPR
# 2. Token expired - check expiration
# 3. Token for wrong peer - check 'sub' field matches your pubkey

# Debug token
echo "wop1://..." | base64 -d | jq
```

### Connection Timeout

```bash
# Increase timeout for slow networks
wopr p2p send alice --session main --message "test" --timeout 30s

# Check if peer is online
wopr p2p discover --topic myproject
```

### Rate Limited

```bash
# Wait for rate limit to expire (1-2 hours)
# Or check peer status
wopr p2p status --peer <peer-id>
```

### Unauthorized

```bash
# Check your grants to the peer
wopr p2p grants --peer alice

# Verify session access
# Grants must include the target session or "*"
```

---

## Protocol Reference

### Message Types

| Type | Purpose |
|------|---------|
| `hello` | Initial handshake with version negotiation |
| `hello-ack` | Handshake acknowledgment |
| `inject` | Send encrypted payload to session |
| `ack` | Message accepted |
| `reject` | Message rejected (with reason) |
| `claim` | Claim invite token |
| `key-rotation` | Notify of key rotation |

### Protocol Versions

| Version | Features |
|---------|----------|
| 1 | Basic messaging, static key encryption |
| 2 | Forward secrecy, ephemeral keys, version negotiation |

### Exit Codes

| Code | Constant | Meaning |
|------|----------|---------|
| 0 | `EXIT_OK` | Success |
| 1 | `EXIT_OFFLINE` | Peer offline |
| 2 | `EXIT_REJECTED` | Request rejected |
| 3 | `EXIT_INVALID` | Invalid request |
| 4 | `EXIT_RATE_LIMITED` | Rate limited |
| 5 | `EXIT_VERSION_MISMATCH` | Protocol version mismatch |
| 6 | `EXIT_PEER_OFFLINE` | Specific peer offline |
| 7 | `EXIT_UNAUTHORIZED` | Not authorized |

---

## Session Mirror

Sessions are **permanent records** - they never compact or truncate. The AI's context window may summarize, but the session file (`*.conversation.jsonl`) contains every message forever.

### Accessing the Full Mirror

```bash
# Quick summary (truncated, last 10 messages)
sessions_history(session="my-session")

# Full mirror with pagination
sessions_history(session="my-session", full=true)              # First 100 messages
sessions_history(session="my-session", full=true, offset=100)  # Next 100
sessions_history(session="my-session", full=true, limit=50)    # 50 per page
```

**Response format (full mode):**
```json
{
  "session": "my-session",
  "total": 847,
  "offset": 0,
  "pageSize": 100,
  "returned": 100,
  "hasMore": true,
  "nextOffset": 100,
  "history": [
    {
      "ts": 1234567890,
      "iso": "2024-01-15T12:00:00.000Z",
      "from": "user",
      "type": "message",
      "content": "Complete untruncated content...",
      "channel": { "type": "p2p", "id": "..." }
    }
  ]
}
```

**Key insight:** Even when the AI's working memory compacts, the session file remains complete. Use `sessions_history(full=true)` to access the full record.

---

## Data Storage

All P2P data stored in `~/.wopr/p2p/`:

| File | Permissions | Content |
|------|-------------|---------|
| `identity.json` | `0600` | Private keys, public keys |
| `access.json` | `0600` | Access grants |
| `peers.json` | `0600` | Known peers |

---

## Related Documentation

- [WOPR Security Model](/skills/wopr-security/SKILL.md)
- [WOPR Core](/skills/wopr/SKILL.md)
- [Hyperswarm Documentation](https://github.com/hyperswarm/hyperswarm)
