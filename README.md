# Network Troubleshooting Log 🔎

Write-ups from networking labs where I did not know the fault in advance.

Each entry records what I saw, what I checked and in what order, what was actually wrong, and how I confirmed the fix. Entries are written as I worked through the problem, including checks that turned out to be fine.

## Sources

- Witty Networks CCNA troubleshooting labs (Packet Tracer)
- Broken labs shared by others (Packet Tracer / PingMyNetwork)

## Log

| # | Date | Source | Issue | Root Cause |
|---|------|--------|-------|------------|
| [01](01-port-security-violation.md) | 2026-11-17 | Witty Networks: Night Jackal | Media 1 can't retrieve a file from the File Server | Port security violation |

## Entry Format

Each entry follows the same structure. See [TEMPLATE.md](TEMPLATE.md).

1. **Ticket:** the problem as it was reported
2. **What I saw:** initial symptoms
3. **What I checked:** each step, in order, with commands and results
4. **Root cause:** what was actually wrong
5. **Fix:** the change made
6. **Verification:** how I confirmed it was resolved
7. **What I'd do differently:** optional
