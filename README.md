# Network Troubleshooting Log 🔎

Write-ups from the [Witty Networks CCNA Packet Tracer Troubleshooting Labs](https://wittynetworks.net/courses/ccna-packet-tracer-troubleshooting-labs/), where I did not know the fault in advance.

Each entry records what I saw, what I checked and in what order, what was actually wrong, and how I confirmed the fix. Entries are written as I worked through the problem, including checks that turned out to be fine.

## Log

| # | Date | Lab | Issue | Root Cause |
|---|------|--------|-------|------------|
| [01](Labs/01-port-security-violation.md) | 2026-11-17 | Night Jackal | Media 1 can't retrieve a file from the File Server | Port security violation |
| [02](Labs/02-missing-nat.md) | 2026-11-18 | Nova Delta | Sales 2 can't reach a website | Missing NAT on the edge router |
| [03](Labs/03-misconfigured-static-route.md) | 2026-11-22 | Silent Stack | Sales PCs can't reach an internet webpage | Misconfigured static route on BR1-RTR1 |
