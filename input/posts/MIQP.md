Title: It’s a proportional allocation - how hard can it be? Going from Water filling to QP
Published: 16/4/2025 10:00
Tags: [MIQP, MILP, LinearProgramming] 
---

# It’s a proportional allocation - how hard can it be? MIQP

Alice, Bob and Charlie buy a pizza and they each put down a part of the price, respectively 50%, 30% and 20%. Pizza arrives and they slice it up and eat it. But Alice gets full after eating 40% of her 50% slice, how do we allocate the remaining 10% slice to Bob and Charlie? Her 10% can be cut up into 2% slices and we proportionally give 3 to Bob (total 36%) and 2 to Charlie (total 24%). 

We have a total allocation of 100 MW of power to allocate to three products, in the ideal allocation with (0.5, 0.3, 0.2) ratio and each product has a maximum of 40MW. 

These two problems are the same.

## Waterfilling algorithm 

There exists a well known algorithm for solving this problem, the water filling algorithm. 

In essence, we are looking at finding the level of water across three containers that is flat along the allocation amount. 

### definitions:

- $w_i$ is the weight from product i 
- $w'_i$ is the updated weight when we have fewer products due to saturation
- $M_i$ is the maximum of product i
- $x'_i$ is the initial / ideal allocation in case the products are not saturated
- $x_i$ is the current allocation to the product 
- $T$ is total allocation amount to spread over the products
- $T_r$ is the remaining allocation to spread over products after the saturated products are removed from T
- $saturated$ : means that the allocation is >= max on the product. 
- $unsaturated$ : means that the allocation is < max on the product

### Water-Filling (Iterative) Algorithm:

- Step 1: Compute the ideal allocations $x'_i = w_i \cdot T$
- Step 2: For any product i for which $x'_i >= M_i$ (saturated), set $x_i =M_i$, otherwise $x_i = x'_i$. 
- Step 3: Compute the remaining capacity by removing the capacity of saturated $x_i$ : $T_r = T − \sum_{i_{\text{saturated}}} M_i$.
- Step 4: For the remaining (unsaturated) products, redistribute $T_r$ proportionally based on their weights normalized over the unsaturated set $x_i = w^{\prime}_i \cdot T_r$ where $w^{\prime}_i = \frac{w_i}{\sum_j w_j}$ with $j$ representing the unsaturated products
- Step 5: Repeat the process if additional products get saturated during the redistribution, ie. from Step 2. 

The reason this terminates is that because we remove saturated products from the list, the next products get allocated and the set gets reduced, either a new product is saturated and the cycle continues or the final allocation is done and terminates.

```python 
import numpy as np

def iterative_waterfilling(T, weights, max_allocations):
    n = len(weights)
    allocations = np.zeros(n)
    unsaturated = np.array([True] * n)
    remaining_T = T

    while True:
        # Calculate proportional weights for unsaturated products
        current_weights = np.array(weights) * unsaturated
        total_current_weight = np.sum(current_weights)
        
        # Calculate ideal allocation for unsaturated products
        ideal_allocations = (current_weights / total_current_weight) * remaining_T

        # Check for saturation
        newly_saturated = ideal_allocations >= max_allocations
        
        # Update allocations and saturation status
        if not np.any(newly_saturated & unsaturated):
            allocations[unsaturated] = ideal_allocations[unsaturated]
            break
        
        for i in range(n):
            if unsaturated[i] and newly_saturated[i]:
                allocations[i] = max_allocations[i]
                unsaturated[i] = False
                remaining_T -= allocations[i]

    return allocations

T = 100
weights = [0.5, 0.3, 0.2]
max_allocations = [40, 40, 40]

allocations_result = iterative_waterfilling(T, weights, max_allocations)
print("Iterative Waterfilling Result:", allocations_result)
```


### Examples 
Example 1 - total 100, allocation (0.5,0.3,0.2), maximum (40,40,40), result (40,36,24)

Example 2 - total 100, allocation (0.5,0.3,0.2), maximum (40,34,40), result (40,34,26)

The problem of this algorithm is that while it works wonderfully for simple proportional problems, as soon as you start adding more constraints (minimums) and relations between the allocations, this iterative algorithm doesn’t work that great.

So how do we solve this as an optimization problem? We want to find a solution that when unconstrained (unsaturated) falls back to the proportional allocation and when constrained (saturated maximum) falls back to the waterfilling algorithm. 

## From Waterfilling to QP : Mixed integer quadratic programming

While the iterative waterfilling algorithm effectively solves basic proportional allocation problems, it struggles under more complex scenarios involving additional constraints, such as minimum allocation limits or relational constraints between allocations. To robustly handle these real-world complexities, we leverage Mixed Integer Quadratic Programming (MIQP). MIQP elegantly generalizes the waterfilling logic into an optimization framework, allowing precise specification of constraints and objectives. By translating allocation decisions into a mathematical optimization problem, we ensure optimal, constraint-respecting allocations, making it suitable for applications demanding reliability and flexibility.

### Definitions:
- $T$: total
- $w_i$: weight
- $M_i$: Maximum of allocation
- $d_i$: product saturated marker ($\in \mathbb{N}, binary \in \lbrace 0,1 \rbrace $), 0 unsaturated, 1 saturated
- $x_i$: allocation amount
- $v_i$: target water level for unsaturated product (shared identity)
- $U$: upper large bound

### Objective:

$$\min \sum_i (x_i - w_i T)^2$$

### Constraints:

#### (I) Basic allocation limits

$$\forall i \quad x_i \geq 0 \quad (a)$$
$$\forall i \quad x_i \leq M_i \quad (b)$$

#### (II) Total allocation constraint

$$\sum_i x_i \leq T$$

