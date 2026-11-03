---
name: herdr-app-guide
description: Read the local Herdr app guide before answering or acting on requests about its features. Use when the agent is running in the Herdr app and the user asks how to use or operate panes, splits, tabs, spaces, terminals, agents, layouts, keybindings, panels, or mods, including questions such as "how do I split a pane?" or "how do I use Herdr's functions?".
---

# Herdr App Guide

Before explaining or using a Herdr app feature, read the current user's home-directory guide:

```sh
cat "$HOME/.config/herdr-app/AGENTS.md"
```

Resolve `$UserHome` to the current user's actual home directory (`$HOME` in a Unix shell). Use an available file-reading tool instead of `cat` if appropriate. Read the whole guide, continuing in chunks if the output is truncated.

Follow the guide's relevant instructions for the user's request. Use the command names, arguments, configuration files, and validation steps it documents. Read any referenced type definitions or help needed for the specific feature. The app rewrites this guide on startup, so read the installed copy rather than relying on remembered commands or embedding a copy in this skill.

Distinguish the desktop app's `herdr-app` commands from herdr-core's `hc` commands as the guide explains. Do not substitute commands from an older `herdr` CLI skill without verifying that they apply to the installed app.

For a how-to question, explain the documented steps. For an action request, carry out the requested operation and verify the result using the guide's instructions. Keep actions within the user's authorized scope.

If the guide is missing or unreadable, report the exact path and read error. Use `herdr-app --skill` as a fallback when available; otherwise explain that local documentation is unavailable rather than guessing commands. Do not create or overwrite `AGENTS.md`.
