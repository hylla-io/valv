# Valv architecture — starting points

**Read first:** Matt Pocock, "How to make codebases AI agents love"
(https://aihero.dev/how-to-make-codebases-ai-agents-love). Build Valv as a few DEEP modules: a
lot of behaviour behind small, simple interfaces, with fast tests on each interface.

**Ruled** items come from the developer (Hylla rulings DL-621, DL-622, and his answers on the
board). Everything marked **(open)** is still for the developer to settle in the planning
walkthrough.

## What Valv is (ruled)
1. A vault: it holds keys and the user's sensitive values. Each is stored once.
2. A release channel: Valv puts a held secret into an outgoing call (for example its header) as
   the call leaves. This works for OAuth and API keys. Nothing that asked for the call holds the
   secret.
3. A guard: it redacts held secrets and personal data in outgoing calls, and it stops agents
   from reading what Valv holds.
4. A sandbox around what it starts. It stops what could reach held secrets or harm the user's
   system.
5. A logger: every call passes through Valv, so the consuming app can get all of it for its
   audit, stripped of secrets.
6. Local only. Valv never sends user data to an outside service to check it.
7. A crate. Hylla links it into its own process. It has its own repo only so others can use it.
   A separate process comes only if evidence after MVP warrants it.

## What Valv is not (ruled)
1. Not a router, model picker or LLM gateway. Those belong to the consuming app.
2. Not a network gate. Agents keep the internet, search and tools. Traffic passes through Valv
   so it can redact and log, but Valv does not block hosts.
3. Not a second "auto mode". At MVP it guards only the secrets it is told to hold, plus actions
   that could harm the system. The docs and CONTRIBUTING ask whether it should guard more.
4. Not a secret viewer. No API returns a secret value.
5. Not the audit store. The consuming app stores, counts and searches.

## Goals, in order
1. **Secure.** Only the user can read, see or edit a secret, through password-controlled
   interactive surfaces (Touch ID on the Mac, Face ID on the phone).
   Agents can confirm a secret exists (name, description, length), never read it.
2. **Fast.** The checks on every call aim for well under a millisecond.
3. **Loud.** A blocked call fails with a clear error that explains why.
4. **General.** Valv is not fitted to Hylla. Another project can use one part, all of it, or
   replace a part (including the level model). Defaults are documented, and each consumer can
   configure them.

## Parts
1. **Vault:** `valv.db`, separate from any app's settings database, sealed and unlocked only by
   the user.
   - Each key and its identity is stored ONCE. Levels hold mappings that point to it, so a
     rotated key is updated in one place. (ruled)
   - Several stored keys may map to the same name per project (for example two OpenRouter keys,
     each mapped to `OPENROUTER_API_KEY` in a different project). A mapping is set only with the
     user's explicit approval. Agents see only key names and descriptions. (ruled)
   - Export is possible. Valv errors and warns loudly, and the user decides. (ruled)
   - Unlock: a passphrase typed only in the app's secure field (never a terminal) until the app
     is Developer-ID signed. Then Touch ID, with the key in the Secure Enclave (this needs a
     Developer ID provisioning profile). (ruled)
   - **(open)** The exact encryption, tamper checks and recovery.
2. **Levels:** a chain the consumer declares (Hylla's include global, project, branch, space,
   agent and applet; Hylla decides their order). Access is only ever narrowed when handed down (*capability attenuation*). The
   consumer passes the level at spawn. (ruled: per-level key sets at MVP)
3. **Release:** a secret reaches the call that needs it (for example an API header), never the
   agent's environment, files or output. The model route reaches Valv through its base URL.
   **(open)** How Valv sees the bodies of other HTTPS calls. A local certificate authority
   would also let Valv sit in the path without changing the base URL, which keeps Remote Control
   working.
4. **Guard:** fast checks on outgoing calls. (ruled defaults, configurable per consumer)
   - Held secrets and stored personal data are redacted, the call goes on, and the user gets a
     loud warning.
   - Unknown look-alike keys fail loud.
   - Clearly fake test values made by Valv pass in every build, so agents can test Valv for real.
   - **(open)** The format of fake values; how a repeated hit is escalated.
5. **Capture:** every call through Valv is handed to the consumer, stripped. Valv only strips,
   catches and hands over; it does no analysis itself. **(open)** Whether Hylla's watcher
   stores it (Hylla item 14).
6. **Sandbox and system access, per level:** what Valv starts runs in a macOS sandbox.
   - It can't read what Valv holds.
   - It can't take actions that harm the system.
   - Folders and macOS permissions the host app holds (Screen Recording, Accessibility,
     Automation, Full Disk Access) are narrowed per level.
   - A human's terminal is strict by default and configurable by the consumer. (ruled)
   - **(open)** The exact sandbox floor, and the deliberate "reveal" step for a human (Touch ID
     in the app is the recommendation; a typed Valv key can be replayed by an AI that controls
     the computer).
7. **Phone:** the user can approve, add and edit keys on the phone with Face ID, over the
   encrypted link. **(open)** This needs Hylla's DL-349 item 4 changed.

## Rules for the shape
- The public API is small and stable. Internals stay private.
- No dependency on Hylla or lagom.
- Fixed rules (no secret value returned, grants only narrow, writes need the user) live in core
  types, never behind a Cargo feature.
- A standalone command-line tool or TUI must be easy to add later as a thin layer over the same
  API. It is roadmap, not MVP.
- Writing a secret is an interactive user action only. An agent or a harness can ASK for a
  change, but only the user approves it.
- Mac only for now; Linux and Windows are roadmap.

## How Hylla uses it
- Hylla depends on the crate and uses every part. Valv is part of Hylla's MVP, and Hylla's MVP
  waits on it. Hylla's own development keeps today's setup until Valv ships.
- Hylla's Settings is the user's interactive surface for Valv, labelled "Valv (Vault)". It is
  password protected, with Face ID on the phone.
- Keys and rules can differ per level.
- Claude Code profiles (for example work and personal): Hylla owns the layout. Valv gives
  primitives: detect, link, check, and launch with `CLAUDE_CONFIG_DIR`.
