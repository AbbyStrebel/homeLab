# homeLab

An embedded/systems engineer's home network — run on hardware I own, and
operated and documented like a production system. The focus is **networking and
security**: a real firewall, working toward proper segmentation, with the
*reasoning* behind each choice written down rather than left implicit.

I build embedded Linux and firmware for a living; this lab is where I keep the
systems, networking, and security muscles exercised outside of work — and where
I practice the operational discipline (decision records, threat modeling, tested
recovery) that good systems work depends on.

> **Honesty note:** some specifics (public hostnames, exact firewall rules, IPs,
> VLAN IDs) are intentionally omitted from this public repo. What's here is the
> architecture and the decisions, not the secrets. The roadmap is honest about
> what's done versus in progress — a lab you can't be honest about isn't a lab
> you've actually learned from.

---

## Why this exists

1. **Run and own real infrastructure** on hardware I built and maintain, instead
   of renting it — so I'm responsible for the whole stack, edge to host.
2. **Keep systems/networking/security skills sharp** beyond the embedded day job:
   firewalling, segmentation, threat modeling, and recovery discipline.
3. **Document decisions, not just setup.** Every non-obvious choice gets an
   [ADR](docs/decisions/) explaining the context and the tradeoff, so the lab is
   legible to someone else (or to me, six months later).

---

## Architecture at a glance

```text
Internet
   │
   ▼
[ ISP gateway: modem + router + Wi-Fi ]   still in the path — NOT yet bridged
   │                                       (double-NAT today; see roadmap)
   ▼
[ OPNsense firewall ]        dedicated appliance (WAN sits behind the ISP router)
   │
   ▼
[ Managed switch ] ──────────┬───────────────┬──────────────────────┐
   │                         │               │                      │
[ Wi-Fi AP ]        [ Server (Debian) ]   [ Desktop (Arch) ]     (VLANs: in progress)
                     Ryzen 7 3700X          self-built
                     general-purpose host   workstation
                     (+ discrete GPU)
```

> **Current state:** the ISP gateway is still doing NAT and Wi-Fi, so OPNsense
> sits behind it (double-NAT). Putting the ISP gateway in bridge mode is a
> tracked roadmap item — my attempts kept taking the network down, so it's
> sequenced *after* internal-network stability. VLAN segmentation is in progress
> and is the current priority. This diagram is the *current* topology, not the
> target state.

<!-- TODO - add ADR linkage once it's realized
> **Current state:** the ISP gateway is still doing NAT and Wi-Fi, so OPNsense
> sits behind it (double-NAT). Putting the ISP gateway in bridge mode is a
> tracked roadmap item — my attempts kept taking the network down, so it's
> sequenced *after* internal-network stability (see
> [ADR 0004](docs/decisions/0004-isp-gateway-bridge-mode.md)). VLAN segmentation
> is in progress and is the current priority. This diagram is the *current*
> topology, not the target state.
-->

---

## Networking & security

This is the heart of the lab.

<!-- TODO, I liked the LLM blob, but it's not ready or completely accurate

- **Dedicated firewall (OPNsense)** at the network edge rather than trusting the
  ISP gateway's consumer firmware — full control of rules, routing, and the
  segmentation work below. See [ADR 0001](docs/decisions/0001-use-opnsense.md).
- **Segmentation (in progress, current priority):** a managed switch plus
  OPNsense, working toward proper VLAN separation (e.g. trusted / IoT / lab) so a
  compromised device can't freely reach the rest of the network. Plan,
  rollback strategy, and maintenance-window discipline are written down before I
  touch the live network — see [ADR 0006](docs/decisions/0006-network-segmentation.md)
  and the [VLAN runbook](docs/runbooks/vlan-segmentation.md).
- **Edge (current):** the ISP gateway is still in the path doing NAT, so the lab
  is double-NAT'd today. Bridging it is deliberately deferred until the internal
  network is stable — stacking two failure domains at once is how you take the
  whole house offline. See [ADR 0004](docs/decisions/0004-isp-gateway-bridge-mode.md).
- **Threat model & posture:** [`SECURITY.md`](SECURITY.md) states what I'm
  protecting, what's mitigated, and — deliberately — what's still open. For a
  learning lab, naming the gaps honestly is more useful than pretending there
  aren't any.
