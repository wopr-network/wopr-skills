# WOPR Skills Registry

Official skill registry for [WOPR](https://github.com/TSavo/wopr).

## Usage

Add this registry:
```bash
wopr skill registry add wopr github:TSavo/wopr-skills/skills
```

Search for skills:
```bash
wopr skill search <query>
```

Install a skill:
```bash
wopr skill install github:TSavo/wopr-skills/skills/<skill-name>
```

## Available Skills

| Skill | Description |
|-------|-------------|
| `wopr` | Complete WOPR CLI reference - all commands for sessions, skills, plugins, providers, middleware |
| `meta-wopr` | Installation, configuration, daemon setup, Docker deployment, troubleshooting |
| `wopr-security` | Security model - trust levels, capabilities, sandbox isolation, gateway sessions |
| `wopr-p2p` | P2P networking - Hyperswarm, identity, discovery, access grants, security scenarios |

## Quick Install All

```bash
wopr skill registry add wopr github:TSavo/wopr-skills/skills
wopr skill install github:TSavo/wopr-skills/skills/wopr
wopr skill install github:TSavo/wopr-skills/skills/meta-wopr
wopr skill install github:TSavo/wopr-skills/skills/wopr-security
wopr skill install github:TSavo/wopr-skills/skills/wopr-p2p
```

## Contributing

1. Fork this repo
2. Create `skills/<skill-name>/SKILL.md`
3. Follow the [Agent Skills spec](https://github.com/anthropics/agent-skills)
4. Submit a PR
