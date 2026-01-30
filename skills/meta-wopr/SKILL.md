---
name: meta-wopr
description: Installation, configuration, and deployment guide for WOPR - the AI agent orchestration system.
---

# WOPR Installation & Configuration Guide

This skill covers installing, configuring, and deploying WOPR from scratch.

## Prerequisites

### System Requirements

- **Node.js**: Version 20.0.0 or higher (LTS recommended)
- **npm**: Version 9.0.0 or higher (comes with Node.js)
- **Operating System**: Linux, macOS, or Windows with WSL2
- **Memory**: Minimum 512MB RAM available
- **Disk Space**: ~100MB for installation, additional space for sessions/plugins

### Verify Prerequisites

```bash
node --version    # Should be v20.0.0 or higher
npm --version     # Should be 9.0.0 or higher
```

### Install Node.js (if needed)

**Using nvm (recommended):**
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc  # or ~/.zshrc
nvm install 20
nvm use 20
```

**Using package manager:**
```bash
# Ubuntu/Debian
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs

# macOS with Homebrew
brew install node@20

# Windows (use installer from nodejs.org)
```

## Installation

### Global Installation (Recommended)

```bash
npm install -g @tsavo/wopr
```

### Verify Installation

```bash
wopr --version
wopr --help
```

### Update WOPR

```bash
npm update -g @tsavo/wopr
```

### Uninstall WOPR

```bash
npm uninstall -g @tsavo/wopr
rm -rf ~/.wopr  # Remove all data (optional)
```

## First-Time Setup

### Interactive Onboarding

Run the onboarding wizard for guided setup:

```bash
wopr onboard
```

The wizard will:
1. Create the `~/.wopr/` directory structure
2. Prompt for authentication method
3. Configure default AI provider
4. Set up initial session
5. Install recommended skills

### Manual Setup

If you prefer manual setup:

```bash
# Create directory structure
mkdir -p ~/.wopr/{sessions,skills,plugins,config,logs,cache}

# Initialize configuration
wopr configure
```

## Configuration Wizard

Run the configuration wizard anytime to update settings:

```bash
wopr configure
```

### Configuration Options

The wizard covers:

1. **Authentication**: OAuth login or API key
2. **Default Provider**: Which AI model to use by default
3. **Session Defaults**: Default context and behavior
4. **Plugin Settings**: Auto-enable plugins
5. **Daemon Settings**: Auto-start, logging level

### Re-run Specific Sections

```bash
wopr configure --section auth        # Just authentication
wopr configure --section providers   # Just providers
wopr configure --section daemon      # Just daemon settings
```

## Authentication Options

### Option 1: OAuth Login (Claude Max/Pro)

For users with Claude Max or Pro subscriptions:

```bash
wopr auth login
```

This opens a browser for OAuth authentication with Anthropic. After successful login, credentials are stored securely in `~/.wopr/config/credentials.json`.

### Option 2: API Key

For direct API access:

```bash
# Interactive (masked input)
wopr auth api-key

# Direct (not recommended for security)
wopr auth api-key sk-ant-api03-xxxxx
```

### Check Authentication Status

```bash
wopr auth              # Show current auth status
wopr auth --verbose    # Show detailed auth info
```

### Switch Authentication Methods

```bash
wopr auth logout       # Clear current credentials
wopr auth login        # Re-authenticate with OAuth
# or
wopr auth api-key      # Switch to API key
```

### Multiple Providers

Configure additional AI providers:

```bash
wopr providers add openai sk-xxxxx
wopr providers add google AIzaSyxxxxx
wopr providers add anthropic sk-ant-xxxxx
```

## Daemon Setup and Management

The daemon runs in the background to handle cron jobs, webhooks, and inter-session communication.

### Start the Daemon

```bash
wopr daemon start
```

### Daemon Commands

```bash
wopr daemon status     # Check if daemon is running
wopr daemon stop       # Stop the daemon
wopr daemon restart    # Restart the daemon
wopr daemon logs       # View daemon logs
wopr daemon logs -f    # Follow logs in real-time
```

### Auto-Start on Boot

**Linux (systemd):**
```bash
# Create service file
sudo tee /etc/systemd/system/wopr.service << 'EOF'
[Unit]
Description=WOPR AI Agent Daemon
After=network.target

