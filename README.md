# Outer Approximation for Sparse Portfolio Optimization

## Overview

This repository contains a C++ implementation of an **Outer Approximation** algorithm for solving sparse portfolio optimization problems with linear constraints.

The implementation follows the methodology developed in my Bachelor's Thesis:

> [**Outer Approximation Method for Sparse Portfolio Optimization**](https://www.linkedin.com/feed/update/urn:li:activity:7487539192193957888/)

The algorithm is inspired by the Outer Approximation framework of **Duran & Grossmann**, adapted to the sparse regression reformulation proposed by **Bertsimas & Cory-Wright**.

---

## Optimization Problem

The implemented algorithm solves the regularized sparse multiple regression problem

$$
\min_{x\in\mathbb{R}^n} \quad
\frac{1}{2}|Xx-y|^2
+
\frac{1}{2\gamma}|x|^2
+
d^Tx
$$

subject to

$$l \le Ax \le u$$
$$e^Tx = 1$$
$$|x|_0 \le k$$

where

* $X$ is the Cholesky factor of the covariance matrix $\Sigma$,
* $\mu$ is the expected return vector
* $y=(XX^T)^{-1}X\mu$,
* $d = \left(X^\top (XX^\top)^{-1}X - I\right)\mu$,
* $A$, $l$, and $u$ define the linear constraints,
* $k$ is the maximum number of selected assets,
* $\gamma > 0$ is the regularization parameter.



This equivalent formulation is obtained by reformulating the original sparse portfolio optimization problem as a regularized multiple linear regression problem.

---

## Algorithm

The implementation follows the classical **Outer Approximation** scheme.

For a fixed binary support vector $z$,

1. Solve the dual continuous optimization problem.
2. Obtain

   * the objective value $f(z)$, (an upper bound),
   * a valid subgradient.
3. Add a cutting plane to the master MILP.
4. Solve the master problem for $\theta$ to obtain a new support vector $z$.
5. Repeat until

$$
f(z)-\theta \le \varepsilon.
$$

The master problem accumulates cuts of the form

$$
\theta \ge
f(z^t)+
\nabla f(z^t)^T(z-z^t),
$$

providing progressively tighter lower bounds.

---

## Code Structure

The implementation is organized into several classes.

### `Datos`

Stores the complete optimization instance:

* dimensions
* matrices
* vectors
* regularization parameter
* sparsity level

---

### `variablesDual`

Builds and solves the dual optimization problem for a fixed support vector.

Responsibilities:

* construct the dual model only once,
* update the objective according to the current support,
* solve using Gurobi,
* return:

  * objective value,
  * optimal dual variables,
  * gradient information.

---

### `variablesMin`

Implements the master Outer Approximation problem.

Responsibilities:

* binary variables (z),
* lower bound variable (\theta),
* cardinality constraint,
* addition of OA cuts,
* solution of the MILP after each iteration.

---

### `AlgoritmoOA`

Coordinates the complete algorithm.

Main loop:

1. solve dual problem,
2. update incumbent solution,
3. compute optimality gap,
4. add OA cut,
5. solve master problem,
6. stop when convergence is reached.

---

## Dependencies

* C++20
* Gurobi Optimizer
* Gurobi C++ API

---

## Input Data

The example currently included in `main()` contains a complete benchmark instance directly embedded in the source code.

The user must define:

* matrix `X`
* vector `y`
* vector `d`
* matrix `A`
* lower bounds `l`
* upper bounds `u`
* ridge parameter `gamma`
* sparsity level `k`

The implementation can easily be adapted to load data from external files.

---

## Output

During execution, the algorithm reports the progress of the Outer Approximation procedure.

For each iteration, the following information is displayed:

* **Iter**: current iteration number.
* **f(z) [UB]**: objective value of the current feasible solution, providing the **upper bound**.
* **theta [LB]**: value of the master problem, providing the **lower bound**.
* **Gap**: difference between the upper and lower bounds, used as the stopping criterion.

Once the optimality gap reaches the prescribed tolerance, the algorithm reports:

* the iteration at which convergence is achieved,
* the optimal support $z^*$ (selected assets),
* the optimal objective value $f(z^*)$,
* the optimal portfolio weights $x_i$ for the selected assets,
* the execution end time,
* the total running time.

---

## References

* D. Bertsimas, R. Cory-Wright, *A Scalable Algorithm for Sparse Portfolio Selection*.
* M. A. Duran, I. E. Grossmann, *An Outer Approximation Algorithm for a Class of Mixed-Integer Nonlinear Programs*.
* [Bachelor's Thesis: *Outer Approximation Method for Sparse Portfolio Optimization*.](https://www.linkedin.com/feed/update/urn:li:activity:7487539192193957888/)

---

## Notes

This implementation was as part of a Bachelor's Thesis in Mathematics. [Project Overview]([https://www.gurobi.com/](https://www.linkedin.com/feed/update/urn:li:activity:7487539192193957888/))

