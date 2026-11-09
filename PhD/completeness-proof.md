# Completeness (EVEN wins ⇒ SAT)

## Statement
Let M be the full model: for every layer k, the NOC clauses, exactly-one and C_NOC_k, plus C_Trap.

**Theorem (completeness).** If EVEN has a winning strategy from init for B_1 ∧ … ∧ B_k, then M is satisfiable.

Together with soundness (SAT ⇒ EVEN wins): **EVEN wins from init ⇔ M is SAT**. By determinacy, UNSAT ⇒ ODD wins.

Notation (same as soundness):
- sel(v): edges of EVEN vertex v selected in some layer (V_k[v] ∧ E_k[v→w] TRUE for some k).
- R: vertices reachable from init using selected edges and all ODD edges.
- Closed set X: every EVEN v ∈ X has sel(v) ⊆ X; every ODD v ∈ X has a successor in X.
- Trap for B_j: a closed set X ⊆ R \ B_j. C_Trap is satisfied iff no trap exists for any j.

## Idea
Build one explicit solution of M from EVEN's winning region W, using one attractor ("move closer to B_k") strategy per layer.
- Each layer satisfies C_NOC_k because attractor ranks strictly decrease outside B_k, so every cycle meets B_k.
- The union of layers has no trap because the minimum-rank vertex of any candidate trap would have to leave it.

## Setup
Let W be EVEN's winning region for B_1 ∧ … ∧ B_k; init ∈ W.

**Fact 0 (winning regions).** Every ODD vertex in W has all its successors in W; every EVEN vertex in W has at least one successor in W. So the restriction G[W] is an arena with no dead ends, and EVEN wins B_1 ∧ … ∧ B_k from every vertex of G[W].

## Lemma A: each attractor covers W
For each k, let A_k be EVEN's attractor to B_k ∩ W inside G[W]. Then A_k = W.

*Proof.* Suppose U = W \ A_k ≠ ∅. The complement of an EVEN-attractor is a trap for EVEN: from U, ODD keeps the play in U forever. U contains no B_k vertex, so ODD wins co-Büchi B_k from U, and EVEN cannot win B_1 ∧ … ∧ B_k there. Contradiction with U ⊆ W. ∎

**Ranks.** Each v ∈ W gets an attractor rank r_k(v) ∈ ℕ, the number of steps EVEN needs to force the play into B_k. r_k(v) = 0 iff v ∈ B_k. For v ∉ B_k:
- EVEN v: some successor in W has rank < r_k(v);
- ODD v: every successor has rank < r_k(v).

**Strategy σ_k (positional, on W).**
- EVEN v ∉ B_k: a successor of lower rank.
- EVEN v ∈ B_k: any successor in W.

## The witness assignment
For every layer k:
- V_k[v] = TRUE iff v ∈ W;
- E_k[v → σ_k(v)] = TRUE for every EVEN v ∈ W; all other E_k variables FALSE.

Hence sel(v) = {σ_1(v), …, σ_k(v)} ⊆ W for every EVEN v ∈ W, and |sel(v)| ≤ k.

## Lemma B: each layer's constraints hold
- **V_k[init]:** init ∈ W.
- **Exactly-one (EVEN):** each EVEN v ∈ W has exactly one TRUE edge; EVEN v ∉ W has V_k FALSE and all E_k FALSE.
- **E_k[v→w] → V_k[w]:** σ_k(v) ∈ W.
- **ODD, V_k[v] → V_k[w]:** Fact 0.
- **C_NOC_k:** in layer k's graph (σ_k edges at EVEN vertices, all edges at ODD vertices, inside W), every step outside B_k strictly lowers r_k. A cycle returns to its start, so it cannot lower the rank at every step; hence every cycle meets B_k. This is what C_NOC_k requires for Büchi (B_k priority 2, others priority 1). ∎

## Lemma C: no trap
- **R ⊆ W:** init ∈ W; selected edges stay in W; ODD edges from W stay in W (Fact 0).
- Suppose X is a non-empty closed set with X ⊆ R \ B_j. Take v ∈ X with minimum r_j. Since v ∉ B_j, r_j(v) ≥ 1.
  - EVEN v: σ_j(v) ∈ sel(v) ⊆ X (X closed), and r_j(σ_j(v)) < r_j(v).
  - ODD v: X closed gives a successor w ∈ X, and r_j(w) < r_j(v).

  Either way X contains a vertex of lower rank than v, contradicting minimality. So no trap exists and C_Trap is satisfied. ∎

## Proof of the theorem
The witness satisfies every layer constraint (Lemma B) and C_Trap (Lemma C), so M is satisfiable. ∎

## Intuition
r_j counts the floors above an exit (B_j). Any closed room has a lowest point, and from there a staircase always goes down: EVEN's layer-j edge, or every ODD move. So no closed room can avoid the exit forever.

## Remarks
- Uses only attractors and Fact 0: no positional result for ODD and no determinacy (determinacy is only needed for "UNSAT ⇒ ODD wins").
- C_NOC_k matters here only to check the witness passes it: the layers filter solutions but never remove every winning one. (Soundness does not use C_NOC_k at all.)
- The witness uses one edge per layer, so it fits exactly-one and gives |sel(v)| ≤ k.
- To check against the implementation:
  - C_NOC_k is exactly "every cycle in layer k meets B_k", including how it treats TRUE V_k vertices not reachable from init in layer k;
  - the at-most-one encoding allows all E_k[v][·] FALSE when V_k[v] is FALSE.