[Service]
Type=simple
User=$USER
ExecStart=/usr/bin/wopr daemon start --foreground
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# Enable and start
sudo systemctl daemon-reload
sudo systemctl enable wopr
sudo systemctl start wopr
```

**macOS (launchd):**
```bash
# Create plist file
cat > ~/Library/LaunchAgents/com.wopr.daemon.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.wopr.daemon</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/bin/wopr</string>
        <string>daemon</string>
        <string>start</string>
        <string>--foreground</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>
EOF

# Load the service
launchctl load ~/Library/LaunchAgents/com.wopr.daemon.plist
```

## Directory Structure

WOPR stores all data in `~/.wopr/`:

```
~/.wopr/
├── config/
│   ├── config.json         # Main configuration
│   ├── credentials.json    # Authentication credentials (encrypted)
│   └── providers.json      # AI provider configurations
├── sessions/
│   └── <session-name>/
│       ├── session.json    # Session metadata
│       ├── history.json    # Conversation history
│       ├── SOUL.md         # Agent personality
│       ├── AGENTS.md       # Agent definitions
│       └── USER.md         # User context
├── skills/
│   └── <skill-name>/
│       └── SKILL.md        # Skill definition
├── plugins/
│   └── <plugin-name>/      # Installed plugins
├── logs/
│   ├── daemon.log          # Daemon logs
│   ├── error.log           # Error logs
│   └── audit.log           # Audit trail
├── cache/
│   ├── registries/         # Cached registry data
│   └── responses/          # Response cache
└── data/
    ├── cron.json           # Scheduled jobs
    └── middleware.json     # Middleware config
```

### Backup Your Data

```bash
# Full backup
tar -czf wopr-backup-$(date +%Y%m%d).tar.gz ~/.wopr

# Sessions only
tar -czf wopr-sessions-$(date +%Y%m%d).tar.gz ~/.wopr/sessions

# Restore from backup
tar -xzf wopr-backup-20250130.tar.gz -C ~/
```

## Environment Variables

Configure WOPR behavior via environment variables:

```bash
# Core settings
export WOPR_HOME=~/.wopr                    # Data directory (default: ~/.wopr)
export WOPR_CONFIG=~/.wopr/config           # Config directory
export WOPR_LOG_LEVEL=info                  # Logging: debug, info, warn, error

# Authentication
export ANTHROPIC_API_KEY=sk-ant-xxxxx       # Anthropic API key
export OPENAI_API_KEY=sk-xxxxx              # OpenAI API key
export GOOGLE_API_KEY=AIzaSyxxxxx           # Google AI API key

# Daemon settings
export WOPR_DAEMON_PORT=7420                # Daemon port (default: 7420)
export WOPR_DAEMON_HOST=127.0.0.1           # Daemon host (default: localhost)

# Performance
export WOPR_MAX_CONTEXT=100000              # Max context tokens
export WOPR_CACHE_TTL=3600                  # Cache TTL in seconds
export WOPR_TIMEOUT=120000                  # Request timeout in ms

# Development
export WOPR_DEBUG=true                      # Enable debug mode
export WOPR_TRACE=true                      # Enable trace logging
```

### Persist Environment Variables

Add to your shell profile (`~/.bashrc`, `~/.zshrc`, etc.):

```bash
# WOPR Configuration
export ANTHROPIC_API_KEY="sk-ant-xxxxx"
export WOPR_LOG_LEVEL="info"
```

## Docker Deployment

### Using Pre-built Image

```bash
# Pull the image
docker pull tsavo/wopr:latest

# Run with mounted config
docker run -d \
  --name wopr \
  -v ~/.wopr:/root/.wopr \
  -e ANTHROPIC_API_KEY=$ANTHROPIC_API_KEY \
  -p 7420:7420 \
  tsavo/wopr:latest

