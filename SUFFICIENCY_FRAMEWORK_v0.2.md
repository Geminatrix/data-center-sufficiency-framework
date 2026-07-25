# Data Center Sufficiency Framework v0.2

**Status:** v0.2 draft. Bite horizons in §8 are typical ranges (→S), not measurements. No worked example yet.

*A gated, time-resolved instrument for scoring efficiency, sufficiency, and capability across failure domains.*

---

## 0. What this is for

Existing rating schemes (PUE, WUE, Tier certification, ERF quotas) measure **efficiency** and **topology**. None of them measure what survives when an input disappears, and none of them prevent a facility from compensating for catastrophic fragility with excellent throughput numbers.

This framework does four things those don't:

1. **Separates three axes that conflict** and refuses to aggregate them.
2. **Treats sufficiency as a curve over time**, not a checkbox.
3. **Gates rather than penalizes** — a critical row with no independent survival pathway caps every dependent claim above that boundary.
4. **Binds row scores to the joint case before they are accepted**, so common-mode cannot be soft-pedaled after the fact.

---

## 0.1 Order of operations — binding

The sequence is not cosmetic. Evaluating rows first and common-mode second permits an operator to score every row cleanly and then treat the joint case as commentary. **The joint curves are drawn first and bound the row scores. They are not a follow-up review.**

```
1.  Enumerate external-contact dependencies      §8  register
2.  Draw joint event traces                      §7  ← HARD PREREQUISITE
3.  Score rows, bounded by step 2                §6
4.  Apply tier gates                             §5
5.  Compute binding horizon                      §5
```

**No row score is final until every joint trace touching that row has been drawn.**

A row with incomplete joint coverage is recorded as `UNRESOLVED` — not scored, not partially credited, not carried into any summary. `UNRESOLVED` ranks **below** a low score, because a low score is knowledge and this is not.

---

## 1. The three axes

| Axis | Question | Unit |
|---|---|---|
| **Efficiency (E)** | How much input becomes useful output? | ratio, dimensionless |
| **Sufficiency (S)** | How little is needed at all, and for how long without resupply? | curve over time |
| **Capability (C)** | What still functions when an input dies? | fraction of nominal, at a stated horizon |

### Scoring rules

- **Never sum. Never average.** Record as a vector: `E | S | C`.
- E and C **anti-correlate by default** — coupling for efficiency is coupling for fragility. A row scoring high on both is either exceptional or mis-measured. **Treat high-high as an audit flag, not a win.** It usually indicates a sufficiency test that was never actually run.
- **No bare "Sufficient."** Every sufficiency claim carries a deprivation and a horizon or it is void.

---

## 2. Sufficiency is a curve

> Sufficiency is not a scalar without time.
> It is remaining capability × isolation duration under specified deprivation.

### Notation

```
S[D] @ t = c
```

- `D` — the specified deprivation (what was removed)
- `t` — isolation duration since deprivation began
- `c` — fraction of nominal capability retained, 0.0–1.0

Example: `S[grid] @72h = 0.00` — under grid loss, zero capability at 72 hours.

A facility does not have *a* sufficiency curve. It has **a family of curves, one per deprivation**, plus **joint curves** for simultaneous deprivations (§7), which take precedence.

### Reading the curve

Plot `c` against `t` for each `D`.

- **Slope** is graceful degradation. Shallow is good.
- **Discontinuities are the finding.** Any cliff in the curve *is* a single point of failure, and its position on the t-axis tells you exactly which reserve ran out. A vertical drop at t=6h is a six-hour reserve with nothing behind it.

> **The derivative of the sufficiency curve locates the single points of failure.**
> This is the primary diagnostic output of the framework. Smooth curves mean layered fallbacks. Cliffs mean a lone dependency that nothing catches.

### Summary statistics

| Statistic | Meaning |
|---|---|
| `t_first_cliff` | Time to the first discontinuity of magnitude >0.2. **The most important single number.** |
| `t_50` | Time until 50% capability |
| `t_floor` | Time until capability drops below the mission-critical floor |
| `c(T_n)` | Capability at each ladder horizon |

