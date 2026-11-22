# 02 | Sales 2 can't reach a website (missing NAT on the edge router)

**Date:** 2026-11-18
**Source:** [Witty Networks CCNA Packet Tracer Troubleshooting Labs: Nova Delta](https://wittynetworks.net/courses/ccna-packet-tracer-troubleshooting-labs/lessons/nova-delta/)

**Ticket:**
> The user at Sale 2 is stating that they are unable to get to the website wittynetworks.net. Investigate and resolve the issue, if possible.

---

## Step 1 | Confirm the issue

From the Sales 2 browser, I tried to reach wittynetworks.net. It failed with `hostname unresolved`. The issue is confirmed on Sales 2.

## Step 2 | Determine the scope

I ran the same check from the other PCs. All of them returned the same `hostname unresolved` error, so the issue affects every PC, not just Sales 2.

## Step 3 | Check the Sales 2 network configuration

```
ipconfig /all
```

Sales 2 had a correct IP address and default gateway, and its DNS server was 8.8.8.8. Since the PC configuration was correct, I moved to Layer 3 to check routing.

## Step 4 | Ping test along the path

From Sales 2, I pinged each hop on the way out:

| # | Target | Result |
|---|--------|--------|
| 1 | Default gateway | Successful |
| 2 | DSW1 | Successful |
| 3 | CSW | Successful |
| 4 | Edge router inside interface | Successful |
| 5 | Edge router outside interface (172.32.1.1) | Successful |
| 6 | DNS server (8.8.8.8) | Failed |
| 7 | wittynetworks.net (by name) | Failed |
| 8 | wittynetworks.net (by IP) | Failed |

Traffic reached the edge router's outside interface but nothing beyond it, so the connection stops at the edge router.

## Step 5 | Check the edge router

From the Admin PC, I SSH'd into the edge router. From my CCNA studies, I knew the inside network would need NAT to reach the internet.

```
show run
```

NAT was not configured. My theory was that missing NAT was the issue.

I didn't configure it right away because I wasn't confident I remembered how. I checked the lab's answer, which only said that NAT needed to be configured on the edge router. It didn't include any commands. That confirmed my diagnosis.

## Step 6 | Fix the issue

Later, without looking at any commands or guidance, I configured NAT on the edge router on my own:

- `ip nat inside` and `ip nat outside` on the interfaces
- A standard ACL to match the inside traffic:

```
access-list 1 permit any
```

- PAT using the outside interface:

```
ip nat inside source list 1 interface g0/0/0 overload
```

## Step 7 | Verify

I tested from the PCs, and all of them could reach wittynetworks.net.

---

## Root cause

NAT was not configured on the edge router, so traffic from the private inside network could not reach the internet. Because the PCs couldn't reach the DNS server (8.8.8.8), they also couldn't resolve wittynetworks.net, which caused the `hostname unresolved` error.

## What I learned

- When every PC has the same problem, the cause is likely somewhere they all share, not on one PC.
- Pinging each hop in order shows exactly where traffic stops.
- A private inside network needs NAT on the edge router to reach the internet. Packets can leave with a private source address, but replies can't route back to it.
- Pinging the edge router's outside interface still worked because it stays inside my network. Anything past it needs a reply from the internet, which needs NAT.
- `hostname unresolved` means DNS failed. Pinging the website's IP tests connectivity without DNS. When both fail, the problem is reaching the internet, and the DNS failure is a symptom.
- A standard ACL only takes a source, so it's `access-list 1 permit any` with no `ip` keyword.
- `show ip nat translations` and `show ip nat statistics` verify NAT is working.
- I could configure NAT/PAT from memory once I tried.

## What I'd do differently

- Test my theory by configuring the fix myself before checking the answer.
- Use `tracert` from Sales 2 instead of pinging each hop one at a time. It would show where traffic stops in one command.
