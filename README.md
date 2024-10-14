# Behind Nova: cross-terms computation for high degree gates

**Author**: Danny Willems
**First version**: Aug 14th 2024.
**Last edit**: Oct 14th 2024.


In this document, we will describe how to generalize Nova to support high degree gates, and also give formulas and tricks to compute the cross-terms in the case of high-degree $d$.
This document is used as a complimentary documentation to [Arrabiata](https://github.com/o1-labs/proof-systems/pull/2433).

## Background

[Nova](https://eprint.iacr.org/2021/370) introduced the concept of folding schemes, a novel accumualion scheme for NP relations described by R1CS. Without going into the details of Nova (we suggest the reader to have a look for the security of the accumulation scheme), we will show how to generalize to multi-variate polynomials of higher degree. This document is not describing a novel method, as far as we know, and we think it is simply a more detailed description of what is described in [ProtoStar](https://eprint.iacr.org/2023/620) and maybe some descriptive notes of the computations required for the cross-terms if someone wants to implement these accumulation schemes. **There is no claim of novelty in this document.**

Let start with multi-variate polynomials of maximum degree d. Let use the notation $\mathbb{F}^{\le d}[X_{1}, ..., X_{n}]$ for the vector space.

As a motivation, we start with two variables, and maximum degree 2. The vector space is described by the set
$$
\mathbb{F}^{\le 2}[X_{1}, X_{2}] = \left\{ a_{0} + a_{1} X_{1} + a_{2} X_{2} + a_{3} X_{1} X_{2} + a_{4} X_{1}^2 + a_{5} X_{2}^2 \, | \, a_{i} \in \mathbb{F} \right\}
$$

Imagine a relation described by a polynomial $P(X_{1}, X_{2}) \in \mathbb{F}^{\le 2}[X_{1}, X_{2}]$, and let say the prover has access to two set of witnesses $(x_{1}, y_{1})$ and $(x_{2}, y_{2})$. The prover wants to prove they know a random linear combination of these two witnesses, i.e. $(x_{1} + r \, x_{2}, y_{1} + r \, y_{2})$. They can provide commitments to $P(x_{1} + r \, x_{2}, y_{1} + r \, y_{2})$. However, folding schemes aim to describe this evaluation on the random linear combination in terms of the individual components of the combinaison, i.e. $P(x_{1}, y_{1})$ and $P(x_{2}, y_{2})$, where $r$ is a random coin by the verifier.



### Homogeneous polynomial/relaxed R1CS

In the Nova paper, the authors introduce the concept of "relaxed R1CS". In this document, we will instead work with "homogeneous" polynomials (which is a generalisation of "relaxed R1CS"). Note that ProtoStar already describes this concept.

A polynomial $P$ is homogeneous of degree $d$ if for any evaluation $(x_{1}, \cdots, x_{n})$, and any random scalar $r$, the following equality is true:
$$
P(r \, x_{1}, \cdots, r \, x_{n}) = r^d P(x_{1}, \cdots, x_{n})
$$

> Geometrically, lines are homegeneous polynomials of degree 1. The value $r$ is used to take a "random point on the line formed by the vector $(x_{1}, \cdots, x_{n})$". Homogeneous polynomials are common: the projective coordinates of an elliptic curve given in short Weierstrass form ($X^3 + a X Z^2 + b Z^3 - Y^2 Z = 0$) is the homogeneous version of the affine coordinates ($X^3 + a X + b - Y^2 = 0$).

The set of homogeneous polynomials is described by the sum of all monomials of degree $d$. For instance, if you take $\mathbb{F}^{\le 2}[X_{1}, X_{2}]$, the homogeneous polynomials of degree 2 of this vector space are $\left\{ a_{3} X_{1} X_{2} + a_{4} X_{1}^2 + a_{5} X_{2}^2 \right\}$.

There is a direct observation that any polynomial can be homogeneized by adding a new variable.
For instance, if you have $P(X_{1}, X_{2}) \in \mathbb{F}^{\le 2}[X_{1}, X_{2}]$, you can make it homogeneous of degree $2$ by adding a variable $Y$. The polynomial 

$$
\begin{align}
P(X_{1}, X_{2}) = \, & a_{0}  & + \\
                   & a_{1} \, X_{1} & + \\
                  & a_{2} \, X_{2} & + \\ 
                  & a_{3} \, X_{1} X_{2} & + \\
                  & a_{4} \, X_{1}^2 & + \\
                  & a_{5} \, X_{2}^2
\end{align}
$$

becomes

$$
\begin{align}
P'(X_{1}, X_{2}, Y) = \, & a_{0} \, Y^2 & + \\
                     & a_{1} \, X_{1} Y & + \\
                     & a_{2} \, X_{2} Y & + \\ 
                     & a_{3} \, X_{1} X_{2} & + \\
                     & a_{4} \, X_{1}^2 & + \\
                     & a_{5} \, X_{2}^2
\end{align}
$$

The definition of "relaxed R1CS" used in the Nova paper is the equivalent of the homogeneized version of the polynomial (of degree 2) described by the matrices of the system. The value $u$ is the equivalent of the additional variable $Y$ used here.

### From homogeneous polynomials to folding schemes

Let come back to the evaluations of $P(x_{1} + r x_{2}, y_{1} +r y_{2})$. Let use the homogeneized version of $P$, coined $P'$. We will use the notation $u_{1} + r u_{2}$ for the evaluation of the third variable.

First, we evaluate $P'$ on the random linear combination.

$$
\begin{align}
P'(x_{1} + r \, x_{2},
  y_{1} + r \, y_{2},
  u_{1} + r \, u_{2}) = \, & a_{0} (u_{1} + r u_{2})^2 + \\
                        & a_{1} (x_{1} + r \, x_{2}) \, (u_{1} + r u_{2}) + \\
                                    & a_{2} (y_{1} + r \, y_{2}) \, (u_{1} + r u_{2}) + \\
                                    & a_{3} (x_{1} + r \, x_{2}) \, (y_{1} + r \, y_{2}) + \\
                                    & a_{4} (x_{1} + r \, x_{2})^{2} + \\
                                    & a_{5} (y_{1} + r \, y_{2})^{2}
\end{align}
$$

By developping the coefficients, we get:


$$
\begin{align}
P'(x_{1} + r x_{2},
  y_{1} + y_{2},
  u_{1} + r u_{2}) = & P'(x_{1}, y_{1}, u_{1}) + \\
                     & P'(r \, x_{2}, r \, y_{2}, r u_{2}) + \, \\
                     & r \, [ a_{0} \, 2 \, u_1 u_2 \, + \, \\
                     & \,\,\,\,\, a_{1} (x_{1} u_{2} + x_{2} u_{1}) \, + \\
                     & \,\,\,\,\, a_{2} (y_{1} u_{2} + y_{2} u_{1}) \, + \\
                     & \,\,\,\,\, a_{3} (x_{1} y_{2} + x_{2} y_{1}) \, + \\
                     & \,\,\,\,\, a_{4} (2 x_{1} x_{2}) \, + \\
                     & \,\,\,\,\, a_{5} (2 y_{1} y_{2})]
