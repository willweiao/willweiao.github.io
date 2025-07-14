+++
date = '2025-07-15T00:19:19+02:00'
draft = false
title = '3 Classic Optimization Methods Comparison'
summary = 'This post is a record for one of my projects'
math = true
showToc = true
tags = ["Gradient descent", "Newton Method", "BFGS", "Numerical Optimization"]
categories = ["Optimization", "Math", "Research"]
+++

This post is a record for my project ["classic-optimization-methods-lab"](https://github.com/willweiao/classic-optimization-methods-lab), where I implemented and compared three classic optimization algorithms: Gradient Descent with line search, Newton's method, and BFGS. These methods form the foundation of numerical optimization, and I wanted to see how they behave on several test problems in practice — not just in theory.

## 1 Introduction

In this project, I implemented three classic optimization methods: **Gradient Descent**, **Newton's Method**, and **BFGS**. All three use a common backtracking line search routine to determine the step size at each iteration. First I want to recall a little bit of the three methods. Below is a brief overview of each algorithm and how it was implemented. 

### 1.1 Gradient Descent with Line Search

Gradient descent updates the current point using the negative gradient direction:

\\[
x_{k+1} = x_k - \alpha_k \nabla f(x_k)
\\]

Here, \\( \alpha_k \\) is the step size determined using **backtracking line search** to satisfy the **Armijo condition**:

\\[
f(x + \alpha p) \leq f(x) + c \alpha \nabla f(x)^\top p
\\]

where \\( p = -\nabla f(x) \\) is the steepest descent direction. The Python code I used is:

```python
c = 0.1
tau = 0.8
alpha_init = 1.0

alpha = alpha_init
while f(x + alpha * p) > f(x) + c * alpha * np.dot(grad_f(x), p):
    alpha *= tau
return alpha
```

This method is easy to implement and works reliably, but it may converge slowly—especially on ill-conditioned functions like Rosenbrock.

### 1.2 Newton's Method 

Newton's method uses second-order derivatives to speed up convergence by approximating the function locally as a quadratic. At each iteration, it directly jumps towards the local minimum of this approximation:

\\[
x_{k+1} = x_k - \alpha_k \left[ \nabla^2 f(x_k) \right]^{-1}\nabla f(x_k)
\\]

Here, \\( \nabla^2 f(x_k) \\) is the Hessian matrix (second-order derivatives). In practice, instead of explicitly computing the inverse Hessian, we solve the linear system:

\\[
\nabla^2 f(x_k)\, p_k = -\nabla f(x_k)
\\]

In my implementation, I used a straightforward Newton step without line search (setting \\(\alpha_k = 1\\)), meaning the update directly follows the solved direction. While simple and extremely fast when close to the solution, this approach can struggle if the starting point is far or the Hessian is ill-conditioned.

### 1.3 BFGS Method

The BFGS method is a popular quasi-Newton approach that approximates the Hessian matrix iteratively, avoiding direct computation of second derivatives. At each iteration, it updates the approximate inverse Hessian \( H_k \) using gradient differences.

The update step is given by:

\[
x_{k+1} = x_k + \alpha_k p_k,\quad \text{where } p_k = -H_k \nabla f(x_k)
\]

I used a backtracking line search with Armijo condition (same as gradient descent), initializing \(\alpha = 1.0\), and reducing it by \(0.8\) each iteration if needed.

If the curvature condition \(\mathbf{y}_k^\top \mathbf{s}_k > 10^{-10}\) fails, the algorithm terminates early to avoid numerical instability.

## 2 Test Problems & Setup

I tested the optimization methods on two classic problems:

- **Rosenbrock function** (nonconvex, challenging for gradient-based methods):
\[
f(x, y) = (1 - x)^2 + 100(y - x^2)^2
\]

- **Quadratic function** (convex, ideal for testing Newton’s method):
\[
f(x, y) = x^2 + 2y^2
\]

### Setup:

- **Initial point**: \((0, 0)\)
- **Tolerance**: \(\|\nabla f(x)\| \leq 10^{-6}\)
- **Maximum iterations**: 100
- **Metrics recorded**: Objective values, gradient norms, and iteration count.

## 3 Results & Observations

### Rosenbrock Function

Starting from the initial point \\((0,0)\\):

| Method           | Iterations | Converged?  | Final Point           | Objective |
|------------------|------------|-------------|-----------------------|-----------|
| Gradient Descent | 1000       | No (slow)   | \\((0.903, 0.815)\\)  | 0.0095    |
| Newton           | 2          | Yes (fast)  | \\((1.000, 1.000)\\)  | 0.0000    |
| BFGS             | 22         | Yes         | \\((1.000, 1.000)\\)  | 0.0000    |

**Observations:**  
- **Newton's method** rapidly converged in only 2 iterations, finding the exact minimum.
- **BFGS** also converged quickly (22 iterations), though it triggered a curvature condition warning, indicating numerical sensitivity.
- **Gradient Descent** was very slow, unable to fully converge even after 1000 iterations.

### Quadratic Function

Starting from the initial point \\((3,4)\\):

| Method           | Iterations | Converged? | Final Point                                 | Objective |
|------------------|------------|------------|---------------------------------------------|-----------|
| Gradient Descent | 37         | Yes        | \\((9.85\ast 10^{-28}, -2.46\ast 10^{-07})\\) | ≈0        |
| Newton           | 1          | Yes (fast) | \\((0.000, 0.000)\\)                        | 0.0000    |
| BFGS             | 7          | Yes        | \\((-9.91\ast 10^{-07}, -4.40\ast 10-07)\\) | ≈0        |

**Observations:**  
- **Newton's method** again converged immediately after 1 iteration.
- **BFGS** quickly reached a very accurate solution in just 7 iterations, despite a curvature warning.
- **Gradient Descent** converged reliably, though slower, taking 37 iterations.

###  Key Takeaways:

- **Newton’s method** provides the fastest convergence near optima, but requires the Hessian and has higher computational cost.
- **BFGS** balances computational efficiency and convergence speed effectively, making it a common choice in practice.
- **Gradient Descent** is simple but significantly slower, particularly on challenging functions like Rosenbrock.

The [plots](https://github.com/willweiao/classic-optimization-methods-lab/tree/main/plots) further illustrate these differences clearly. You can click on the link and check them in my projects.

## 4 Conclusion

In this short project, I explored three foundational optimization algorithms—Gradient Descent, Newton's Method, and BFGS. While Newton's method excels near optima with rapid convergence, BFGS offers a robust and efficient alternative without explicit second derivatives. Gradient Descent, though simpler, often requires significantly more iterations. This experience underscored the practical trade-offs of choosing optimization methods, highlighting BFGS as a versatile and reliable option for typical use cases. For codes and detailed experiments and results please be free to check on my [project](https://github.com/willweiao/classic-optimization-methods-lab). Again it's not a formal project and I am plan to add more methods that the textbook didn't mentioned but useful in practice to it and also extend this post. If you have any questions or any thoughts, please contact me via e-mails or start a issue in that repository. Thank you for reading!




