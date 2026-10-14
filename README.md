<!-- solve-problem:3c41dc1c-8386-4e32-a8fb-659dda525cc5 -->
# Liveness under content-destruction defenses

A [Tangle Solve problem](https://solve.tangle.tools/problems/3c41dc1c-8386-4e32-a8fb-659dda525cc5).

## What is the problem?

Content-destruction attacks are blockchain consensus-layer attacks in which an adversary withholds its blocks from the public chain and releases competing blocks only when needed to force *ties*: equal-length (or equal-weight) competing chains/tips in a DAG or tree ledger. If the fork-choice rule discards or under-rewards losing sides, honest miners' content (transactions, blocks) gets excluded from the main ledger or its rewards, and the computational work behind it is *wasted*. MAD-DAG (Bar-Zur, Tamar, Eyal, 2025; (verify arXiv:2511.21552 — treat details as unverified)) proposes a ledger rule for DAG ledgers that makes such attacks *reward-extraction resistant*: an attacker cannot gain reward by creating competing chains. However, reward resistance is not liveness: an attacker might still cheaply *suppress* honest content — paying a small (or even zero) reward cost while causing honest blocks/transactions to be excluded or their execution wasted.

Formal setting (most reasonable interpretation): a DAG ledger (blocks reference multiple parents; ordering/inclusion determined by a fork-choice and ledger-ordering rule, cf. GHOSTDAG/inclusive protocols). Parties: honest miners following the rule, and a probabilistically-bounded adversary (resource fraction β). Define per-round metrics: (i) *exclusion rate* — fraction of honestly produced blocks/transactions not included (or delayed beyond a bound) in the final ledger; (ii) *wasted execution* — honest work or transaction processing that is discarded; (iii) adversary cost. The open problem: prove a bound (ideally matching upper and lower bounds) on exclusion rate and wasted execution as a function of β under the MAD-DAG rule (or any rule satisfying its reward-extraction-resistance property), and characterize whether a rule can simultaneously achieve reward-extraction resistance and bounded liveness loss.

Ambiguity note: the census does not define the exact metrics or whether "liveness" means eventual inclusion with delay bound or eventual inclusion at all; both variants should be treated.

## Why does it matter?

DAG ledgers (PHANTOM, GHOSTDAG, SPECTRE, inclusive protocols) are deployed and proposed precisely to improve throughput and reward fairness, and content-destruction/tie-forcing attacks (generalizing selfish mining and stubborn mining to DAGs) undermine exactly that promise. If a defense like MAD-DAG stops reward theft but lets an adversary suppress transactions or waste honest execution at near-zero cost, it enables cheap censorship and denial-of-service economics against miners and users. A proven liveness bound would be the missing half of a complete incentive-compatibility-plus-liveness guarantee for DAG ledgers.

## What's already known?

- **Inclusive protocols** (Lewenberg, Sompolinsky, Zohar, FC 2015): include off-chain blocks in the ledger order and reward them proportionally, mitigating selfish mining incentives by rewarding losing-fork work. This is the baseline idea that tie-forcing attacks exploit — inclusion rules must decide *which* competing content counts.
- **Selfish/stubborn mining literature** (Eyal–Sirer 2014; Sapirshtein et al. 2016 (verify)): shows tie-forcing and temporary withholding extract rewards in linear chains; these generalize to DAG tie-forcing at tips.
- **DAG ordering** (SPECTRE, PHANTOM, GHOSTDAG by Sompolinsky et al. (verify)): give fast/volume-safe ordering with eventual inclusion, but were analyzed mostly for safety/liveness under *no* incentive manipulation, not under rational attackers gaming reward structure.
- **MAD-DAG** (Bar-Zur, Tamar, Eyal 2025 (verify)): proposes a ledger/reward rule that is reward-extraction resistant — attackers gain nothing from creating competing chains. The census states this leaves the *inclusion/liveness* objective open: bounded exclusion and wasted execution under repeated equal-length chain triggering. To our knowledge no bound is published.

## What would count as solving it?

1. **Upper bound:** A proof that under the MAD-DAG ledger rule (or a specified rule with its reward-resistance property), with adversary resource fraction β and network delay parameter, the fraction of honest blocks/transactions excluded beyond time T is at most f(β, T) → 0, and wasted execution is bounded — with explicit constants.
2. **Lower bound:** A matching adversarial strategy showing no content-destruction-resistant-with-rewards rule can do better, i.e., an inherent trade-off curve between reward-extraction resistance and liveness.
3. **Negative result (variant):** An explicit attack showing a non-negligible exclusion/waste at adversary cost strictly less than the honest loss — proving reward resistance ≠ liveness and motivating a modified rule.
4. **Construction:** A modified ledger rule with provable simultaneous reward-extraction resistance and O(β)-bounded exclusion, with a simulation study (event-driven DAG simulation replicating GHOSTDAG-style settings) demonstrating the bound is meaningful.

A referee would want a formal model, explicit attack strategies, and either the theorem or a reproducible experimental demonstration.

## How to start

1. Read Lewenberg–Sompolinsky–Zohar (2015) Sections on reward distribution to fix the inclusive-reward formalism, then obtain and study the MAD-DAG paper (arXiv:2511.21552, verify) — extract its exact ledger rule, threat model, and what "reward-extraction resistance" formally means; the gap should be stated or inferable there.
2. Formalize the tie-forcing attack as a stochastic process: honest blocks arrive as Poisson, attacker withholds and releases equal-fork-weight sets; compute the steady-state probability that the ledger rule ranks honest content last/never under repeated ties.
3. Compare against the selfish-mining Markov-chain machinery (Eyal–Sirer style state graphs, adapted to DAG tips) — the main obstacle is that DAG orderings couple many blocks at once, so the state space is larger; consider mean-field/large-population approximations first.
4. Prototype a simulator (ns-3 or discrete-event) with GHOSTDAG-like parameters to numerically measure exclusion under the MAD-DAG rule before attempting proofs.

## Key references

- R. Bar-Zur, O. Tamar, I. Eyal. *MAD-DAG* (2025), arXiv:2511.21552 (verify — identifier from census; content unverified). Proposes a 2025 DAG ledger/reward rule that resists reward extraction by attackers creating competing chains; does not address liveness/exclusion, which is the open problem here. https://arxiv.org/abs/2511.21552
- Y. Lewenberg, Y. Sompolinsky, A. Zohar. *Inclusive Block Chain Protocols*. Financial Cryptography 2015, DOI:10.1007/978-3-662-47854-7_33. Introduces including off-chain blocks in the ledger order with proportional rewards to mitigate selfish mining — the foundational inclusion-reward framework for DAGs.
- (Verify as needed) Y. Sompolinsky et al., SPECTRE/PHANTOM/GHOSTDAG papers — DAG ordering rules with formal safety/liveness analyses under partial synchrony, absent incentive manipulation.
- (Verify) I. Eyal, E.G. Sirer, *Majority is not Enough* (FC 2014); A. Sapirshtein et al. (FC 2016) — tie-forcing and withholding attack analyses in linear chains, methodological template for the DAG attack.

## Warnings

- The MAD-DAG citation comes from a census record dated after the claimed arXiv ID's plausibility window; verify the paper exists and its exact model before building on it — if unavailable, the problem reduces to: "does there exist a DAG ledger rule that is simultaneously reward-extraction resistant and liveness-preserving under tie-forcing?"
- "Equal-length competing chains" is ambiguous in DAGs: equal weight, equal depth, or competing anticone sets? Fix a definition and state it.
- Do not conflate eventual inclusion with *timely* inclusion — inclusive protocols eventually include almost all blocks, so any nontrivial result must concern delay, reward exclusion, or wasted execution, not bare eventual inclusion.
- Common dead end: proving reward resistance implies liveness by fiat; the census explicitly says they are separate objectives, and a cheap-suppression counterexample is plausibly the intended first result.

---
*`BCGT-000252` · flagged for review*

## Acceptance guidance

1. **Upper bound:** A proof that under the MAD-DAG ledger rule (or a specified rule with its reward-resistance property), with adversary resource fraction β and network delay parameter, the fraction of honest blocks/transactions excluded beyond time T is at most f(β, T) → 0, and wasted execution is bounded — with explicit constants.
2. **Lower bound:** A matching adversarial strategy showing no content-destruction-resistant-with-rewards rule can do better, i.e., an inherent trade-off curve between reward-extraction resistance and liveness.
3. **Negative result (variant):** An explicit attack showing a non-negligible exclusion/waste at adversary cost strictly less than the honest loss — proving reward resistance ≠ liveness and motivating a modified rule.
4. **Construction:** A modified ledger rule with provable simultaneous reward-extraction resistance and O(β)-bounded exclusion, with a simulation study (event-driven DAG simulation replicating GHOSTDAG-style settings) demonstrating the bound is meaningful.

A referee would want a formal model, explicit attack strategies, and either the theorem or a reproducible experimental demonstration.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/3c41dc1c-8386-4e32-a8fb-659dda525cc5?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000252.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000252)