**Do not compute area under the curve.** AUC reintroduces exactly the averaging this framework exists to prevent — it lets a long shallow tail buy back a cliff. Report the curve and the cliff positions.

---

## 3. Duration ladder

Every sufficiency test is evaluated at fixed horizons. Unbound, every test answers "yes" trivially.

| Rung | Horizon | Character | What it exercises |
|---|---|---|---|
| **T0** | 0–60 min | Buffer | Stored mass, inertia, capacitance |
| **T1** | 24 h | Reserve | On-site energy stock, tank volume |
| **T2** | 72 h | Resupply | Local logistics, regional function |
| **T3** | 30 d | Autonomy | Independence from the regional economy |
| **T4** | 1 yr | Sovereignty | Independence from the supply chain itself |

Most facilities are excellent at T0, adequate at T1, thin at T2, and **undefined past T2**. Undefined is a result, not a blank — record it as `S[D] @T3 = undefined (untested)`.

> **T3 and T4 are where the §8 register bites.** Most external-contact dependencies have expiry clocks measured in weeks to months, so they produce cliffs that no T2 test will ever surface. A program that stops at 72 hours cannot see them by construction.

---

## 4. Evidence class

Every cell carries an evidence tag. Nothing enters as verified without an evidence reference.

| Tag | Meaning |
|---|---|
| `→M` | **Measured** — full-facility test actually executed under the stated deprivation |
| `→S` | **Simulated / modeled** — analysis with stated assumptions |
| `→V` | **Vendor-claimed** — datasheet or contract language |
| `→A` | **Assumed** — inherited belief, no artifact |

**The canary rule:** a sufficiency claim tagged `→V` or `→A` cannot support a Capability score. Capability requires `→M` or a `→S` with an explicit, reviewable model.

> **Why this row matters most here:** monthly generator runs are load-bank tests, not full-facility failover with the utility breaker physically open. The claim and the test are different objects, and standard paperwork treats them as one.

---

## 5. The gate mechanism

**You cannot average your way out of a single point of systemic death.**

### Dependency tiers

| Tier | Rows | Note |
|---|---|---|
| **T0 — Substrate** | Energy, Control | Everything fails without these |
| **S — Stock** | Water, Materials | Consumable inputs; set the ceiling on duration downstream |
| **T1 — Enabling** | Cooling, Networking, Security | Depend on Substrate |
| **T2 — Delivered** | Compute, Data | Depend on Substrate + Stock + Enabling |
| **X — Cross-cutting** | Environment, Waste | Modify horizons across all rows |
| **D — Derived** | Recovery | The integral of everything above; never scored independently |

### Joint bound — applied before the tier gate

```
C(row) = min( C[D_single] , min over all joint traces J touching row of C[J] )
```

A row's capability is **defined as** the minimum across its single-deprivation case and every joint trace that strikes it. The clean single-input number is never the score; it is only a candidate for the score.

If any joint trace touching the row is undrawn, the row is `UNRESOLVED` and contributes nothing to the summary.

### Tier gate

```
effective_horizon(row) = min( own_horizon, min over upstream dependencies, §8 bite horizon )
```

**No row may claim Capability above its effective horizon.** If Control has no autonomous survival pathway past 4 hours, then Compute's "graceful degradation to 1% power over 30 days" is not partial credit — it is **void**, because nothing will be executing the degradation.

### Gate conditions (hard)

A row is **gated** — dependent claims above the boundary receive no credit — when any of these hold:

- No independent survival pathway can be demonstrated at horizon `t`
- The only survival pathway shares a common mode with the deprivation being tested
- The pathway exists but is `→V` or `→A` evidence class
- The pathway depends on a single external counterparty
- The pathway requires external contact to initiate or sustain (§8)

### Binding horizon

```
facility_horizon = min( effective_horizon(r) ) for all r in {T0 ∪ S}
register_cap     = earliest unbypassed bite horizon in §8
binding_horizon  = min( facility_horizon, register_cap )
```

