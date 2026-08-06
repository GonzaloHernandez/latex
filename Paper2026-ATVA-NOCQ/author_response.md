# Global Response to the Reviewers

We thank the reviewers for their feedback and positive remarks regarding our modular architecture, clarity of writing, and empirical evaluation against state-of-the-art solvers. Because NOCQ is the first tool to address $\omega$-regular games from a Constraint Programming (CP) approach and is intended to be continuously and fully supported, the editorial points raised across the board are very much appreciated, as they have allowed us to improve this tool paper.

# Response to Reviewer A:

**Q1:** As we describe in the first paragraph of Section 4.2, on page 10, NOCQ only computes memoryless strategy solutions and, as such, solves games under that restriction, which means that optimal strategies may not be computed in some cases where qualitative and quantitative goals are required (see further clarifications in the responses below); various examples of this situation can be found in the literature. 

*Memoryful Path Exploration via the LCG Propagator:* While the CP variables can be used to produce a positional strategy for EVEN, what the tool computes is the winning regions of the game. To achieve this, the custom Lazy Clause Generation (LCG) propagator does  memoryful (but finite) reasoning using such variables:
- Dynamic Path Unrolling: During filtering, the propagator executes an on-the-fly graph traversal tracking crossed edges and the exact accumulated energy (pathW).
- Cycle Detection: When a cycle closes, the propagator evaluates the multi-objective condition over the entire path history. By exploring this unrolled path space, the propagator performs a memoryful analysis to anticipate a memoryful (but finite) opponent strategy. If a violation is found (e.g., ODD forcing a negative energy loop), it generates a conflict explanation clause to prune the opponent's illegal variable choices. When optimal strategies are memoryless, such information is sufficient to synthesize optimal strategies; otherwise, it may not, even if the computed winning strategies are correctly identified.

**Q2:** Tooling Scope: We thank the reviewer for pointing out ltlsynt (Spot) and Totzke’s solvers. Our evaluation intentionally focused on Oink as it represents the direct state of the art in our targeted parity game tracks, consistently winning recent SYNTCOMP editions. While generalized synthesis engines like ltlsynt handle parity structures, they rely on full LTL synthesis and heavy automata manipulation, making direct performance comparisons outside the scope of this tool paper. We will update our Related Work to formally contextualize NOCQ against these other frameworks.

# Response to Reviewer B

**Q1:** To clarify the boundary between generic CP engineering and our specialized contributions:

*Generic CP & LCG Infrastructure:* Boolean tracking, conflict analysis, clause learning, and backtracking are native to the Chuffed solver.

*Specialized Propagators & Parallel Novelty:* Standard solvers cannot reason about infinite plays or cyclical energy weights. Our technical novelty lies in the custom propagator performing lazy, path-unrolled cycle detection. Crucially, our architecture solves for both player perspectives in parallel, allowing the solver to immediately exploit the fastest path to a solution from either side.

# Response to Reviewer C

**Q1:** NOCQ represents a major architectural leap that goes far beyond the plain parity game solver in [21]:

*New Quantitative Engine:* Supporting energy and mean-payoff objectives required a complete core redesign and engineering a complex, custom LCG propagator for dynamic path-unrolling since these new conditions were completely absent in [21].

*Tooling Maturity:* NOCQ is built from scratch as a standalone toolchain that features a robust parser, modular backends, and fully integrated benchmarking suites.

# Response to Reviewer D

**Q1:** Experimental Presentation: We agree that the identical values in Table 3 are an artifact of PAR2 timeout penalties hitting the threshold in the same subsets. If preferred, we will change the presentation to a cumulative survival plot (% of problems solved against time) for a penalty-independent comparison. We will also add an explanatory footnote.

**Q2:** Corrections: We will fix the typo ("Goas" $\rightarrow$ "Goals") and update Algorithm 1 so that NOCEager correctly reads NOCFilter.

**Q3:** Hardware Requirements: While our benchmarking cluster has 250 GB of RAM per node, our Slurm execution scripts strictly limit each instance to 8 GB of RAM. NOCQ does not require massive hardware; we will clarify this memory cap in the instructions. 