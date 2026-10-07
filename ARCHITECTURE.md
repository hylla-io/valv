# Valv architecture — starting points

**Read first:** Matt Pocock, "How to make codebases AI agents love"
(https://aihero.dev/how-to-make-codebases-ai-agents-love). Build Valv as a few DEEP modules: a
lot of behaviour behind small, simple interfaces, with fast tests on each interface.

Everything below marked **(open)** is for the developer and the Valv orchestrator to settle.

## Goals, in order
1. **Secure.** Only the user can read, see or edit a secret, through password-controlled
   interactive surfaces (Touch ID on the Mac, Face ID on the phone).
   Agents can confirm a secret exists (presence, length), never read it.
2. **Fast.** The checks on every call aim for well under a millisecond.
3. **Loud.** A blocked call fails with a clear error that explains why.
4. **General.** Valv is not fitted to Hylla. Another project can use one part, all of it, or
   replace a part (including the level model).

## Parts (proposed): each its own Cargo feature, so users take only what they want
1. **Vault:** `valv.db`, separate from any app's settings database, encrypted and unlocked only
   by the user. **(open)** How it is encrypted; one global vault with levels inside, or one per
   project.
2. **Levels:** global → project → branch → agent. Access is only ever narrowed when handed
   down. The industry term is *capability attenuation* (least privilege, applied to delegation).
3. **Inject:** a secret reaches the request that needs it (for example an API header), never the
   agent's environment or output. **(open)** Shell wrapper, local proxy, or both.
4. **Guard:** fast checks that catch secrets and personal data in outbound traffic and replace
   them with a presence marker, a length, or a clear refusal. Built-in rules plus user rules.
   Clearly fake test values pass in dev builds, so agents can test Valv for real.
   **(open)** The format of fake values; how a repeated hit is escalated.
5. **Capture:** outbound API traffic is captured, stripped and handed to the caller (Hylla's
   audit system). Valv only strips, catches and routes; it does no analysis itself.
6. **Folder access (later):** user-approved folder access per level, narrowed the same way.

## Rules for the shape
- The public API is small and stable. Internals stay private.
- No dependency on Hylla or lagom.
- A standalone command-line tool or TUI must be easy to add later as a thin layer over the same
  API. It is roadmap, not MVP. A TUI would require sudo.
- Writing a secret is an interactive user action only. An agent or a harness can ASK for a
  change, but only the user approves it.
- Mac only for now; Linux and Windows are roadmap.

## How Hylla uses it
- Hylla depends on the crate and uses every part.
- Hylla's Settings is the user's interactive surface for Valv: password protected, with Face ID
  on the phone.
- Keys and rules can differ per level: global, project, profile, agent.
- Claude Code profiles (for example work and personal) go through Valv. Each gets its own
  `CLAUDE_CONFIG_DIR`, and the user may share one real Claude folder (memories, history) across
  profiles with symlinks.
