# CombinOptHeuristics.jl
**CombinOptHeuristics** provides heuristic solvers for combinatorial optimization. It additionally provides types and constructors for problem instances, as well as some public domain benchmark problems. Currently only quadratic unconstrained binary optimization (QUBO) is supported, but the plan is to eventually support quadratic assignment problems (QAP) as well.

- The type `QUBO` represents a QUBO problem.
- Convenience functions `ising` and `maxcut` facilitate the construction of Ising energy minimization and MAXCUT problems.
- The `solve` function generates heuristic solutions for a given problem.

More details can be found in the source documentation.



## Installation
In the Julia terminal, enter package mode (type `]`) and enter

```add https://github.com/benninkrs/CombinOptHeuristics.jl```


## Example Usage
This example shows how to construct and solve a simple QUBO problem involving 3 variables: 
```
julia> using CombinOptHeuristics

# create a maximization QUBO problem over {0,1}^3
julia> A = [0 3 -4; 3 0 6; -4 6 0]; b = [0.2, -0.7, 0];
julia> q = QUBO(A, b, 0, (0,1), max);

# evaluate the objective function at [1,0,1]
julia> q([1,0,1])                       
-7.8

# find a good solution
julia> (f,x) = solve(q, 100) 
Progress: 100%|████████████████████████████████████████████████████| Time: 0:00:00
  Best:  11.3           
(11.3, [0.0, 1.0, 1.0])

julia> q(x)                             
11.3
```

This example shows how to solve a benchmark problem:
```
# read a MAXCUT problem from BiqBin
julia> q = read_biqbin_maxcut("G1");

julia> (f,x) = solve(q,100)
Progress: 100%|████████████████████████████████████████████████████| Time: 0:00:05
  Best:  11539.0
(11539.0, [0.0, 1.0, 1.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 1.0  …  0.0, 0.0, 1.0, 1.0, 0.0, 1.0, 0.0, 0.0, 1.0, 0.0])
```
(Because the heuristic is randomized, you may obtain a slightly different result.)

The function `predict` quickly and cheaply predicts the optimal value based on the problem parameters:
```
julia> q = read_biqbin_maxcut("G1");

julia> predict(q)
11546.366666382983
```


## Technical Details

### Problem Types

Currently, solvers are provided only for quadratic unconstrained binary optimization (QUBO), which is the task of maximizing or minimizing 
  $$  f(x) = x^T A x + b^T x + c $$
  over $x \in \{\text{lo},\text{hi}\}^n$, where $\{\text{lo},\text{hi}\}$  is typically either $\{0,1\}$ or $\{-1,1\}$. Specialized types of QUBO include MAXCUT, MAXSAT, and Ising energy minimization. 


### Algorithm

QUBO problems are solved by a variant of the method described in [Boros2007].  In this approach the search space is relaxed to a continuous domain. Starting from a random point in the interior of the domain, the components of a candidate solution $x$ are progressively discretized using a greedy heuristic until a local optimum is reached. This search is fast, scaling as $O(n^2)$, and has a high probability of generating a pretty good solution. In practice one typically generates a large number of candidate solutions and selects the best one.