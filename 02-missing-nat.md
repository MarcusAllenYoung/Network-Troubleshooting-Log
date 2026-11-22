# 02 | Sale 2 can't reach a website (missing NAT on the edge router)

**Date:** 2026-11-18
**Source:** [Witty Networks CCNA Packet Tracer Troubleshooting Labs: Nova Delta](https://wittynetworks.net/courses/ccna-packet-tracer-troubleshooting-labs/lessons/nova-delta/)

**Ticket:**
> The user at Sale 2 is stating that they are unable to get to the website wittynetworks.net. Investigate and resolve the issue, if possible.

## What I checked
1. Remembered from my CCNA studies that the inside network needs NAT to reach the internet.
2. Checked the edge router and found NAT was not configured.

## Root cause
NAT was missing on the edge router, so traffic from the inside network could not reach the website.

## Fix
At first I didn't configure it because I felt like I had forgotten how, so I looked up the answer. That only confirmed NAT was the issue. Later, without looking at any commands, I configured NAT on the edge router myself:

- `ip nat inside` and `ip nat outside` on the interfaces
- `access-list 1` to match the inside traffic
- PAT using the outside interface:

```
ip nat inside source list 1 interface g0/0/0 overload
```

## Verification
- After NAT was configured, the PCs could reach the website.
