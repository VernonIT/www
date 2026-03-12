---
title: "Split-Horizon DNS for Azure App Service Environments"
date: 2026-03-11
draft: false
tags: ["azure", "dns", "networking", "infoblox", "app-service-environment"]
categories: ["infrastructure"]
description: "Resolving cross-tenant ASE domains on your on-premises network using Infoblox conditional forwarding."
showToc: true
---

## The Problem

You manage a network that connects multiple companies — or simply your corporate network alongside one or more Azure cloud tenants. Each Azure App Service Environment (ASE) gets its own DNS zone (e.g. `company-ase.appserviceenvironment.net`). By default, that zone is resolved by Azure's internal private DNS, which your on-premises clients can't reach.

The result: your users get `NXDOMAIN` for any ASE-hosted app. This guide walks through three progressively powerful DNS configurations in Infoblox to solve this cleanly.

**Who this is for:** Network engineers managing on-premises DNS (specifically Infoblox) who need to resolve Azure ASE domains for one or more tenants — including cross-company or multi-tenant scenarios.

---

## Background: How Azure ASE DNS Works

An Internal Load Balancer (ILB) ASE exposes a single private IP address — the ILB IP — that all apps in the environment resolve to. Azure automatically creates a private DNS zone for the environment, so that:

```
myapp.company-ase.appserviceenvironment.net      →  10.x.x.x (ILB IP)
myapp.scm.company-ase.appserviceenvironment.net  →  10.x.x.x (ILB IP)
```

This works fine for clients inside Azure — they use Azure's internal DNS. On-premises clients, however, query your corporate DNS (Infoblox), which has no knowledge of these private zones. You need to bridge that gap.

There are two approaches:

- **Host A records yourself in Infoblox** — simple, but requires manual updates
- **Forward DNS queries to Azure's DNS Private Resolver** — better; Azure maintains the answers automatically

> **Recommended approach:** Use Infoblox Forward Zones pointing to each tenant's Azure DNS Private Resolver Inbound Endpoint. This way you never need to update Infoblox when ILB IPs change.

---

## Option 1: Authoritative Zone with A Records

Use this if you don't have Azure DNS Private Resolver deployed, or if you need a quick static solution.

### Step 1: Find the ILB IP

In the Azure Portal, navigate to your App Service Environment and find the inbound IP address under **IP Addresses**. Or use the Azure CLI:

```bash
az appservice ase show \
  --name <your-ase-name> \
  --resource-group <rg> \
  --query "virtualNetwork"
```

### Step 2: Create an Authoritative Zone in Infoblox

- **Zone FQDN:** `company-ase.appserviceenvironment.net`
- **Zone Type:** Authoritative Primary
- Add a wildcard A record: `Name = *`, `Address = <ILB IP>`
- Add a root A record: `Name = @` (or blank), `Address = <ILB IP>`

**Why the wildcard?** ASE apps use subdomains like `myapp.company-ase.appserviceenvironment.net` and `myapp.scm.company-ase.appserviceenvironment.net` (for Kudu). A wildcard `*` A record covers all of these with a single entry.

### Limitations

- You must manually update the A record if the ILB IP ever changes
- You must repeat this for every ASE across every tenant
- No automatic failover or health-aware resolution

---

## Option 2: Forward Zones to Azure DNS Private Resolver (Recommended)

This is the cleaner, production-grade approach. Instead of hosting A records, you delegate resolution to each Azure tenant's DNS Private Resolver, which already has authoritative answers from the private DNS zone.

### Prerequisite

Each Azure tenant must have a **DNS Private Resolver** deployed with an **Inbound Endpoint**. The Inbound Endpoint IP is what Infoblox will forward queries to.

### Step 1: Get Each Tenant's Inbound Endpoint IP

In the Azure Portal: **DNS Private Resolvers** → select the resolver → **Inbound Endpoints**. Note the IP address (e.g. `10.1.0.4`). Each company/tenant that owns an ASE needs to provide you with this IP.

### Step 2: Create Forward Zones in Infoblox

For each ASE, create a **Forward Zone** (not an Authoritative Zone):

