---
title: "Liouville's Theorem on Conformal Rigidity"
date: 2021-01-23
draft: false
tags: ["geometry","analysis","conformal", "ODE", "Liouville's Theorem"]
cotegories: ["geometry", "analysis", "conformal"]
katex: true 

---
In this post I summarize the content and proof of Liouville's Theorem on Conformal Rigidity, which I learned in 2018 from Professor Alex Austin (now at RIT) in his class at UCLA. 

## Conformal Maps
A conformal transformation is one that preserves angles. In two dimensions, this is equivalent to being holomorphic and having a non-vanishing derivative.
There are tons of these transformations (my personal favorites are Mobius transformations). 
In fact, the famous [Riemann mapping Theorem](https://en.wikipedia.org/wiki/Riemann_mapping_theorem) asserts that any simply connected domain \\( U \subset \mathbb{C} \\) admits a bijective conformal map \\(f: U \to \mathbb{D}\\) to the unit disc. 

{{< figure src="https://i.stack.imgur.com/c6nSk.jpg" caption="A conformal map in the complex plane" >}}

In this note we will show that the only conformal maps in dimensions \\(n \ge 3\\) are built from inversions and reflections. It turns out that 2 dimensions is the exception!

First, what exactly does "preserve angles" mean? The nicest way to talk about angles is via an inner product.
Given a vector space \\(V\\) with an inner product \\(\langle \cdot, \cdot \rangle\\), the angle \\(\theta\\) between two vectors \\(v,w \in V\\) is defined by

$$ \cos \theta = \dfrac{ \langle v, w \rangle}{\sqrt{ \langle v,v \rangle \langle w, w \rangle}}.$$

In Euclidean space with the standard inner product, this matches the formula you know. Thus a linear transformation \\(A: V \to V\\)
preserves angles if \\(\langle Av, Aw \rangle = \lambda^2 \langle v, w \rangle\\) for all \\(v,w\\) and some \\(\lambda > 0\\). Note that since \\(\lambda^2 > 0\\) we are requiring the orientation of the angle to be preserved as well.

Thus a conformal linear transformation is just an orientation and distance preserving map (called a special orthogonal matrix)
followed by a dilation. Well, that's pretty boring. But now we can define more general conformal maps by requiring its derivative to
be a conformal linear transformation!

**Definition** 
Let \\(U \subset \mathbb{R}^n\\) be open. A \\(C^1\\) map \\(f:U \to \mathbb{R}^n\\). \\(f\\) is said to be **conformal** if \\(D_x f:T_xU \to T_{f(x)}\mathbb{R}^n\\) is conformal for all \\(x \in U\\).

### Inversions and Reflections
A nice example of a conformal map is spherical inversion. Denote the one-point compactification \\(\mathbb{R}^n \cup \{\infty\}\\) of \\(\mathbb{R}^n\\) by \\(\widehat{\mathbb{R}}^n\\). Let \\(S(a,r) \subset \mathbb{R}^n\\) be the sphere of radius \\(n\\) with center \\(a\\). The inversion about \\(S(a,r)\\) is defined as 

$$\varphi_{a,r}(x) = a + r\frac{x-a}{|x-a|^2}, \quad x \ne a,\infty$$

and \\(\varphi_{a,r}(a) = \infty\\), \\(\varphi_{a,r}(\infty) = a.\\) 

\\(\varphi_{r,a}\\) swaps the interior and exterior of the sphere and fixes \\(S(a,r)\\). One can check that \\(\varphi_{a,r}\\) is an involution, i.e. \\(\varphi_{a,r} \circ \varphi_{a,r} = \text{id}.\\)

We can verify that \\(\varphi_{a,r}\\) is conformal with direct computation. 
First consider \\(\varphi = \varphi_{0,1}\\). We calculate that

$$D_x \varphi = \frac{1}{|x|^2}( I - 2 Q_x )$$

where \\(Q\\) is the symmetric matrix with entries \\(\dfrac{x_i x_j}{|x|^2}\\). 
Note that \\(Q\\) satisfies \\(Q^2=Q\\) since 

$$\begin{aligned}
Q^2_{ij} &= \sum_{k=1}^n \frac{x_k x_j}{|x|^2} \frac{x_i x_k}{|x|^2}\cr
&= \left( \sum_{k=1}^n\frac{x_k^2}{|x|^2}\right)\frac{x_i x_j}{|x|^2} = \frac{x_i x_j}{|x|^2}.
\end{aligned}$$

Hence we get \\((D_x\varphi)^T D_x \varphi = \frac{1}{|x|^4}(I-4Q-x+4Q_x^2) = \frac{1}{|x|^4}I\\) so \\(\varphi\\) is conformal. 
For general \\(\varphi_{r,a}\\), we use an affine transformation \\(\psi(x) = rx +a\\), which is obviously conformal.
Then \\(\varphi_{r,a}=\psi \circ \varphi \circ \psi^{-1}\\) is also conformal.

Another type of transformation we will need is reflections.
Fix \\(a \in \mathbb{R}^n\\) and \\(s>0\\) and let \\(P(a,s) = \\{x \in \mathbb{R}^n| a \cdot x =s\\} \cup \\{\infty\\} \\) be the corresponding plane in \\(\widehat{\mathbb{R}}^n\\). The reflection in \\(P(a,s)\\) is defined as 

$$r_{a,s}(x) = x-2(a \cdot x -s) \frac{a}{|a|^2}, \qquad x \in \mathbb{R}^n$$

and \\(r_{a,s}(\infty) =\infty\\). We define a Mobius transformation on \\(\widehat{\mathbb{R}}^n\\) for \\(n\ge 3\\) 
as a finite composition of reflections in planes and inversions about spheres.

## Liouville's Theorem
Now we can precisely state what we want to prove.

**Liouville's Theorem[^footnote]**: Let \\(U \subset \mathbb{R}^n\\) be open with \\(0 \in U\\) and \\(n \ge 3\\).    
If \\(f:U \to \mathbb{R}^n\\) is \\(C^4\\) and conformal, then \\(f\\) is the restriction of a Mobius transformation. 

[^footnote]: You may have heard of Liouville's Theorem from complex analysis that says every bounded entire function is constant. Many rigidity results of this type are called "Liouville Theorems" 

Actually, the theorem holds for much lower regularity, but we assume \\(C^4\\) for simplicity. The strategy we will use is to
study vector fields whose flow is conformal. The **flow** of a \\(v\\) is a function \\(f_t\\) parametrized by time that satisfies

$$\begin{aligned}
f_0(x) &= x\cr
\frac{d}{dt}f_t(x) &= v(f_t(x))\end{aligned}$$

for every \\(x\\). The point \\(x\\) is "flowing down the river" determined by the vector field (for this reason I've heard the [Lie derivative](https://en.wikipedia.org/wiki/Lie_derivative) called the "fisherman's derivative"!)
Therefore if we can transfer the problem of characterizing conformal maps into one about vector fields and differential equations.
This tactic can be applied in many other problems!

**Lemma**
Let \\(v:\mathbb{R}^n \to \mathbb{R}^n\\) be a \\(C^1\\) vector field. Then the flow of \\(v\\) is conformal if and only if
$$(Dv)^T+Dv = \frac{2}{n} \operatorname{tr}(Dv) I.$$

_Proof_: Let \\(f_t\\) be the local flow of \\(v\\) and \\(p \in \mathbb{R}^n\\). For convenience set \\(A=D_p f_t\\), \\(B=D_{f_t(p)}v\\). We differentiate the relation\
$$A^T A = (\det A)^{2/n} \; I$$
with respect to \\(t\\) and use [Jacobi's formula](https://en.wikipedia.org/wiki/Jacobi%27s_formula) to obtain

$$\begin{aligned} A^TB^TA+A^TBA &= \frac{2}{n}(\det (A))^{2/n -1} \det A \operatorname{tr} \left(A^{-1} \frac{dA}{dt}\right)\cr
&=\frac{2}{n} (\det A)^{2/n} \operatorname{tr} (B) I.\end{aligned}$$

Multiplying by \\((A^T)^{-1}\\) on the left and \\(A^{-1}\\) on the right we're left with 
$$\begin{aligned} B^T+B &= \frac{2}{n}(\det A)^{2/n} \operatorname{tr} (B) \underbrace{(A^T)^{-1}A^{-1}}_{=(AA^T)^{-1}}\cr
&= \frac{2}{n} \operatorname{tr}(B) I\end{aligned}$$

as desired. \\(\blacksquare\\)

**Theorem** 
If the flow of \\(v\\) is conformal, then \\(v\\) is of the form 

$$v(x) = a+Bx+2(c \cdot x)x-|x|^2 c$$

for some \\(c\in \mathbb{R}^n\\), where \\(B\in M_{n \times n}\\) satisfies \\(B+B^T = \frac{2}{n} \operatorname{tr}(B) I_n\\).

_Proof_: Using the lemma we get \\(\partial_i v_i = \partial_j v_j\\)  for all \\(i,j\\)
and \\(\partial_i v_j = - \partial_j v_i\\) for all \\(i \ne j\\). By repeatedly using these facts we can deduce that all third order partial derivatives of \\(v\\) vanish.
Therefore,

$$v_i(x) = a_i + \sum_j b_{ij}x_j + \sum_{j,k} c_{ijk}x_jx_k$$

for some coefficients \\(c_{ijk}\\) such that \\(c_{ijk}=c_{ikj}\\). Next we calculate

$$\begin{aligned} \partial_i v_i(x) &= b_j + 2 \sum_{k} c_{ijk}x_k\cr
\partial_k \partial_j v_i(x)& = 2c_{ijk}\end{aligned}$$

We know \\(c_{iik}=c_{jjk}\\) for all \\(i,j\\) so set \\(c_k:=c_{iik}\\) and \\(c = (c_1,\dots,c_n\\)). Using the symmetries again we get \\(c_{ikk} = -c_{kik} = -c_{kki}=-c_i\\). 
Now we can rewrite \\(v_i\\) as

$$\begin{aligned}
v_i(x) &= a_i + \sum_{j}b_{ij} x_j + 2 \left(\sum_k c_k x_k \right)x_i -c_i \sum_{k}x^k \cr
&=a_i +\sum_{j} b_{ij}x_j + 2(c \cdot x) x_i -|x|^2 c_i\end{aligned}$$

Combining the equations for \\(i=1,2,\dots,n\\) we get what we want. \\(\blacksquare\\)

Let \\(v(x) =e_j\\) and \\(\varphi_t(x) = x+te_j\\) its flow, where \\(e_j\\) is the \\(j\\)th standard basis vector. 
Let \\(g=f^{-1}\\) and consider \\(h_t=g \circ \varphi_t \circ f : V \to U\\), with \\(V \subset U\\) chosen so that \\(\varphi_t(f(V)) \subset f(U)\\). Then we have

$$\frac{d}{dt}h_t(x) = D_{f(h_t(x))}g \cdot v(f(x)).$$

We conclude \\(h_t\\) is the flow of the vector field \\(w(x) =D_{f(x)}g \cdot v(f(x))\\).
Now we can execute our plan to prove Liouville's theorem.

_Proof of Liouville_: 
By composing with an affine map we can assume \\(f(0)=0\\) and  \\(D_0f =I\\). 
Set \\(\varphi = \varphi_{0,1}\\) as before and define \\(G = \varphi \circ f\\), which is conformal on \\(U\\). It suffices to show that \\(G\\) is a M\"obius trasformation. We will embed \\(G\\) in the flow of \\(v = (DG)^{-1}e_i.\\) We get 

$$\begin{aligned}
D_xG &= D_{f(x)}\varphi D_x f\cr
&= \frac{1}{|f(x)|^2}(I-2Q_{f(x)})D_x f\end{aligned}$$

We know \\((I-2Q_x)^2 = I,\\) so \\((I-2Q_x)^{-1} = I-2Q_x\\). With this we can calculate 

$$\begin{aligned}
(D_xG)^{-1}e_i &= (D_x f)^{-1}(D_{f(x)}\varphi)^{-1}e_i\cr 
&= (D_xf)^{-1} \cdot |f(x)|^2 (I -2Q_{f(x)})e_i\cr
&= |f(x)|^2 (D_xf)^{-1}e_i - 2(f(x) \cdot e_i)
\end{aligned}$$

Using the Theorem on conformal vector fields, we can write 
\\((D_x G)^{-1}e_i = a+Bx +2(c\cdot x)x-|x|^2c.\\)

\\(f(0)=0\\) implies \\((D_0G)^{-1}e_i = 0\\), so \\(a=0\\). We contend that \\(B \equiv 0\\) as well. 
Let \\(u \in \mathbb{R}^n\\) be unit length, and \\(\epsilon>0\\). We consider  \\(\epsilon B(u)\\) as \\(\epsilon \to 0\\):  


$$\begin{aligned}
B( \epsilon u) &= (D_{\epsilon u}G)^{-1}e_i-2(c \cdot \epsilon u) \epsilon u - |\epsilon u|^2c\cr 
&= |f(\epsilon u)|^2(D_{\epsilon u}f)^{-1}\left(I-2Q_{f(\epsilon u)}\right)e_i -\epsilon^2((c\cdot u)u -c)\cr
&= \epsilon^2 \left| u + \frac{o(\epsilon)}{\epsilon}\right| (D_{\epsilon u}f)^{-1}\left(I-2Q_{f(\epsilon u)}\right)e_i -\epsilon^2((c\cdot u)u -c) 
.\end{aligned}$$

where we used linear approximation \\(f(x) = x+ o(|x|)\\). Dividing through by \\(\epsilon\\), 
the LHS is independent of \\(\epsilon\\) while the RHS has a factor of \\(\epsilon\\). As \\(\epsilon \to 0\\), the RHS converges to \\(0\\). 
Consequently, we must have \\(B \equiv 0\\), which proves the contention. To find \\(c\\) we use the same argument with \\(x = \epsilon c\\) to get 

$$|c|^2c = \left|c + \frac{o(|\epsilon c|)}{|\epsilon c|}|c| \right|^2(D_{\epsilon c}f)^{-1}(I-2Q_{c})e_i$$

We can write \\(Q_{f(\epsilon c)}=Q_{\epsilon c} + E(s)\\) where \\(E(s) \to 0\\) as \\(s\to 0\\) (in the space of matrices). Therefore \\(|c|^2c = |c|^2(I-2Q_{c})e_i\\).
Solving for \\(e_i\\) we have

$$e_i = (E-2Q_c)c = c-2\frac{c\cdot c}{|c|^2}c = -c.$$

We have shown that 
$$\begin{aligned}
(D_xG)^{-1}e_i &= - 2(e_i \cdot x)x - |x|^2e_i\cr
&=|x|^2(I-2Q_x)e_i\cr
&= (D_x \varphi)^{-1}e_i
\end{aligned}$$

Thus we conclude that \\(D_x G = D_x \varphi.\\) Hence \\(f(x) = \varphi(\varphi(x) +d)\\) for some constant \\(d\in \mathbb{R}^n,\\)
which is indeed a Mobius transformation. \\(\blacksquare\\)

