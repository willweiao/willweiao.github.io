+++
date = '2025-07-13T01:09:19+02:00'
draft = false
title = 'nonlinear optimization lecture notes'
showToc=true
math=true

tags = ["KKT", "SQP", "Nonlinear Programming"]
categories = ["Optimization", "Math", "Notes"]
+++

This is a personal lecture notes from a TUM course *Nonlinear Optimization* in WS23-24. This isn't a full course recap—just a way for me to organize the core ideas I want to remember. I've picked out the topics that I found most important and useful.

## 1. Problem Setting and Definitions

1. The general NLP (nonlinear program) is:

$$
\text{min}_{x \in \mathbb{R}^n}f(x) \quad \text{s.t.} \quad g(x) \leq 0, h(x) = 0
$$

2. define feasible set as \\(X\\), and for a \\(x\in X\\) define *the index set of active inequality constraints* \\(\mathcal{A}(x)\\) and also *the index set of inactive inequality constraints* \\(\mathcal{I}(x)\\):

$$
\mathcal{A}(x)=\lbrace i; 1\leq i\leq m, g_i(x)=0\rbrace,
$$
$$
\mathcal{I}(x)=\lbrace i; 1\leq i\leq m, g_i(x)<0\rbrace.
$$

3. The *tangent cone* of \\(M\\) at a point \\(x \in M\\) is:

$$
T(M,x) = \lbrace d\in \mathbb{R}^n; \exist \eta_k > 0, x^k \in M: \lim\limits_{k\rarr\inf}x^k=x, \lim\limits_{k\rarr\inf}\eta_k(x^k-x)=d\rbrace.
$$

4. The *linearized tangent cone* is:

$$
T_l(g,h,x)=\lbrace d \in \mathbb{R}^n; \nabla g_i(x)^Td \leq 0, i \in \mathcal{A}(x), \nabla h(x)^Td =0\rbrace.
$$

## 2. First-order necessary optimality conditions

1. **ACQ condition**:   \\(T_l(g,h,x)=T(X,x)\\)

2. **GCQ condition**:   \\(T_l(g,h,x)^\circ=T(X,x)^\circ\\)

A condition that implies GCQ is called **constraints qualification(CQ)**. Obviously ACQ is a CQ.

3. **KKT conditions**: There exist Lagrange multipliers \\(\overline{\lambda} \in \mathbb{R}^m\\) and \\(\overline{\mu} \in \mathbb{R}^p\\) such that:

$$
\begin{aligned}
& \text{(a)} \qquad \nabla f(\overline{x})+\nabla g(\overline{x})\overline{\lambda}+\nabla h(\overline{x})\overline{\mu} = 0;\\\\
& \text{(b)} \qquad h(\overline{x})=0;\\\\
& \text{(c)} \qquad \bar{\lambda} \geq 0, g(\bar{x}) < 0, \bar{\lambda}^T g(\bar{x}) = 0.
\end{aligned}
$$

