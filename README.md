# cs258-pres

CS 258 / EE 227 (Reinforcement Learning, UCR) paper presentation:
**Decision Transformer: Reinforcement Learning via Sequence Modeling** (Chen et al., NeurIPS 2021).

9 slides, about 8 minutes, plus one backup slide for Q&A, following the course structure:
Motivation & Problem → Main Idea → Method → Results → Conclusion.

**Thread through the talk:** credit flows *backward*, actions flow *forward*. TD learns the backward pass through Bellman backups. DT computes it once, in hindsight, as return-to-go, and learns only the forward pass. Small ← / → tags on the slides mark each side. [BACKGROUND.md](BACKGROUND.md) has the full paper overview and lineage in the same frame.

![Overview of all slides](screenshots/overview.png)

## Contents

- `deck/` — the slide source (one HTML section per slide, shared `deck.css`, `deck.json` index). Speaker notes with timings are in each slide's `<aside>`.
- `BACKGROUND.md` — paper overview and lineage (looking back / looking forward).
- `screenshots/` — rendered previews.

## Timing

| # | Slide | Time |
|---|---|---|
| 1 | Cover / pitch | 0:30 |
| 2 | Motivation & problem (hospital example, deadly triad) | 1:00 |
| 3 | Main idea & contributions | 1:00 |
| 4 | Tokens & return-to-go (Gₜ with γ = 1) | 0:50 |
| 5 | Architecture & training (return-conditioned BC) | 1:00 |
| 6 | Test-time loop | 0:50 |
| 7 | Benchmark results | 1:10 |
| 8 | Credit assignment results | 0:50 |
| 9 | Conclusion, limitations & the backward/forward thread | 0:50 |
|   | **Total** | **≈8:15–8:30** |
| 10 | Backup: lineage, looking back / forward | Q&A only |

All numbers are from Chen et al. (2021), Tables 1, 2, 6 and 7. The hospital scenario is illustrative; the paper does not evaluate it.
