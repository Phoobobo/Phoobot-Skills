# Phoobot-Skills

Reusable personal agent skills for local Mac maintenance and agent coordination.

## Skills

| Skill | Purpose |
| --- | --- |
| [hybrid-agent-coordination](skills/hybrid-agent-coordination/SKILL.md) | Coordinate independent agent sessions, worktrees, handoffs, and cleanup in the standalone Herdr CLI only. |
| [herdr-agent-comm](skills/herdr-agent-comm/SKILL.md) | Send agent messages and collect replies in the standalone Herdr CLI only. |
| [herdr-app-guide](skills/herdr-app-guide/SKILL.md) | Read the local Herdr app guide before explaining or using panes, tabs, layouts, and other features. |
| [mo-clean](skills/mo-clean/SKILL.md) | Preview and run Mole clean with protected paths, explicit scope, and verified cleanup results. |
| [bytebox-login](skills/bytebox-login/SKILL.md) | Open or reuse an authorized Bytebox SSH session through a user-provided Kerberos jump host, with Herdr Core support. |

The coordination and maintenance skill instructions are written in Chinese; the Herdr app guide skill is written in English. They preserve user authorization and distinguish sending a message from completing a task.

## Install

```sh
npx skills add Phoobobo/Phoobot-Skills
```

Or select a single skill:

```sh
npx skills add Phoobobo/Phoobot-Skills --skill hybrid-agent-coordination
npx skills add Phoobobo/Phoobot-Skills --skill herdr-agent-comm
npx skills add Phoobobo/Phoobot-Skills --skill herdr-app-guide
npx skills add Phoobobo/Phoobot-Skills --skill mo-clean
npx skills add Phoobobo/Phoobot-Skills --skill bytebox-login
```

## Requirements

`herdr-agent-comm` and `hybrid-agent-coordination` are for the standalone [Herdr CLI](https://github.com/herdrdev/herdr) only. They require `HERDR_ENV=1`, an empty or unset `HC_TERMINAL_ID`, and `herdr` on `PATH`. Make the upstream `herdr` skill available and check the installed CLI help. In Herdr Core, use `hc --skill` instead. For `herdr-app-guide`, run the agent inside the Herdr desktop app and provide its generated `$HOME/.config/herdr-app/AGENTS.md` guide.

This repository contains personal agent skills, not the third-party Herdr skill or private integration configuration. For `bytebox-login`, supply your approved jump host, Kerberos principal, and target IP when starting a new connection.

## Public backup

This is a sanitized source snapshot: no credentials, private endpoints, local account paths, real session IDs, internal project files, or conversation archives are included. Example agent names and MR numbers are illustrative.

## Discovery

[skills.sh](https://skills.sh) indexes GitHub-hosted skills through Skills CLI installation telemetry. See its [listing documentation](https://skills.sh/docs/faq). Indexing may not be immediate.
