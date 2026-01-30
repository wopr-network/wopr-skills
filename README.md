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
| `wopr` | WOPR CLI commands for session management, skills, plugins, and inter-agent communication |

## Contributing

1. Fork this repo
2. Create `skills/<skill-name>/SKILL.md`
3. Follow the [Agent Skills spec](https://github.com/anthropics/agent-skills)
4. Submit a PR