# Check status
docker logs wopr
```

### Docker Compose

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  wopr:
    image: tsavo/wopr:latest
    container_name: wopr
    restart: unless-stopped
    volumes:
      - ~/.wopr:/root/.wopr
      - /var/run/docker.sock:/var/run/docker.sock  # Optional: for Docker-in-Docker
    environment:
      - ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
      - WOPR_LOG_LEVEL=info
      - WOPR_DAEMON_HOST=0.0.0.0
    ports:
      - "7420:7420"
    healthcheck:
      test: ["CMD", "wopr", "daemon", "status"]
      interval: 30s
      timeout: 10s
      retries: 3
```

Run with:
```bash
docker-compose up -d
docker-compose logs -f
```

### Build from Source

```dockerfile
# Dockerfile
FROM node:20-alpine

RUN npm install -g @tsavo/wopr

WORKDIR /root

ENTRYPOINT ["wopr"]
CMD ["daemon", "start", "--foreground"]
```

```bash
docker build -t wopr:local .
docker run -d --name wopr -v ~/.wopr:/root/.wopr wopr:local
```

## Troubleshooting

### Common Issues

#### "command not found: wopr"

The npm global bin directory is not in your PATH.

```bash
# Find npm global bin
npm config get prefix

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH="$(npm config get prefix)/bin:$PATH"

# Reload shell
source ~/.bashrc
```

#### "EACCES permission denied"

Fix npm permissions:

```bash
# Option 1: Use nvm (recommended)
nvm install 20
nvm use 20

# Option 2: Fix npm prefix
mkdir ~/.npm-global
npm config set prefix '~/.npm-global'
export PATH=~/.npm-global/bin:$PATH
npm install -g @tsavo/wopr
```

#### "Error: No API key configured"

Configure authentication:

```bash
wopr auth login           # For OAuth
# or
wopr auth api-key         # For API key
```

#### "Daemon failed to start"

Check for port conflicts:

```bash
# Check if port is in use
lsof -i :7420

# Use different port
export WOPR_DAEMON_PORT=7421
wopr daemon start
```

#### "Session not found"

List available sessions:

```bash
wopr session list
```

#### "Plugin installation failed"

Clear cache and retry:

```bash
wopr plugin cache clear
npm cache clean --force
wopr plugin install <plugin-name>
```

#### "Out of memory"

Increase Node.js memory:

```bash
export NODE_OPTIONS="--max-old-space-size=4096"
wopr daemon start
```

### Debug Mode

Enable verbose logging for troubleshooting:

```bash
export WOPR_DEBUG=true
export WOPR_LOG_LEVEL=debug
wopr daemon start
```

### Check System Health

```bash
wopr providers health-check    # Check AI provider connectivity
wopr daemon status             # Check daemon status
wopr config list               # Verify configuration
```

### View Logs

```bash
wopr daemon logs               # Recent logs
wopr daemon logs -f            # Follow logs
cat ~/.wopr/logs/error.log     # Error log
cat ~/.wopr/logs/daemon.log    # Full daemon log
```

### Reset Configuration

If configuration is corrupted:

```bash
# Backup first
cp -r ~/.wopr ~/.wopr.backup

# Reset config only
rm ~/.wopr/config/*.json
wopr configure

# Full reset (loses all data)
rm -rf ~/.wopr
wopr onboard
```

### Get Help

```bash
wopr --help                    # General help
wopr <command> --help          # Command-specific help
wopr doctor                    # Run diagnostics (if available)
```

## Quick Start Checklist

1. [ ] Install Node.js 20+
2. [ ] Run `npm install -g @tsavo/wopr`
3. [ ] Run `wopr onboard`
4. [ ] Configure authentication (`wopr auth login` or `wopr auth api-key`)
5. [ ] Start daemon (`wopr daemon start`)
6. [ ] Create first session (`wopr session create main "You are a helpful assistant"`)
7. [ ] Test with a message (`wopr session inject main "Hello!"`)

## Next Steps

After installation:

- Install skills: `wopr skill search` and `wopr skill install`
- Configure providers: `wopr providers add`
- Set up scheduled tasks: `wopr cron add`
- Explore plugins: `wopr plugin search`

See the `wopr` skill for complete CLI reference.
