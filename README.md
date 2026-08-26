# agent-network-vultr

Desired state for a minimal
[NetBird Agent Network](https://docs.netbird.io/agent-network) demo on Vultr:
keyless, identity-gated LLM access, demonstrated by an agent that cannot
reach anything else.

- **https://agent-network.bigconfig.online** — dashboard (agent-network
  view), REST API, management and signal gRPC, relay WebSocket, embedded IdP
- **https://\<label\>.agent-network.bigconfig.online** — the generated
  agent-network endpoint: tunnel-only, keyless
- **`agent-network-agent`** — the isolated agent container: NetBird client +
  headless Claude Code on an internal Docker network with no internet route

Built by the
[`agent-network`](https://github.com/getcolors/agent-network) Package Skill:
OpenTofu manages the instance, its firewall and two unproxied Cloudflare `A`
records (the name and its wildcard); Ansible converges the gateway stack,
bootstraps the control plane headlessly, builds and starts the agent, and
proves the demo's claims — isolation both ways, the keyless call, both
denial classes, attribution, limits, and the endpoint refusing callers
outside the overlay.

## Use

```sh
direnv allow               # once, after cloning
./green build              # render .colors/agent-network-vultr/
./green create --dry-run   # walk the workflow, no side effects
./green create             # converge
```

## Fake-key mode

This deployment runs with a **deliberately fake Anthropic key**: acceptance
expects Anthropic's own 401 relayed through the proxy, proving isolation,
tunnel, policy and server-side key injection with nothing billable. Put a
real key in `.envrc.private` and re-run `./green create` to upgrade the demo
to real completions.

## Watching the demo

```sh
ssh agent-network-vultr agent-network-status    # containers, endpoint, usage
ssh agent-network-vultr agent-network-smoke     # re-run every acceptance gate
docker exec agent-network-agent claude -p 'hi'  # on the host: the governed path
```

Dashboard sign-in: `claude@ululi.it` with the host-generated password at
`/etc/agent-network/secrets/admin_password`.

## Deleting

```sh
COLORS_PAR_COMPUTE_PREVENT_DESTROY=false ./green delete
```

No backups exist, by design; a later `create` rebuilds everything and mints a
fresh endpoint hostname.
