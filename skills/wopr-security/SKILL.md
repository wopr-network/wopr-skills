---
name: wopr-security
description: WOPR security configuration reference covering trust levels, capabilities, sandbox isolation, gateway sessions, and multi-agent security best practices.
---

# WOPR Security Configuration

WOPR implements a three-layer security model: Trust Levels (who), Capabilities (what), and Sandbox (where). This skill covers security configuration and best practices.

## Trust Levels

Trust levels determine the base security context for any injection source.

| Level | Source | Default Behavior |
|-------|--------|------------------|
| `owner` | CLI, daemon, local | Full access to all capabilities |
| `trusted` | Explicit P2P grants, verified users | Scoped access per grant configuration |
| `semi-trusted` | Channel users, time-limited P2P | Limited tools, optional sandbox |
| `untrusted` | P2P discovery, unknown sources | Sandboxed, minimal tools, gateway only |

### Trust Level Hierarchy

```
owner > trusted > semi-trusted > untrusted
```

Higher trust levels can grant capabilities to lower levels. Lower trust levels cannot escalate their own privileges.

## Capabilities System

Capabilities control what actions a session or agent can perform.

### Available Capabilities

| Capability | Description |
|------------|-------------|
| `inject` | Send messages to sessions |
| `inject.tools` | Use MCP tools during execution |
| `inject.network` | Make HTTP requests (`http_fetch`) |
| `inject.exec` | Execute shell commands (`exec_command`) |
| `session.spawn` | Create new sessions |
| `cross.inject` | Inject messages into other sessions |
| `config.write` | Modify WOPR configuration |
| `a2a.call` | Use agent-to-agent communication tools |

### Capability Profiles by Trust Level

```bash
# Owner: Full access
owner: ["*"]

# Trusted: Scoped operational access
trusted: ["inject", "inject.tools", "session.spawn", "a2a.call"]

# Semi-trusted: Basic interaction only
semi-trusted: ["inject", "inject.tools"]

# Untrusted: Message injection only (no tool use)
untrusted: ["inject"]
```

### Granting Capabilities

```bash
# Grant specific capabilities to a peer
wopr security grant <peer-key> inject,inject.tools,a2a.call

# Use a preset profile
wopr security grant <peer-key> --profile trusted
wopr security grant <peer-key> --profile semi-trusted

# Revoke all access
wopr security revoke <peer-key>

# List current grants
wopr security list-grants
```

## Session Provider Security

WOPR supports per-session model selection, allowing different trust levels to use different AI providers.

### Per-Session Provider Configuration

```bash
# Set provider for a specific session
wopr session set-provider gateway-session openai --model gpt-4o

# Set provider with fallback chain
wopr session set-provider code-executor anthropic --model claude-sonnet-4-20250514 --fallback openai:gpt-4o

# Create session with specific provider
wopr session create sandbox-agent "You are a sandboxed assistant" --provider openai
```

### Provider Trust Considerations

| Trust Level | Recommended Provider Strategy |
|-------------|------------------------------|
| `owner` | Use premium models (Claude Opus, GPT-4) for complex tasks |
| `trusted` | Standard models with rate limiting |
| `semi-trusted` | Cost-efficient models, lower token limits |
| `untrusted` | Sandboxed sessions only, minimal model access |

## Docker Sandbox Isolation

Untrusted sources run in hardened Docker containers matching Clawdbot's security model.

### Sandbox Configuration

```bash
docker run \
  --read-only \                     # Read-only root filesystem
  --tmpfs /tmp --tmpfs /var/tmp \   # Ephemeral temp dirs
  --network none \                  # No network (prevents exfiltration)
  --cap-drop ALL \                  # Drop all Linux capabilities
  --security-opt no-new-privileges \  # Prevent privilege escalation
  --security-opt seccomp=wopr-seccomp.json \  # Syscall filtering
  --security-opt apparmor=wopr-sandbox \      # AppArmor profile
  --pids-limit 100 \                # Limit process count
  --memory 512m \                   # Memory limit
  --memory-swap 512m \              # No swap
  --cpus 0.5 \                      # CPU limit
  --ulimit nofile=1024:1024 \       # File descriptor limits
  -v /workspace:/workspace:ro \     # Session workspace read-only
  wopr-sandbox:latest
```

### Sandbox Architecture