`binding_horizon` is the honest headline number. Everything claimed beyond it is unscoreable.

---

## 6. The rows

Twelve domains. Each carries an efficiency question, a sufficiency test phrased as a **subtraction** (what happens when X is removed — removal tests are hard to fake in a way ratings are not), and a closed-loop target.

*Scored only after §7 and §8 are complete.*

---

### 1. Energy — *T0 Substrate*

- **Efficiency:** How much input exergy becomes useful compute? (not just PUE — useful work per joule at the workload layer)
- **Sufficiency test:** Remove each power source individually, then jointly. Utility, generation, storage, onsite renewables.
- **Target:** Multiple *independent* sources + storage + graceful load reduction.
- **Watch:** UPS is a bridge, not a reserve. A 100 MW hall with 2–15 MWh of lithium buys 1–9 minutes. The only real store on site is usually fuel in a tank — an import, and therefore a T2/T3 supply-chain dependency wearing an energy costume.
- **Also record:** average draw vs. contracted capacity (the reservoir), curtailable fraction in <5 min.

### 2. Cooling / Heat — *T1 Enabling*

- **Efficiency:** Exergy destroyed in heat removal. Is exit grade preserved or blended away?
- **Sufficiency test:** Persist without grid power. Without pumps. Without chillers. Without external water. Each alone, then together.
- **Target:** Passive fallback + thermal buffering + heat reuse to **≥2 structurally different offtakers**.
- **Watch:** Stream segregation. Blending 100°C junction/VRM streams into one 50°C return is unforced irreversibility — a 90°C stream carries roughly 5× the exergy per unit mass of a 50°C one.
- **Convergence note:** fluid mass is thermal ride-through. Immersion buys minutes where air buys seconds — failure absorbed by material property rather than by control loop.

### 3. Water / Fluids — *S Stock*

- **Efficiency:** How much is consumed or lost? (WUE, reported not modeled)
- **Sufficiency test:** How long does operation continue without replenishment?
- **Target:** Capture, purification, recirculation, leak recovery. Zero evaporative draw.
- **Watch:** This is the one input that is politically contested, drought-exposed, and unbuyable at any price in the wrong place. **Closed dielectric loop scores on both the efficiency and the sovereignty rubric simultaneously** — the rare move with no trade.

### 4. Compute — *T2 Delivered*

- **Efficiency:** Compute delivered per joule. **Erasure count per unit output** — a program property, currently unmeasured, and a compiler-layer number.
- **Sufficiency test:** What capability survives at 75%, 50%, 10%, 1% available power?
- **Target:** Progressive degradation, not binary shutdown.
- **Also record:** batch tolerance and deferrable fraction — work you can decline is sufficiency, not just scheduling.
- **Register check:** subscription-gated silicon (feature-on-demand licensing) is an external-contact dependency inside the compute layer itself. See §8.

### 5. Data — *T2 Delivered*

- **Efficiency:** Bits stored and moved per joule.
- **Sufficiency test:** Survive corruption, isolation, hardware loss, EMP-class events.
- **Target:** Geographic + hardened + offline redundancy.
- **Watch — split the electromagnetic threats, they have opposite mitigations:**
  - **E1 HEMP** — fast rise, couples to short conductors, kills chips. Requires Faraday shielding. Essentially nobody commercial does this.
  - **GMD / Carrington-class** — quasi-DC, couples only to *long* conductors, saturates HV transformers. **Does not touch your servers.** It removes the grid for months because bespoke transformers have year-plus lead times. The facility failure mode is therefore **fuel logistics at T3**, not electronics.

### 6. Networking — *T1 Enabling*

- **Efficiency:** Useful bandwidth per unit energy and resource cost.
- **Sufficiency test:** Terrestrial connectivity disappears.
- **Target:** Multiple **physically independent** paths.
- **Watch:** Different carriers riding the same conduit is redundancy that isn't. Independence must be at the **medium** layer — fiber / RF / satellite — not the contract layer.

### 7. Materials — *S Stock*

