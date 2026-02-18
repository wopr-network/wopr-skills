# wopr-skills

Claude Code skills for the WOPR project — custom slash commands available to agents working in WOPR repos.

## Structure

```
skills/
  meta-wopr/    # Meta skills about WOPR itself
  wopr/         # Core WOPR development skills
  wopr-p2p/     # P2P-specific development skills
  wopr-security/ # Security audit and review skills
```

## Key Details

- Skills here are available to Claude Code agents when this repo is in the workspace
- These are the skills that power `/wopr:sprint`, `/wopr:groom`, `/wopr:auto`, etc.
- To add a new skill: create a `.md` file in the appropriate subdirectory following the existing format
- Skills use YAML frontmatter (`name`, `description`) followed by the skill instructions

## Issue Tracking

All issues in **Linear** (team: WOPR). Issue descriptions start with `**Repo:** wopr-network/wopr-skills`.