#### (III) Saturation constraints $\forall i$

$$x_i = w_i \cdot v_i + M_i \cdot d_i \quad (a)$$
$$v_i \leq U \cdot (1 - d_i) \quad (b) \quad \text{with } v_i = 0 \text{ when saturated}$$
$$v_i \geq 0 \quad (c)$$
$$v_i \leq \frac{M_i}{w_i} \cdot (1 - d_i) + U \cdot d_i \quad (d)$$

#### (IV) Water level equality constraints across unsaturated products

$$\forall i (1 \rightarrow n), \quad \forall j (i+1 \rightarrow n)$$

$$v_i - v_j \leq U \cdot (d_i + d_j)$$
$$v_j - v_i \leq U \cdot (d_i + d_j)$$

*If both are unsaturated, it means $d_i = d_j = 0$, thus forcing $v_i = v_j$.*



### Explanation of $v_i$

This set of equations sets up $v_i$ as a shared value $z$ across all unsaturated products.

$$40 + (w_1 + w_2) \cdot z = 100$$
$$(0.3 + 0.2) z = 60$$
$$z = 120$$
$$x_1 = 0.3 \cdot 120 = 36$$
$$x_2 = 0.2 \cdot 120 = 24$$

### Implementation in Python 

```python 
#pip install numpy
#pip install cvxpy
#pip install ecos

import cvxpy as cp
import numpy as np

# Problem parameters
T = 100.0
w = [0.5, 0.3, 0.2]          # target weights for products 1, 2, and 3
M_max = [40.0, 40.0, 40.0]     # maximum allocation for each product

# Number of products
n = len(w)

# A sufficiently large constant U (big-M) for enforcing water-level equality.
U = 500.0

# Decision variables:
# x: allocation amounts
x = cp.Variable(n)
# v: auxiliary "water-level" variables for unsaturated products
v = cp.Variable(n)
# delta: binary variables; delta[i] = 1 means product i is saturated (x[i] = M_max[i])
delta = cp.Variable(n, boolean=True)

constraints = []

# For each product, define the allocation as the sum of the unsaturated part and the saturation term.
for i in range(n):
    # If not saturated (delta[i] = 0) then x[i] = w[i]*v[i].
    # If saturated (delta[i] = 1) then x[i] = M_max[i].
    constraints.append(x[i] == w[i] * v[i] + M_max[i] * delta[i])
    
    # Force v[i] = 0 when saturated, by bounding v[i] to 0 when delta[i]=1.
    constraints.append(v[i] <= U * (1 - delta[i]))
    # Ensure nonnegativity of v.
    constraints.append(v[i] >= 0)
    # When unsaturated (delta[i]=0), we must have x[i] = w[i]*v[i] ≤ M_max[i]. 
    # Throught: Why not T here instead of M_max[i]? -> it would be correct, but M_max[i] is a more restrictive boundary so helps convergence
    constraints.append(v[i] <= (M_max[i] / w[i]) * (1 - delta[i]) + U * delta[i])

# Enforce that all unsaturated products share the same water-level.
# For every pair (i,j), if both are unsaturated (delta[i] = delta[j] = 0) then v[i] must equal v[j].
for i in range(n):
    for j in range(i+1, n):
        constraints.append(v[i] - v[j] <= U * (delta[i] + delta[j]))
        constraints.append(v[j] - v[i] <= U * (delta[i] + delta[j]))

# Total allocation constraint: the sum of all allocations must equal the available capacity.
constraints.append(cp.sum(x) == T)

# Ensure each x[i] does not exceed its maximum. (These could be built in via the definition of x.)
for i in range(n):
    constraints.append(x[i] >= 0)
    constraints.append(x[i] <= M_max[i])

# Define the target allocation for each product (unconstrained ideals)
target = np.array([w_i * T for w_i in w])

# Objective: minimize squared deviation from the ideal allocation.
objective = cp.Minimize(cp.sum_squares(x - target))

# Define and solve the MIQP.
# Use a MIQP-capable solver such as GUROBI, CPLEX, ECOS_BB, etc.
prob = cp.Problem(objective, constraints)
result = prob.solve(solver=cp.ECOS_BB)


print("Status:", prob.status)
print("Optimal value:", result)
print("Optimal allocations:")
for i in range(n):
    print(f"  x[{i+1}] = {x.value[i]:.4f}")
print("Water-level (v) values:")
for i in range(n):
    print(f"  v[{i+1}] = {v.value[i]:.4f}")
print("Saturation indicators (delta) values:")
for i in range(n):
    print(f"  delta[{i+1}] = {delta.value[i]:.4f}")

# Expected behavior for this example:
# - For product 1, the unconstrained target is 50 but M_max[0]=40, so we expect it to be saturated (delta[0]=1, x[0]=40).
# - For products 2 and 3 (unsaturated, delta = 0), they share the same water-level z.
#   The total allocation constraint becomes: 40 + (w[1] + w[2]) * z = 100  -->  (0.3+0.2)*z = 60, so z = 120.
#   Hence, x[2] = 0.3 * 120 = 36 and x[3] = 0.2 * 120 = 24.
```

## Summary

In business applications, we often see allocations that should be "preference based" if unconstrained, but then with different constraints it becomes complicated to tease out the preferences in an optimal way. This QP application shows a way to have an allocation identical to the water filling, but with the flexibility of QP. 

If you were simplifying the QP constraints to remove the equations (IV), the output would be (40, 35, 25) since that minimises the objective function. 

This example here is taken out of an abstration of a practical problem of allocating ancillary service capacity to a series of contracts, with trader preferences. This chapter is part of my book "Energy - From Asset to Cashflow" (not yet published). 
