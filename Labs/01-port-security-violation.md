# 01 | Media 1 can't retrieve a file from the File Server (port security violation)

**Date:** 2026-11-17
**Source:** [Witty Networks CCNA Packet Tracer Troubleshooting Labs: Night Jackal](https://wittynetworks.net/courses/ccna-packet-tracer-troubleshooting-labs/lessons/night-jackal/)

**Ticket:**
> The User at Media 1 is unable to retrieve the file Lab.txt from the File Server. Investigate the issue and resolve, if possible. Use the Admin PC for remote access to network devices.

---

## Step 1 | Confirm the issue on Media 1

From the Media 1 command prompt, I tried to retrieve Lab.txt from the File Server over FTP.

```
C:\>ftp 192.168.2.250
%Error opening ftp://192.168.2.250/ (Timed out)
```

The issue is confirmed.

## Step 2 | Determine the scope

I ran the same FTP check from Media 2-4 and Sales 1-4. All of them could retrieve Lab.txt.

- The issue only affects Media 1.
- Since every other PC can reach the File Server, this is likely not a routing issue.

## Step 3 | Check the Media 1 configuration

```
ipconfig /all
```

Media 1 had an APIPA address (169.254.173.34), so it was not getting a working IP.

## Step 4 | Try to get a new IP on Media 1

```
ipconfig /release
ipconfig /renew
```

The DHCP request failed. Since all the other PCs can get IPs and retrieve the file, I ruled out a Layer 3 routing issue and moved to Layers 1 and 2.

## Step 5 | Check Layers 1 and 2 on ASW1

From the Admin PC, I SSH'd into ASW1, the access switch Media 1 connects to.

| # | Command | Result |
|---|---------|--------|
| 1 | `show interfaces status` | All ports connected and in the correct VLAN (10) |
| 2 | `show run` | Disabled for this lab |
| 3 | `show ip interface brief` | Nothing useful |
| 4 | `show cdp neighbors detail` | F0/1 and F0/2 connect to DSW1 and DSW2, so F0/3-6 connect to the PCs. The topology doesn't show port numbers, so I didn't know which port was Media 1 |
| 5 | `show interfaces f0/3` through `f0/6` | Nothing that stood out |
| 6 | `show port-security` | Port security configured. **F0/3 had 9 security violations**, all other ports had 0 |

After check 5, I stopped and thought about what else could block a single PC at the access port, and port security came to mind.

Since F0/3 was the only port with violations, my working assumption was that F0/3 was the Media 1 port.

## Step 6 | Fix the issue

From my CCNA studies, I knew that `shutdown` / `no shutdown` resets a port after a port security violation. Those commands were disabled in this lab.

I searched for other ways to fix a port security violation. The first suggestions were shut / no shut, or removing port security with `no switchport port-security`, which was also disabled. When I asked again and said I couldn't use `shutdown`, `switchport`, or `no`, it suggested:

```
clear port-security sticky
```

## Step 7 | Verify

On Media 1:

```
ipconfig /renew
```

DHCP worked and Media 1 received an IP. I retried the FTP connection to the File Server and was able to retrieve Lab.txt.

---

## Root cause

Port security on ASW1 F0/3 (the Media 1 port) had already learned a sticky MAC address that wasn't Media 1's. That filled the allowed MAC limit, so the switch dropped Media 1's frames, including its DHCP requests. Media 1 never got an IP and could not reach the File Server.

`clear port-security sticky` deleted the learned address, so the port could learn Media 1's MAC.

## What I'd do differently

- Check port security earlier when a single PC on an access switch has no IP and everything else works.
- Check the violation mode and sticky MACs with `show port-security interface` before trying to reset the port.
