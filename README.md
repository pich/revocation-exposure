# Revocation Exposure

**English** | [中文](README.zh-CN.md)

**A token can be cryptographically valid and operationally unauthorized. The gap between those two states is measurable — and most systems never measure it.**

Revocation Exposure is a concept coined by [Michał Piszczek](https://piszczek.pl/michal-piszczek):
the measurable window between a revocation decision and system-wide enforcement, during which a
credential remains cryptographically valid but is no longer authorized.

> Canonical definition & FAQ: **https://piszczek.pl/glossary/revocation-exposure**

---

## Definition

**Revocation Exposure** is the time between the moment authority is withdrawn and the moment every
enforcement point in a distributed system stops honoring it. During that window the credential
still verifies — correct signature, valid proof, not expired — while the authority behind it no
longer exists.

```
RevocationExposure = t(last enforcement point rejects) − t(revocation decision)
```

## Why it is not solved by sender-constraining

DPoP (RFC 9449) and mTLS-bound tokens answer one question: *is the caller the party this token was
issued to?* They say nothing about the second question: *does this authority still exist?*

Theft and stale authority are **two separate security axes**. Sender-constraining collapses the
theft axis and leaves the staleness axis untouched. A system can be fully DPoP-compliant and still
honor withdrawn authority for seconds or minutes.

## The propagation race

Revocation is not an endpoint call. It is a race across every component that independently decides
whether to honor a credential:

```
$ auth revoke client_7f3a9c
ok — revoked at t+0ms

edge-cache-03    ACCEPT  stale · t+420ms
worker-pool-11   ACCEPT  stale · t+1.2s
api-gw-eu-1      ACCEPT  stale · t+2.8s
batch-runner     ACCEPT  stale · t+9.4s
```

Every one of those `ACCEPT`s is correct cryptography and wrong authority.

## Why AI agents make it sharper

Autonomous agents hold credentials and act at machine speed, in parallel. A nine-second revocation
lag is not nine seconds of theoretical risk — it is nine seconds of an unauthorized agent doing
real work with valid credentials. As agent concurrency rises, exposure stops being a latency
footnote and becomes a blast-radius multiplier.

## How to measure it

Treat it as a service level objective, not a feature checkbox:

| Dimension | Question | Failure mode when ignored |
| --- | --- | --- |
| **Lag** | How long until every enforcement point rejects? | Unknown blast radius during incidents |
| **Coverage** | Which validators see revocation at all? | Caches and offline validators silently exempt |
| **Verification** | Is enforcement observed or assumed? | Green dashboards, stale acceptance in production |
| **Blast radius** | What can be done inside the window? | Exposure measured in seconds, damage in permissions |

Chaos-test it the way you test failover: revoke a real credential in a controlled environment and
observe when each enforcement point actually stops accepting it.

## Related concepts

- [Joule Wars](https://github.com/pich/joule-wars) — competing on useful intelligence per joule
- [Proof-Adjusted Autonomy](https://github.com/pich/proof-adjusted-autonomy) — autonomy discounted by evidence

## Source

Full essay: **https://piszczek.pl/blog/token-revocation-is-not-an-endpoint**
Video (25s): https://youtube.com/shorts/rF7HCQHZEqE

## Citation

See [CITATION.cff](CITATION.cff). Licensed [CC BY 4.0](LICENSE) — reuse with attribution.
