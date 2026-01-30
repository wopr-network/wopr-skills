---
name: wopr
description: WOPR CLI commands for session management, skills, plugins, configuration, and inter-agent communication.
---

# WOPR CLI

WOPR is an AI agent orchestration system. Use these commands to manage sessions, skills, plugins, and communicate with other agents.

## Session Management

```bash
# List all sessions
wopr session list

# Create a new session with optional context
wopr session new <name> [context]

# Delete a session
wopr session delete <name>

# View session details
wopr session show <name>

# View conversation history
wopr session history <name> [--limit N]
```

## Injecting Messages

```bash
# Send a message to a session and get a response
wopr inject <session> "your message here"

# Log a message without getting a response (for context)
wopr log <session> "message" [--from source]
```

## Skills

```bash
# List installed skills
wopr skill list

# Search registries for skills
wopr skill search <query>

# Install a skill from registry or URL
wopr skill install <source> [name]

# Create a new skill
wopr skill create <name> [description]

# Remove a skill
wopr skill remove <name>

# Clear skill cache
wopr skill cache clear

# Manage registries
wopr skill registry list
wopr skill registry add <name> <url>
wopr skill registry remove <name>
```

## Plugins

```bash
# List installed plugins
wopr plugin list

# Install a plugin
wopr plugin install <source>

# Remove a plugin
wopr plugin remove <name>

# Enable/disable a plugin
wopr plugin enable <name>
wopr plugin disable <name>

# Search for plugins
wopr plugin search <query>

# Manage plugin registries
wopr plugin registry list
wopr plugin registry add <name> <url>
wopr plugin registry remove <name>
```

## Configuration

```bash
# View all configuration
wopr config

# Get a specific config value
wopr config get <key>

# Set a config value
wopr config set <key> <value>

# Reset configuration to defaults
wopr config reset
```

## Cron Jobs (Scheduled Messages)

```bash
# List scheduled jobs
wopr cron list

# Add a cron job
wopr cron add <name> <schedule> <session> "message"
# Options: --now (run immediately), --once (run once then delete)

# Remove a cron job
wopr cron remove <name>
```

## Providers (AI Models)

```bash
# List configured providers
wopr provider list

# Add provider credentials
wopr provider add <provider-id> <api-key>

# Remove provider
wopr provider remove <provider-id>

# Check provider health
wopr provider health
```

## Daemon Management

```bash
# Start the daemon
wopr daemon start

# Stop the daemon
wopr daemon stop

# Check daemon status
wopr daemon status

# View daemon logs
wopr daemon logs [--follow]
```

## Cross-Session Communication

To communicate with another session:

```bash
# Inject a message into another session
wopr inject other-session "Hello from this session"
```

To spawn a new session for a subtask:

```bash
# Create session, inject task, get result
wopr session new subtask "You are a helper agent"
wopr inject subtask "Do this specific task"
wopr session delete subtask
```

## Examples

### Install and use a skill
```bash
wopr skill search git
wopr skill install github:tsavo/wopr-skills/skills/git-essentials
wopr skill list
```

### Create a scheduled reminder
```bash
wopr cron add daily-standup "0 9 * * *" main "Time for standup! What's on the agenda?"
```

### Multi-agent workflow
```bash
# Create a researcher agent
wopr session new researcher "You research topics thoroughly"
wopr inject researcher "Research the latest on quantum computing"

# Create a writer agent
wopr session new writer "You write clear summaries"
wopr inject writer "Summarize: [paste researcher output]"

# Cleanup
wopr session delete researcher
wopr session delete writer
```
