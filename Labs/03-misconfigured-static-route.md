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

On BR1-RTR1, there was a static route for 10.30.30.0/30 pointing to 12.12.12.1, which is toward the internet. Traffic for the Sales network should go to 10.10.10.2, the Edge-SW. The correct OSPF route for the Sales network was also in the routing table.

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

BR1-RTR1 had a static route for 10.30.30.0/30 pointing to 12.12.12.1 (toward the internet). The Sales network is 10.30.30.0/24 and was correctly learned through OSPF, but the router chooses the most specific matching prefix before it looks at administrative distance. Traffic to 10.30.30.0 through 10.30.30.3 matched the /30 and was sent toward the internet.

The Sales gateway falls inside that /30. The DHCP relay sends requests with the gateway's IP as the relay address, and the DHCP server replies to that address. BR1-RTR1 sent those replies toward the internet, so the Sales PCs never received an IP address. Not reaching 13.13.13.1 was a symptom of DHCP failing.

## What I'd do differently

- Run `show ip route 10.30.30.1` on each hop to see exactly which route the Sales gateway address uses, instead of reading the full routing table.
- After the fix, run `show ip route 10.30.30.1` on BR1-RTR1 to confirm the Sales network now uses the OSPF route.
