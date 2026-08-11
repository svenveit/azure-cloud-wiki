# Border Gateway Protocol (BGP) in Azure

## Overview

**Border Gateway Protocol (BGP)** is a dynamic routing protocol used to exchange routing information between different networks.

In Azure, BGP is particularly relevant when Azure networks need to exchange routes dynamically with external networks or network appliances.

Typical Azure services that use BGP include:

- Azure VPN Gateway
- Azure ExpressRoute
- Azure Route Server

The main benefit is simple:

> Instead of maintaining network prefixes manually on both sides, BGP allows connected networks to advertise and learn routes dynamically.

---

## Why BGP?

Without dynamic routing, network prefixes must be configured manually.

Consider an Azure environment connected to an on-premises network.

The on-premises environment contains:

```text
10.10.0.0/16
10.20.0.0/16
```

Azure contains:

```text
10.100.0.0/16
```

With static routing, both sides must know which networks are reachable through the connection.

If another on-premises network is added:

```text
10.30.0.0/16
```

the routing configuration may need to be updated manually.

With BGP, the new prefix can be advertised dynamically.

```text
On-Premises
10.10.0.0/16
10.20.0.0/16
10.30.0.0/16
        │
        │ BGP Route Advertisement
        ▼
      Azure
10.100.0.0/16
```

This becomes increasingly useful as network environments grow or change frequently.

---

## Core BGP Concepts

For working with BGP in Azure, a few concepts are particularly important.

### Autonomous System

An **Autonomous System (AS)** represents a network or group of networks operating under a common routing administration.

Each Autonomous System participating in BGP is identified by an:

**Autonomous System Number (ASN)**

Example:

```text
Azure
ASN 65515

On-Premises
ASN 65001
```

The ASN allows BGP to identify the routing domains exchanging routes.

Private ASNs are commonly used for private and hybrid networking scenarios.

---

### BGP Peer / Neighbor

Two devices that establish a BGP connection are called **BGP peers** or **BGP neighbors**.

Example:

```text
On-Premises Router
ASN 65001
        │
        │ BGP Peering
        │
        ▼
Azure VPN Gateway
ASN 65515
```

Both sides require the information necessary to establish the BGP session, including the peer IP address and ASN.

Once the BGP session is established, the peers can exchange routing information.

---

### Route Advertisement

BGP peers advertise the network prefixes they can reach.

For example, the on-premises router could advertise:

```text
10.10.0.0/16
10.20.0.0/16
```

Azure can then learn that these networks are reachable through the connection.

Azure can advertise its own reachable prefixes in the opposite direction.

```text
                BGP
        Route Advertisement
               ↕
┌─────────────────────────────┐
│                             │
│ On-Premises          Azure  │
│                             │
│ 10.10.0.0/16    10.100.0.0/16
│ 10.20.0.0/16                │
│                             │
└─────────────────────────────┘
```

This dynamic exchange of prefixes is the central idea behind using BGP in Azure hybrid networking.

---

## BGP in Azure

BGP appears in several Azure networking scenarios.

| Azure Service | BGP Usage |
|---|---|
| Azure VPN Gateway | Dynamic route exchange between Azure and external networks |
| ExpressRoute | Route exchange between Azure and external networks through private connectivity |
| Azure Route Server | BGP route exchange between Azure and network virtual appliances |

The detailed configuration of these services is outside the scope of this page.

The important point is that the same fundamental BGP concepts apply:

```text
BGP Peer
    │
    ├── ASN
    ├── Peer IP
    └── Advertised Prefixes
```

---

## Example: BGP with Azure VPN Gateway

A common Azure scenario is a Site-to-Site VPN connection between Azure and an on-premises environment.

Without BGP, the on-premises address spaces can be configured statically.

With BGP enabled, Azure and the on-premises VPN device can dynamically exchange routing information.

Example:

```text
On-Premises
ASN 65001

Networks:
10.10.0.0/16
10.20.0.0/16
        │
        │
        │ BGP
        │
        ↕
        │
Azure VPN Gateway
ASN 65515
        │
        │
        ▼
Azure VNet
10.100.0.0/16
```

The on-premises router advertises its reachable networks to Azure.

Azure can then learn:

```text
10.10.0.0/16
10.20.0.0/16
```

through BGP.

Azure advertises its reachable Azure network prefixes in the opposite direction.