- **Dynamic DNS (currently paused):** my ISP won't give me a static IP, so a cron
  job runs [`porkbun-ddns`](https://github.com/mietzen/porkbun-ddns) to keep my
  Porkbun domain pointed at my public IP. It's **paused**: behind the ISP router,
  OPNsense's WAN sees a *private* address, so DDNS was publishing the wrong IP.
  The real fix is bridging the gateway. Documented honestly because the bug *is*
  the lesson. See [ADR 0003](docs/decisions/0003-dynamic-dns.md).
-->

- **Dedicated firewall (OPNsense)** at the network edge rather than trusting the
  ISP gateway's consumer firmware — full control of rules, routing, and the
  segmentation work below.
- **Segmentation (in progress, current priority):** a managed switch plus
  OPNsense, working toward proper VLAN separation (e.g. trusted / IoT / lab) so a
  compromised device can't freely reach the rest of the network. Plan,
  rollback strategy, and maintenance-window discipline are written down before I
  touch the live network.
- **Edge (current):** the ISP gateway is still in the path doing NAT, so the lab
  is double-NAT'd today. Bridging it is deliberately deferred until the internal
  network is stable — stacking two failure domains at once is how you take the
  whole house offline.
- **Threat model & posture:** [`SECURITY.md`](SECURITY.md) states what I'm
  protecting, what's mitigated, and — deliberately — what's still open. For a
  learning lab, naming the gaps honestly is more useful than pretending there
  aren't any.
- **Dynamic DNS (currently paused):** my ISP won't give me a static IP, so a cron
  job runs [`porkbun-ddns`](https://github.com/mietzen/porkbun-ddns) to keep my
  Porkbun domain pointed at my public IP. It's **paused**: behind the ISP router,
  OPNsense's WAN sees a *private* address, so DDNS was publishing the wrong IP.
  The real fix is bridging the gateway.

---

## Hardware

### Network edge

<!-- TODO: Get actual devices/specs? -->
| Component       | Detail                                                        |
|-----------------|---------------------------------------------------------------|
| Firewall/router | OPNsense on a dedicated small-form appliance (purpose-bought) |
| Switch          | Managed switch (VLAN-capable)                                 |
| Wireless        | Standalone access point                                       |

### Server (repurposed desktop)

| Component   | Detail                                                        |
|-------------|---------------------------------------------------------------|
| Role        | General-purpose home server (self-hosted services, lab work)  |
| CPU         | AMD Ryzen 7 3700X — 8 core / 16 thread                        |
| RAM         | 16 GiB DDR4 (2× 8 GiB, G.Skill F4-3600C16-8GVKC)              |
| GPU         | Discrete GPU (available for GPU workloads — see roadmap)      |
| Storage     | Inland QN450 NVMe SSD                                         |
| Motherboard | ASUS ROG STRIX B450-F GAMING                                  |
| OS          | Debian (evaluating Proxmox — see roadmap)                     |

### Workstation (self-built)

| Component | Detail                             |
|-----------|------------------------------------|
| Role      | Daily-driver desktop (built by me) |
| OS        | Arch Linux                         |

<!-- TODO
### Laptop

| Component | Detail |
|-----------|--------|
| Role | |
| OS | |
-->

### RF

#### Miscellaneous

- HackRF-1 + Portapack ([Meyhem Firmware](https://hackrf.app))
- Flipper Zero
- RTL-SDR

#### HTs

- 2 x TD-H3 (2M/70cm, 5W)
- TD-H9 (2M/70cm, 10W, GNSS, APRS capable)

#### Other Transceivers

- tr(u)SDX

---

## Decisions (ADRs)

The reasoning behind the setup lives in [`docs/decisions/`](docs/decisions/).
Start here if you want to know *why*, not just *what*. These are the most
representative part of the repo: short records of the non-obvious calls and what
each one cost.

---

<!--  TODO make an actual roadmap page. And edit the blurb
## Roadmap

Deliberately public — this lab is a learning platform, and the roadmap *is* part
of the point. Items are honest about status; current priorities first.

**Networking & security (priority):**

- [ ] **VLAN segmentation.** Real separation on the managed switch + OPNsense.
      Plan and rollback are written down ([ADR 0006](docs/decisions/0006-network-segmentation.md)
      + [runbook](docs/runbooks/vlan-segmentation.md)); execution is the current
      focus.
- [ ] **Backups.** Stand up a real, *tested* backup for the server and its data —
      local-first, with a restore I've actually run. Currently none; highest-
      priority ops gap. Plan in [ADR 0007](docs/decisions/0007-backup-strategy.md).
- [ ] **Bridge the ISP gateway** to kill the double-NAT and give OPNsense the real
      public IP. Sequenced *after* VLANs are stable. **Unblocks DDNS.**
- [ ] **Re-enable dynamic DNS** once the gateway is bridged.
- [ ] **SSH hardening.** Key-only auth, password login disabled.
- [ ] **Patch cadence.** Move from ad-hoc updates to a schedule (or
      unattended-upgrades on the server).
- [ ] **Observability.** Start with Uptime Kuma; grow into Prometheus + Grafana.
- [ ] **TLS audit.** Track down which machine issued the cert for my `.dev`
      domain and whether it has expired, then consolidate cert management.

**Host / services (secondary):**

- [ ] **Evaluate Proxmox** on the server for VM/container isolation and
      snapshots. Decision will land as an ADR.
- [ ] **Containerized services.** Begin expressing services as Compose files for
      reproducibility. A local LLM host on the server's GPU is one candidate
      workload — a *sketch*, not a running service yet (see
      [`services/llm/`](services/llm/), clearly marked as such).

**Lightweight CI (supporting, not a centerpiece):**

- [ ] **Repo linting at home.** GitHub Actions lints the Markdown / YAML / shell
      *in this repo* so docs and scripts stay clean; later, run it on a
      self-hosted runner on the server. This is modest by design — the serious
      firmware CI/CD (build, unit tests, static analysis) lives with my embedded
      work, not here.
-->
