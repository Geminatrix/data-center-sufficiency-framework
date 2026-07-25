# How to Score a Facility with the Sufficiency Framework

This is a minimal practical guide. Use the full worksheets in the main document for formal scoring.

## Prerequisites (do these first or the scores are invalid)

1. **§8 Register** — List every external-contact dependency (attestation, license servers, certificates, SSO, cloud control planes, NTP, DNS, telemetry, firmware channels, subscription silicon, etc.). For each: present? bite horizon? offline bypass demonstrated under physical isolation (→M only)?  
   Any `present=Y` + `offline_bypass=N` caps the whole facility at that bite horizon.

2. **§7 Joint traces** — For every event in the table (grid loss, water cutoff, network isolation, GMD, HEMP, supply-chain freeze, trust-root compromise, credential/attestation expiry, heat dome, offtaker failure, plus any site-specific), draw one joint capability curve of the rows it strikes simultaneously.  
   A row cannot be scored until every joint event that lists it has a drawn curve. Incomplete coverage → `UNRESOLVED` (worse than a low score).

## Then score each row

Use Appendix A worksheet.

- Efficiency (E): measured or modeled ratio. Tag evidence class.
- Sufficiency (S): family of curves under single deprivations + the joint bounds. Record t_first_cliff (most important), t_50, t_floor, and capability at each ladder rung (T0–T4).
- Capability (C): min of the clean single-deprivation C and the worst joint C that touches the row. Then apply the tier gate and §8 bite. Claims above the effective horizon are void.

## Headline number

`binding_horizon = min(facility_horizon over T0+S rows, earliest unbypassed §8 bite)`

Everything beyond that is unscoreable.

## Flags to raise

UNRESOLVED, PHONE-HOME, HIGH-HIGH, PAPER, MONOCULTURE, CLIFF, UNDEFINED-T3, SOLE COUNTERPARTY.

## Notes

- Never average or sum the three axes.
- High E + high C on the same row is an audit flag, not a success.
- Degeneracy (structurally different means) > redundancy (N identical units).
- Materials and program-level erasure metrics are still open frontiers.

Start with one real facility (or a published design). The first worked example will teach more than another ten pages of theory.
