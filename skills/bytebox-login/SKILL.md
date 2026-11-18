---
name: bytebox-login
description: Log in to a user-specified Bytebox IP through Kerberos and an approved SSH jump host, defaulting the remote username to root, or reuse an existing authenticated SSH terminal. Use when the user asks to log in to Bytebox, open a Bytebox terminal, connect to an IP through the ByteDance jump server, or resume a Bytebox session. Support Herdr Core terminals and pause for human input when kinit or jump authentication requires it.
---

# Bytebox login

## Determine the destination and terminal

Accept any user-specified Bytebox IP; do not restrict destinations to the examples below. Use `root` as the remote username unless the user explicitly supplies another username, including in `user@IP`. Do not ask for a username when it is omitted. Keep the remote username separate from the source Kerberos principal: defaulting to `root` does not change the Kerberos identity.

Ask for the IP only when it is missing or ambiguous and cannot be determined from a clearly requested existing session. Validate the IP as an IPv4 or IPv6 address and quote the resulting SSH destination as one shell argument. For a new connection, use the user's approved jump host and intended Kerberos principal; ask if either is not known from the request or trusted local configuration. Do not connect to destinations the user is not authorized to access.

Use an interactive terminal visible to the user for authentication. When Herdr Core is available, read `hc --skill` and relevant command help first. Use `hc ls`, then `hc read <terminal> --lines 120` to identify existing sessions. Terminal IDs are ephemeral; discover them each time. Core `hc` controls terminal processes; desktop `herdr-app` controls pane placement. Do not apply the older `herdr` CLI's environment requirements to `hc`.

Reuse an authenticated terminal at the desired Bytebox when available. Read its screen before sending input; ensure it is an available shell, not a running command, editor, agent, password prompt, or unfinished user draft. Do not interrupt the user's work. If a new visible pane is needed, use the installed `herdr-app --skill` documentation; if the desktop API is unavailable, ask the user to open a terminal rather than guessing layout commands.

## Authenticate on the source machine

Run `klist` on the source machine, outside the jump and Bytebox shells. Reuse an unexpired ticket for the intended identity. An unavailable `klist` or an expired ticket requires authentication; it is not evidence of valid credentials.

If reauthentication is required, substitute the intended Kerberos principal:

```sh
kinit '<kerberos-principal>'
```

If a password or other interactive challenge appears, tell the user which terminal needs input and pause dependent commands until they complete authentication. Let the user enter secrets directly in that terminal. Do not ask for passwords, MFA codes, or tickets in chat, store them in the skill, or automate their entry. Check `klist` after completion; continue only when the intended ticket is valid.

## Enter the jump server

From that authenticated source shell, run:

```sh
ssh -K '<approved-jump-host>'
```

Use `-K` only for the approved jump host when the organization's policy permits Kerberos credential delegation. Do not replace the approved route with direct Bytebox SSH or invent a ProxyJump configuration.

If jump login asks for a password, MFA, confirmation, or interactive selection, let the user complete it in the terminal. Read the terminal again after the user reports completion. Confirm that the terminal is at the approved jump host's shell before proceeding. A submitted command alone does not prove that login succeeded.

## Enter the Bytebox

At the jump shell, substitute the requested IP and the resolved remote username into this command. Use `root` when the user supplied only an IP:

```sh
ssh '<remote-user>@<bytebox-ip>'
```

For example, a request to log in to `192.0.2.10` produces `ssh 'root@192.0.2.10'` (documentation address only). Honor an explicit username override instead of replacing it with `root`. Replace all placeholders before sending input.

If authentication or host-key verification requires human input, pause for the user. Do not disable host-key verification or accept a changed host key automatically.

Wait for the destination shell and verify with a read-only command:

```sh
hostname; whoami; pwd
```

Verify that `whoami` matches the resolved remote username. If the destination's hostname is already known, check it; otherwise record the returned hostname without guessing it from the IP. Confirm the connection used the requested IP from the submitted SSH command and the resulting session. Report the requested IP, terminal address, and verified host/user; leave the connected terminal available to the user. For a reused session, establish that it belongs to the requested destination and perform the same verification without repeating authentication.

## Herdr Core input and recovery

Use explicit discovered terminal addresses with `hc send <terminal> '<command>'`. Inspect the screen after each step with `hc read`; never queue all login commands together. Do not send the next SSH command while a password prompt or jump challenge is active. Use `hc wait <terminal> --output <expected-new-output>` only when its documented behavior fits the next output; a persistent shell normally never exits, so plain `hc wait` is unsuitable for shell readiness.

On a timeout or failed submission, read the terminal before retrying because the command may already have arrived. On Kerberos failures, check the source ticket and request human reauthentication. On denied Bytebox access, report the target and observed error; do not change accounts, destinations, SSH policy, or permissions to bypass it. Keep each session intact unless the user asks to disconnect.
