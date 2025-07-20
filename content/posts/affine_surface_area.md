+++
date = '2025-07-18T13:23:28+02:00'
draft = true
title = 'Affine surface area is affine invariant'
summary = 'This post is a extra script from my presentation in the seminar: Convex Geometry.'
math = true
showToc = true
tags=["affine differential geometry", "gauss curvature", "adjoint operator"]
category=["Differential Geometry", "Convex Geometry", "Seminar"]
+++


## Affine Surface Area Introduction

Affine surface area is a fundamental notion in convex geometry that refines classical surface area by making it invariant under volume-preserving affine transformations. For a smooth convex body \\( C \subset \mathbb{R}^d \\), its affine surface area is defined as:

\\[
\text{A}(C) = \int_{\partial C} \kappa(x)^{\frac{1}{d+1}} dS (x),
\\]

where \\( \kappa(x) \\) is the Gaussian curvature at boundary point \\( x \\), and \\( dS(x) \\) is the usual surface area measure.

Unlike the standard surface area, affine surface area captures more subtle geometric information and plays a central role in affine isoperimetric inequalities, approximation theory, and valuations. It also appears naturally when studying the efficiency of random polytopes approximating convex bodies.

In my presentation, it's an important concept to understand the asymptotic formula. In this post I will prove it's affine invariant under some affine transformations.

## Theorem

The theorem is stated as following:

> the Affine Surface Area is invariant under volume-preserving affine transformation, i.e., \\(A(TC)=A(C) \\) when \\(C\subset \mathbb{R}^d\\) is a convex body and \\(T \in SL(d)\\).

## Proof

For the proof, first we want to show that:

> (1) \\( dS^{\prime}(x)=\lvert \det T_x \rvert dS(x)\\), where \\( T_x: T_x(\partial C)\to T_{Tx}(\partial TC)\\), i.e., the map between the original tangent space at \\(x\\) and the tangent space at point \\(Tx\\) on the transformed convex body \\(TC\\) (or the \\(\partial TC\\) more precisely).

To show (1), parametrize the manifold \\(\partial C\\) as \\(x(u) \in \partial C, u \in \mathbb{R}^{d-1}\\). Since \\(\partial C \in \mathbb{R}^{d-1}\\), the surface area is equivalent to the volume in the \\(d-1\\) dimension. We denote \\(\sigma(u)=S(x)\\), then from differential geometry we know:

\\[
    d\sigma(u) = \sqrt{\det G(u)} du
\\]

where \\( G(u)= J(u)^{\top}J(u)\\), \\(J(u)\\) is the Jacobian matrix for \\(x=x(u)\\). Then after transformation we know:

\\[
    x^{\prime}=Tx
\\]

thus:

\\[
    J^{\prime}(u)=TJ(u)
\\]

and

$$
\begin{aligned}
G^{\prime}(u)  &= J(u)^{\top}T^{\top}T J(u)  \\\\
&= J(u)^{\top}T^{\top}J(u)^{-\top}J(u)^{\top}J(u)J(u)^{-1}T J(u) \\\\
&= T_x^{\top}G(u)T_x
\end{aligned}
$$

Therefore:

\\[
    \det G^{\prime}(u) = (\det T_x)^2\det G(u) \\\\
    d\sigma^{\prime}(u)=\lvert \det T_x \rvert d\sigma(u)
\\]

---

Now we expolre the relation between \\(\det T\\) and \\(\det T_x\\), we have the following:

> (2) \\( \lvert \det T \rvert = \lvert \det T_x \rvert \frac{1}{\lVert T^{\ast}n \rVert}\\)

we can see \\(\det T\\) as the volume of a parallelotope, and \\(\det T_x\\) is the bottom's area. So we need to decide the height.

Notice that:

\\[
    \langle Tx,n \rangle=\langle x, T^{\ast}n \rangle
\\]

To be the new normal vector, we need to normalise it: \\(n^{\prime} = \frac{1}{\lVert T^{\ast}n \rVert}\\).

Then the height is the strenching factor \\(\lVert T^{\ast}n \rVert\\).

---