- **Efficiency:** Mass per delivered FLOP-year. Service life before replacement. Refurb rate vs. shred rate.
- **Sufficiency test:** Which failures require outside replacement parts?
- **Target:** Modular repair, reuse, standardized components, recoverable materials.
- **Watch:** Spares held on site vs. spares *assumed orderable*. This is the softest loop in the entire stack and nobody scores it. An organism turns over its own substrate continuously; a data center has **no metabolic recycling of its own body** — dead accelerators are shredded and the gallium and rare earths landfilled.
- **Status:** no accepted metric exists. Open frontier.

### 8. Waste — *X Cross-cutting*

- **Efficiency:** How much output becomes unusable waste?
- **Sufficiency test:** Can waste streams become inputs locally?
- **Target:** Heat / material / water cascading into secondary uses — **with buffers at every interface.**
- **Watch:** Every cascade coupling is a new external dependency. Score it in **both** directions: it raises efficiency and lowers capability. See §10.

### 9. Control — *T0 Substrate*

- **Efficiency:** Coordination overhead per MW.
- **Sufficiency test:** Operate safely without cloud control, without upstream authentication, without humans.
- **Target:** Local autonomous control with **manual fallback**.
- **Why this is Substrate:** Control decides whether every other row's fallback actually fires. A gate here voids capability claims everywhere downstream.
- **Register check:** DCIM/BMS cloud control planes and SSO-gated operator login are the two most common hidden external-contact dependencies in this row. See §8.

### 10. Security — *T1 Enabling*

- **Efficiency:** Protection per unit complexity and cost.
- **Sufficiency test:** **Can compromise of one trust root cascade everywhere?**
- **Target:** Compartmentalization and independent recovery roots.
- **Watch:** This is degeneracy applied to trust, and it is where monoculture is most total — one CA, one signing key, one BMC vendor, one hypervisor, one firmware image across every "redundant" unit.
- **Register check:** certificate validity and revocation checking are a *time-bombed* external dependency. See §8.

### 11. Environment — *X Cross-cutting*

- **Efficiency:** How well does the site exploit local conditions? (free-cooling hours, ambient gradient, seasonal swing)
- **Sufficiency test:** Adapt to drought, heat, cold, smoke, flooding.
- **Target:** Site-specific adaptive infrastructure.
- **Watch:** **Exergy is reference-state dependent.** A 50°C stream is a resource in Reykjavik in February and garbage in Phoenix in August. Same stream. This row is why scores are not portable between sites and must never be benchmarked across climates.

### 12. Recovery — *D Derived*

- **Efficiency:** How quickly and cheaply does capacity return, per unit restored?
- **Sufficiency test:** What fraction of capacity returns with **no external parts, no external personnel, no upstream network**? Can it cold-start with the grid still down?
- **Target:** Black-start capable. Restart executable by on-site staff from **local** documentation. **Boots and runs with zero external contact** — scored via the §8 register, not as a note in this row.
- **Note:** Recovery is the integral of the other eleven, not a peer. Score it as derived. Its ceiling is `binding_horizon`.

---

## 7. Joint event traces — prerequisite, not review

**The table format implies rows are independent. They are not.** Row-by-row testing systematically understates risk, because real events strike multiple rows at once — and typically take out Control, which disables the fallbacks in the others.

The table is the **inventory**. The joint traces are the **test**, and per §0.1 they are drawn **before** row scores are accepted.

For each event: trace every row it touches simultaneously, and produce **one joint curve** for the event as a whole. That curve, not the row's clean single-deprivation curve, bounds the capability of every row it strikes.

