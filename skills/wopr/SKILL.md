---
name: wopr
description: Complete WOPR CLI reference for session management, skills, plugins, providers, middleware, and inter-agent communication.
---

# WOPR CLI Reference

WOPR is an AI agent orchestration system. This is the complete command reference.

## Setup & Configuration

```bash
wopr onboard                              # Interactive onboarding wizard
wopr configure                            # Re-run configuration wizard
```

## Authentication

```bash
wopr auth                                 # Show auth status
wopr auth login                           # Login with Claude Max/Pro (OAuth)
wopr auth api-key <key>                   # Use API key instead
wopr auth logout                          # Clear credentials
```

## Sessions

```bash
wopr session list                         # List all sessions
wopr session create <name> [context]      # Create session with optional context
wopr session create <name> --provider <id> [--fallback chain]  # Create with provider
wopr session show <name> [--limit N]      # Show details and conversation history
wopr session delete <name>                # Delete a session
wopr session set-provider <name> <id> [--model name] [--fallback chain]  # Update provider
wopr session init-docs <name>             # Initialize SOUL.md, AGENTS.md, USER.md
wopr session inject <name> <message>      # Inject message, get AI response
wopr session log <name> <message>         # Log message (no AI response)
```

## Skills

```bash
wopr skill list                           # List installed skills
wopr skill search <query>                 # Search registries for skills
wopr skill install <source> [name]        # Install from registry or URL
wopr skill create <name> [description]    # Create a new skill
wopr skill remove <name>                  # Remove a skill
wopr skill cache clear                    # Clear registry cache
wopr skill registry list                  # List configured registries
wopr skill registry add <name> <url>      # Add a skill registry
wopr skill registry remove <name>         # Remove a registry
```

## Plugins

```bash
wopr plugin list                          # List installed plugins
wopr plugin install <source>              # Install (npm pkg, github:u/r, ./local)
wopr plugin remove <name>                 # Remove a plugin
wopr plugin enable <name>                 # Enable a plugin
wopr plugin disable <name>                # Disable a plugin
wopr plugin search <query>                # Search npm for plugins
wopr plugin registry list                 # List plugin registries
wopr plugin registry add <name> <url>     # Add a plugin registry
wopr plugin registry remove <name>        # Remove a plugin registry
```

## Providers (AI Models)

```bash
wopr providers list                       # List all providers and status
wopr providers add <id> [credential]      # Add/update provider credential
wopr providers remove <id>                # Remove provider credential
wopr providers health-check               # Check health of all providers
wopr providers default <id> [options]     # Set global provider defaults
wopr providers show-defaults [id]         # Show global provider defaults
```

## Cron Jobs (Scheduled Messages)

```bash
wopr cron list                            # List scheduled jobs
wopr cron add <name> <sched> <sess> <msg> # Add cron [--now] [--once]
wopr cron once <time> <session> <message> # One-time job (now, +5m, +1h, 09:00)
wopr cron now <session> <message>         # Run immediately (no scheduling)
wopr cron remove <name>                   # Remove a cron job
```

### Cron History (A2A Tool)

Agents can view execution history using the `cron_history` tool:

```
cron_history                              # View recent executions
cron_history name="job-name"              # Filter by job name
cron_history session="target-session"     # Filter by target session
cron_history failedOnly=true              # Show only failures
cron_history successOnly=true             # Show only successes
cron_history since=1706745600000          # Filter by timestamp (ms)
cron_history limit=10 offset=20           # Pagination
```

Returns: timestamp, job name, session, status, duration, error (if failed), and full message.

## Configuration

```bash
wopr config get [key]                     # Show config (all or specific key)
wopr config set <key> <value>             # Set config value
wopr config list                          # List all config values
wopr config reset                         # Reset to defaults
```

## Middleware

```bash
wopr middleware list                      # List all middleware
wopr middleware chain                     # Show execution order
wopr middleware show <name>               # Show middleware details
wopr middleware enable <name>             # Enable middleware
wopr middleware disable <name>            # Disable middleware
wopr middleware priority <name> <n>       # Set middleware priority
```

## Context Providers

```bash
wopr context list                         # List all context providers
wopr context show <name>                  # Show context provider details
wopr context enable <name>                # Enable context provider
wopr context disable <name>               # Disable context provider
wopr context priority <name> <n>          # Set context provider priority
```

## Daemon

```bash
wopr daemon start                         # Start the daemon
wopr daemon stop                          # Stop the daemon
wopr daemon status                        # Check if daemon is running
wopr daemon logs                          # Show daemon logs
```

## Common Workflows

### Send a message to another session
```bash
wopr session inject other-session "Hello from this session"
```

### Spawn a helper agent for a subtask
```bash
wopr session create helper "You are a research assistant"
wopr session inject helper "Research quantum computing advances in 2025"
# ... use the response ...
wopr session delete helper
```

### Schedule a daily check-in
```bash
wopr cron add daily-standup "0 9 * * *" main "Good morning! What's the plan for today?"
```

### Set up a one-time reminder
```bash
wopr cron once +30m main "Reminder: Check on the build status"
```

### Install skills from registry
```bash
wopr skill registry add wopr github:TSavo/wopr-skills/skills
wopr skill search git
wopr skill install github:TSavo/wopr-skills/skills/git-essentials
```

### Switch AI model for a session
```bash
wopr session set-provider my-session anthropic --model claude-sonnet-4-5-20250929
```

## A2A Tools Reference

These tools are available to agents within WOPR sessions:

### Session Management
- `sessions_list` - List all sessions
- `sessions_spawn` - Create a new session (requires session.spawn capability)
- `sessions_send` - Send message to another session (requires cross.inject capability)
- `sessions_terminate` - Terminate a session

### Cron/Scheduling
- `cron_schedule` - Schedule a recurring cron job
- `cron_once` - Schedule a one-time job
- `cron_list` - List all scheduled cron jobs
- `cron_cancel` - Cancel a scheduled cron job
- `cron_history` - View cron execution history with filtering and pagination

### Memory
- `memory_read` - Read from persistent memory
- `memory_write` - Write to persistent memory
- `memory_list` - List memory keys
- `memory_delete` - Delete memory entries

### Events
- `event_emit` - Emit a custom event
- `event_subscribe` - Subscribe to events
- `event_unsubscribe` - Unsubscribe from events

### Network
- `http_fetch` - Make HTTP requests (requires inject.network capability)

### Execution
- `exec_command` - Execute shell commands (requires inject.exec capability)

### Utility
- `security_whoami` - Show current trust level and capabilities
