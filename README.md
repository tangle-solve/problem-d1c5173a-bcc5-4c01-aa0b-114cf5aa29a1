<!-- solve-problem:d1c5173a-bcc5-4c01-aa0b-114cf5aa29a1 -->
# Foreknowledge-free whale-transaction attacks

A [Tangle Solve problem](https://solve.tangle.tools/problems/d1c5173a-bcc5-4c01-aa0b-114cf5aa29a1).

## What is the problem?

In Bitcoin-style proof-of-work systems, a "whale transaction" is one carrying an unusually large fee. Bar-Zur et al. (2023) showed that a rational miner who *knows* a whale transaction has appeared (or is coming) can profitably deviate from honest mining — e.g., withholding its own blocks to fork the chain and orphan/capture the whale fee — and they trained a deep-RL agent ("WeRLman") to execute this attack profitably.

The open problem: **WeRLman's measured attack profitability may be an artifact of information leakage in its training/evaluation simulator.** The RL agent is fed state features (e.g., the whale transaction's presence and fee) that implicitly encode *future* transaction arrivals — information no real miner possesses at decision time. The task is to **measure the true value of advance transaction information** by comparing attack policies under an information-diet experiment:

- Formal setting: a stochastic blockchain mining game (block arrival as a Poisson process, transaction/fee arrival process, attacker with hash power α < 0.5, reward = fees + block rewards, observable state = chain state and mempool *at decision time*).
- Constraint: two attack evaluations must be **observationally identical** — the baseline attacker and the informed attacker see the same history up to time t, with no leakage of arrivals at times > t.
- Deliverable: quantify the gap between attack profitability with and without foreknowledge; equivalently, re-derive (via RL or analysis) a profitable withholding policy that uses only legitimately observable information, or show none exists for realistic parameters.

Precise interpretation: "foreknowledge-free" means the attacker's policy is measurable with respect to the natural filtration of the arrival process up to the current decision time (a *causal/non-anticipative* policy), whereas WeRLman's simulator features break this.

## Why does it matter?

If whale attacks remain profitable without foreknowledge, high-fee transactions systematically expose permissionless blockchains to rational withholding attacks — a concrete security argument for fee-smoothing mechanisms (e.g., monotonic fee rules as in Ethereum EIP-1559-style designs). If profitability evaporates without foreknowledge, then WeRLman-style results overstate the threat and the community's threat model must be revised. Methodologically, this is a flagship case of *benchmark/simulator leakage* invalidating RL-derived security claims, so the answer affects how all RL-based blockchain attack studies are evaluated.

## What's already known?

- **Bar-Zur et al. (2023), WeRLman:** trains a deep-RL attacker in a Bitcoin-like environment with whale transactions; the learned policy withholds blocks around whales and reports substantial relative reward gain over honest mining (verify exact figures from the paper). The state includes whale-transaction features.
- **Sarenche et al. (2025), arXiv:2411.11702:** a follow-up analysis identifying that WeRLman's model contains **implicit reward predictability** — features/simulator dynamics that effectively reveal future fee events to the agent, i.e., non-causal information. This sharpens the problem but (per the census) does not complete the causal re-evaluation; it identifies the leakage channel. (Verify what, if anything, Sarenche et al. already repair or re-measure before redoing it.)
- Classical baselines: selfish mining (Eyal–Sirer 2014; Sapirshtein et al. 2016 MDP formulation) is well understood for block rewards, but fee-based/whale-conditional withholding without future knowledge is not characterized.
- Gap: no published measurement of whale-attack profitability under a strictly causal information structure.

## What would count as solving it?

An **experimental/theoretical characterization of the value of foreknowledge** in the whale-attack game, specifically:

