# Phoobot-Skills

Reusable personal agent skills for local Mac maintenance and agent coordination.

## Skills

| Skill | Purpose |
| --- | --- |
| [hybrid-agent-coordination](skills/hybrid-agent-coordination/SKILL.md) | Coordinate independent agent sessions, isolated worktrees, task handoffs, results, and cleanup. |
| [herdr-agent-comm](skills/herdr-agent-comm/SKILL.md) | Send scoped agent messages with clear sender identity, bounded waits, and asynchronous replies. |
| [mo-clean](skills/mo-clean/SKILL.md) | Preview and run Mole clean with protected paths, explicit scope, and verified cleanup results. |

The skill instructions are written in Chinese. They preserve user authorization and distinguish sending a message from completing a task.

## Install

```sh
npx skills add Phoobobo/Phoobot-Skills
```

Or select a single skill:

```sh
npx skills add Phoobobo/Phoobot-Skills --skill hybrid-agent-coordination
npx skills add Phoobobo/Phoobot-Skills --skill herdr-agent-comm
npx skills add Phoobobo/Phoobot-Skills --skill mo-clean
```

## Requirements

For Herdr operations, install [Herdr](https://github.com/herdrdev/herdr) and make its `herdr` skill available to your agent. Run Herdr communication from a Herdr-managed agent session (`HERDR_ENV=1`). Use the installed CLI help to verify supported commands. Application-based coordination requires that application's supported session tools.

This repository contains personal agent skills, not the third-party Herdr skill or company-specific integrations.

## Public backup

This is a sanitized source snapshot: no credentials, private endpoints, local account paths, real session IDs, internal project files, or conversation archives are included. Example agent names and MR numbers are illustrative. The original local skills are not modified.

## Discovery

[skills.sh](https://skills.sh) indexes GitHub-hosted skills through Skills CLI installation telemetry. See its [listing documentation](https://skills.sh/docs/faq). Indexing may not be immediate.