The result is dynamic route exchange between both environments.

---

## BGP and the Local Network Gateway

When configuring a Site-to-Site VPN in Azure, the **Local Network Gateway** represents the remote network side.

With static routing, the reachable on-premises networks are defined through address spaces.

Conceptually:

```text
Local Network Gateway

Address Spaces
├── 10.10.0.0/16
├── 10.20.0.0/16
└── 10.30.0.0/16
```

Azure uses these configured prefixes to understand which networks exist behind the remote VPN device.

With BGP, routing information can instead be exchanged dynamically between the BGP peers.

The BGP configuration includes information such as:

```text
ASN
BGP Peer IP Address
```

The conceptual difference is:

```text
Static Routing

Administrator
     │
     ▼
Configure Prefixes
     │
     ▼
Local Network Gateway
     │
     ▼
Azure knows remote networks
```

versus:

```text
BGP

On-Premises Router
     │
     │ Advertises Prefixes
     ▼
Azure VPN Gateway
     │
     ▼
Azure learns remote routes
```

This is one of the most practical ways to understand why BGP can be useful in Azure hybrid networking.

---

## Static Routing vs. BGP

Both approaches can be valid depending on the environment.

| Static Routing | BGP |
|---|---|
| Routes configured manually | Routes exchanged dynamically |
| Simple for small environments | Useful for larger environments |
| Changes may require manual updates | Route changes can be advertised |
| Fewer routing components | Requires BGP configuration |
| Suitable for stable network prefixes | Suitable for dynamic or complex environments |

A small environment with only a few stable networks may not require BGP.

For example:

```text
Azure
   │
   │ Site-to-Site VPN
   ▼
On-Premises
10.10.0.0/16
```

If the network rarely changes, configuring the address space statically can be perfectly reasonable.

BGP becomes more valuable when:

- multiple networks exist
- network prefixes change
- multiple locations are connected
- redundant connectivity exists
- dynamic route propagation is required
- routing is managed across larger hybrid environments

---

## BGP Routes in Azure

When BGP is enabled, Azure can learn routes from BGP peers.

These routes become part of Azure's routing decision process.

A simplified route table could contain routes from different sources:

```text
Address Prefix      Next Hop / Source

10.100.0.0/16       Virtual Network
10.10.0.0/16        BGP
10.20.0.0/16        BGP
0.0.0.0/0           Internet
```

For troubleshooting, it is therefore important to understand **where a route came from**.

Azure networking can involve routes originating from:

- system routes
- user-defined routes
- BGP propagation

The effective routing behavior is determined by the routes available to the affected network interface or subnet and Azure's route selection rules.

---

## BGP Route Propagation

Azure route tables can interact with routes learned through BGP.

One important setting is **BGP route propagation**.

When BGP route propagation is enabled, routes learned through BGP can propagate to the relevant Azure routing context.

This is especially important in hybrid network architectures.

Conceptually:

```text
On-Premises
10.10.0.0/16
      │
      │ BGP
      ▼
Azure Gateway
      │
      │ Route Propagation
      ▼
Azure Network
```

If BGP route propagation is disabled on a route table, the expected BGP-learned routes may not be available to resources associated with that routing configuration.

This is therefore an important setting to verify during troubleshooting.

---

## BGP Does Not Replace Azure Routing

BGP should not be confused with Azure routing itself.

BGP is primarily a mechanism for **exchanging routing information**.

Azure still determines how traffic is forwarded based on the routes available in its routing system.

For example:

```text
BGP
 │
 └── Learns that 10.10.0.0/16 exists
              │
              ▼
       Azure Routing Table
              │
              ▼
       Routing Decision
              │
              ▼
          Next Hop
```

User-defined routes can also influence the resulting traffic path.

This distinction becomes particularly important when troubleshooting scenarios where BGP routes are successfully learned but traffic still does not follow the expected path.

---

## BGP with Azure Route Server

Azure Route Server also uses BGP.

In this scenario, BGP is commonly used between Azure and **Network Virtual Appliances (NVAs)**.

Conceptually:

```text
Network Virtual Appliance
          │
          │ BGP
          ↕
Azure Route Server
          │
          ▼
     Azure Network
```

This allows routing information to be exchanged dynamically instead of maintaining routes manually for every network change.

The detailed architecture and configuration of Azure Route Server should be treated as a separate topic.

---

