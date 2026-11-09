# Context: Generalized Büchi games by layers + round-robin + C_MSCC (sonar-cp / mnoc)

## 1. Problem
- Two-player game on a finite arena: vertices V = V_E ∪ V_O (EVEN / ODD), edges E, initial vertex init, no dead ends.
- EVEN's objective: **conjunction of Büchi conditions** (generalized Büchi): visit every set B_1..B_k infinitely often.
- ODD's objective (dual): eventually avoid some B_j forever (disjunction of co-Büchi).
- Solver: `mnoc` in sonar-cp (C++17 lazy clause generation: CDCL SAT core + CP propagators, 1-UIP learning, no restarts). Solves for EVEN only: **SAT ⇒ EVEN wins from init**, **UNSAT ⇒ ODD wins**.

## 2. Encoding: one layer per condition
- Layer k has its own copy of the NOC variables and clauses for condition B_k:
  - V_k[v]: vertex v is reached in layer k; E_k[v][i]: EVEN selects edge i of v in layer k.
  - V_k[init].
  - EVEN v: V_k[v] → **exactly one** E_k[v][i] (at-least-one clause + at-most-one encoding; pairwise or sequential for high degree); E_k[v][i] → V_k[w].
  - ODD v: V_k[v] → V_k[w] for every successor w.
  - NOC propagator of layer k (Büchi as parity: forbids cycles in layer k that miss B_k), with path-based reason clauses.
- Layers are independent: no switching between layers during a play.
- **Exactly-one per layer** (changed from at-least-one): costs nothing for completeness, since each layer's own condition (Büchi; later parity/Rabin) is positional for EVEN. It bounds |sel(v)| ≤ k and keeps R small. Caveat: it would break completeness if a single layer carried a non-positional condition (Streett, Muller).

## 3. EVEN's combined policy: round-robin
- sel(v) = { w : V_k[v] ∧ E_k[v→w] for some layer k }, so 1 ≤ |sel(v)| ≤ k for reached EVEN v.
- Each EVEN vertex v has a counter c_v and a fixed order w_0..w_{m-1} of sel(v) (m = |sel(v)|). At each visit to v, EVEN moves to w_{c_v} and sets c_v := (c_v + 1) mod m. **c_v changes only when the play is at v.**
- Memory: one counter per EVEN vertex (product of |sel(v)|, arena-dependent but compact). A textbook arena-independent alternative can be read off the same layers: a global phase counter j ∈ {1..k}; play layer j's edge until B_j is visited, then j := j+1.

## 4. C_MSCC (MSCCPropagator): check on the union of the layers
- Layers alone are unsound for conjunctions: ODD may trap the combined play in a set missing some B_j.
- For each j:
  1. R = vertices reachable from init using TRUE selected edges plus all ODD edges.
  2. Peel R \ B_j: remove an EVEN vertex if one of its edges leaving the current set can still be selected (not FALSE in some layer); remove an ODD vertex if it has no successor left in the set.
  3. Non-empty remainder S_j = **trap** ⇒ conflict.
- Reason clause: ¬(TRUE path literals from init into S_j) ∨ (FALSE escape literals: for each EVEN vertex in S_j, edge leaving S_j, and layer k, whichever of V_k[v] / E_k[v][i] is FALSE). Root-fixed literals dropped; empty clause ⇒ UNSAT at root.
- **Eager mode** (default): runs at every propagation fixpoint on partial assignments (a trap reached through TRUE edges with all escapes FALSE survives every completion). Must also run on the final complete assignment. **Lazy mode**: only in `finalCheck` on complete assignments.
- Brancher (ReachBrancher): layer by layer, tries TRUE on the first undecided edge of a reached EVEN vertex with no selected edge; with C_MSCC active, a completion pass decides remaining V/E variables FALSE one decision at a time (so reason clauses are fully assigned; learning can later flip them).
- C_MSCC is **conservative**: a reported trap need not be a real loss for round-robin (the path into it may require specific EVEN choices). This only affects search, not soundness; completeness is argued separately.

## 5. Definitions used in the proofs
- **Closed set** X ⊆ V (w.r.t. the current selection): (C1) every EVEN v ∈ X has sel(v) ⊆ X; (C2) every ODD v ∈ X has a successor in X. Intuition: once inside X, EVEN's selected edges cannot leave and ODD can always stay. Closed alone is not losing; **closed + inside R \ B_j = trap** (ODD wins).
- S_j = result of peeling R \ B_j. C_MSCC passes ⇔ S_j = ∅ for all j.

