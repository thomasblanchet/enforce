# enforce

A Stata package to enforce accounting identities between variables.

## Usage

Install the command in Stata by typing `ssc install enforce`.

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

The command is designed to enforce an arbitrary set of accounting identities intelligently, while performing a series of auxiliary checks and adjustments. To understand the command, first assume an identity <img src="/tex/20f5744c1176c626bd1d0d2c8841b6d7.svg?invert_in_darkmode&sanitize=true" align=middle width=64.86657704999999pt height=22.831056599999986pt/>. The simplest, naïve way of enforcing it is to multiply both b and c by the same constant, <img src="/tex/1e4c4e7504b35492d6a32200f7447ce8.svg?invert_in_darkmode&sanitize=true" align=middle width=63.953589149999985pt height=24.65753399999998pt/>. The main problem with that approach is that it only works well with positive variables. If <img src="/tex/d7390019e5f9d9dcee82a92b3e0a5375.svg?invert_in_darkmode&sanitize=true" align=middle width=38.82599489999999pt height=21.18721440000001pt/>, it will set both <img src="/tex/4bdc8d9bcfb35e1c9bfb51fc69687dfc.svg?invert_in_darkmode&sanitize=true" align=middle width=7.054796099999991pt height=22.831056599999986pt/> and <img src="/tex/3e18a4a28fdee1744e5e3f79d13b9ff6.svg?invert_in_darkmode&sanitize=true" align=middle width=7.11380504999999pt height=14.15524440000002pt/> to zero, instead of enforcing the more general condition <img src="/tex/0dc6093397852d2a4a38bb8512e8dd5e.svg?invert_in_darkmode&sanitize=true" align=middle width=48.871665149999984pt height=22.831056599999986pt/>. If <img src="/tex/1be8477d80a78ef8876e828308cd38f5.svg?invert_in_darkmode&sanitize=true" align=middle width=64.3966323pt height=22.831056599999986pt/>, then it will not work at all, and more generally if <img src="/tex/2cc5b615c6d7d9246884e590191f6895.svg?invert_in_darkmode&sanitize=true" align=middle width=68.51954669999998pt height=22.831056599999986pt/> (because <img src="/tex/1880fe0ecaa115a4450051a030db2a5d.svg?invert_in_darkmode&sanitize=true" align=middle width=37.19163689999999pt height=22.831056599999986pt/> or <img src="/tex/99e7c2e44652ae6d8b48eec21bd200b5.svg?invert_in_darkmode&sanitize=true" align=middle width=37.25064419999999pt height=21.18721440000001pt/>), it can easily lead to absurd adjustments.

One way to fix that first issue is to calculate the discrepancy <img src="/tex/d233813a2931377ad10f2ae9352f5d59.svg?invert_in_darkmode&sanitize=true" align=middle width=92.6232714pt height=22.831056599999986pt/>, and redistribute it proportionally to the absolute value of <img src="/tex/4bdc8d9bcfb35e1c9bfb51fc69687dfc.svg?invert_in_darkmode&sanitize=true" align=middle width=7.054796099999991pt height=22.831056599999986pt/> and <img src="/tex/3e18a4a28fdee1744e5e3f79d13b9ff6.svg?invert_in_darkmode&sanitize=true" align=middle width=7.11380504999999pt height=14.15524440000002pt/>. That is, we redefine <img src="/tex/4bdc8d9bcfb35e1c9bfb51fc69687dfc.svg?invert_in_darkmode&sanitize=true" align=middle width=7.054796099999991pt height=22.831056599999986pt/> as <img src="/tex/f69f80b84236ca3cfefaccdfa7ec5c87.svg?invert_in_darkmode&sanitize=true" align=middle width=44.40057434999999pt height=22.831056599999986pt/> and <img src="/tex/3e18a4a28fdee1744e5e3f79d13b9ff6.svg?invert_in_darkmode&sanitize=true" align=middle width=7.11380504999999pt height=14.15524440000002pt/> as <img src="/tex/68c90711d9f3014f1e01c720ad72ca02.svg?invert_in_darkmode&sanitize=true" align=middle width=85.55541554999999pt height=24.65753399999998pt/>, where <img src="/tex/26fd3086c5caedfa45c6ee771bd60412.svg?invert_in_darkmode&sanitize=true" align=middle width=121.223289pt height=24.65753399999998pt/>. If <img src="/tex/4bdc8d9bcfb35e1c9bfb51fc69687dfc.svg?invert_in_darkmode&sanitize=true" align=middle width=7.054796099999991pt height=22.831056599999986pt/> and <img src="/tex/3e18a4a28fdee1744e5e3f79d13b9ff6.svg?invert_in_darkmode&sanitize=true" align=middle width=7.11380504999999pt height=14.15524440000002pt/> are both positive, this is equivalent to the naïve approach, but otherwise it behaves much more reasonably.

