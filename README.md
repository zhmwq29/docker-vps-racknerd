# Docker VPS hosting: Is Docker preinstalled, do you get a dedicated IPv4, and how much RAM does a Compose stack need? (Plan-checking checklist plus what RackNerd's KVM specials include)

Three things decide whether a Docker VPS works out: how Docker actually gets onto the machine, whether the plan hands you a dedicated IPv4 you can bind to ports 80 and 443, and whether the RAM tier matches the stack you plan to run. A spec sheet that says "Docker supported" answers none of them.

RackNerd sells KVM VPS specials with full root access, a 1 Gbps port, and a control panel called ctrl where Docker with Portainer installs in one click. You can also skip the button and install Docker Engine yourself over SSH. This article covers what the sizing rules actually are, what RackNerd documents about its plans, and the five things worth confirming before you pay anyone.

> **Quick sizing answer:** one small app plus a database fits in 2 GB of RAM. A production stack with several services wants 2–4 vCPU and 4–8 GB. RackNerd rotates its specials, so match the current tier list against those numbers rather than buying the biggest box on the page.
>
> 👉 [See RackNerd's current VPS specials and tiers](https://bit.ly/RacKnerd)

## What a Docker VPS is, and why shared hosting can't run one

A container packages an application with its libraries, runtime, and configuration, and shares the host's Linux kernel instead of booting its own operating system. That sharing is the whole economic argument: a virtual machine burns 1–2 GB of RAM just to run a guest OS, while Docker's own overhead measures in single-digit megabytes. The same physical box holds far more containers than VMs.

The kernel sharing also explains the hosting requirement. The Docker daemon needs root or sudo to manage containers, networks, namespaces, and cgroups. Shared hosting accounts don't grant that, so a VPS or dedicated server is the normal home for Docker. A 2 GB VPS running a handful of containers is a completely standard small-team setup, usually driven by one `docker-compose.yml` and a single `docker compose up`.

## What RackNerd actually documents about its VPS specials

RackNerd sells these as rotating specials rather than a fixed catalogue, and the tier list changes from batch to batch. Printing a price table that goes stale in a few weeks helps nobody, so what follows is the part that stays constant across plans, which is also the part that determines whether containers run properly.

| Included on every VPS special | Detail | Why it matters for Docker |
| --- | --- | --- |
| Virtualization | KVM | Docker's namespaces and cgroups behave normally; no OpenVZ-style restrictions |
| Access | Full root | Required for the daemon, custom networks, and volume mounts |
| Network | 1 Gbps port | Image pulls and registry pushes stop being the bottleneck |
| Control panel | ctrl: reboot, OS reinstall, HTML5 VNC console, one-click apps, inbound firewall rules, root password reset, upgrades | Recover a broken container host without opening a ticket |
| Docker path | One-click Docker with Portainer from ctrl, or install Docker yourself over SSH | These are two different experiences; know which one you're buying |
| OS choice | CentOS, Rocky Linux, AlmaLinux, Fedora, Debian, Ubuntu; custom ISO on request via support ticket | Docker Engine is packaged for all of them |
| Provisioning | Right after payment | Minutes, not a waitlist |
| Management level | Unmanaged, with 24/7 support | You own the stack, the updates, and the backups |

That last row is the one people skip. RackNerd's specials are self-managed. Support covers the platform, the network, and hardware; it does not run your Compose files for you. If you want someone else to debug a container that won't start, this isn't that product.

👉 [Check which RackNerd tiers are in the current specials batch](https://bit.ly/RacKnerd)

## Match the RAM tier to the workload, not to the marketing

Docker Engine itself idles around 200 MB of RAM with minimal CPU, so on a 2 GB box you have roughly 1.8 GB for actual work. That's enough for one small application and a database, or a reverse proxy in front of a single service.

Three rules cover most buying decisions:

**One app plus a database: 2 GB.** A container running a small API or a self-hosted tool, plus Postgres or MySQL beside it. Add Caddy or Nginx Proxy Manager and you're still inside the budget.

**Several services in one Compose file: 2–4 vCPU and 4–8 GB.** This is the realistic spec for a busy production stack: Nextcloud with its database and Redis, or Immich with machine learning containers, or a handful of internal tools behind Traefik. RackNerd's own documentation for its one-click apps lines up with this, which is a useful cross-check on what vendors mean by "enough RAM."

**Write-heavy databases: watch the storage, not the core count.** A slow disk shows up first as slow Docker volume I/O, long before the CPU graph moves. If your workload is mostly database writes, storage quality matters more than an extra vCPU.

RackNerd publishes recommended RAM for the apps installable from ctrl, and the numbers map cleanly onto the tiers above:

| One-click app | RAM RackNerd recommends |
| --- | --- |
| n8n | from 2 GB |
| Coolify | from 2 GB |
| Hermes Agent | at least 4 GB |
| AnythingLLM | at least 4 GB |
| Ollama | at least 4 GB |
| LibreChat | at least 4 GB |
| OpenClaw | at least 8 GB |

Anyone planning to serve a local model should read that table before ordering. A 2 GB VPS is the right buy for n8n; it is the wrong buy for Ollama.

## Five things to check before you pay

**1. "Docker VPS" can mean three different products.** Preinstalled means the engine is running when the box boots. OVHcloud describes its Docker VPS that way. A one-click template means you pick a Docker image in the panel and wait a minute, which is how Cloudzy and Hostinger frame it. RackNerd sits in a third spot: Docker with Portainer installs as a one-click app from ctrl, or you install Engine yourself. None of these is worse, but "ready in minutes" means different amounts of work depending on which one you bought.

**2. A dedicated IPv4 is not automatic.** Running a reverse proxy that answers on 80 and 443 for several subdomains requires a public IP you control, plus working Let's Encrypt validation. Cloudzy states every Docker VPS includes a dedicated static IPv4 and IPv6 with no shared NAT. RackNerd's specials page doesn't spell this out in the material it publishes, so ask before you plan a multi-domain setup around it.

**3. Add-on fees.** Cloudzy lists Floating IPs at $2.50/month and charges for snapshots beyond the free quota. Providers that don't list extras usually don't have them, but "usually" isn't a contract. Read the order form.

**4. Refund terms.** Cloudzy publishes a 14-day money-back guarantee. RackNerd's specials page, as published, doesn't state a refund window, so read the terms at checkout rather than assuming one exists. If you're testing an unfamiliar stack, a provider with a written refund window is worth a slightly higher monthly price.

**5. Promo prices are not regular prices.** Hostinger's $6.49/month is a Black Friday price against a $19.49 list price, and the same page shows a renewal at $11.99/month after an initial 24-month prepayment. Cloudzy's $2.48/month entry point is a 50%-off promotional figure. LightNode's $7.71/month is its ordinary starting price. Putting those three numbers in a row and declaring a winner compares a discount, a bigger discount, and a list price.

## The Docker Desktop licence, which most hosting pages skip

Docker Engine, the CLI, and the open container format are free to use. The paid commercial licence applies only to Docker Desktop, the GUI for macOS and Windows. On a Linux VPS you install Engine from the command line and work in a shell, so the Desktop licence generally never enters the picture. Docker Desktop itself is free for personal use, education, non-commercial open source, and businesses under 250 employees with annual revenue below $10 million. If procurement asks whether running containers on a VPS creates a licensing cost, that's the answer.

## Where your data actually lives

A container's writable layer disappears when the container is recreated or upgraded. Databases, uploaded files, and configuration have to sit in a named volume, which lives on the host filesystem outside the container lifecycle:

yaml
services:
  db:
    image: postgres:16
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:


Back up the volume, not the container. Snapshotting a running container captures an image definition, not the rows your application wrote last night. This is also why storage quality matters: every write your database makes goes through the volume to the underlying disk.

👉 [Start with a RackNerd VPS special and install Docker from ctrl](https://bit.ly/RacKnerd)

## How the alternatives compare on paper

Entry prices are only meaningful next to the configuration they buy, and these four are not equivalent products. No provider below is linked, and none of these figures are endorsement.

| Provider | Entry price as listed | Entry configuration | Docker-relevant detail |
| --- | --- | --- | --- |
| LightNode | $7.71/mo, or $0.012/hr hourly | 2 GB DDR4, 1 vCPU, 50 GB NVMe, 1 TB transfer | Docker preinstalled and enabled by default; 40+ locations |
| Cloudzy | $2.48/mo (50%-off promotional price) | 1 GB DDR5, 1 vCPU, 25 GB NVMe, 1 TB transfer | One-click Engine + Compose, dedicated IPv4 + IPv6, 14-day money-back, 99.95% SLA |
| Hostinger | $6.49/mo Black Friday price (list $19.49) | Top advertised config: 8 vCPU, 32 GB RAM, 400 GB NVMe, 32 TB | Docker Manager with Traefik routing, Ubuntu 24.04 + Docker template; renews at $11.99/mo |
| OVHcloud | Not published on its Docker VPS page | Dedicated CPU, RAM, and storage | Docker preinstalled and operational at provisioning, full root, anti-DDoS |

Two things stand out. Cloudzy is the only one of the four that states a refund window and a dedicated IP guarantee on the same page. OVHcloud publishes no price at all on the Docker VPS page, which makes it impossible to compare without requesting a quote.

## Questions people actually ask

**Does Docker have to come preinstalled?**
No. Any VPS with full root access lets you install Docker Engine yourself in a few commands, and RackNerd supports both routes. Preinstallation saves about ten minutes on day one and nothing after that.

**How many containers fit in 2 GB of RAM?**
Engine takes roughly 200 MB. The rest covers one small application plus a database, or a reverse proxy in front of a couple of tiny services. It does not cover a 4 GB-floor app like Ollama or AnythingLLM.

**Can I run Compose, Swarm, or K3s?**
Compose works anywhere you have root and a Compose binary. Cloudzy's Docker image ships Compose and states Swarm and K3s are supported on the same box; multi-node clusters need additional VPS instances and networking. RackNerd's ctrl offers Docker with Portainer as a one-click app, and full root access means any orchestrator you can install is fair game.

**Do I need a dedicated IP address?**
For a reverse proxy serving real domains, practically yes. Shared NAT breaks HTTP validation for Let's Encrypt and makes binding 80/443 awkward. Confirm the plan includes a public IPv4 before you design around one.

**Will container data survive an upgrade?**
Data in a named volume survives container recreation, image updates, and `docker compose down`. Data written inside the container's writable layer does not.

## Which plan to buy

Running a single application with a database: take the smallest tier that gives you 2 GB and a dedicated IPv4, and don't pay for cores you won't schedule. Running several services from one Compose file: 2–4 vCPU and 4–8 GB, which is also the floor for anything model-related. Running short-lived build jobs or CI runners a few hours a week: hourly billing beats a monthly plan, and both LightNode and Cloudzy publish hourly rates.

Before you check out anywhere, confirm two details in writing: how Docker arrives on the box (preinstalled, one-click template, or your own install), and what the refund terms are. Those two answers decide how painful the first bad afternoon will be.

👉 [Compare RackNerd's current VPS specials and pick a tier](https://bit.ly/RacKnerd)
