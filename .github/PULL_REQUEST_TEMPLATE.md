## What & why

<!-- What does this change, and why? -->

## Checklist

- [ ] `pytest -q` is green
- [ ] New transforms are reversible with a round-trip test (`decrypt(encrypt(x)) == x`)
- [ ] New/changed steps handle hostile input with a categorized `DecryptError`
- [ ] Examples/tests use **synthetic data only** (no real hosts, keys, or names)
- [ ] No frozen contract was renamed/removed (additive only) — see `CONTRACTS.md`
- [ ] Docs updated if behavior changed