But there are still other problems. First, if forces us to define a reference variable (in this case <img src="/tex/44bc9d542a92714cac84e01cbbb7fd61.svg?invert_in_darkmode&sanitize=true" align=middle width=8.68915409999999pt height=14.15524440000002pt/>) that will remain unchanged, which may or may not be desirable. Second, it is not clear how to generalize this adjustment to more complex settings. In practice, we must simultaneously satisfy dozens of accounting identities, with variables presents in several of them, so that adjustments must be performed across several dimensions. We formalize that problem as follows. Assume that we have a vector <img src="/tex/e3faeb81e09238ac1b274854c40e338a.svg?invert_in_darkmode&sanitize=true" align=middle width=125.04348119999997pt height=24.7161288pt/> of variables to be adjusted, and we seek an adjusted vector <img src="/tex/b9be78399537676ecc52f66764f4eb5a.svg?invert_in_darkmode&sanitize=true" align=middle width=120.66008129999999pt height=24.7161288pt/> that must satisfy a set of accounting identities. We will minimize:

<p align="center"><img src="/tex/86874363127dac408d99820ddb109f1b.svg?invert_in_darkmode&sanitize=true" align=middle width=97.10765294999999pt height=44.89738935pt/></p>

subject to the accounting identities. The convex cost function <img src="/tex/a5e17e19978af11d23f1ca5d8b791c17.svg?invert_in_darkmode&sanitize=true" align=middle width=67.82918339999998pt height=26.76175259999998pt/> at the numerator ensure that the differences between raw and adjusted variables are as low as possible, and that they are spread equitably across all the variables. The <img src="/tex/4cff995c577a5a941bf5c42a37d86847.svg?invert_in_darkmode&sanitize=true" align=middle width=24.000233399999992pt height=24.65753399999998pt/> at the denominator ensures that adjustments are proportional to the initial value of the variable. In simple cases, this is equivalent to the procedure explained above.

Assume that the set of accounting identities can be written as a linear system <img src="/tex/592ea153158af3875f115636da383551.svg?invert_in_darkmode&sanitize=true" align=middle width=62.44850534999999pt height=22.465723500000017pt/>. The problem can be written in matrix form as:

<p align="center"><img src="/tex/3fb0b4d0fe28bc0760e4d522074d99e6.svg?invert_in_darkmode&sanitize=true" align=middle width=409.8254556pt height=32.990165999999995pt/></p>