Denote \\( L(x, \lambda, \mu) = f(x）+ \lambda^T g(x) +\mu^T h(x)\\) as **Lagrangian function**.

4. The point \\( x\in X\\) is said to satisfy **MFCQ**, if:

$$
\begin{aligned}
& \text{(a)} \qquad  \nabla h(x) \text{is full rank,} \\\\
& \text{(b)} \qquad \exists d \in \mathbb{R}^n \text{with} \quad \nabla g_i(x)^Td<0, i\in \mathcal{A}(x), \nabla h(x)^Td=0
\end{aligned}
$$

5. The point \\( x\in X\\) is said to satisfy **PLICQ**, if:

$$
\begin{aligned}
& \text{(a)} \qquad  \nabla h(x) \text{is full column rank,} \\\\
& \text{(b)} \qquad  \exists \text{no vectors} u \in \mathbb{R}^m, v \in \mathbb{R}^p \text{with} \quad \nabla g(x)u + \nabla h(x)v =0, u_{\mathcal{A}(x)} \geq 0,  u_{\mathcal{A}(x)} \not= 0, u_{\mathcal{I}(x)} = 0.
\end{aligned}
$$

Note that **PLICQ** \\(\Leftrightarrow\\) **MFCQ**.

6. Another CQ is **LICQ** that is stronger than **PLICQ**, we say a point \\( x\in X\\) satisfy **LICQ** is:

$$
\text{if all columns of the matrix} (\nabla g_{\mathcal{A}(x)}(x), \nabla h(x))\text{are linearly independent.}
$$

7. Overview of the **CQs**:

$$
LICQ \rArr PLICQ \Leftrightarrow MFCQ \rArr ACQ \rArr GCQ
$$

## 3. Second-order necessary optimality conditions

1. We say that \\( x,\lambda, \mu\\) satisfies a *second-order necessary optimality conditions(CQ2)* if for all \\(d \in T_+(g,h,x,\lambda)\\) there exists an open interval \\(J \supset \lbrace 0 \rbrace\\) and a twice continuously differentiable curve \\( \gamma: J \rarr \mathbb{R}^n\\) such that:

$$
\gamma (0) =x, \gamma^'(0)=d, g_{\mathcal{A}_0(x, d)}(\gamma (t))=0, h(\gamma (t))=0 \quad \forall t\in J, t\geq 0, 
$$

where \\(\mathcal{A}_0(x, d):=\lbrace i \in \mathcal{A}(x); \nabla g_i(x)^Td=0 \rbrace\\).

2. Let f, g, h be twice continuously differentiable. Suppose that \\(\bar(x)\\) is a local solution where the GCQ holds. Then there exist Langrange multipliers \\(\bar(\lambda) \in \mathbb{R}^m\\) and \\(\bar(\mu) \in \mathbb{R}^p\\) with:

$$
\begin{aligned}
& \text{(a)} \quad \nabla_x L(\bar(x), \bar(\lambda), \bar(\mu))=0,\\\\
& \text{(b)} \quad h(\bar(x))=0,\\\\
& \text{(c)} \quad \bar(\lambda) \geq 0, g(\bar(x)) \leq 0, \bar(\lambda)^Tg(\bar(x)) = 0,\\\\
& \text{(d)} \quad \text{if CQ2 is satisfied, then: }d^T\nabla_xx L(\bar(x), \bar(\lambda), \bar(\mu))d \geq 0 \quad \text{for all}d \in T_+(g,h, \bar(x), \bar(\lambda)).
\end{aligned}
$$

## 4. Optimization Algorithms

1. **Penalty Methods** and **augmented Lagrangian methods**:
$$
P_\alpha (x) = f(x)+\frac{\alpha}{2}\displaystyle\sum_{i=1}^p h_i(x)^2 = f(x) + \frac{\alpha}{2} \lVert h(x)\rVert^2.
$$
For an equality constrained problem:
$$
L_\alpha (x, \mu) = L(x, \mu) + \frac{\alpha}{2} \lVert h(x)\rVert^2 = f(x) + \mu^Th(x) + \frac{\alpha}{2} \lVert h(x)\rVert^2.
$$
Then the method is to solve a series of unconstrained optimization problems: \\( \min\limits_{x \in \mathbb{R}^n} P_{\alpha_k} (x)\\) for increasing values \\(\alpha_k > 0\\).

2. **SQP Method**:

Sequential Quadratic Programming (SQP) is a powerful method for solving nonlinear constrained optimization problems of the form:

$$
\begin{aligned}
\min_{x \in \mathbb{R}^n} \quad & f(x) \\\\
\text{s.t.} \quad & g_i(x) \leq 0, \quad i = 1, \dots, m \\\\
& h_j(x) = 0, \quad j = 1, \dots, p
\end{aligned}
$$

At each iteration, SQP solves a **Quadratic Programming (QP)** subproblem that locally approximates the original nonlinear problem. The basic idea is:

- Approximate the objective function \\( f(x) \\) by a quadratic model using the gradient and Hessian.
- Linearize the constraints \\( g_i(x), h_j(x) \\) using a first-order Taylor expansion.
- Solve the resulting QP to get a search direction.

The QP subproblem at iteration \\( k \\) is typically:

$$
\begin{aligned}
\min_d \quad & \nabla f(x_k)^T d + \frac{1}{2} d^T B_k d \\\\
\text{s.t.} \quad & g_i(x_k) + \nabla g_i(x_k)^T d \leq 0 \\\\
& h_j(x_k) + \nabla h_j(x_k)^T d = 0
\end{aligned}
$$

where \\( B_k \\) is a positive definite approximation to the Hessian of the Lagrangian.

After solving the QP, we update the iterate with a step \\( x_{k+1} = x_k + \alpha_k d_k \\), where \\( \alpha_k \\) is found via a line search.

3. **Barrier Methods and interior point methods**:

Barrier methods are a class of algorithms for solving constrained optimization problems by **replacing inequality constraints with a smooth penalty term** that discourages infeasibility. This allows the problem to be solved as a sequence of **unconstrained** (or equality-constrained) problems.

Consider a constrained problem:

$$
\begin{aligned}
\min_{x \in \mathbb{R}^n} \quad & f(x) \\\\
\text{s.t.} \quad & g_i(x) \leq 0, \quad i = 1, \dots, m
\end{aligned}
$$

We construct a **barrier function**:

$$
\Phi(x) = -\tau \sum_{i=1}^m \ln(-g_i(x))
$$

and define the **barrier subproblem** (with parameter \\( t > 0 \\)):

$$
\min_{x}\quad B_\tau (x) = f(x) + \Phi(x)
$$

As \\( \tau \\) decrease to 0, the solution to the barrier problem approaches the solution of the original constrained problem, while staying strictly inside the feasible region.

This leads to the **interior point method** framework: starting from a feasible interior point, we solve a sequence of barrier problems with increasing \\( t \\), each one guiding us closer to the true optimum while avoiding the constraint boundaries.

Another view on interior point method is based on a perturbed KKT system. 

In interior point methods, inequality constraints are often rewritten using **slack variables** to convert the problem into an equality-constrained form.

we introduce a **slack vector** \\( s \in \mathbb{R}^m \\) such that:

$$
g(x) + s = 0, \quad s > 0
$$

This transforms the inequality into an equality plus a positivity constraint. Then The KKT conditions for this system become:

1. \\( \nabla f(x) + \nabla g(x)^\top \lambda = 0 \\)

2. \\( g(x) + s = 0 \\)

3. \\( \lambda \geq 0, \quad s > 0 \\)

4. \\( \lambda_i s_i = 0 \quad \forall i \\)

Interior point methods then consider the **perturbed KKT conditions** by relaxing the complementarity condition to:

$$
\lambda_i s_i = \mu \quad \text{for all } i, \quad \mu > 0
$$

In vector form:

$$
\Lambda S e = \mu e
$$

where \\( \Lambda = \text{diag}(\lambda), \quad S = \text{diag}(s), \quad e = \text{vector of ones} \\).

This defines a smooth nonlinear system:

$$
\begin{cases}
\nabla f(x) + \nabla g(x)^\top \lambda = 0 \\\\
g(x) + s = 0 \\\\
\Lambda S e = \mu e
\end{cases}
$$

We solve this system using **Newton's method**, and gradually decrease \\( \mu \\) toward zero. The iterates remain strictly inside the feasible region (i.e., \\( s > 0, \lambda > 0 \\)) and converge to a KKT point as \\( \mu \to 0 \\).

4. **Active Set Method**:

This is the method popular for solving quadratic problems. We consider:

$$
\min\limits_{x \in \mathbb{R}^n} c^Tx+\frac{1}{2}x^THx \qquad s.t. \qquad Ax+\alpha \leq 0, Bx+\beta = 0.
$$
with positive definite H. The idea of active set methods is to estimate the active set of the solution iteratively and handle these estimated active inequalities as equalities while ignoring the remaining inequalities. At the calculation of \\(x^{k+1}\\), the active set \\(\mathcal{A}(x^k)\\) of the actual point x^k is approximated, i.e. choosing \\(\mathcal{A}_k \sub \mathcal{A}(x^k)\\) suitably. At the k-th iteration we solve:

$$
\begin{aligned}
\min_{x \in \mathbb{R}^n}  \quad \frac{1}{2}& x^T H x + c^T x \\\\
\text{s.t.} \quad  A_{\mathcal{A_k}}&  x = -\alpha_{\mathcal{A_k}}, \\\\
 B x  & = -\beta
\end{aligned}
$$

This results in a method where we solve a series of equality constrained quadratic problems.