| Event | Rows struck simultaneously | Joint curve drawn? |
|---|---|---|
| **Grid loss** | Energy, Cooling, Networking, Control | ☐ |
| **Municipal water cutoff** | Water, Cooling, Environment | ☐ |
| **Regional network isolation** | Networking, Control, Data, Security | ☐ |
| **GMD / Carrington-class** | Energy (months), Materials (transformer lead time), Recovery | ☐ |
| **E1 HEMP** | Data, Control, Security, Compute | ☐ |
| **Supply chain freeze** | Materials, Energy (fuel), Recovery | ☐ |
| **Trust root compromise** | Security, Control, Data, Recovery | ☐ |
| **Credential / attestation expiry** | Control, Security, Compute, Data, Recovery | ☐ |
| **Heat dome / drought** | Environment, Water, Cooling, Energy | ☐ |
| **Offtaker failure** (district heat / greenhouse) | Waste, Cooling, and any ERF compliance claim | ☐ |

**Coverage rule:** a row may not be scored until every event listing that row has a drawn joint curve. Add site-specific events freely; **never remove a row from an event's strike list to make coverage easier.**

### Degeneracy ratio

For each event, compute:

```
degeneracy = (functions served by structurally different means)
           ÷ (functions served by N-identical units)
```

**Redundancy is N identical units** — same firmware, same vendor, same batch, same bug. It survives random failure and dies instantly to common-mode.
**Degeneracy is structurally different elements performing the same function** (Edelman & Gally). Biology runs almost entirely on the latter; data centers almost entirely on the former.

Tier III/IV certifies topology, **not decorrelation.** Nobody scores common-mode exposure, which is why the failure mode is invisible in the paperwork.

Current industry degeneracy ratio is near zero across nearly every row. Expect it.

---

## 8. Standing register — external-contact dependencies

**Permanent, facility-wide, enumerated before anything else is scored.**

This register exists because these dependencies are **invisible in almost every current resilience review, and growing.** They are acquired through procurement and licensing rather than through engineering, so they never appear in the resilience document. They are not a Recovery footnote — they cut across Control, Security, Compute, Data, and Networking, and each carries its own clock.

### Why they need their own instrument

Every entry below is an **expiry, not a drain.**

A fuel tank empties predictably and visibly; someone watches the gauge. A certificate simply stops validating on a date nobody wrote in the runbook. Because most expiry windows sit between 30 and 365 days, **these are the cliffs that only appear at T3 and T4** — structurally invisible to any program that tests to 72 hours.

They also fail *silently and remotely*. There is no local symptom preceding them and no local action that causes them. A capability present yesterday is absent today with no change in local state — which is precisely the signature the rest of this framework is built to catch and which no other row detects.

### The register

| Dependency | Typical bite horizon | Failure mode when isolated |
|---|---|---|
| **Remote attestation** (TPM / DICE / vendor service) | boot | Hardware refuses to enter trusted state |
| **License / entitlement server** | days–months | Software degrades or halts at renewal check |
| **Subscription-gated silicon** (feature-on-demand) | months | Cores, ports, or accelerator features silently disable |
| **Certificate validity** | 30–398 d | Internal TLS fails wholesale; no renewal path offline |
| **Revocation checking** (OCSP / CRL) | hours–days | Fail-closed stacks halt; fail-open stacks are now insecure |
| **Cloud KMS / key escrow** | hours | Encrypted volumes cannot be unsealed |
| **Identity provider / SSO** | hours | **Operators locked out of their own facility** |
| **Time source** (NTP / GNSS) | hours–days | Clock skew breaks Kerberos and TLS; GNSS is jammable |
| **DNS resolution** | minutes–hours | Internal service discovery collapses |
| **Cloud control plane** (DCIM / BMS / OOB management) | immediate | Loss of orchestration and remote hands |
| **Telemetry heartbeat** | hours–days | "Call home or degrade" firmware behavior |
| **Firmware / microcode channel** | months | No patch path; forced choice between known-vulnerable and unsupported |

### Scoring

For every entry, record:

```
present? [Y/N]   bite_horizon = ____   offline_bypass? [Y/N]   evidence = [→M/→S/→V/→A]
```

Two rules make the register bind:

1. **`offline_bypass` must be `→M`.** A vendor's statement that the product "supports air-gapped operation" is `→V` and does not count. The test is a physical disconnect, held past the bite horizon.
2. **Any entry with `present=Y` and `offline_bypass=N` caps `binding_horizon` at that entry's bite horizon** — regardless of every other row's performance. This is the fuel-tank rule applied to licensing.

