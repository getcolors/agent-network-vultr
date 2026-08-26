# CLAUDE.md

## Repository

`agent-network-vultr` is desired state only: no source code. It configures the
[`agent-network`](https://github.com/getcolors/agent-network) Package Skill to
run a minimal NetBird Agent Network demo on one Vultr instance, serving
`agent-network.bigconfig.online` and, one generated label beneath it, the
keyless agent-network endpoint.

`colors.yml` is the only file to edit. Everything else is either generated
(`.colors/`), secret (`.envrc.private`), or a copy of the installed skill.

## The launcher is a copy, not a symlink

Root `./green` is a copy of `.agents/skills/package-agent-network-green/green`.
`npx skills update -p` rewrites the payload and leaves the root file alone, so
the project keeps running the old pin while `skills-lock.json` claims the new
one. After any update:

```sh
npx skills update -p
cp .agents/skills/package-agent-network-green/green green
```

## What converges here

The gateway stack (Traefik, the combined `netbird-server`, the dashboard in
agent-network-only mode, the NetBird reverse proxy in private mode), a
headlessly bootstrapped control plane (admin account, endpoint, Anthropic
provider claiming two models, a guardrail allowlisting one, a policy with
per-group caps on the `agents` peer group, an account-wide global limit), and
the **isolated agent**: a container on an internal Docker network with no
internet route, running the NetBird client and headless Claude Code.
Acceptance proves the isolation (positive and negative space), the keyless
path, both denial classes, attribution, the configured limits, and that the
endpoint refuses callers outside the overlay.

## The Anthropic key is deliberately fake

`COLORS_PAR_ANTHROPIC_API_KEY` in this deployment's `.envrc.private` is a
fake value by the operator's choice (2026-08-26). The acceptance suite runs
in fake-key mode: the keyless probes expect Anthropic's own 401 relayed
through the proxy, which proves everything NetBird owns with nothing
billable. To upgrade the demo to real completions, put a real key in
`.envrc.private` and re-run `./green create` — the provider PUT rotates the
stored key and the same gates then require completions.

## Do not

- Read or print `.envrc.private`.
- Edit, read as source, or commit `.colors/`.
- Export `COLORS_PAR_PROFILE`; the package refuses to run when it is set.
- Weaken `compute-prevent-destroy` in committed desired state.
- Run a real `create` or `delete` without explicit authorization.
- Regenerate anything under `/etc/agent-network/secrets` on the host. Those
  are create-once: a new datastore encryption key orphans the peer database
  while the stack still looks healthy.

## Disposability

There are no backups, on purpose: this deployment is a demo and nothing on
the box is worth outliving it. Recovery is a guarded `delete`
(`COLORS_PAR_COMPUTE_PREVENT_DESTROY=false` for one run) followed by
`create`, which regenerates the endpoint hostname and every peer identity.
The dashboard admin password is generated on the host:
`ssh agent-network-vultr cat /etc/agent-network/secrets/admin_password`.