1. A modified environment (or proof of leakage in the original) where the attacker's observation is restricted to the natural filtration — no feature or simulator dynamic that correlates with not-yet-arrived transactions beyond what is statistically inferable from the past.
2. A retrained/analytically-derived causal attack policy, with relative reward vs. honest mining reported across α and whale frequency/size parameters, using identical seeds and arrival processes for the with/without-foreknowledge comparison (paired evaluation).
3. A quantified gap: profit(oracle-informed attacker) − profit(causal attacker), with confidence intervals, and a verdict on whether the causal attacker still beats honest mining.

A referee accepts: reproduction of WeRLman's leaky result, demonstration of the leakage channel, and a leakage-free experiment showing either attenuated or preserved profitability.

## How to start

1. Read Bar-Zur et al. (2023) Sections on the environment/state design and reward accounting; enumerate every state feature and simulator assumption, flagging anything not computable from a real miner's view.
2. Read arXiv:2411.11702 for the specific predictability mechanism identified (whether it is the fee feature, deterministic arrival scheduling, or reward shaping).
3. Rebuild the environment with a strict interface: `action_t` may depend only on events with timestamp ≤ t; obtain WeRLman's code if released (verify) and diff the state construction.
4. Train matching RL agents (leaky vs. causal) with common random numbers on the arrival processes; bootstrap CIs on the reward gap. Main obstacle: RL variance — use paired seeds and long horizons.

## Key references

- Bar-Zur, R., Bar-Zur, R., et al. (2023). "WeRLman: To Tackle Whale (Transactions), Go Deep (RL)." IEEE S&P 2023. DOI: 10.1109/SP46215.2023.10179444. Trains a deep-RL selfish-mining variant that exploits high-fee ("whale") transactions in a Bitcoin-like simulator and reports profitable deviations. https://doi.org/10.1109/SP46215.2023.10179444
- Sarenche et al. (2025), arXiv:2411.11702. Analyzes WeRLman's model and identifies implicit reward predictability/leakage channels motivating this problem. https://arxiv.org/abs/2411.11702
- (Background, add as needed) Eyal & Sirer 2014, "Majority is not Enough" (selfish mining baseline); Sapirshtein, Sompolinsky & Zohar 2016 (optimal selfish-mining MDP) — standard causal-information baselines. (verify authors/venues before citing in output.)

## Warnings

- The leakage is subtle: it need not be a literal "future fee" feature; deterministic transaction scheduling, reward shaping that correlates with imminent whales, or fixed-seed training can all make the environment predictable. Audit dynamics, not just feature names.
- Do not compare leaky and causal agents across different random seeds; the arrival process must be paired, else the gap estimate is noise-dominated.
- WeRLman's environment details (exact observable features, arrival model) matter enormously; if the code is unavailable, reconstruction choices may change conclusions — document them.
- "Profitable" is relative reward vs. honest mining under the same α; check whether WeRLman's metric includes block rewards and difficulty adjustment, and match it.
- Verify precisely what Sarenche et al. (2025) already fixed — the census record is ambiguous on whether they only diagnosed or also repaired the leakage.

---
*`BCGT-000007` · flagged for review*

## Acceptance guidance

An **experimental/theoretical characterization of the value of foreknowledge** in the whale-attack game, specifically:

1. A modified environment (or proof of leakage in the original) where the attacker's observation is restricted to the natural filtration — no feature or simulator dynamic that correlates with not-yet-arrived transactions beyond what is statistically inferable from the past.
2. A retrained/analytically-derived causal attack policy, with relative reward vs. honest mining reported across α and whale frequency/size parameters, using identical seeds and arrival processes for the with/without-foreknowledge comparison (paired evaluation).
3. A quantified gap: profit(oracle-informed attacker) − profit(causal attacker), with confidence intervals, and a verdict on whether the causal attacker still beats honest mining.

A referee accepts: reproduction of WeRLman's leaky result, demonstration of the leakage channel, and a leakage-free experiment showing either attenuated or preserved profitability.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/d1c5173a-bcc5-4c01-aa0b-114cf5aa29a1?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000007.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000007)
