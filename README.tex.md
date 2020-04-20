# enforce

A Stata package to enforce accounting identities between variables.

## Usage

Install the command in Stata by typing `ssc install Stata`.

Then use it as:
```stata
enforce (identity) [(identity) ...], {replace|suffix(string)|prefix(string)} [options]
```
where `identity` is:
```stata
[+|-] {varname|0} [{+|-}{varname|0} ...] = [+|-] {varname|0} [{+|-}{varname|0} ...]
```
You can type `help enforce` for details.

## Example

```stata
set obs 100
drawnorm a1 b1 c1 a2 b2 c2 x y z
foreach v in a1 b1 c1 x y z {
    // Create missing values at random
    replace `v' = . if (uniform() < 0.1)
}
enforce (a1 = b1 + c1) (a2 = a1 + x) (b2 = b1 + y) (c2 = c1 + z) (x = y + z), fixed(a2) replace
```

## Details

The command is designed to enforce an arbitrary set of accounting identities intelligently, while performing a series of auxiliary checks and adjustments. To understand the command, first assume an identity $a=b+c$. The simplest, naïve way of enforcing it is to multiply both b and c by the same constant, $a/(b+c)$. The main problem with that approach is that it only works well with positive variables. If $a=0$, it will set both $b$ and $c$ to zero, instead of enforcing the more general condition $b=-c$. If $b+c=0$, then it will not work at all, and more generally if $b+c \ll a$ (because $b<0$ or $c<0$), it can easily lead to absurd adjustments.

One way to fix that first issue is to calculate the discrepancy $\varpepislon=a-b-c$, and redistribute it proportionally to the absolute value of $b$ and $c$. That is, we redefine $b$ as $b+\lambda \varepsilon$ and $c$ as $c+(1-\lambda)\varepsilon$, where $\lambda=|b|/(|b|+|c|)$. If $b$ and $c$ are both positive, this is equivalent to the naïve approach, but otherwise it behaves much more reasonably.

But there are still other problems. First, if forces us to define a reference variable (in this case $a$) that will remain unchanged, which may or may not be desirable. Second, it is not clear how to generalize this adjustment to more complex settings. In practice, we must simultaneously satisfy dozens of accounting identities, with variables presents in several of them, so that adjustments must be performed across several dimensions. We formalize that problem as follows. Assume that we have a vector $X=(x_1,\dots,x_n)'$ of variables to be adjusted, and we seek an adjusted vector $Y=(y_1,\dots,y_n)'$ that must satisfy a set of accounting identities. We will minimize:

$$
\sum_{i=1}^n \frac{(y_i - x_i)^2}{|x_i|}
$$

subject to the accounting identities. The convex cost function $(y_i-x_i)^2$ at the numerator ensure that the differences between raw and adjusted variables are as low as possible, and that they are spread equitably across all the variables. The $|x_i|$ at the denominator ensures that adjustments are proportional to the initial value of the variable. In simple cases, this is equivalent to the procedure explained above.

Assume that the set of accounting identities can be written as a linear system $AX=B$. The problem can be written in matrix form as:

$$
\text{minimize} \qquad \frac{1}{2}X' Q X + C' X \qquad \text{subject to} \qquad AX=B
$$

where $Q=\mathrm{diag}(1/|x_1|,\dots,1/|x_n|)$ and $C=(\mathrm{sign}(x_1),\dots,\text{sign}(x_n))'$. This is a standard, quadratic programming problem with equality constraints and a positive definite matrix $Q$. Thus, the result is the solution of the linear system (see [Wikipedia for details](https://en.wikipedia.org/wiki/Quadratic_programming#Equality_constraints)):

$$
\begin{bmatrix}
Q & A' \\
A & 0
\end{bmatrix} \begin{bmatrix}
X \\ \lambda
\end{bmatrix} = \begin{bmatrix}
-C \\
B
\end{bmatrix}
$$

where $\lambda$ is a set of Lagrange multipliers determined alongside $X$. (Variables that are set fixed or equal to zero are removed from the vector $X$ and included in $B$.) This problem is solved via QR decomposition, so it will return an optimal solution in the least-squares sense if the system of identities is technically infeasible. The command checks wether this is the case, and stops by default is the system of equalities is found to be infeasible for some observations. You can override this behavior with the force option. Note that variables initially equal to zero are implicitly fixed, so sometimes they can be the reason behind infeasibility.

This is the main task performed by the command enforce, although it also has several additional functionalities.

First, it takes advantage of accounting identities to fill in any missing value that can technically be calculated from nonmissing variables, even though it was initially absent from the raw data.

Second, it pays specific attention to the way constraints are defined to overcome missing value problems. Indeed, assume for example that we have the constraints $a=b+c$, $\alpha=a+x$, $\beta=b+y$, $\gamma=c+z$, and $x=y+z$. Clearly, this implies that $\alpha=\beta+\gamma$. However, if the data were to only contain nonmissing values for $\alpha$, $\beta$ and $\gamma$, a naïve treatment of missing values would lead us to dismiss all the constraints as irrelevant to our data. The command is designed to be aware of the fact that the system of identities implicitly imposes $\alpha=\beta+\gamma$.

Third, the command analyzes the system of identities to find any implausibility (e.g. variable always equal to zero) or incompatibility with the data (in case of fixed variables).

Fourth, it provides extensive reporting on the magnitude of the discrepancies and the adjustments.
