---
jupytext:
  text_representation:
    extension: .md
    format_name: myst
kernelspec:
  display_name: Julia
  language: julia
  name: julia-1.8
---

(ree)=
```{raw} html
<div id="qe-notebook-header" style="text-align:right;">
        <a href="https://quantecon.org/" title="quantecon.org">
                <img style="width:250px;display:inline;" src="https://assets.quantecon.org/img/qe-menubar-logo.svg" alt="QuantEcon">
        </a>
</div>
```

# {index}`Rational Expectations Equilibrium <single: Rational Expectations Equilibrium>`

```{contents} Contents
:depth: 2
```

```{epigraph}
"If you're so smart, why aren't you rich?"
```

## Overview

This lecture introduces the concept of *rational expectations equilibrium*.

To illustrate it, we describe a linear quadratic version of a famous and important model
due to Lucas and Prescott {cite}`LucasPrescott1971`.

This 1971 paper is one of a small number of research articles that kicked off the *rational expectations revolution*.

We follow Lucas and Prescott by employing a setting that is readily "Bellmanized" (i.e., capable of being formulated in terms of dynamic programming problems).

Because we use linear quadratic setups for demand and costs, we can adapt the LQ programming techniques described in {doc}`this lecture <../dynamic_programming/lqcontrol>`.

We will learn about how a representative agent's problem differs from a planner's, and how a planning problem can be used to compute rational expectations quantities.

