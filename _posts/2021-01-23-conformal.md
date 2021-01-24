---
title: 'Conformal Rigidity'
date: 2021-01-23
permalink: /posts/2021/01/conformal-rigidity/
tags:
  - geometry 
  - analysis 
---
A conformal transformation is one that preserves angles. In two dimensions, this is equivalent to being holomorphic and having a non-vanishing derivative.
There are tons of these transformations (my personal favorites are Mobius transformations). In fact, the famous [Riemann mapping Theorem](https://en.wikipedia.org/wiki/Riemann_mapping_theorem)
asserts that any simply connected domain $U \subset \mathbb{C}$ admits a bijective conformal map $f: U \to \mathbb{D}$ to the unit disc.\
It turns out that 2 dimensions is the exception. In this note we will show that 
the only conformal maps in dimensions $n \ge 3$ are built from inversions and reflections.

First, what exactly does "preserve angles" mean? The nicest way to talk about angles is via an inner product.
Given a vector space $V$ with an inner product $\langle \cdot, \cdot \rangle$, the angle $\theta$ between two vectors $v,w \in V$ is defined by\
$$ \cos \theta = \dfrac{ \langle v, w \rangle}{\sqrt{ \langle v,v \rangle \langle w, w \rangle}}.$$\
In Euclidean space with the standard inner product, this matches the formula you know. Thus a linear transformation $A: V \to V$
preserves angles if $\langle Av, Aw \rangle = \lambda^2 \langle v, w \rangle$ for all $v,w$ and some $\lambda > 0$.\
Note that since $\lambda^2 > 0$ we are requiring the orientation of the angle to be preserved as well.\

Thus a conformal linear transformation is just an orientation and distance preserving map (called a special orthogonal matrix)
followed by a dilation. Well, that's pretty boring. But now we can define more general conformal maps by requiring its derivative to
be a conformal linear transformation!

**Definition** 
Let $U \subset \mathbb{R}^n$ be open. A $C^1$ map $f:U \to \mathbb{R}^n$. $f$ is said to be **conformal** if $D_x f:T_xU \to T_{f(x)}\mathbb{R}^n$ is conformal for all $x \in U$.

How strong is this restriction on the derivative? As it turns out, pretty strong.\


**Lemma**
Let $v:\mathbb{R}^n \to \mathbb{R}^n$ be a $C^1$ vector field. Then the flow of $v$ is conformal if and only if \
$$(Dv)^T+Dv = \frac{2}{n} \operatorname{tr}(Dv) I$$\

_Proof_: Let $f_t$ be the local flow of $v$ and $p \in \mathbb{R}^n$. For convenience set $A=D_p f_t$, $B=D_{f_t(p)}v$. We differentiate the relation\
$$A^T A = (\det A)^{2/n} \; I$$\
with respect to $t$ and use [Jacobi's formula](https://en.wikipedia.org/wiki/Jacobi%27s_formula) to obtain\
$$A^TB^TA+A^TBA = \frac{2}{n}(\det (A))^{2/n -1} \det A \tr \left(A^{-1} \frac{dA}{dt}\right)$$\
$$=\frac{2}{n} (\det A)^{2/n} \tr (B) I.$$\

Multiplying by $(A^T)^{-1}$ on the left and $A^{-1}$ on the right we're left with 
$$ B^T+B = \frac{2}{n}(\det A)^{2/n} \tr (B) \underbrace{(A^T)^{-1}A^{-1}}_{=(AA^T)^{-1}}\\
= \frac{2}{n} \tr(B) I$$\\
as desired. $\qedsymbol$

**Theorem** 
If the flow of $v$ is conformal, then $v$ is of the form 
$$v(x) = a+Bx+2(c \cdot x)x-|x|^2c$$\
for some $c\in \mathbb{R}^n$, where $B\in M_{n \times n}$ satisfies $B+B^T = \frac{2}{n} \tr(B) I_n$.\

_Proof_: Using the lemma we get $\partial_i v_i = \partial_j v_j$  for all $i,j$
and $\partial_i v_j &= - \partial_j v_i$ for all $i \ne j$. By repeatedly using these facts we can deduce that all third order partial derivatives of $v$ vainsh
Therefore,\
$$v_i(x) = a_i + \sum_j b_{ij}x_j + \sum_{j,k} c_{ijk}x_jx_k$$\
for some coefficients $c_{ijk}$ such that $c_{ijk}=c_{ikj}$. Next we calculate\
$$\partial_i v_i(x) &= b_j + 2 \sum_{k} c_{ijk}x_k\\
\partial_k \partial_j v_i(x)& = 2c_{ijk}.$$\

We know $c_{iik}=c_{jjk}$ for all $i,j$ so set $c_k:=c_{iik}$ and $c = (c_1,\dots,c_n$). Using the symmetries again we get $c_{ikk} = -c_{kik} = -c_{kki}=-c_i$. 
Now we can rewrite $v_i$ as\
$$v_i(x) = a_i + \sum_{j}b_{ij} x_j + 2 \left(\sum_k c_k x_k \right)x_i -c_i \sum_{k}x^k \\
=a_i +\sum_{j} b_{ij}x_j + 2(c \cdot x) x_i -|x|^2 c_i.$$\
Writing $(B)_{ij}=b_{ij}$ and combining the equations for $i=1,2,\dots,n$ we get what we want. $\qedsymbol$

Let $v(x) =e_j$ and $\varphi_t(x) = x+te_j$ its flow, where $e_j$ is the $j$th standard basis vector. 
Let $g=f^{-1}$ and consider $h_t=g \circ \varphi_t \circ f : V \to U$, with $V \subset U$ chosen so that $\varphi_t(f(V)) \subset f(U)$. Then we have\ 
$$\frac{d}{dt}h_t(x) = D_{f(h_t(x))}g \cdot v(f(x)).$$
We conclude $h_t$ is the flow of the vector field $w(x) =D_{f(x)}g \cdot v(f(x))$.