```
+---------------------------------------------+
|  Docker Container (sandbox)                 |
|  +-------------------------------------+    |
|  |  claude-code                        |    |
|  |  - Can use Read, Write, Edit, Bash  |    |
|  |  - BUT filesystem is read-only      |    |
|  |  - AND network is blocked           |    |
|  +----------------+--------------------+    |
|                   | MCP over unix socket    |
+-------------------+-------------------------+
                    |
+-------------------v-------------------------+
|  WOPR Host (a2a-mcp server)                 |
|                                             |
|  Tool calls filtered by SecurityContext:    |
|  - sessions_send -> DENIED (no cross.inject)|
|  - http_fetch -> DENIED (no inject.network) |
|  - memory_read -> ALLOWED (filtered paths)  |
|  - security_whoami -> ALLOWED (always)      |
+---------------------------------------------+
```

### Key Sandbox Properties

- Claude-code runs inside container with restricted filesystem/network
- A2A tools are served by WOPR host (outside container)
- WOPR filters which A2A tools are available based on session's capabilities
- Even if claude-code tries `http_fetch`, WOPR denies it server-side
- Container isolation + capability filtering = defense in depth

## Gateway Session Model

Untrusted sources cannot directly inject into privileged sessions. They must go through a gateway.

### Gateway Flow

```
+-----------------+     +-----------------+     +-----------------+
|  P2P Peer       |---->|  Gateway        |---->|  Privileged     |
|  (untrusted)    |     |  (semi-trusted) |     |  (owner/trusted)|
+-----------------+     +-----------------+     +-----------------+
     inject              decides & forwards       executes
```

### Gateway Configuration

```json
{
  "security": {
    "gateway": {
      "gatewaySessions": ["p2p-gateway", "public-api"],
      "forwardRules": {
        "p2p-gateway": {
          "allowForwardTo": ["code-executor", "research-agent"],
          "allowActions": ["query", "search", "analyze"],
          "requireApproval": false,
          "rateLimit": { "perMinute": 10 }
        },
        "public-api": {
          "allowForwardTo": ["api-handler"],
          "allowActions": ["query"],
          "requireApproval": true,
          "rateLimit": { "perMinute": 5 }
        }
      }
    }
  }
}
```

### Gateway CLI Commands

```bash
# List gateway sessions
wopr security gateway list

# Designate a session as a gateway
wopr security gateway add <session>

# View forward rules for a gateway
wopr security gateway rules <session>

# View pending approval requests
wopr security gateway queue

# Approve a pending request
wopr security gateway approve <request-id>

# Deny a pending request
wopr security gateway deny <request-id>
```

### Gateway Responsibilities

1. **Receive** - Accept injections from untrusted sources
2. **Validate** - Check request against policy (allowed actions, rate limits)
3. **Transform** - Sanitize/restructure requests before forwarding
4. **Forward** - Inject into appropriate privileged session
5. **Respond** - Return results to original requester

## Plugin Security

Plugins must declare their security requirements.

### Plugin Security Declaration

```typescript
const plugin: WOPRPlugin = {
  name: "my-plugin",

  // Declare required capabilities
  requiredCapabilities: ["inject.network", "a2a.call"],

  // Declare what trust level plugin operates at
  trustLevel: "trusted",

  async init(ctx) {
    // Plugin's inject calls carry its trust level
    ctx.inject("session", "message"); // Uses plugin's trust context
  }
};
```

### Plugin Security Best Practices

1. **Minimal Capabilities**: Request only capabilities you need
2. **Trust Level Appropriateness**: Don't request `owner` unless absolutely necessary
3. **Input Validation**: Validate all external inputs before processing
4. **Error Handling**: Don't leak sensitive information in error messages
5. **Audit Logging**: Log security-relevant events

## P2P Security

P2P introduces additional security considerations. See `wopr-p2p` skill for detailed P2P security configuration.

### P2P Security Defaults

```json
{
  "security": {
    "p2p": {
      "discoveryTrust": "untrusted",
      "autoAccept": false,
      "keyRotationGracePeriod": "24h",
      "payloadSizeLimit": 1048576,
      "rateLimits": {
        "perSession": { "perMinute": 30 },
        "perPeer": { "perMinute": 100 }
      }
    }
  }
}
```

### Critical P2P Security Rules

1. **Discovery default**: All discovered peers are `untrusted`
2. **Auto-accept disabled**: Peers must be explicitly granted access
3. **Gateway routing**: Discovered peers can ONLY inject to gateway sessions
4. **Key rotation**: 24-hour grace period for rotated keys (not 7 days)
5. **Rate limiting**: Per-session and per-peer rate limits enforced

## Security Policy Configuration

### Full Configuration Example

