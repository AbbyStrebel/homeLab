# Security posture

This is a threat model for my homelab. It states what I'm protecting, what I've
done, and — deliberately — what I know is still open.

> **Note:** this file intentionally contains no secrets, no exact rules, no IPs,
> and no hostnames. It describes posture, not configuration.

## What I'm protecting (assets)

- My home network and the devices on it (personal data, workstation).
- The server and whatever it hosts (a general-purpose host; see the roadmap).
- My domain and DNS (registered at Porkbun).

## Who/what I'm defending against (threat model)

- **Opportunistic internet scanning** — automated bots probing exposed ports.
  This is the realistic, constant threat for any home connection.
- **Lateral movement from a compromised device** — e.g. an IoT gadget or a
  guest device being used to reach things it shouldn't. (This is the motivation
  for the VLAN work.)
- *Not* in scope: targeted attacks by a determined adversary. This is a homelab;
  I'm defending against the ambient internet, not a nation-state.

## Current mitigations

- **Dedicated firewall (OPNsense)** at the network layer rather than relying on
  the ISP gateway's firmware.
- **SSH is not exposed to the internet** — port 22 is closed at the edge and
  there's no VPN into the lab yet, so SSH is reachable only from inside the LAN.
- **Secrets kept out of this repo** — API credentials (e.g. Porkbun), keys, and
  config with real values are not committed; the repo documents architecture only.
- **Inbound:** no ports are forwarded from the internet. This may change in the
  future, research (e.g reverse proxy) is a roadmap item.

<!-- TODO Update when ADR exists?? reread?
- **Dedicated firewall (OPNsense)** at the network layer rather than relying on
  the ISP gateway's firmware. See [ADR 0001](docs/decisions/0001-use-opnsense.md).
- **SSH is not exposed to the internet** — port 22 is closed at the edge and
  there's no VPN into the lab yet, so SSH is reachable only from inside the LAN.
- **Secrets kept out of this repo** — API credentials (e.g. Porkbun), keys, and
  config with real values are not committed; the repo documents architecture only.
- **Inbound:** to the best of my knowledge no ports are forwarded from the
  internet. The old game servers (Minecraft/Terraria) may have had forwards once,
  but those are dormant — verifying and confirming "zero inbound" is a roadmap
  item.
-->

## Known gaps (honest — these are open)

- **SSH still uses password auth.** Key-only auth with passwords disabled is
  planned but not done. Mitigated for now by SSH not being internet-reachable
  (port 22 closed, no VPN), but on-LAN it's weaker than it should be. Roadmap.
- **Patching is ad-hoc.** I update/upgrade machines when I remember to, not on a
  schedule. An unpatched always-on box is a real risk — a regular cadence (or
  unattended-upgrades) is roadmapped.
- **No backups.** There is currently no backup of the server or its data, tested
  or otherwise. This is the gap I'd fix first for anything I'd be sad to lose.
  Roadmap, high priority.
- **No network segmentation yet.** VLANs aren't working, so a compromised device
  isn't isolated from the rest of the LAN. See roadmap.
- **No monitoring/alerting.** I wouldn't currently know if something was wrong —
  observability is roadmapped (Uptime Kuma → Prometheus/Grafana).
- **TLS state uncertain.** The cert for my `.dev` domain needs auditing (which
  machine, whether expired). Tracked in the roadmap.
- **Secrets management is manual.** No vault/SOPS yet; credentials live in
  protected files on the hosts, not in a managed secrets system.

> Note: being double-NAT'd behind the ISP gateway incidentally reduces inbound
> exposure today, but that's a side effect of an unfinished setup, not a designed
> control — I don't count it as a mitigation.

## Responsible disclosure

This is a personal lab with nothing hosted for others, so there's no formal
disclosure process. If you spot something concerning in this *repo* (e.g. a
secret I accidentally committed), please open an issue without posting the secret
itself.