## 6. Soundness (SAT ⇒ EVEN wins): configuration proof
- **Lemma 1 (peeling keeps closed sets).** If X is closed and X ⊆ R \ B_j, then X ⊆ S_j. Induction on peeling steps: an EVEN v ∈ X has no selected edge leaving the current set (sel(v) ⊆ X); an ODD v ∈ X keeps a successor in X.
- **Lemma 2 (play stays in R, never stuck).** Every vertex of R has V_k true in some layer (induction: V_k[init]; E_k[u→w] → V_k[w]; ODD V_k[u] → V_k[w]), so exactly-one gives sel(v) ≠ ∅ for EVEN v ∈ R; EVEN only uses selected edges, R is closed under them and under ODD edges.
- **Configuration** = (current vertex, all counter values). Finitely many: ≤ |V| · ∏ |sel(u)|. Under round-robin EVEN's move is determined by the configuration; only ODD chooses.
- **Key lemma (repeated configuration ⇒ closed set).** If a play segment starts and ends in the same configuration, the set X of vertices visited in it is closed. (C1): c_u returns to its start value and moves only on visits to u, so u is visited a multiple of m_u times, hence ≥ m_u consecutive visits, which use every edge of sel(u); each leads to the next vertex of the segment, in X. (C2): every ODD vertex in the closed walk is followed by a vertex in X.
- **Theorem.** If C_MSCC passes, every round-robin play from init visits every B_j infinitely often, against any ODD strategy (with or without memory).
  Proof: suppose the play avoids B_j from step t. By pigeonhole on configurations, some configuration repeats at t ≤ s < s′. The segment's vertex set X is closed (key lemma), non-empty, ⊆ R (Lemma 2), avoids B_j; by Lemma 1, X ⊆ S_j = ∅. Contradiction. ∎
- Corollary: if a losing play existed, it would have a lasso of total length ≤ |V| · ∏ |sel(u)| (useful for a counterexample checker).
- Note: the proof uses only the structural clauses and C_MSCC, **not the per-layer NOC propagators**. For generalized Büchi these act as dominance/guidance constraints (worth benchmarking mnoc with them off).

## 7. Completeness (UNSAT ⇒ ODD wins): sketch, to formalize
- Let W be EVEN's winning region (init ∈ W). For each k take σ_k: a positional Büchi strategy for B_k inside W (attractor to B_k ∩ W, and on B_k vertices any edge staying in W).
- Set V_k = W and E_k = σ_k (exactly one edge per vertex per layer). Clauses hold since σ_k and ODD edges stay in W.
- Layer NOC holds: a cycle avoiding B_k would need the B_k-attractor rank to decrease forever.
- No trap: R ⊆ W. If S ⊆ R \ B_j were a trap, take its vertex of minimum B_j-attractor rank: if EVEN, σ_j leaves S to lower rank (selectable escape) so it is peeled; if ODD, all successors have lower rank, so none in S and it is peeled. Contradiction.
- To write carefully: vertices in B_k, rank definition inside W.

## 8. Memory facts (literature)
- EVEN (conjunction side) needs memory: generalized Büchi k, Streett k! (index appearance record), Muller up to |C|! (LAR).
- ODD (disjunction side) is **positional**: disjunction of co-Büchi / of parity is Rabin-type, and Rabin objectives are positional for their holder (Klarlund; Zielonka's criterion: positional for a player iff her losing sets are closed under union — EVEN's generalized Büchi winning sets are upward-closed). Generalized parity: Chatterjee, Henzinger, Piterman, FoSSaCS 2007.
- Consequence: an UNSAT certificate can be one edge per ODD vertex.

## 9. Status
- Verified against nested-attractor reference solver (utils/gbuchi.py): 0 unsound, 0 incomplete on tens of thousands of vertices (n=10–200, k=1–5).
- Eager C_MSCC: two n=500, k=3 instances 9.8 s → 0.05 s and >300 s → 1.1 s. Full benchmark pending (slurm).
- Lazy-mode bottlenecks: completion pass redone after every final-check conflict; ReachBrancher rescans all layers from vertex 0 each call.
- Decided, to implement: exactly-one (AMO) per layer.

## 10. Open questions / next steps
- Formalize the completeness proof.
- Forcing propagation in C_MSCC (force the only remaining selectable escape TRUE).
- Benchmark with per-layer NOC propagators disabled.
- Policy output + independent checker (configuration-graph lasso search; ODD positional certificate for UNSAT).
- Brancher order and polarity.
- **Parity / Rabin conjunctions**: not upward-closed, so the closed-set argument is not enough. C_MSCC must rule out closed sets whose maximum priority for some condition j is odd (not just "avoids B_j"). Per-layer exactly-one stays complete (parity/Rabin positional for EVEN).