We will also learn about how a rational expectations equilibrium can be characterized as a [fixed point](https://en.wikipedia.org/wiki/Fixed_point_%28mathematics%29) of a mapping from a *perceived law of motion* to an *actual law of motion*.

Equality between a perceived and an actual law of motion for endogenous market-wide objects captures in a nutshell what the rational expectations equilibrium concept is all about.

Finally, we will learn about the important "Big $K$, little $k$" trick, a modeling device widely used in macroeconomics.

Except that for us

* Instead of "Big $K$" it will be "Big $Y$"
* Instead of "little $k$" it will be "little $y$"

### The Big $Y$, little $y$ trick

This widely used method applies in contexts in which a "representative firm" or agent is a "price taker" operating within a competitive equilibrium.

We want to impose that

* The representative firm or individual takes *aggregate* $Y$ as given when it chooses individual $y$, but $\ldots$.
* At the end of the day, $Y = y$, so that the representative firm is indeed representative.

The Big $Y$, little $y$ trick accomplishes these two goals by

* Taking $Y$ as beyond control when posing the choice  problem of who chooses $y$;  but $\ldots$.
* Imposing $Y = y$ *after* having solved the individual's optimization  problem.

Please watch for how this strategy is applied as the lecture unfolds.

We begin by applying the  Big $Y$, little $y$ trick in a very simple static context.

#### A simple static example of the Big $Y$, little $y$ trick

Consider a static model in which a collection of $n$ firms produce a homogeneous good that is sold in a competitive market.

Each of these $n$ firms sells output $y$.

The price $p$ of the good lies on an inverse demand curve

```{math}
:label: ree_comp3d_static

p = a_0 - a_1 Y
```

where

* $a_i > 0$ for $i = 0, 1$
* $Y = n y$ is the market-wide level of output

Each firm has total cost function

$$
c(y) = c_1 y + 0.5 c_2 y^2,
\qquad c_i > 0 \text{ for } i = 1,2
$$

The profits of a representative firm are $p y - c(y)$.

Using {eq}`ree_comp3d_static`, we can express the problem of the representative firm as

```{math}
:label: max_problem_static

\max_{y} \Bigl[ (a_0 - a_1 Y) y - c_1 y - 0.5 c_2 y^2 \Bigr]
```

In posing problem {eq}`max_problem_static`, we want the firm to be a *price taker*.

We do that by regarding $p$ and therefore $Y$ as exogenous to the firm.

The essence of the Big $Y$, little $y$ trick is *not* to set $Y = n y$ *before* taking the first-order condition with respect
to $y$ in problem {eq}`max_problem_static`.

This assures that the firm is a price taker.

The first order condition for problem {eq}`max_problem_static` is

```{math}
:label: BigYsimpleFONC

a_0 - a_1 Y - c_1 - c_2 y = 0
```

At this point, *but not before*, we substitute $Y = ny$ into {eq}`BigYsimpleFONC`
to obtain the following linear equation

```{math}
:label: staticY

a_0 - c_1 - (a_1 + n^{-1} c_2) Y = 0
```

to be solved for the competitive equilibrium market wide output $Y$.

After solving for $Y$, we can compute the competitive equilibrium price $p$ from the inverse demand curve {eq}`ree_comp3d_static`.

### Further Reading

References for this lecture include

* {cite}`LucasPrescott1971`
* {cite}`Sargent1987`, chapter XIV
* {cite}`Ljungqvist2012`, chapter 7



```{code-cell} julia
using LinearAlgebra, Statistics
```

## Defining Rational Expectations Equilibrium

```{index} single: Rational Expectations Equilibrium; Definition
```

Our first illustration of a rational expectations equilibrium involves a market with $n$ firms, each of which seeks to maximize the discounted present value of profits in the face of adjustment costs.

The adjustment costs induce the firms to make gradual adjustments, which in turn requires consideration of future prices.

Individual firms understand that, via the inverse demand curve, the price is determined by the amounts supplied by other firms.

Hence each firm wants to  forecast future total industry supplies.

In our context, a forecast is generated by a belief about the law of motion for the aggregate state.

Rational expectations equilibrium prevails when this belief coincides with the actual
law of motion generated by production choices induced by this belief.

We formulate a rational expectations equilibrium in terms of a fixed point of an operator that maps beliefs into optimal beliefs.

(ree_ce)=
### Competitive Equilibrium with Adjustment Costs

```{index} single: Rational Expectations Equilibrium; Competitive Equilbrium (w. Adjustment Costs)
```

To illustrate, consider a collection of $n$ firms producing a homogeneous good that is sold in a competitive market.

Each of these $n$ firms sells output $y_t$.

The price $p_t$ of the good lies on the inverse demand curve

```{math}
:label: ree_comp3d

p_t = a_0 - a_1 Y_t
```

where

* $a_i > 0$ for $i = 0, 1$
* $Y_t = n y_t$ is the market-wide level of output

(ree_fp)=
#### The Firm's Problem

Each firm is a price taker.

While it faces no uncertainty, it does face adjustment costs

In particular, it chooses a production plan to maximize

```{math}
:label: ree_obj

\sum_{t=0}^\infty \beta^t r_t
```

where

```{math}
:label: ree_comp2

r_t := p_t y_t - \frac{ \gamma (y_{t+1} - y_t )^2 }{2},
\qquad  y_0 \text{ given}
```

Regarding the parameters,

* $\beta \in (0,1)$ is a discount factor
* $\gamma > 0$ measures the cost of adjusting the rate of output

Regarding timing, the firm observes $p_t$ and $y_t$ when it chooses $y_{t+1}$ at at time $t$.

To state the firm's optimization problem completely requires that we specify dynamics for all state variables.

This includes ones that the firm cares about but does not control like $p_t$.

We turn to this problem now.

#### Prices and Aggregate Output

In view of {eq}`ree_comp3d`, the firm's incentive to forecast the market price translates into an incentive to forecast aggregate output $Y_t$.

Aggregate output depends on the choices of other firms.

We assume that $n$ is such a large number  that the output of any single firm has a negligible effect on aggregate output.

That justifies firms in regarding their forecasts of aggregate output as being unaffected by their own output decisions.

#### The Firm's Beliefs

We suppose the firm believes that market-wide output $Y_t$ follows the law of motion

```{math}
:label: ree_hlom

Y_{t+1} =  H(Y_t)
```

where $Y_0$ is a known initial condition.

The *belief function* $H$ is an equilibrium object, and hence remains to be determined.

#### Optimal Behavior Given Beliefs

For now let's fix a particular belief $H$ in {eq}`ree_hlom` and investigate the firm's response to it.

Let $v$ be the optimal value function for the firm's problem given $H$.

The value function satisfies the Bellman equation

```{math}
:label: comp4

v(y,Y) = \max_{y'} \left\{ a_0 y - a_1 y Y - \frac{ \gamma (y' - y)^2}{2}   + \beta v(y', H(Y))\right\}
```

Let's denote the firm's optimal policy function by $h$, so that

```{math}
:label: comp9

y_{t+1} = h(y_t, Y_t)
```

where

```{math}
:label: ree_opbe

h(y, Y) := \mathop{\mathrm{arg\,max}}_{y'}
\left\{ a_0 y - a_1 y Y - \frac{ \gamma (y' - y)^2}{2}   + \beta v(y', H(Y))\right\}
```

Evidently $v$ and $h$ both depend on $H$.

#### First-Order Characterization of $h$

In what follows it will be helpful to have a second characterization of $h$, based on first order conditions.

The first-order necessary condition for choosing $y'$ is

```{math}
:label: comp5

-\gamma (y' - y) + \beta v_y(y',H(Y)) =0
```

An important useful envelope result of Benveniste-Scheinkman  {cite}`BenvenisteScheinkman1979` implies that to
differentiate $v$ with respect to $y$ we can naively differentiate
the right side of {eq}`comp4`, giving

$$
v_y(y,Y) = a_0 - a_1 Y + \gamma (y' - y)
$$

Substituting this equation into {eq}`comp5` gives the *Euler equation*

```{math}
:label: ree_comp7

-\gamma (y_{t+1} - y_t) + \beta [a_0 - a_1 Y_{t+1} + \gamma (y_{t+2} - y_{t+1} )] =0
```

The firm optimally sets  an output path that satisfies {eq}`ree_comp7`, taking {eq}`ree_hlom` as given, and  subject to

* the initial conditions for $(y_0, Y_0)$
* the terminal condition $\lim_{t \rightarrow \infty } \beta^t y_t v_y(y_{t}, Y_t) = 0$

This last condition is called the *transversality condition*, and acts as a first-order necessary condition "at infinity".

The firm's decision rule solves the difference equation {eq}`ree_comp7` subject to the given initial condition $y_0$ and the transversality condition.

Note that solving the Bellman equation {eq}`comp4` for $v$ and then $h$ in {eq}`ree_opbe` yields
a decision rule that automatically imposes both the Euler equation {eq}`ree_comp7` and the transversality condition.

#### The Actual Law of Motion for $\{Y_t\}$

As we've seen, a given belief translates into a particular decision rule $h$.

Recalling that $Y_t = ny_t$, the *actual law of motion* for market-wide output is then

```{math}
:label: ree_comp9a

Y_{t+1} = n h(Y_t/n, Y_t)
```

Thus, when firms believe that the law of motion for market-wide output is {eq}`ree_hlom`, their optimizing behavior makes the actual law of motion be {eq}`ree_comp9a`.

(ree_def)=
### Definition of Rational Expectations Equilibrium

A *rational expectations equilibrium* or *recursive competitive equilibrium*  of the model with adjustment costs is a decision rule $h$ and an aggregate law of motion $H$ such that

1. Given belief $H$, the map $h$ is the firm's optimal policy function.
1. The law of motion $H$ satisfies $H(Y)= nh(Y/n,Y)$ for all.
   $Y$

Thus, a rational expectations equilibrium equates the perceived and actual laws of motion {eq}`ree_hlom` and {eq}`ree_comp9a`.

#### Fixed point characterization

As we've seen, the firm's optimum problem induces a mapping $\Phi$ from a perceived law of motion $H$ for market-wide output to an actual law of motion $\Phi(H)$.

The mapping $\Phi$ is the composition of two operations, taking a perceived law of motion into a decision rule via {eq}`comp4`--{eq}`ree_opbe`, and a decision rule into an actual law via {eq}`ree_comp9a`.

The $H$ component of a rational expectations equilibrium is a fixed point of $\Phi$.

## Computation of an Equilibrium

```{index} single: Rational Expectations Equilibrium; Computation
```

Now let's consider the problem of computing the rational expectations
equilibrium.

### Misbehavior of $\Phi$

Readers accustomed to dynamic programming arguments might try to address this problem by choosing some guess $H_0$ for the aggregate law of motion and then iterating with $\Phi$.

Unfortunately, the mapping $\Phi$ is not a contraction.

In particular, there is no guarantee that direct iterations on $\Phi$ converge [^fn_im].

Fortunately, there is another method that works here.

The method exploits a general connection between equilibrium and Pareto optimality expressed in
the fundamental theorems of welfare economics (see, e.g, {cite}`MCWG1995`).

Lucas and Prescott {cite}`LucasPrescott1971` used this method to construct a rational expectations equilibrium.

The details follow.

(ree_pp)=
### A Planning Problem Approach

```{index} single: Rational Expectations Equilibrium; Planning Problem Approach
```

Our plan of attack is to match the Euler equations of the market problem with those for a  single-agent choice problem.

As we'll see, this planning problem can be solved by LQ control ({doc}`linear regulator <../dynamic_programming/lqcontrol>`).

The optimal quantities from the planning problem are rational expectations equilibrium quantities.

The rational expectations equilibrium price can be obtained as a shadow price in the planning problem.

For convenience, in this section we set $n=1$.

We first compute a sum of  consumer and producer surplus at time $t$

```{math}
:label: comp10

s(Y_t, Y_{t+1})
:= \int_0^{Y_t} (a_0 - a_1 x) \, dx - \frac{ \gamma (Y_{t+1} - Y_t)^2}{2}
```

The first term is the area under the demand curve, while the second measures the social costs of changing output.

The *planning problem* is to choose a production plan $\{Y_t\}$ to maximize

$$
\sum_{t=0}^\infty \beta^t s(Y_t, Y_{t+1})
$$

subject to an initial condition for $Y_0$.

### Solution of the Planning Problem

Evaluating the integral in {eq}`comp10` yields the quadratic form $a_0
Y_t - a_1 Y_t^2 / 2$.

As a result, the Bellman equation for the planning problem is

```{math}
:label: comp12

V(Y) = \max_{Y'}
\left\{a_0  Y - {a_1 \over 2} Y^2 - \frac{ \gamma (Y' - Y)^2}{2} + \beta V(Y') \right\}
```

The associated first order condition is

```{math}
:label: comp14

-\gamma (Y' - Y) + \beta V'(Y') = 0
```

Applying the same Benveniste-Scheinkman formula gives

$$
V'(Y) = a_0 - a_1 Y + \gamma (Y' - Y)
$$

Substituting this into equation {eq}`comp14` and rearranging leads to the Euler
equation

```{math}
:label: comp16

\beta a_0 + \gamma Y_t - [\beta a_1 + \gamma (1+ \beta)]Y_{t+1} + \gamma \beta Y_{t+2} =0
```

### The Key Insight

Return to equation {eq}`ree_comp7` and set $y_t = Y_t$ for all $t$.

(Recall that for this section we've set $n=1$ to simplify the
calculations)

A small amount of algebra will convince you that when $y_t=Y_t$, equations {eq}`comp16` and {eq}`ree_comp7` are identical.

Thus, the Euler equation for the planning problem matches the second-order difference equation
that we derived by

1. finding the Euler equation of the representative firm and
1. substituting into it the expression $Y_t = n y_t$ that "makes the representative firm be representative"

If it is appropriate to apply the same terminal conditions for these two difference equations, which it is, then we have verified that a solution of the planning problem is also a rational expectations equilibrium quantity sequence

It follows that for this example we can compute equilibrium quantities by forming the optimal linear regulator problem corresponding to the Bellman equation {eq}`comp12`.

The optimal policy function for the planning problem is the aggregate law of motion
$H$ that the representative firm faces within a rational expectations equilibrium.

#### Structure of the Law of Motion

As you are asked to show in the exercises, the fact that the planner's
problem is an LQ problem implies an optimal policy --- and hence aggregate law
of motion --- taking the form

```{math}
:label: ree_hlom2

Y_{t+1}
= \kappa_0 + \kappa_1 Y_t
```

for some parameter pair $\kappa_0, \kappa_1$.

Now that we know the aggregate law of motion is linear, we can see from the
firm's Bellman equation {eq}`comp4` that the firm's problem can also be framed as
an LQ problem.

As you're asked to show in the exercises, the LQ formulation of the firm's
problem implies a law of motion that looks as follows

```{math}
:label: ree_ex5

y_{t+1} = h_0 + h_1 y_t + h_2 Y_t
```

Hence a rational expectations equilibrium will be defined by the parameters
$(\kappa_0, \kappa_1, h_0, h_1, h_2)$ in {eq}`ree_hlom2`--{eq}`ree_ex5`.

## Exercises

(ree_ex1)=
### Exercise 1

Consider the firm problem {ref}`described above <ree_fp>`.

Let the firm's belief function $H$ be as given in {eq}`ree_hlom2`.

Formulate the firm's problem as a discounted optimal linear regulator problem, being careful to describe all of the objects needed.

Use the type `LQ` from the [QuantEcon.jl](http://quantecon.org/quantecon-jl) package to solve the firm's problem for the following parameter values:

$$
a_0= 100, a_1= 0.05, \beta = 0.95, \gamma=10, \kappa_0 = 95.5, \kappa_1 = 0.95
$$

Express the solution of the firm's problem in the form {eq}`ree_ex5` and give the values for each $h_j$.

If there were $n$ identical competitive firms all behaving according to {eq}`ree_ex5`, what would {eq}`ree_ex5`  imply for the *actual* law of motion {eq}`ree_hlom` for market supply.

(ree_ex2)=
### Exercise 2

Consider the following $\kappa_0, \kappa_1$ pairs as candidates for the
aggregate law of motion component of a rational expectations equilibrium (see
{eq}`ree_hlom2`).

Extending the program that you wrote for exercise 1, determine which if any
satisfy {ref}`the definition <ree_def>` of a rational expectations equilibrium

* (94.0886298678, 0.923409232937)
* (93.2119845412, 0.984323478873)
* (95.0818452486, 0.952459076301)

Describe an iterative algorithm that uses the program that you wrote for exercise 1 to compute a rational expectations equilibrium.

(You are not being asked actually to use the algorithm you are suggesting)

(ree_ex3)=
### Exercise 3

Recall the planner's problem {ref}`described above <ree_pp>`

1. Formulate the planner's problem as an LQ problem.
1. Solve it using the same parameter values in exercise 1
    * $a_0= 100, a_1= 0.05, \beta = 0.95, \gamma=10$
1. Represent the solution in the form $Y_{t+1} = \kappa_0 + \kappa_1 Y_t$.
1. Compare your answer with the results from exercise 2.

(ree_ex4)=
### Exercise 4

A monopolist faces the industry demand curve {eq}`ree_comp3d`  and chooses $\{Y_t\}$ to maximize $\sum_{t=0}^{\infty} \beta^t r_t$ where

$$
r_t = p_t Y_t - \frac{\gamma (Y_{t+1} - Y_t)^2 }{2}
$$

Formulate this problem as an LQ problem.

Compute the optimal policy using the same parameters as the previous exercise.

In particular, solve for the parameters in

$$
Y_{t+1} = m_0 + m_1 Y_t
$$

Compare your results with the previous exercise.  Comment.

## Solutions

### Exercise 1

```{code-cell} julia
---
tags: [remove-cell]
---
using Test
```

```{code-cell} julia
using QuantEcon, Printf, LinearAlgebra
```

To map a problem into a [discounted optimal linear control
problem](http://quant-econ.net/jl/lqcontrol.html), we need to define

- state vector $x_t$ and control vector $u_t$
- matrices $A, B, Q, R$ that define preferences and the law of
  motion for the state

For the state and control vectors we choose

$$
x_t = \begin{bmatrix} y_t \\ Y_t \\ 1 \end{bmatrix},
\qquad
u_t = y_{t+1} - y_{t}
$$

For $, B, Q, R$ we set

$$
A =
\begin{bmatrix}
    1 & 0 & 0 \\
    0 & \kappa_1 & \kappa_0 \\
    0 & 0 & 1
\end{bmatrix},
\quad
B = \begin{bmatrix} 1 \\ 0 \\ 0 \end{bmatrix} ,
\quad
R =
\begin{bmatrix}
    0 & a_1/2 & -a_0/2 \\
    a_1/2 & 0 & 0 \\
    -a_0/2 & 0 & 0
\end{bmatrix},
\quad
Q = \gamma / 2
$$

By multiplying out you can confirm that

- $x_t' R x_t + u_t' Q u_t = - r_t$
- $x_{t+1} = A x_t + B u_t$

We'll use the module `lqcontrol.jl` to solve the firm's problem at the
stated parameter values.

This will return an LQ policy $F$ with the interpretation
$u_t = - F x_t$, or

$$
y_{t+1} - y_t = - F_0 y_t - F_1 Y_t - F_2
$$

Matching parameters with $y_{t+1} = h_0 + h_1 y_t + h_2 Y_t$ leads
to

$$
h_0 = -F_2, \quad h_1 = 1 - F_0, \quad h_2 = -F_1
$$

Here's our solution

```{code-cell} julia
# model parameters
a0 = 100
a1 = 0.05
β = 0.95
γ = 10.0

# beliefs
κ0 = 95.5
κ1 = 0.95

# formulate the LQ problem
A = [1  0  0
     0 κ1 κ0
     0  0  1]

B = [1.0, 0.0, 0.0]

R = [    0   a1/2 -a0/2
      a1/2      0     0
     -a0/2      0     0]

Q = 0.5 * γ

# solve for the optimal policy
lq = QuantEcon.LQ(Q, R, A, B; bet = β)
P, F, d = stationary_values(lq)

hh = h0, h1, h2 = -F[3], 1 - F[1], -F[2]

@printf("F = [%.3f, %.3f, %.3f]\n", F[1], F[2], F[3])
@printf("(h0, h1, h2) = [%.3f, %.3f, %.3f]\n", h0, h1, h2)
```

```{code-cell} julia
---
tags: [remove-cell]
---
@testset begin
  @test F[1] ≈ -2.3625545964023384e-15
  @test F[2] ≈ 0.04628205128205243
  @test h0 ≈ 96.94871794872053
end
```

The implication is that

$$
y_{t+1} = 96.949 + y_t - 0.046 \, Y_t
$$

For the case $n > 1$, recall that $Y_t = n y_t$, which,
combined with the previous equation, yields

$$
Y_{t+1}
= n \left( 96.949 + y_t - 0.046 \, Y_t \right)
= n 96.949 + (1 - n 0.046) Y_t
$$

### Exercise 2

To determine whether a $\kappa_0, \kappa_1$ pair forms the
aggregate law of motion component of a rational expectations
equilibrium, we can proceed as follows:

- Determine the corresponding firm law of motion
  $y_{t+1} = h_0 + h_1 y_t + h_2 Y_t$.
- Test whether the associated aggregate law
  :$Y_{t+1} = n h(Y_t/n, Y_t)$ evaluates to
  $Y_{t+1} = \kappa_0 + \kappa_1 Y_t$.

In the second step we can use $Y_t = n y_t = y_t$, so that
$Y_{t+1} = n h(Y_t/n, Y_t)$ becomes

$$
Y_{t+1} = h(Y_t, Y_t) = h_0 + (h_1 + h_2) Y_t
$$

Hence to test the second step we can test $\kappa_0 = h_0$ and
$\kappa_1 = h_1 + h_2$.

The following code implements this test

```{code-cell} julia
candidates = ([94.0886298678, 0.923409232937],
              [93.2119845412, 0.984323478873],
              [95.0818452486, 0.952459076301])

for (k0, k1) in candidates
    A = [1  0  0
         0 k1 k0
         0  0  1]
    lq = QuantEcon.LQ(Q, R, A, B; bet=β)
    P, F, d = stationary_values(lq)
    hh = h0, h1, h2 = -F[3], 1 - F[1], -F[2]

    if isapprox(k0, h0; atol = 1e-4) && isapprox(k1, h1 + h2; atol = 1e-4)
        @printf("Equilibrium pair = (%.6f, %.6f)\n", k0, k1)
        @printf("(h0, h1, h2) = [%.6f, %.6f, %.6f]\n", h0, h1, h2)
    end
end
```

```{code-cell} julia
---
tags: [remove-cell]
---
@testset begin
  @test h0 ≈ 95.081891001308
end
```

The output tells us that the answer is pair (iii), which implies
$(h_0, h_1, h_2) = (95.0819, 1.0000, -.0475)$.

(Notice we use `isapprox` to test equality of floating point numbers,
since exact equality is too strict)

Regarding the iterative algorithm, one could loop from a given
$(\kappa_0, \kappa_1)$ pair to the associated firm law and then to
a new $(\kappa_0, \kappa_1)$ pair.

This amounts to implementing the operator $\Phi$ described in the
lecture.

(There is in general no guarantee that this iterative process will
converge to a rational expectations equilibrium)

### Exercise 3

We are asked to write the planner problem as an LQ problem.

For the state and control vectors we choose

$$
x_t = \begin{bmatrix} Y_t \\ 1 \end{bmatrix},
\quad
u_t = Y_{t+1} - Y_{t}
$$

For the LQ matrices we set

$$
A = \begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix},
\quad
B = \begin{bmatrix} 1 \\ 0 \end{bmatrix},
\quad
R = \begin{bmatrix} a_1/2 & -a_0/2 \\ -a_0/2 & 0 \end{bmatrix},
\quad
Q = \gamma / 2
$$

By multiplying out you can confirm that

- $x_t' R x_t + u_t' Q u_t = - s(Y_t, Y_{t+1})$
- $x_{t+1} = A x_t + B u_t$

By obtaining the optimal policy and using $u_t = - F x_t$ or

$$
Y_{t+1} - Y_t = -F_0 Y_t - F_1
$$

we can obtain the implied aggregate law of motion via
$\kappa_0 = -F_1$ and $\kappa_1 = 1-F_0$

```{code-cell} julia
# formulate the planner's LQ problem
A = I + zeros(2, 2)
B = [1.0, 0.0]

R = [ a1 / 2.0  -a0 / 2.0
     -a0 / 2.0        0.0]

Q = γ / 2.0

# solve for the optimal policy
lq = QuantEcon.LQ(Q, R, A, B; bet=β)
P, F, d = stationary_values(lq)

# print the results
κ0, κ1 = -F[2], 1 - F[1]
println("κ0=$κ0\tκ1=$κ1")
```

```{code-cell} julia
---
tags: [remove-cell]
---
@testset begin
    @test κ0 ≈ 95.081845248600 rtol = 4
    @test κ1 ≈ 0.952459076301 rtol = 4
end
```

The output yields the same $(\kappa_0, \kappa_1)$ pair obtained as
an equilibrium from the previous exercise.

### Exercise 4

The monopolist's LQ problem is almost identical to the planner's problem
from the previous exercise, except that

$$
R = \begin{bmatrix}
    a_1 & -a_0/2 \\
    -a_0/2 & 0
\end{bmatrix}
$$

The problem can be solved as follows

```{code-cell} julia
# formulate the monopolist's LQ problem
A = I + zeros(2, 2)
B = [1.0, 0.0]

R = [       a1   -a0 / 2.0
     -a0 / 2.0         0.0]

Q = γ / 2.0

# solve for the optimal policy
lq = QuantEcon.LQ(Q, R, A, B; bet=β)
P, F, d = stationary_values(lq)

# print the results
m0, m1 = -F[2], 1 - F[1]
println("m0=$m0\tm1=$m1")
```

```{code-cell} julia
---
tags: [remove-cell]
---
@testset begin
  @test m0 ≈ 73.47294403502833
end
```

We see that the law of motion for the monopolist is approximately
$Y_{t+1} = 73.4729 + 0.9265 Y_t$.

In the rational expectations case the law of motion was approximately
$Y_{t+1} = 95.0819 + 0.9525 Y_t$.

One way to compare these two laws of motion is by their fixed points,
which give long run equilibrium output in each case.

For laws of the form $Y_{t+1} = c_0 + c_1 Y_t$, the fixed point is
$c_0 / (1 - c_1)$.

If you crunch the numbers, you will see that the monopolist adopts a
lower long run quantity than obtained by the competitive market,
implying a higher market price.

This is analogous to the elementary static-case results

[^fn_im]: A literature that studies whether models populated  with agents
who learn can converge  to rational expectations equilibria features
iterations on a modification of the mapping $\Phi$ that can be
approximated as $\gamma \Phi + (1-\gamma)I$. Here $I$ is the
identity operator and $\gamma \in (0,1)$ is a *relaxation parameter*.
See {cite}`MarcetSargent1989` and {cite}`EvansHonkapohja2001` for statements
and applications of this approach to establish conditions under which
collections of adaptive agents who use least squares learning converge to a
rational expectations equilibrium.