## BGP with ExpressRoute

BGP is also a fundamental part of **Azure ExpressRoute**.

Routes between Azure and external networks are exchanged through BGP.

Conceptually:

```text
Customer Network
       │
       │ BGP
       ↕
ExpressRoute
       │
       ▼
     Azure
```

While the underlying connectivity model differs from a VPN Gateway, the fundamental BGP concepts remain the same:

- BGP peers
- ASNs
- advertised prefixes
- learned routes

The detailed ExpressRoute configuration is outside the scope of this page.

---

## Practical Troubleshooting

When troubleshooting BGP connectivity in Azure, the problem can usually be approached step by step.

### 1. Is the BGP Session Established?

First verify whether both BGP peers successfully established a session.

If the BGP session is not established, no dynamic route exchange can take place.

Check:

- BGP peer IP addresses
- ASN configuration
- network connectivity between peers
- VPN or underlying connectivity

---

### 2. Which Prefixes Are Advertised?

An established BGP session does not automatically mean that the expected routes are being exchanged.

Verify which prefixes each side advertises.

Example:

```text
Expected from On-Premises:

10.10.0.0/16
10.20.0.0/16
```

If Azure does not receive these prefixes, the problem may be on the advertising side.

---

### 3. Which Routes Has Azure Learned?

Verify whether the expected BGP routes are visible in Azure.

For example:

```text
10.10.0.0/16 → learned through BGP
```

If the route is missing, investigate the BGP advertisement and propagation configuration.

---

### 4. Which Routes Does On-Premises Learn?

Routing must work in both directions.

Even if Azure knows how to reach the on-premises network, the on-premises router must also know how to return traffic to the Azure networks.

Example:

```text
Azure advertises:

10.100.0.0/16
```

The remote router should learn the appropriate Azure prefixes.

---

### 5. Check BGP Route Propagation

If a route table is associated with the affected subnet, verify whether BGP route propagation is enabled or intentionally disabled.

Unexpected propagation settings can prevent learned routes from being used as expected.

---

### 6. Check User-Defined Routes

A BGP route may exist while another route influences the actual traffic path.

Check:

- User-defined routes
- effective routes
- next-hop selection
- overlapping prefixes

The existence of a BGP route alone does not guarantee that it becomes the route used for the traffic.

---

## Common Issues

Typical BGP-related issues in Azure include:

- Incorrect ASN
- Incorrect BGP peer IP
- BGP session not established
- Expected prefixes are not advertised
- BGP route propagation is disabled
- User-defined routes influence the traffic path
- Return routes are missing
- Overlapping address spaces
- Different routing information exists on each side

A useful troubleshooting mindset is:

```text
Is the BGP session established?
        │
        ▼
Are the expected prefixes advertised?
        │
        ▼
Did Azure learn the routes?
        │
        ▼
Did On-Premises learn the Azure routes?
        │
        ▼
Which route is actually selected?
        │
        ▼
Is the return path available?
```

---

## Practical Azure Perspective

For an Azure engineer, the most important point is not understanding every internal detail of the BGP protocol.

The practical questions are:

- Which systems are BGP peers?
- What ASN does each side use?
- Which prefixes does each side advertise?
- Which routes has Azure learned?
- Which Azure routes are advertised to the remote network?
- Is BGP route propagation enabled where required?
- Are user-defined routes changing the expected traffic path?
- Does a valid return route exist?

If these questions can be answered, many BGP-related Azure networking problems become significantly easier to understand.

---

## Key Takeaways

- **BGP dynamically exchanges routing information between networks.**
- BGP peers establish a session and advertise reachable network prefixes.
- Each BGP routing domain is identified by an **ASN**.
- Azure uses BGP in services such as **VPN Gateway, ExpressRoute, and Azure Route Server**.
- With Azure VPN Gateway, BGP can dynamically exchange routes between Azure and on-premises networks.
- Static address spaces are suitable for simple environments, while BGP becomes valuable as hybrid networks grow or change.
- BGP route propagation determines whether learned routes are propagated into the relevant Azure routing context.
- BGP exchanges routing information but does not replace Azure's routing logic.
- Effective routes, user-defined routes, advertised prefixes, and return paths are important when troubleshooting.
- For Azure engineers, understanding **peers, ASNs, advertisements, learned routes, and route propagation** is more important than deep BGP protocol theory.