```json
{
  "security": {
    "defaults": {
      "sandbox": { "enabled": false },
      "tools": { "deny": ["config.write"] }
    },
    "trustLevels": {
      "owner": {
        "capabilities": ["*"],
        "sandbox": { "enabled": false }
      },
      "trusted": {
        "capabilities": ["inject", "inject.tools", "session.spawn", "a2a.call"],
        "sandbox": { "enabled": false }
      },
      "semi-trusted": {
        "capabilities": ["inject", "inject.tools"],
        "sandbox": { "enabled": true, "network": "restricted" }
      },
      "untrusted": {
        "capabilities": ["inject"],
        "sandbox": { "enabled": true, "network": "none" },
        "tools": { "deny": ["*"] }
      }
    },
    "p2p": {
      "discoveryTrust": "untrusted",
      "autoAccept": false
    },
    "gateway": {
      "gatewaySessions": ["p2p-gateway"],
      "forwardRules": {
        "p2p-gateway": {
          "allowForwardTo": ["*"],
          "allowActions": ["*"],
          "requireApproval": false,
          "rateLimit": { "perMinute": 20 }
        }
      }
    },
    "enforcement": "enforce"
  }
}
```

### Enforcement Modes

| Mode | Behavior |
|------|----------|
| `warn` | Log security violations but allow execution (migration mode) |
| `enforce` | Block security violations (production mode) |

### CLI Security Commands

```bash
# View global security status
wopr security status

# View effective policy for a session
wopr security status <session>

# Show current user's trust level
wopr security whoami

# Configure policy for a session
wopr security policy set <session> --sandbox enabled
wopr security policy set <session> --tools.deny "exec_command,http_fetch"

# Configure trust level defaults
wopr security policy set --trust-level untrusted --sandbox enabled

# View security audit log
wopr security audit
wopr security audit --denied
wopr security audit --last 100
```

## A2A Security Tools

Agents can query their security context via A2A tools.

### Available Security Tools

```typescript
// Query own security context
"security_whoami"     // Returns trust level, capabilities, sandbox status

// Check if action is allowed before attempting
"security_check"      // Check specific capability

// Request privilege elevation (queued for owner approval)
"security_request"    // Request elevation

// Gateway-only tools
"gateway_forward"     // Forward request to privileged session
"gateway_respond"     // Send response back to requester
"gateway_queue"       // View pending requests
```

### Example: Checking Capabilities

```typescript
// Agent checks if it can make network requests
const result = await security_check({ capability: "inject.network" });
if (result.allowed) {
  // Safe to proceed with http_fetch
} else {
  // Find alternative approach
}
```

## Multi-Agent Security Best Practices

### 1. Principle of Least Privilege

Grant only the capabilities each agent needs.

```bash
# Research agent: needs network, no exec
wopr security grant researcher-agent inject,inject.tools,inject.network

# Code executor: needs exec, no network
wopr security grant code-agent inject,inject.tools,inject.exec

# Review agent: read-only, no tools
wopr security grant review-agent inject
```

### 2. Session Isolation

Use separate sessions for different trust levels.

```bash
# Create isolated sessions
wopr session create untrusted-handler "Handle untrusted requests"
wopr session create trusted-executor "Execute trusted operations"

# Set appropriate providers
wopr session set-provider untrusted-handler openai --model gpt-4o-mini
wopr session set-provider trusted-executor anthropic --model claude-sonnet-4-20250514
```

### 3. Gateway Pattern for External Input

Route all external input through gateway sessions.

```bash
# Set up gateway
wopr security gateway add external-gateway

# Configure forward rules
wopr config set security.gateway.forwardRules.external-gateway.allowForwardTo '["internal-processor"]'
wopr config set security.gateway.forwardRules.external-gateway.rateLimit.perMinute 10
```

### 4. Audit Everything

Enable comprehensive audit logging.

```bash
# View recent security events
wopr security audit

# Filter by action type
wopr security audit --denied
wopr security audit --capability inject.exec

# Export for analysis
wopr security audit --format json --output security-events.json
```

### 5. Regular Security Reviews

```bash
# List all grants
wopr security list-grants

# Review gateway rules
wopr security gateway rules --all

# Check for stale grants
wopr security audit --stale-grants
```

## Verification Checklist

Before enabling enforcement mode, verify:

- [ ] All sessions have appropriate trust levels assigned
- [ ] Gateway sessions are configured for external input
- [ ] P2P auto-accept is disabled
- [ ] Required capabilities are granted to trusted peers
- [ ] Sandbox is enabled for untrusted/semi-trusted
- [ ] Rate limits are configured
- [ ] Audit logging is enabled
- [ ] Forward rules are restrictive

## Migration from Permissive Mode

```bash
# Step 1: Enable warning mode
wopr config set security.enforcement warn

# Step 2: Monitor audit log for violations
wopr security audit --denied --since "1 week ago"

# Step 3: Fix violations by granting needed capabilities
wopr security grant <peer> <capabilities>

# Step 4: Enable enforcement
wopr config set security.enforcement enforce
```
