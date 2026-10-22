# Betting Platform Security Research

Independent security research on Indian betting exchange platforms.

## What This Is

Penetration testing findings from 70+ cricket betting exchange sites across 4 different white-label platform architectures. All testing conducted under explicit authorization.

## Findings

| # | Finding | Severity | Scope |
|---|---------|----------|-------|
| 01 | [Zero-Auth Odds Source Override](findings/01-checkOddsUrl-zero-auth.md) | Critical | 50+ sites |
| 02 | [Encrypted Score Feed — Hardcoded Key](findings/02-encrypted-score-feed.md) | Medium | Shared infra |

More findings published regularly.

## Architectures Covered

- **Node.js + MongoDB** — Express, Socket.IO, white-label exchanges
- **Spring Boot + MySQL** — Java REST API platforms
- **Laravel + MySQL** — PHP panel-based systems  
- **ASP.NET WebForms** — Legacy .NET platforms

## Testing Methodology

- JS bundle analysis for API endpoint discovery
- WebSocket protocol reverse engineering
- Odds feed manipulation & validation bypass
- Casino game timing analysis
- Admin panel access control testing
- Cross-platform credential reuse

## Contact

**Telegram:** [@swarmesh_net](https://t.me/swarmesh_net)

Async only. No calls.

---

*No live credentials, active exploit code, or specific site identifiers published.*


## Defensive Articles

The following generalized articles cover defensive lessons from authorized security assessments. They contain no target-specific findings or active exploit details.

- [A Leaked Feed Isn't Yet a Betting Exploit](https://topdrift.github.io/betting-security-research/articles/feed-visibility-and-server-side-controls.html)
- [Authorization Boundaries in Multi-Tenant Betting Systems](https://topdrift.github.io/betting-security-research/articles/multitenant-authorization-boundaries.html)
- [Evidence Without Exposure: Safer Security Reporting](https://topdrift.github.io/betting-security-research/articles/evidence-without-exposure.html)

[Open the article site](https://topdrift.github.io/betting-security-research/).
