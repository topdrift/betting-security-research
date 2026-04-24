# Decrypting Encrypted Live Score Feeds in Betting Platforms

**Category:** Information Disclosure / Weak Cryptography  
**Affected:** Betting platforms using scorecache.tresting.com  
**Discovered:** April 2026

---

## Summary

Multiple betting platforms transmit live cricket scores via WebSocket using AES encryption with a **hardcoded key embedded in the client-side JavaScript**. Since the key is publicly accessible in the minified JS bundle, the encryption provides zero actual security — it's security through obscurity.

## The Encryption

The score WebSocket (`wss://scorecache.tresting.com`) sends encrypted payloads:

```json
{
  "data": "U2FsdGVkX1+fs2XXK9H...",
  "isEncrupted": true
}
```

The payload is CryptoJS AES with OpenSSL-compatible format (`Salted__` prefix).

## Finding the Key

The decryption key is in the score iframe's JS bundle:

```javascript
// score.tresting.com/static/js/main.*.js
e.isEncrupted ? (e => {
  try {
    const a = CryptoJS.AES.decrypt(e, "?l5V-37g~<Li").toString(CryptoJS.enc.Utf8);
    return a && "" !== a ? a : e
  } catch(e) { return e }
})(e.data) : e;
```

**Key: `?l5V-37g~<Li`**

## Decrypted Data

The decrypted payload contains the full live scorecard:

```json
{
  "team1": {
    "shortName": "RCB",
    "score": "184-5",
    "overs": "16.5"
  },
  "team2": {
    "shortName": "GT", 
    "score": "205-3",
    "overs": "20.0"
  },
  "batsman": [...],
  "bowler": {...},
  "lastBalls": ["1", "4", "6", "1", "1", "6"],
  "needByBall": "RCB needs 22 runs in 19 balls to win",
  "targetRun": 206
}
```

Ball-by-ball data, batsman stats, bowler figures, partnership details — everything.

## Security Impact

1. **Score data is effectively public** — anyone who reads the JS source can decrypt it
2. **Competitive advantage** — automated systems can consume score data faster than the UI renders
3. **The encryption adds latency** without security benefit — decrypt/encrypt on every message
4. **Key rotation is impossible** without breaking all deployed clients simultaneously

## Remediation

1. **Use server-side session tokens** for score access instead of a static key
2. **Per-user encryption** with session-derived keys
3. **Or remove encryption entirely** — if the key is in the JS, it's public anyway. Just serve plaintext and save the CPU cycles.

---

*Published as security research. Contact: @swarmesh_net on Telegram*