\end{align}
$$

and as the polynomial P is homogeneous, we do have:
$$
P'(r \, x_{2}, r \, y_{2}, r u_{2}) = r^2 P'(x_{2}, y_{2}, u_{2})
$$

In Nova, the last term in `r` is called the "cross-terms", named $T$.
Then, we have, as in Nova,

$$
\begin{align}
P'(x_{1} + r x_{2},
  y_{1} + y_{2},
  u_{1} + r u_{2}) = & P'(x_{1}, y_{1}, u_{1}) + \\
                     & r^2 \, P'(x_{2}, y_{2}, u_{2}) + \, \\
                     & r \, T(x_{1}, y_{1}, u_{1}, x_{2}, y_{2}, u_{2})
\end{align}
$$

Then, we have an expression for the random linear combination in terms of the separated terms $(x_{1}, y_{1}, u_{1})$ and $(x_{2}, y_{2}, u_{2})$ that the prover can commit to.

We notice that in the expression for the cross-terms, we ordered by monomials. It means that if our polynomial is the sum of the monomials $a_{1} X_{1}$ and $a_{3} X_{1} X_{2}$, we can only care about the cross-terms that are induced by the coefficients $a_{1}$ and $a_{3}$, and we can skip the rest.

It is important to notice that because in practical cases, we do know in advance the form of our constraint, and therefore, we can optimize the computation while building the witness.


### Generalisation to degree $3$

What happens now if we want to use homogeneous polynomials of degree 3?
We could generalize the process we describe above. And by developping the coefficients, we will get two cross-terms: one wrapped by `r`, and one by `r^2`.
The accumulation scheme described by Nova would simply be generalized to $E_{1} - r T_{1} - r^2 T_{2} + E_{2}$, and the random combiner $r$ will be generated after $T_{1}$ and $T_{2}$ have been committed and added to the transcript. Also, the cross-terms can also be computed by focusing on the monomials.

About the formula of the cross-terms, we suggest to describe directly the following generalisation to degree $d$.

### Generalisation to degree $d$

Naturally, the generalisation to degree $d$ is done by having $d - 1$ cross-terms, all wrapped by a power of $r$. The question now is how do we compute them? Does it exist a generic formula?


