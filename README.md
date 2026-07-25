# Data Center Sufficiency Framework

A gated, time-resolved instrument for scoring **efficiency**, **sufficiency**, and **capability** across failure domains in data centers.

Most rating schemes (PUE, WUE, Tier, ERF) measure efficiency and topology. None of them measure what survives when an input disappears. This framework does four things those do not:

1. Separates three axes that conflict and refuses to aggregate them (`E | S | C`).
2. Treats sufficiency as a curve over time, not a checkbox.
3. Gates rather than penalizes — a critical row with no independent survival pathway caps every dependent claim.
4. Binds row scores to joint (common-mode) event traces before they are accepted.

**Core idea:** You cannot average your way out of a single point of systemic death. Cliffs in the sufficiency curve locate the single points of failure. External-contact dependencies (licenses, certificates, attestation, SSO, etc.) are treated as first-class expiry clocks that can bind the entire facility horizon.

## Status

v0.2 draft. Bite horizons in the external-contact register (§8) are typical ranges (→S), not measurements. No worked facility example yet.

## Files

- `SUFFICIENCY_FRAMEWORK_v0.2.md` — the full framework, worksheets, and appendices.
- `HOW_TO_SCORE.md` — minimal practical guide.

## Quick start

1. Enumerate external-contact dependencies (§8 register).
2. Draw joint event traces for the listed common-mode events (§7) — these are a hard prerequisite.
3. Score each row, bounded by the joint curves and the register.
4. Apply tier gates. The binding horizon is the honest headline number.

## Why this exists

Exergy efficiency is coupling, and coupling is fragility. Every cascade that raises efficiency adds an external dependency. The framework makes the trade visible instead of hiding it. It is deliberately non-aggregating and deliberately harsh on paper claims and monoculture.

## License

CC BY-SA 4.0 (or choose your preferred open license). Attribution to Dane Fritz / PlanetaryS936.

## Contact / Feedback

Open issues or reach out via X: [@PlanetaryS936](https://x.com/PlanetaryS936)

Contributions, critiques, and worked examples are welcome. The missing →S measurements and the open Materials / Erasure metrics are intentional frontiers.
