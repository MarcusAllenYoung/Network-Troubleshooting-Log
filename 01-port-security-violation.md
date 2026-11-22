# 01 | Media 1 can't retrieve a file from the File Server (port security violation)

**Date:** 2026-11-17
**Source:** [Witty Networks CCNA Packet Tracer Troubleshooting Labs: Night Jackal](https://wittynetworks.net/courses/ccna-packet-tracer-troubleshooting-labs/lessons/night-jackal/)

**Ticket:**
> The User at Media 1 is unable to retrieve the file Lab.txt from the File Server. Investigate the issue and resolve, if possible. Use the Admin PC for remote access to network devices.

## What I saw
- The PC had an APIPA (169.254.x.x) address, so it had no working IP.

## What I checked
1. `ipconfig` on the PC: APIPA address.
2. Used CDP to figure out which switch ports connected to the distribution switches.
3. Port security came to mind as a possible cause. Checked it and found the port with violations.

## Root cause
Port security violation on the switch port, so the port was not passing traffic.

## Fix
I knew `shutdown` / `no shutdown` resets a port after a violation, but those commands were disabled in this lab. I looked up another way to clear it and used:

```
clear port-security sticky
```

## What I'd do differently
- Learn other ways to recover a port from a port security violation besides shut / no shut.
