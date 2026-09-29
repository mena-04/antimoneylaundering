# Hybrid Classical-Quantum AML Suspicious Subgraph Discovery

Team 36-QCI, Quantum Dice Use Case Challenge 2026 (Phase 2 prototype).

Finds suspicious account clusters in a transaction network by combining risk-guided graph expansion with local QUBO problems solved by classical, quantum-simulated, and probabilistic (p-bit) solvers.

## Pipeline
1. **Data:** IBM AML HI-Small (5.08M transactions, 518K accounts), loaded from Hugging Face. A 500K-transaction sample keeps all laundering transactions plus surrounding context, giving a graph of 196,343 accounts and 183,004 edges.
2. **Risk scoring:** edges scored on amount, frequency, bidirectional flow, and payment format; nodes on incident edge risk, degree, flow balance, and activity. Ground-truth labels are not used in scoring, only in evaluation.
3. **Seeds and expansion:** the 10 highest-risk accounts seed a greedy neighborhood expansion (max 30 nodes, depth 3).
4. **QUBO formulation:** each candidate subgraph becomes a QUBO that rewards node risk and suspicious edges and penalizes deviation from a target cluster size.
5. **Solvers compared:** exact exhaustive search (small instances), greedy local search, simulated annealing, QAOA p=1 (statevector simulation, capped at 16 variables), p-bit v1 (linear schedule), and p-bit v2 (parallel tempering).
6. **Consolidation and visualization:** overlapping clusters are merged by Jaccard similarity, and the top case is highlighted on the graph.

## Results (4 candidate regions)
- On the 8-variable region, where exact search is feasible, QAOA, greedy search, and simulated annealing all matched the optimum. P-bit v2 was within 0.415 of the optimum and P-bit v1 within 3.16.
- On the 30-variable regions, greedy search and simulated annealing reached the lowest energies. P-bit v2 came close (within about 0.2-4 energy units) but ran slower (~15 s vs 2-5 s), and P-bit v1 was worse.
- QAOA ran on a 16-node restricted subgraph, so its energies are not directly comparable on the larger regions. It matched exact on the 8-variable one.
- The p-bit solvers did not outperform the classical baselines in this prototype.
- Label precision on larger regions was low (roughly 0.09-0.36 for the classical solvers) while recall was high (up to 1.0), so the clusters over-include accounts.

## Running
Runs in Google Colab. Requires `numpy pandas networkx matplotlib scipy`. The dataset downloads automatically from Hugging Face in the first cells. Run `aml_phase2.ipynb` top to bottom.

## Limitations
- Small candidate regions, so exact ground truth exists only for one of them.
- QAOA and p-bits are simulated on classical hardware, not run on quantum or p-bit devices.
- Single run per solver with fixed seeds, no repeated-trial statistics.
