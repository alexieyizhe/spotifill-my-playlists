__CS 240 |__ January 10, 2019

# Big-O Order Notation

==__Formal Definition__:== $f(n)$ is $O(g(n))$ where:

- $g(n)$ is a upper bound on growth rate (as inputs get bigger and bigger, what _type_ of growth is there)

- $f(n)$ grows no faster than $g(n)​$

- $f(n) \leq c * g(n), \ \forall n \geq n_o​$ where $c \gt 0, n_n \gt 0​$

  - A key observation is that this is not $\forall \ n \in \N​$ - sometimes, $f(n) \geq g(n)​$ for small $n​$ until it changes for larger $n​$:

![Image result for big o graph](./assets/Rt5wO.gif)

Showing that $f(n)$ is $O(g(n))$ __from first principles__ means to _use the definition and find $c$, $n_o$._ Make sure you're deriving them from the formula, not arbitrary trial and error.

Typically, we think that $g(n)$ will "dominate" $f(n)$, but this is not always true.

> __Ex. Prove that $f(n) = (n + 1)^5 \in O(g(n) = n^5)$.__
>
> To show this by first principles, we need to show that $(n + 1)^5 \leq c * n^5 \ \ \forall \ n \geq n_0, c \gt 0$.
> $g(n)$ will never "dominate" $f(n)$, so we need a $c \gt 0$.
>
> Observe:
> $n + 1 \leq 2n$                                            [$\forall n \geq 1$]
> $ (n + 1)^5 \leq (2n)^5 = 2^5 * n^5 = 32n^5$    [$\forall n \geq 1$]
> Thus, letting $c = 32, n_0 = 1$ means that $(n + 1)^5 \in O(n^5)$.

> __Ex. Prove that $f(n) = n^2 + nlog\  n + n \in O(g(n) = n^2)$.__
>
> Observe:
> $n^2 + nlog\ n + n \leq n^2 + n^2 + n^2$      [$\forall n \geq 1$]
> $n^2 + nlog\ n + n \leq 3n^2$                      [$\forall n \geq 1 $]

### Properties of Big-O

1. $af(n) \in O(f(n))$ for any constant $ a \gt 0$.

   __Proof:__
   Choose $c = a, n_0 = 1$.

2. If $f(n) \in O(g(n))$ and $g(n) \in O(h(n))$, then $f(n) \in O(h(n))$.

   __Proof:__ 
   We know that $f(n) \leq c_1 * g(n), \forall n \geq n_1$ and $g(n) \leq c_2 * h(n), \forall n \geq n_2$.
   So, $c_1 * g(n) \leq c_1 *c_2 * h(n)$ and so $f(n) \leq (c_1 * c_2) *h(n)$, for some $n_3$.
   Then, choose $n_3 = max\{n_1, n_2\}$.

3. $max\{f(n), g(n)\} \in O(f(n) + g(n))$.

   **Proof:** Simply show that since $f(n), g(n) \gt 0$, the $RHS \geq LHS$ always.

4. $a_0 + a_1x +a_2x^2 + ...+a_nx^n \in O(x^n), a_i \gt 0$.

5. $n^x \in O(a^n), x \gt 0, a \gt 1$.

6. $(log \ n)^x \in O(n^y), x, y \gt 0$.

## Asymptotic Lower & Upper Bounds

We want a __tight asymptotic bound.__ This leads us to develop two more sets of notations:

### $\Omega$-Notation

- $f$ and $g$ are switched compared to Big-O Notation
- "grows no slower than"
- asymptotic _lower_ bound
- If $f(n) \in \Omega(g(n))​$, then $f(n) \geq c * g(n), \ \forall n \geq n_o​$ where $c \gt 0, n_n \gt 0​$

> __Ex. Show that $n^3 log \  n \in \Omega(n^3)$.__
>
> Observe:
> $n^3 log \  n \geq c * n^3​$, $\forall n \geq n_o​$.
> Let $c = 1, n_o = 2​$ to achieve the desired result.

> __Ex. Show that $2n^2 + 3n + 11 \geq c * n^2$.__
>
> Let $c = 2$. Then:
> $2n^2 + 3n + 11 \geq 2n^2, \forall n \geq 1 = n_o $.

> __Ex. Show that $\frac{1}{2}n^2 - 5n \in \Omega(n^2)​$.__
>
> We need to show that $\frac{1}{2}n^2 - 5n \geq c * n^2, \forall n \geq n_o$ for $c, n_o \gt 0$.
> Find $c \lt \frac{1}{2}$ (the leading term of $f(n)$), and $n_o \gt 10$ (to ensure that $f(n)\gt 0$).
>
> Choose $c = \frac{1}{4}$. Then, we need $\frac{1}{2}n^2 - 5n = \frac{1}{4} n^2 + \frac{1}{4}n^2 - 5n \geq \frac{1}{4}n^2$. 
>
> Now, subtract $\frac{1}{4}n^2$ from both sides to get:
> $\frac{1}{4}n^2 - 5n \geq 0$
> $\frac{1}{4}n \geq 5$
> $n \geq 20$
>
> So, if $c = \frac{1}{4}, n_o = 20$, then we achieve the desired result.

### $\Theta$-Notation: 

- $c_1 *g(n) \leq |f(n)| \leq c_2 * g(n)$
- $f(n)$ grows at the same rate as $g(n)$
- an upper AND lower bound ensures a tight bound



### Strictly Smaller/Larger Asymptotic Bounds

To show strict inequality, we need to introduce two new notations:

#### $o$-notation

==__Formal Definition:__== $f(n) \in o(g(n))​$ if for __all__ constants $c \gt 0​$, there exists a constant $n_o \gt 0​$ such that $|f(n)| \lt c |g(n)|​$ for all $n \geq n_o​$.

- $f$ dominated by $g$
- $f(n)$ grows slower than $g(n)$
- given **any** $c$, we can always find an $n_o \gt 0$ to satisfy the properties

> __Ex. Show that $2019n^2 + 1388n \in o(n^3)$.__
>
> We need to show that $\forall c \gt 0, \exists \ n_o \gt 0$ such that $2019n^2 + 1388n \lt c *n^3$.
>
> _Note that $0 \lt c \lt 1$ can be possible_
> $2019n^2 + 1388n \lt 5000n^2,  \forall \ n \geq 1$
> $2019n^2 + 1388n \leq \frac{5000}{n} * n * n^2,  \forall \ n \geq 1$
> We need $\frac{5000}{n} \leq c$ to find a valid $c$, so just rearrange and for any $c$, choose $n_o = \frac{5000}{c}$.

__Little-o is stronger than Big-O - that is, that when you have a function $\in o(g(n))$, it's also $\in O(g(n))$ as well.__

#### $\omega$-notation

==__Formal Definition: __==$f(n) \in \omega(g(n))​$ if for **all** constants $c \gt 0​$, there exists a constant $n_o \gt 0​$ such that $0 \leq c |g(n)| \lt |f(n)|​$ for all $n \geq n_o​$.

- $g$ dominated by $f$
- $f(n)$ grows faster than $g(n)$
- given **any** $c$, we can always find an $n_o \gt 0$ to satisfy the properties
- _rarely_ proven from first principles

