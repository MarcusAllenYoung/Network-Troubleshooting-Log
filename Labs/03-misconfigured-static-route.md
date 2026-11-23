# 03 | Sales PCs can't reach an internet webpage (misconfigured static route)

**Date:** 2026-11-22
**Source:** [Witty Networks CCNA Packet Tracer Troubleshooting Labs: Silent Stack](https://wittynetworks.net/courses/ccna-packet-tracer-troubleshooting-labs/lessons/silent-stack/)

**Ticket:**
> The user at Sales 2 has reported they are unable to reach an Internet webpage with IP 13.13.13.1. Investigate and resolve the issue, if possible.

---

## Step 1 | Confirm the issue

From the Sales 2 command prompt, I pinged 13.13.13.1. It failed, so the issue is confirmed.

I wanted to confirm it in the browser too, but the browser was locked in this lab.

## Step 2 | Determine the scope

I ran the same ping from the other PCs:

| Department | PCs | Can reach 13.13.13.1 |
|------------|-----|----------------------|
| Marketing | Both | Yes |
| Finance | Both | Yes |
| HR | Both | Yes |
| Sales | Sales 1, Sales 2 | **No** |

The issue affects both Sales PCs, not just Sales 2.

## Step 3 | Check the Sales PCs' network settings

```
ipconfig /all
```

Both Sales PCs had an APIPA address, no default gateway, and no DNS server.

```
ipconfig /renew
```

The DHCP request failed on both.

## Step 4 | Check Layer 2 on ASW2

From the Admin PC, I SSH'd into ASW2, the access switch the Sales PCs connect to.

| Command | Result |
|---------|--------|
| `show interfaces status` | No issues |
| `show port-security` | Not configured |

Layer 2 looked fine, so I moved up the network. I used `show cdp neighbors detail` to find the next hop's IP so I could SSH into it.

## Step 5 | Follow the path and check each routing table

On each device, I ran `show ip route` to check for a route to the Sales network, then `show cdp neighbors detail` to find the next device.

| # | Device | Route to Sales network |
|---|--------|------------------------|
| 1 | DSW2 (Layer 3 switch) | Correct |
| 2 | BR1-Core router | Correct |
| 3 | Edge-SW (Layer 3 switch) | Correct |
| 4 | BR1-RTR1 | **Wrong** |

On BR1-RTR1, a static route sent the Sales network 10.30.30.0/30 to 12.12.12.1, which is toward the internet. It should point to 10.10.10.2, the Edge-SW. There was also a correct OSPF route, but the router was using the static route instead.

## Step 6 | Fix the issue

I removed the misconfigured static route on BR1-RTR1:

```
no ip route 10.30.30.0 255.255.255.252 12.12.12.1
```

## Step 7 | Verify

On both Sales PCs:

```
ipconfig /renew
```

Both received an IP address. I pinged 13.13.13.1 and it was successful. The issue is resolved.

---

## Root cause

A misconfigured static route on BR1-RTR1 sent traffic for the Sales network 10.30.30.0/30 out toward the internet (12.12.12.1) instead of to the Edge-SW (10.10.10.2). The router used it over the correct OSPF route, so traffic going back to the Sales network was sent the wrong way.

## What I'd do differently

- Run `show ip route 10.30.30.0` on each hop instead of reading the full routing table.
- After the fix, check `show ip route` on BR1-RTR1 to confirm the OSPF route was installed.
