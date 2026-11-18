# Background: Decision Transformer, backward and forward

Chen et al., *Decision Transformer: Reinforcement Learning via Sequence Modeling*, NeurIPS 2021.

One idea runs through the whole deck: **credit flows backward, actions flow forward.** Classical RL learns the backward flow; Decision Transformer (DT) computes it once and spends all its learning on the forward flow. The lineage fits the same frame, with what came before and what came after.

**Analogy.** You learn chess only from recorded games, each tagged with the player's final score. TD learning tries to work out the value of every position by passing credit back through the games, move by move. DT sums each game's score once, labels every move with "score still to come," and learns "in this position, a player headed for score X played this." At test time you ask for a grandmaster's score and it imitates such players, move by move, forward.

## Backward: the problem (slides 2 and 8)

Offline RL learns a policy from a fixed dataset with no new interaction. TD methods such as Q-learning learn the backward flow through Bellman backups. Each update pulls value back by one step, using the model's own estimates as targets. Offline, this goes wrong in three ways:

- **Errors compound** because targets are bootstrapped.
- **Untried actions look too good**, because the max in the target favors actions the data never covered.
- **Sparse, delayed rewards travel slowly**, one backup at a time.

In CS 258 terms, offline Q-learning has all three legs of the deadly triad: function approximation, bootstrapping and off-policy data. The paper names the triad as its reason to avoid bootstrapping. Earlier offline methods (BCQ, BEAR, BRAC, CQL) keep the TD backward pass and add constraints or pessimism to make it safe.

## Backward, done once: return-to-go (slide 4)

DT still needs credit to reach early steps, but it gets it without learning:

R̂ₜ = rₜ + rₜ₊₁ + … + r_T

This is computed by summing rewards backward from the end of each logged trajectory. It is exact for that trajectory, involves no estimates, and is undiscounted. It is the Monte Carlo return Gₜ from lecture with γ = 1. One indexing difference: R̂ₜ includes rₜ, while Sutton and Barto's Gₜ starts at rₜ₊₁.

Why this works is **hindsight relabeling**. Every trajectory is a correct demonstration of how to achieve *its own* return, so mixed-quality data becomes correctly labeled training data.

## Forward: the model (slides 5 and 6)

- **Tokens.** The last K timesteps form a sequence (R̂, s, a, R̂, s, a, …), so 3K tokens. K is 20 on Gym and 30 on Atari (50 on Pong).
- **Model.** A small GPT-style causal transformer: one embedding per token type, a conv encoder for Atari frames, and a learned timestep embedding. The causal mask means each token looks back at history and the model predicts forward.
- **Training.** Predict the logged action from the state token. Cross-entropy for discrete actions, MSE for continuous. There is no Bellman update anywhere: this is behavior cloning conditioned on return.
- **Acting.** Pick a target return. The model acts, the environment returns a reward, and the target is reduced by that reward. Repeat. The return-to-go counts down as the episode runs forward, like a GPS's remaining distance.

## Results (slides 7 and 8)

- **Atari (1% of DQN-replay data).** DT is competitive with CQL: ahead on Breakout and Seaquest, close on Pong, clearly behind on Qbert.
- **D4RL Gym.** DT has the best average among the compared methods: 74.7 vs 63.9 for CQL.
- **Percentile BC.** BC on only the top slice of trajectories nearly matches DT on large D4RL datasets. DT's advantage is clearest in low-data settings and on Atari. The paper reports this itself.
- **Credit assignment, where the backward story pays off.**
  - Key-to-Door: about 72% success from 1K random trajectories, against about 13% for CQL.
  - Hopper with all reward moved to the end: DT barely changes while CQL collapses. CQL must crawl that late reward back step by step; DT already has it in every return-to-go token.
- **Context matters.** K = 1 is much worse on Atari.

## Where the backward pass falls short: limitations (slide 9)

DT's backward pass stays inside each trajectory, so it cannot **stitch**. If the data contains trips A→B and B→C but never A→C, dynamic programming can join them at B, because its backward pass crosses trajectories through shared states. DT's return-to-go at B only reflects the trip it came from.

Other limitations:

- **Stochastic environments.** A high return may have been luck, and conditioning on it imitates the luck.
- **Hand-picked target.** The target return is set per task.

## Lineage (backup slide 10)

### Looking back

| Year | Work | What DT inherits |
|---|---|---|
| 1989 | ALVINN (Pomerleau) | Behavior cloning: imitate logged actions |
| 2017 | Hindsight Experience Replay (Andrychowicz et al.) | Relabel data with what was actually achieved (a conceptual ancestor) |
| 2017–20 | Transformer; GPT, GPT-2, GPT-3 | Causal next-token prediction and prompting |
| 2019 | Upside-Down RL (Schmidhuber; Srivastava et al.) | Direct ancestor: command a policy with a desired return and horizon |
| 2019 | Reward-Conditioned Policies (Kumar, Peng, Levine) | Same idea with an MLP, trained by supervised learning |
| 2019–20 | BCQ, BEAR, BRAC, AWR, CQL | The offline TD baselines DT is compared against |
| 2020 | D4RL (Fu et al.); DQN-replay (Agarwal et al.) | The benchmarks |

### Looking forward

| Year | Work | Relation to DT |
|---|---|---|
| 2021 | Trajectory Transformer (Janner, Li, Levine) | Concurrent sibling: also models rewards and plans with beam search |
| 2021 | RvS (Emmons et al.) | A tuned MLP with return or goal conditioning often matches DT; the transformer isn't essential |
| 2022 | Online DT (Zheng, Zhang, Grover) | Fine-tunes DT with online interaction |
| 2022 | Multi-Game DT (Lee et al.) | One model for 41 Atari games |
| 2022 | Brandfonbrener et al. | Return-conditioned SL needs near-deterministic dynamics and good coverage; it can't stitch |
| 2022 | Paster et al., "You Can't Count on Luck" | Stochastic environments mislead return conditioning |
| 2023 | Q-learning DT (Yamagata et al.) | Relabels return-to-go with learned Q-values: **a learned backward pass returns**, to enable stitching |
| 2023 | Elastic DT (Wu et al.) | Addresses stitching by adapting how much history the model sees |

**The arc.** DT removed the learned backward pass and showed how far a forward sequence model could go. Follow-up work found the gains come mainly from return conditioning plus supervised learning, and that the missing backward pass is exactly what stitching needs. The strongest descendants bring some of it back.

This lineage covers work through 2023.