Raise the **`PHONE-HOME`** flag (§9) on every row the entry touches.

> A facility with 30 days of fuel, full water recirculation, and a 90-day certificate has a **90-day binding horizon** and a hard cliff at day 90 that no amount of diesel addresses. Enumerate this register before congratulating anyone on the tank farm.

---

## 9. Audit flags

Raise for review, do not score:

| Flag | Trigger |
|---|---|
| **UNRESOLVED** | Row scored before its joint traces were drawn. Invalidates the summary |
| **PHONE-HOME** | Row depends on external contact to boot, run, or recover (§8) |
| **HIGH-HIGH** | E and C both high in the same row → sufficiency test probably never run |
| **UNBOUND** | A sufficiency claim with no horizon attached |
| **PAPER** | Capability score resting on `→V` or `→A` evidence |
| **SOLE COUNTERPARTY** | Any cascade or offtake dependent on exactly one external party |
| **MONOCULTURE** | Redundant units sharing firmware, vendor, batch, or signing root |
| **CLIFF** | Discontinuity >0.2 in any sufficiency curve — locate and name the exhausted reserve |
| **UNDEFINED-T3** | No answer at 30 days. Common, and it is a finding, not an omission |

---

## 10. The central tension — state it, do not resolve it

**Exergy efficiency is coupling, and coupling is fragility.**

Every cascade stage that raises efficiency adds an external dependency and lowers capability. This is not a flaw in the framework; it is the real trade, and the framework exists to make it visible rather than to make it disappear.

Germany's EnEfG is the live case: energy reuse factor of 10% for facilities starting operation on or after 1 July 2026, rising to 15% (2027) and 20% (2028), with PUE ≤1.2 for new builds. It is the first regulation to measure a **trophic transfer coefficient** — and it mandates the coupling **without mandating the buffer.** The statute is threaded with exemptions precisely because everyone involved knows the coupling is brittle.

Two things resolve the tension, and neither is currently built:

- **Buffers** — thermal and electrical storage as the **decoupling organ**, so neither party's uptime is load-bearing on the other's. Nobody builds them because a buffer is a tank, and a tank is capex with no throughput line item.
- **Degeneracy** — multiple structurally different offtakers (district heat, greenhouse, absorption chilling, process heat) rather than one certified pipe. A web tolerates a dead node. A quota met by a single counterparty is a chain wearing a sustainability certificate.

**Linear trophic cascades are the most fragile arrangement in ecology, not the most robust.** Ecosystems survive because they are loosely coupled with redundant pathways, and they pay for it in efficiency. Design toward the web, budget for the loss.

---

## 11. Open metrics

Two rows have **no accepted measurement instrument at all**. This is where the unclaimed work is.

1. **Materials** — mass per delivered FLOP-year, element recovery rate, on-site spare depth. No standard, no reporting, no comparison possible.
2. **Erasure** — thermodynamic cost of computation at the program layer. Landauer's floor applies only to *erasure*; logically reversible computation has no lower dissipation bound (Bennett). Real switching runs ~10³–10⁴× above kT·ln2. **Erasure count is a program property** — cache eviction, register overwrite, checkpoint discard are software decisions with thermodynamic prices. Nobody profiles code by erasure. Exergy accounting currently stops at the facilities layer; it belongs in the compiler.

A third, weaker gap: **per-token energy figures are nearly all vendor-published, single-number, with no methodology and no boundary definition** — does it include cooling, networking, idle amortization? Until the boundary is declared, centralized-vs-local comparisons are rhetorical. The measurement instrument does not exist, which is why nobody is forced to show the seam.

---

## Appendix A — Worksheet template

