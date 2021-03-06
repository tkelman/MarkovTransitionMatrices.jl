# MarkovTransitionMatrices

[![Build Status](https://travis-ci.org/magerton/MarkovTransitionMatrices.jl.svg?branch=master)](https://travis-ci.org/magerton/MarkovTransitionMatrices.jl)

[![Coverage Status](https://coveralls.io/repos/magerton/MarkovTransitionMatrices.jl/badge.svg?branch=master&service=github)](https://coveralls.io/github/magerton/MarkovTransitionMatrices.jl?branch=master)

[![codecov.io](http://codecov.io/github/magerton/MarkovTransitionMatrices.jl/coverage.svg?branch=master)](http://codecov.io/github/magerton/MarkovTransitionMatrices.jl?branch=master)

This package creates a Markov transition matrix for a discrete process where innovations
are Gaussian. The transition matrix returned is **TRANSPOSED** so that
`P[i,j] = Pr(state_{t+1}=i | state_t = j)`. This is because it is faster to iterate over
the matrix with the current state as the column and next period as the row. The
matrix is also sparse to save space.

# Example: Correlated random walk

To model `x_{t+1} = x_t + u_{t+1}` where `u_{t+1} ∼ N(0,Σ)`, we first pick a grid:
```julia
grid_x1 = 0.0:1.0:10.0
grid_x2 = -1.5:0.5:15.0
```
Internally, the function will use `Base.product()` to iterate through all possible
states. We then we create functions for the mean and variance given a tuple in
the state space.
```julia
μ(s) = [s...]
Σ(s) = [1.0 0.5; 0.5 1.0]
```
Many probabilities are very close to zero and can be eliminated from the matrix
without too much impact on numerical accuracy. Here, we drop all probabilities less
than `1e-8`. Create the transition matrix `P` as
```julia
using MarkovTransitionMatrices
P = markov_transition(μ, Σ, 1e-8, grid_x1, grid_x2)
```

# Example: Markov-switching

We can also model a Markov-switching process with a finite number of regimes
`r ∈ {1,2,…,k}`. Let the transition matrix for the `k` regimes
(*not transposed*) be
```julia
πswitch = [.9 .1; .4 .6]
```
The mean and variance functions take two parameters: the regime and state-tuple
```julia
μswitch(r::Int, s) = r==1 ? [s...] : [s...] .+ ones(2)
Σswitch(r::Int, s) = r==1 ? eye(2) : [1.0 0.5; 0.5 1.0]
Pswitch = markovswitching_transition(μswitch, Σswitch, πswitch, 1e-8, grid_x1, grid_x2)
```