where <img src="/tex/0186ed4f08cc44351611736c1dd36c26.svg?invert_in_darkmode&sanitize=true" align=middle width=200.84737034999998pt height=24.65753399999998pt/> and <img src="/tex/d58c0ddc4c92448c2e4bc108fcf00787.svg?invert_in_darkmode&sanitize=true" align=middle width=205.43410139999997pt height=24.7161288pt/>. This is a standard, quadratic programming problem with equality constraints and a positive definite matrix <img src="/tex/1afcdb0f704394b16fe85fb40c45ca7a.svg?invert_in_darkmode&sanitize=true" align=middle width=12.99542474999999pt height=22.465723500000017pt/>. Thus, the result is the solution of the linear system (see [Wikipedia for details](https://en.wikipedia.org/wiki/Quadratic_programming#Equality_constraints)):

<p align="center"><img src="/tex/2074293b6cf6b6f276445bf6b22cd792.svg?invert_in_darkmode&sanitize=true" align=middle width=163.7054826pt height=39.452455349999994pt/></p>

where <img src="/tex/fd8be73b54f5436a5cd2e73ba9b6bfa9.svg?invert_in_darkmode&sanitize=true" align=middle width=9.58908224999999pt height=22.831056599999986pt/> is a set of Lagrange multipliers determined alongside <img src="/tex/cbfb1b2a33b28eab8a3e59464768e810.svg?invert_in_darkmode&sanitize=true" align=middle width=14.908688849999992pt height=22.465723500000017pt/>. (Variables that are set fixed or equal to zero are removed from the vector <img src="/tex/cbfb1b2a33b28eab8a3e59464768e810.svg?invert_in_darkmode&sanitize=true" align=middle width=14.908688849999992pt height=22.465723500000017pt/> and included in <img src="/tex/61e84f854bc6258d4108d08d4c4a0852.svg?invert_in_darkmode&sanitize=true" align=middle width=13.29340979999999pt height=22.465723500000017pt/>.) This problem is solved via QR decomposition, so it will return an optimal solution in the least-squares sense if the system of identities is technically infeasible. The command checks wether this is the case, and stops by default if the system of equalities is found to be infeasible for some observations. You can override this behavior with the force option. Note that variables initially equal to zero are implicitly fixed, so sometimes they can be the reason behind infeasibility.

This is the main task performed by the command enforce, although it also has several additional functionalities.

First, it takes advantage of accounting identities to fill in any missing value that can technically be calculated from nonmissing variables, even though it was initially absent from the raw data.

Second, it pays specific attention to the way constraints are defined to overcome missing value problems. Indeed, assume for example that we have the constraints <img src="/tex/20f5744c1176c626bd1d0d2c8841b6d7.svg?invert_in_darkmode&sanitize=true" align=middle width=64.86657704999999pt height=22.831056599999986pt/>, <img src="/tex/b1a443b698595a0f2005056c85ce4237.svg?invert_in_darkmode&sanitize=true" align=middle width=70.66946204999998pt height=19.1781018pt/>, <img src="/tex/97017544417da1329b8a74259d214a41.svg?invert_in_darkmode&sanitize=true" align=middle width=67.87837649999999pt height=22.831056599999986pt/>, <img src="/tex/b6161ff1ddfc66e45ba9866a02fd307f.svg?invert_in_darkmode&sanitize=true" align=middle width=66.91410989999999pt height=19.1781018pt/>, and <img src="/tex/77095855395b178659bbe0a6bc3fff8d.svg?invert_in_darkmode&sanitize=true" align=middle width=68.42063744999999pt height=19.1781018pt/>. Clearly, this implies that <img src="/tex/849b99ea48f4b1ced7887bbf204453eb.svg?invert_in_darkmode&sanitize=true" align=middle width=72.17473229999999pt height=22.831056599999986pt/>. However, if the data were to only contain nonmissing values for <img src="/tex/c745b9b57c145ec5577b82542b2df546.svg?invert_in_darkmode&sanitize=true" align=middle width=10.57650494999999pt height=14.15524440000002pt/>, <img src="/tex/8217ed3c32a785f0b5aad4055f432ad8.svg?invert_in_darkmode&sanitize=true" align=middle width=10.16555099999999pt height=22.831056599999986pt/> and <img src="/tex/11c596de17c342edeed29f489aa4b274.svg?invert_in_darkmode&sanitize=true" align=middle width=9.423880949999988pt height=14.15524440000002pt/>, a naïve treatment of missing values would lead us to dismiss all the constraints as irrelevant to our data. The command is designed to be aware of the fact that the system of identities implicitly imposes <img src="/tex/849b99ea48f4b1ced7887bbf204453eb.svg?invert_in_darkmode&sanitize=true" align=middle width=72.17473229999999pt height=22.831056599999986pt/>.

Third, the command analyzes the system of identities to find any implausibility (e.g. variable always equal to zero) or incompatibility with the data (in case of fixed variables).

Fourth, it provides extensive reporting on the magnitude of the discrepancies and the adjustments.
