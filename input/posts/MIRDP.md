Title: Unbounded Integrality Gaps and Additive Hardness for the Skiving Stock Problem (MIRDP)
Published: 07/10/2026
Tags: [Math] 
---

# OpenAI's Math 

OpenAI publishing their [OpenAI Math](https://github.com/openai/math) repository enables us to search for problems that are related to and impacted by these proofs. 


Here I present one that I found and selected by talking to Fable 5.1. 

## Unbounded Integrality Gaps and Additive Hardness for the Skiving Stock Problem

**Eric Winnington**

7 October 2026

### Abstract

We prove that the additive integrality gaps of both the ordinary and the proper
pattern linear programming relaxations of the one-dimensional skiving stock problem are
unbounded. In particular, this refutes Zak’s modified integer round-down conjecture for
the ordinary relaxation and also disproves its analogue for the stronger proper relaxation.
For every fixed integer c ≥0, distinguishing instances that cover B bins from instances
that cannot cover B −c bins is NP-hard. Consequently, a deterministic polynomial-time
algorithm with an absolute additive guarantee exists if and only if P= NP.
The proofs apply a cardinality-saturated complement transfer to the bin-packing
construction of OpenAI (24 September 2026). On an instance with 5B items, the map
bi = 2 −ai, with covering threshold 9, identifies feasible five-item packing and covering
configurations. Saturation of the packing LP forces exact use of every item and preserves
both covering LP values. Pooling all exceptional groups and unused items converts
a covering deficit d into a packing excess at most ⌈3d/2⌉ for the source’s size range.
More generally, we prove an abstract transfer theorem and exact formulas for both
integer optima in terms of a common uniform-hypergraph matching deficiency. The
counterexamples and hardness instances can be restricted to an arbitrarily narrow fixed
neighborhood of one fifth of the covering threshold. The source-independent transfer
is proved here; the source result and the scope of its supplied Lean formalization are
identified separately.

[skiving_stock_integrality_gaps.pdf](https://github.com/user-attachments/files/33156312/skiving_stock_integrality_gaps.pdf)