```
ROW:                    [name]                      TIER: [T0/S/T1/T2/X/D]
═════════════════════════════════════════════════════════════════════════

▸ PREREQUISITE — JOINT COVERAGE            (must clear before scoring)
─────────────────────────────────────────────────────────────────────────
  Events in §7 striking this row:  ______________________________________
  Joint curve drawn for each?      [ ] all   [ ] partial → STOP: UNRESOLVED
  Worst joint capability C[J]:     ____  at event: ______________________

▸ PREREQUISITE — EXTERNAL CONTACT          (§8 register)
─────────────────────────────────────────────────────────────────────────
  Entries touching this row:       ______________________________________
  Any present with offline_bypass = N?   [ ] Y → raise PHONE-HOME
  Earliest bite horizon:           ____   evidence: [→M/→S/→V/→A]

▸ EFFICIENCY
─────────────────────────────────────────────────────────────────────────
  E = [value] [unit]                          evidence: [→M/→S/→V/→A]

▸ SUFFICIENCY
─────────────────────────────────────────────────────────────────────────
  deprivation D = [what was removed]
    S[D] @T0 (60m)  = ____     evidence: ____
    S[D] @T1 (24h)  = ____     evidence: ____
    S[D] @T2 (72h)  = ____     evidence: ____
    S[D] @T3 (30d)  = ____     evidence: ____
    S[D] @T4 (1yr)  = ____     evidence: ____

    t_first_cliff   = ____     exhausted reserve: ____________
    t_50            = ____
    t_floor         = ____

▸ CAPABILITY
─────────────────────────────────────────────────────────────────────────
  C[D_single]                = ____
  C[J] worst joint           = ____
  C = min(above)             = ____  @ horizon ____

  upstream deps:             ______________________________________
  effective_horizon = min(own, upstream, §8 bite) = ____
  GATED? [Y/N]   if Y: claims above ____ are void

▸ FLAGS
─────────────────────────────────────────────────────────────────────────
  [ ] UNRESOLVED   [ ] PHONE-HOME   [ ] HIGH-HIGH   [ ] UNBOUND
  [ ] PAPER        [ ] SOLE COUNTERPARTY            [ ] MONOCULTURE
  [ ] CLIFF        [ ] UNDEFINED-T3
```

## Appendix B — Facility summary

```
▸ GATE CHECKS (complete before reading anything below)
  Joint traces drawn:        ____ / ____ events
  §8 register enumerated:    [ ] Y
  Rows UNRESOLVED:           ______________________________________
      → if any, this summary is INVALID, not partial

▸ HORIZONS
  facility_horizon = min(effective_horizon) over {Energy, Control, Water, Materials}
                   = ____
  register_cap     = ____   (earliest unbypassed §8 bite horizon)
  binding_horizon  = min(above) = ____   ← the honest headline number

▸ VECTOR BY ROW (E | S@T2 | C)
  Energy       ___ | ___ | ___        Materials    ___ | ___ | ___
  Cooling      ___ | ___ | ___        Waste        ___ | ___ | ___
  Water        ___ | ___ | ___        Control      ___ | ___ | ___
  Compute      ___ | ___ | ___        Security     ___ | ___ | ___
  Data         ___ | ___ | ___        Environment  ___ | ___ | ___
  Networking   ___ | ___ | ___        Recovery     ___ | ___ | ___  (derived)

▸ FINDINGS
  Degeneracy ratio (facility-wide):  ____
  Gated rows:                        ______________________________
  PHONE-HOME rows:                   ______________________________
  Cliffs located (t, cause):         ______________________________
```

---

*v0.2 — draft. Changes from v0.1: joint event traces promoted to hard prerequisite (§0.1 order of operations, §5 joint bound, §7 coverage rule, UNRESOLVED flag); external-contact dependencies promoted from a Recovery note to a standing facility-wide register (§8) with its own PHONE-HOME flag and a cap on binding_horizon; "credential / attestation expiry" added as a §7 event. Materials (row 7) and the erasure metric under Compute have no established measurement standard and remain open. Regulatory figures cited from the German EnEfG; a June 2026 Kabinettbeschluss / Novelle proposes moderate PUE relaxations for existing facilities and additional ERF exemptions (including no viable heat network). Verify current statutory text before citing.*