We notice that we can compute the cross-terms by looking at each individual monomials, and sum the individual contributions.
Therefore, we can focus on finding a formula for the polynomials of the form $a_{i} X_{1}^{n_{1}} X_{2}^{n_{2}} Y^{n}$ where $n = d - |n_{1} - n_{2}|$ (i.e. $n$ is the additional degree to make the monomial homogeneous).

For an evaluation point $(x_{1} + r x_{2}, y_{1} + r y_{2}, u_{1} + r u_{2})$ we have to compute:

$$
(x_{1} + r x_{2})^{n_{1}} (y_{1} + r y_{2})^{n_{2}}  (u_{1} + r u_{2})^{n}
$$
which is given by the multi-binomial coefficients


$$
\sum_{i = 0}^{n_{1}} \sum_{j = 0}^{n_{2}} \sum_{k = 0}^{n} \binom{n_{1}}{i} x_{1}^{i} (r x_{2})^{n_{1} - i} \binom{n_{2}}{j} y_{1}^{j} (r y_{2})^{n_{2} - j} \binom{n}{k} u_{1}^{k} (r u_{2})^{n - k}
$$

When developping, we notice that the term of $r^d P(x_{2}, y_{2}, u_{2})$ is reached when $i + j + k = 0$ and the term $P(x_{1}, y_{1}, u_{1})$ is reached when $i + j + k = d$. The other terms are the "cross-terms". The cross-terms for $r$ is when $i + j + k = 1$, and more generally, the cross-terms for $r^l$ are the terms when $i + j + k = l$.

#### How many terms are in the sum?

We could think that the formula contains many terms, and it might be expensive.
However, it is worth noticing the the maximum value of $n_{1} + n_{2} + n = d$. Therefore, if $d = 2$, only maximum 4 terms appear in the sum. More general, maximum $2^d$ terms appear in the sum.


### Generalisation to multi-variate polynomials with more than 2 variables

It is easy to generalize to more than 2 variables. The multi-binomial formula to compute

$$
(x_{1} + r y_{1})^{n_{1}} \cdots (x_{l} + r x_{l})^{n_{l}}
$$

is

$$
\sum_{i_{1} = 0}^{n_{1}} \cdots \sum_{i_{l} = 0}^{n_{l}} \binom{n_{1}}{i_{1}} x_{1}^{i} (r y_{1})^{n_{1} - i_{1}} \cdots \binom{n}{i_{l}} x_{l}^{i_{l}} (r y_{i_{l}})^{n_{l} - i_{l}}
$$

### Algorithms and optimisations

An implementation would work as follow:
- split the constraints into monomials
- compute the cross-terms induced by each monomial (a cache can be used for intermediate powers).
- accumulate the previous cross-term

Idea TBC:
- use an index for each variable, and the index can be used to get the evaluation of $\omega^i$ in a contiguous array. Symbolic computation. The monomials are constructed using indices.

#### Computation of the sums

When transforming the formula into an algorithm, the inner sums
$$
\sum_{i_{1} = 0}^{n_{1}} \cdots \sum_{i_{l} = 0}^{n_{l}}
$$

must be decomposed into a linear stream of indices. See [this method](https://github.com/o1-labs/proof-systems/blob/master/mvpoly/src/utils.rs#L223) in mvpoly.

#### Computation of cross-terms for shifted polynomials

In practical cases, custom gates will be a sum of individual constraints which will be linearly combined using a random challenge often noted $\alpha$.
A question arises from there: given the cross-terms for a list of polynomial, how do we compute the cross-terms of the linearly combined form?

TODO

### Practical examples

#### Elliptic curve addition in short Weierstrass form

Let $E(\mathbb{F})$ be an elliptic curve over the finite field $\mathbb{F}$.
Let define $P_{1} = (X_{1}, Y_{1})$ and $P_{2} = (X_{2}, Y_{2})$ be two *different* points on the curve.
The constraints to compute the addition of the two points require:
- 7 columns: $X_1$, $X_{2}$, $X_{3}$, $Y_{1}$, $Y_{2}$, $Y_{3}$, $\lambda$.
- Three constraints:
$$
\begin{align}
& λ (X_1 - X_2) - Y_1 + Y_2 = 0 \\
& X_3 + X_1 + X_2 - λ^2 = 0 \\
& Y_3 - λ (X_1 - X_3) + Y_1 = 0 \\
\end{align}
$$


## Implementations

Code can be found in the repository [o1-labs/proof-systems](https://github.com/o1-labs/proof-systems/), in the crate [mvpoly](https://github.com/o1-labs/proof-systems/tree/master/mvpoly).

A project using it is [Arrabiata](https://github.com/o1-labs/proof-systems/tree/master/arrabiata) which provides an implementation of a folding argument for arbitrary custom gates.