```
Zone FQDN:   CoA-apps.appserviceenvironment.net
Zone Type:   Forward Zone
Forward To:  10.1.0.4  (CoA's DNS Private Resolver Inbound Endpoint)

Zone FQDN:   CoB-apps.appserviceenvironment.net
Zone Type:   Forward Zone
Forward To:  10.2.0.4  (CoB's DNS Private Resolver Inbound Endpoint)
```

### How It Flows

```
Client: myapp.CoA-apps.appserviceenvironment.net
  → Infoblox: forward zone match
  → CoA Resolver (10.1.0.4:53)
  → Azure Private DNS Zone → returns ILB IP
  → Client connects to CoA ASE
```

If CoA ever migrates their ASE to a new ILB IP, their private DNS zone updates automatically. Your Infoblox configuration never needs to change.

### Network Prerequisites

DNS forwarding is only half the story. Ensure:

- Routing from on-premises to each Azure VNet (via VPN Gateway or ExpressRoute)
- NSG rules allowing UDP/TCP 53 inbound to the DNS Private Resolver subnet from your Infoblox server IPs
- NSG rules allowing 443 (and 80 if needed) to the ASE subnet from your client IP ranges
- ASE-required management NSG rules are in place (see [Azure docs](https://learn.microsoft.com/en-us/azure/app-service/environment/networking))

---

## Option 3: Adding a Catch-All for appserviceenvironment.net

Once you have specific forward zones for each company's ASE, you may also want to resolve your own ASE — or ensure any unknown ASE subdomain goes somewhere sensible.

### The Temptation: Wildcard Zones

It might seem natural to create a forward zone for `*.appserviceenvironment.net` — but DNS doesn't work that way. Wildcards apply to *records within a zone*, not to zone names themselves. You can't create a zone whose name contains a wildcard.

### The Solution: Parent Zone Forwarding

Instead, create a forward zone for the parent domain:

```
Zone FQDN:   appserviceenvironment.net
Zone Type:   Forward Zone
Forward To:  10.0.0.4  (Your own Azure DNS Private Resolver)
```

DNS resolution uses **longest suffix match**, so more specific zones always win:

| Query | Matched Zone | Forwarded To |
|---|---|---|
| `myapp.YourASE.appserviceenvironment.net` | `appserviceenvironment.net` | Your resolver |
| `myapp.CoA-apps.appserviceenvironment.net` | `CoA-apps.appserviceenvironment.net` | CoA's resolver |
| `myapp.CoB-apps.appserviceenvironment.net` | `CoB-apps.appserviceenvironment.net` | CoB's resolver |
| `unknown.other.appserviceenvironment.net` | `appserviceenvironment.net` | Your resolver (NXDOMAIN) |

The specific CoA and CoB zones take priority. Everything else falls through to your resolver, which will return `NXDOMAIN` for zones it doesn't know about — which is exactly the correct behavior.

---

## Final Configuration Summary

With all three layers in place, your Infoblox will have:

```
Forward Zone: CoA-apps.appserviceenvironment.net → 10.1.0.4
Forward Zone: CoB-apps.appserviceenvironment.net → 10.2.0.4
Forward Zone: appserviceenvironment.net          → 10.0.0.4 (your resolver)
```

Adding a new company's ASE in future requires only **one new forward zone entry** in Infoblox. No A records to maintain, no IPs to track.

---

## Deployment Checklist

- [ ] Obtain Inbound Endpoint IP from each Azure tenant's DNS Private Resolver
- [ ] Create Infoblox Forward Zone for each ASE domain, pointing to the correct resolver
- [ ] Create parent Forward Zone for `appserviceenvironment.net` pointing to your resolver
- [ ] Assign zones to correct DNS Views if using multi-view Infoblox configuration
- [ ] Verify UDP/TCP 53 is open from Infoblox to each resolver inbound endpoint
- [ ] Verify layer-3 routing from client subnets to each ASE ILB IP range
- [ ] Verify NSG rules on each ASE subnet and resolver subnet
- [ ] Test resolution from a client on each company's subnet

---

## Further Reading

- [Azure App Service Environment networking](https://learn.microsoft.com/en-us/azure/app-service/environment/networking)
- [Azure DNS Private Resolver overview](https://learn.microsoft.com/en-us/azure/dns/dns-private-resolver-overview)
- [Infoblox Forward Zone configuration](https://docs.infoblox.com)