The next step is to see how the gauss curvature changes after the affine transformation. Recall we define the gauss curvature \\(\kappa(x)\\)as the product of all the **principle curvatures** \\( \kappa_1, \dots, \kappa_{d-1} \\), and the principle curvatures are the eigenvalues of the **Weingarten map** \\( W_x \\). The Weingarten map \\( W_x \\) at a point \\( x \in \partial C \\) is a linear operator on the tangent space that describes how the unit normal vector changes as you move along the surface:

\\[
W_x: T_x(\partial C) \to T_x(\partial C), \quad v \mapsto -D_v n(x),
\\]

where \\( n(x) \\) is the outward unit normal at \\( x \\), and \\( D_v n(x) \\) is the directional derivative of the normal in direction \\( v \\).

i.e. for gauss curvature we have:

\\[
\kappa(x) = \det W_x = \prod_{i=1}^{d-1} \kappa_i(x).
\\]

We further define the Weingarten map on the new surface to be \\(W^{\prime}_y: T_y(\partial TC) \to T_y(\partial TC), v^{\prime} \mapsto -D_w n^{\prime}(y), w=v^{\prime}\\).

Now to determine what it is, we first need to rewrite the new normal vector as:

\\[
    n^{\prime}(y)=F(x)=\frac{1}{a}T^{\ast}n(x)
\\]

Hence we can do:

\\[
    D_{v^{\prime}} n^{\prime}(y) = D_{v^{\prime}}(F\circ T^{-1})(y)= D_{T^{-1}v^{\prime}}F(x)= D_v \frac{1}{a}T^{\ast}n(x)= T^{\ast}n(x)D_v\frac{1}{a}+\frac{1}{a}T^{\ast}D_v n(x)
\\]

Project it onto the tangent space we get:

\\[
    W^{\prime}_y v^{\prime} = \frac{1}{a}(T^{\ast}D_vn(x))^{\top} = \frac{1}{a}(T^{\ast}W_xv)^{\top}=\frac{1}{a}T_x^{\ast}W_xv
\\]

The last equation is coming from the following: for every \\(w\in T_y(\partial TC)\\), we have:

\\[
    \langle T^\ast v,w \rangle=\langle v, Tw \rangle=\langle v, T_xw \rangle=\langle T_x^\ast v, w \rangle
\\]

Then

\\[
    W^{\prime}_y v^{\prime} = \frac{1}{\lVert T^{\ast}n \rVert} T_x^\ast W_x v
\\]
\\[
    \rArr \enspace W^{\prime}_y = \frac{1}{\lVert T^{\ast}n \rVert} T_x^\ast W_x T_x^{-1}
\\]

Note that we denote \\(v^\prime\\) as the parameter after the transformation hence \\(v^\prime = T_x v\\).

This and the above result (2) lead to the gauss curvature after transformation \\(\kappa^\prime\\):

>(3)\\[
    \kappa^\prime = \det W^{\prime}_y = \frac{1}{{\lVert T^{\ast}n \rVert}^{d-1}}\cdot \frac{\kappa}{{(\det T_x)}^2} = \frac{1}{{\lVert T^{\ast}n \rVert}^{d+1}}\cdot \frac{\kappa}{{(\det T)}^2}
\\]

---

Put all the parts (1), (2) and (3) together we can finally get:

$$
\begin{aligned}
{\kappa^\prime (y)}^{\frac{1}{d+1}} d\sigma ^\prime (y) &= {\kappa (x)}^{\frac{1}{d+1}}\cdot \frac{1}{(\det T)^{\frac{2}{d+1}}}\cdot \frac{1}{\lVert T^{\ast}n \rVert}\cdot \lvert \det T\rvert \cdot \lVert T^{\ast}n \rVert d\sigma (x) \\\\
&= (\det T)^{1-\frac{2}{d+1}}{\kappa (x)}^{\frac{1}{d+1}}d\sigma (x) \\\\
&= (\det T)^{\frac{d-1}{d+1}}{\kappa (x)}^{\frac{1}{d+1}}d\sigma (x)
\end{aligned}
$$

And for \\(T \in SL(d)\\), \\(\lvert \det T \rvert=1\\), hence \\(A(TC)=A(C) \\).\\(\square\\)