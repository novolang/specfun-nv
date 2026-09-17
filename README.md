# specfun-nv

The **special functions** are the named functions of mathematical
analysis that are not elementary: the gamma function, the error
function, the incomplete integrals built on them, and the Bessel
functions. This package brings them to novo-lang, with
[scipy.special](https://docs.scipy.org/doc/scipy/reference/special.html)
and the
[Digital Library of Mathematical Functions](https://dlmf.nist.gov/) as
the reference.

**Status: NOT IMPLEMENTED — interface only.** Every function is
declared with its full signature, but every body is a `todo()` that
panics when called. The package is published so its design can be
reviewed and depended on before it is implemented. Version 0.1.0 will
be the first working release.

## What it is

The **gamma function** is the extension of the factorial to the whole
real line. For a positive integer `n`, `gamma(n)` is `(n-1)!`, and
between the integers it is the smooth curve that satisfies
`gamma(x+1) = x * gamma(x)`. Every continuous probability
distribution's normalising constant is written with it. Its logarithm
is the function a likelihood is actually computed with, because
`gamma(172.0)` is larger than any 64-bit floating-point number. Its
derivative is the **digamma** function, written `psi`, and a gradient
in a gamma, beta or Dirichlet model is made of it. The **beta
function** is `gamma(a) * gamma(b) / gamma(a + b)`.

The **error function**, written `erf`, is the cumulative probability of
the normal distribution rewritten so that it runs from -1 to 1 and is
zero at zero. Its **complement**, `erfc`, is one minus it, and it is a
function of its own rather than that subtraction: at an argument of 6,
`erf(x)` is 0.9999999999999999779…, so `1 - erf(x)` in binary floating
point has about two correct digits out of sixteen. A tail probability
IS that difference, so a p-value computed as `1 - erf` is wrong in
every digit that matters.

Stop the gamma function's integral partway and divide by the whole and
you have the **regularised incomplete gamma**, written `P(a, x)` and
`Q(a, x)` for the lower and upper pieces. They run from 0 to 1 and sum
to 1. Doing the same to the beta function gives the **regularised
incomplete beta**, `I_x(a, b)`. These two functions are not
conveniences: `P(a, x)` *is* the cumulative distribution function of
the gamma, chi-squared, Erlang and Poisson distributions, and
`I_x(a, b)` *is* the cumulative distribution function of the beta,
Student's t, F and binomial distributions. Every p-value in those
families is one call to one of them with its arguments rearranged, and
each has an **inverse** in its argument, which is what a quantile and a
critical value are.

The **Bessel functions** are the solutions of Bessel's differential
equation, and they are to problems with circular or cylindrical
symmetry what the sine and cosine are to problems on a line. `J` is the
function of the first kind, which oscillates and is finite at zero. `Y`
is the function of the second kind, which oscillates and is unbounded
at zero. `I` and `K` are the **modified** functions of the first and
second kind, which do not oscillate: `I` grows like `exp(x)` and `K`
decays like `exp(-x)`. They are the diffraction pattern of a circular
aperture, the modes of a drumhead, the von Mises distribution's
normalising constant, and the Matérn covariance a Gaussian process is
fitted with.

## Install

```
novo pkg add specfun-nv
```

## Example

```novo
use specincomplete

fn main() [io]
    // A chi-squared statistic and its degrees of freedom.  The
    // p-value is the chance of seeing a statistic this large or
    // larger when the null hypothesis holds.
    let statistic = 7.815
    let degrees_of_freedom = 3.0

    // That chance is the UPPER regularised incomplete gamma at half
    // the statistic, with half the degrees of freedom as its shape.
    // `gamma_q` computes it directly rather than as `1 - gamma_p`,
    // which is what keeps the digits in a small tail.
    match specincomplete.gamma_q(degrees_of_freedom / 2.0, statistic / 2.0)
        Ok(p)  => println("p = ${p}")           // p = 0.05000...
        Err(e) => println(e.message())
```

Build and test with `novo pkg build` and `novo test`. Today `novo test`
fails on purpose: every test reaches a
`not implemented: specfun-nv.<module>.<fn>` panic. The tests are the
specification the implementation will have to satisfy.

## What the package contains

| Module | Contents |
| --- | --- |
| `specfault` | Every way a function can refuse, with the domain rule it broke and the argument that broke it. |
| `specgamma` | The gamma function, its logarithm and that logarithm's sign, the digamma function, the beta function and two combinatorial helpers. |
| `specerf` | The error function, its complement, the scaled complement, the two inverses and Dawson's integral. |
| `specincomplete` | The regularised incomplete gamma and beta, their complements, their inverses and their logarithms. |
| `specbessel` | The Bessel functions `J`, `Y`, `I` and `K` at integer and at real order, the scaled modified forms, the zeros of `J` and the spherical forms. |

## How to choose an entry point

**Every function comes in an integer-order and a real-order form in
`specbessel`, and they are different functions.** `bessel_j_int(2,
x)` takes an `Int`; `bessel_j(2.5, x)` takes a `Float`. For a whole
number order the three-term recurrence
`J(n-1, x) + J(n+1, x) = (2n/x) * J(n, x)` is exact and cheap, and for
a real order the value comes from a power series, an asymptotic
expansion or a continued fraction depending on the argument. A caller
who knows the order is a count — a drumhead mode, a diffraction ring —
should call the integer form and should not have to write `4.0` where
they mean `4`.

**Use the complement rather than the subtraction.** `erfc` rather than
`1 - erf`, `gamma_q` rather than `1 - gamma_p`, `beta_inc_c` rather
than `1 - beta_inc`. Each complement is computed directly and keeps
its relative accuracy where the subtraction has none.

**Use the logarithmic form when the value will be multiplied by
others.** `log_gamma`, `log_beta`, `log_gamma_p` and `log_beta_inc`
stay finite over ranges where the value itself overflows or underflows
to zero. A likelihood is a sum of logarithms for this reason.

**Use the scaled form when the exponential will cancel.**
`bessel_i_scaled` is `exp(-x) * I(n, x)` and `bessel_k_scaled` is
`exp(x) * K(n, x)`. A von Mises density and a Matérn covariance are
both quotients in which the exponential cancels, and computing them
from the unscaled functions loses the digits before the cancellation
can recover them.

## The rules a user needs

1. **Every function returns a `Result` and none of them returns a
   NaN.** Not one of these functions is defined on the whole real
   line, and a NaN carries no reason, propagates silently through the
   arithmetic that follows it, and leaves the reader several
   operations away from the mistake. A `SpecFault` names the domain
   rule that was broken and carries the argument that broke it.
2. **The gamma function's poles are zero and every negative integer.**
   The limits from the two sides have opposite signs, so there is no
   value to return. `gamma`, `log_gamma`, `log_gamma_sign` and
   `digamma` all answer `SpecGammaPole` there.
3. **`log_gamma` is the logarithm of the ABSOLUTE value, and
   `log_gamma_sign` is the other half.** The gamma function is
   negative on half of the intervals between the negative integers, so
   its logarithm is complex there and one `Float` cannot carry both
   parts. Multiply the two back together to reconstruct a signed
   quantity.
4. **An overflow is not a fault.** `gamma(200.0)` is a positive
   infinity, and `erfc(30.0)` and `bessel_k_int(0, 800.0)` underflow to
   zero. All three are representable results of a well-posed question,
   and `log_gamma`, `erfcx` and `bessel_k_scaled` are the functions
   that avoid them.
5. **`Y` and `K` are undefined at zero and below it**, so their
   argument must be strictly positive. `J` and `I` are finite
   everywhere for an integer order; for a non-integer order they need a
   non-negative argument, since below zero they are complex.
6. **A negative order is exact for an integer and refused for a real
   number.** `J(-n, x)` is `(-1)^n * J(n, x)`, `Y(-n, x)` the same, and
   `I(-n, x)` and `K(-n, x)` are `I(n, x)` and `K(n, x)`. For a real
   order the reflection involves a sine of pi times the order and is
   singular at the integers, so the real-order entry points take a
   non-negative order and answer `SpecNegative` otherwise.
7. **The inverses iterate, so `SpecNoConvergence` is a reachable
   answer.** The continued fractions behind `gamma_p_inv`,
   `gamma_q_inv`, `beta_inc_inv`, `erf_inv` and `erfc_inv` converge for
   every argument in the domain, and "converges" is not "converges in
   a fixed number of steps". The fault carries the argument and the
   number of iterations spent, which is enough to report it.

## Accuracy

The claim is a relative error against `scipy.special`, over the range
where the result is representable, and it is stated per group rather
than implied:

| Group | Relative error target |
| --- | --- |
| `specgamma`, `specerf` | at or below 1e-14 |
| `specincomplete` | at or below 1e-13 |
| `specbessel`, for orders up to 100 and arguments up to 1000 | at or below 1e-13 |

**No function here is claimed to be correctly rounded.** A correctly
rounded result is the representable number nearest the exact value,
and proving that for a function computed by a series or a continued
fraction needs more working precision than this package uses. Where a
caller needs a bound rather than a target, the test suite's assertions
are what the implementation is held to.

**Near a zero of `J` or `Y` the ABSOLUTE error is what holds, not the
relative one.** The value passes through zero, so no relative claim
about it can be true. `bessel_j_zero` gives the zeros themselves to
the relative target above.

## What is not included

- **Complex arguments.** Every function here takes and returns a
  `Float`. The Bessel functions of complex argument, the complex error
  function and the Faddeeva function all need a complex type, and
  novo-lang has no package that declares one yet.
- **The Airy functions, the elliptic integrals, the Legendre and
  Laguerre and Hermite polynomials, the Riemann zeta function and the
  hypergeometric functions.** Each is a body of numerical work of its
  own, and none of them is in the way of a distribution function. A
  later package can add them without a signature here moving.
- **Vectorised forms.** There is no `erf` over an array. These are
  scalar functions, and a caller who wants one elementwise maps it
  across the array they already hold. Taking a dependency on an array
  package to provide that would put it in the closure of everything
  that needs a distribution function.
- **The unregularised incomplete integrals.** They overflow for shape
  parameters a fitting routine reaches in its first few steps. A
  caller who genuinely wants one multiplies the regularised value by
  `gamma(a)` or `beta(a, b)` and can see the overflow happen in their
  own expression.

## Related packages

- **`stats-nv`** holds the distributions themselves — densities,
  sampling, summaries and tests. This package is the arithmetic
  underneath its cumulative distribution and quantile functions, and
  it is the reason `stats-nv` can spell them at all.
- **`ndarray-nv`** is N-dimensional arrays of `Float` and of `Int`.
  It is what a caller maps these functions across; neither package
  depends on the other.
- **`std.math`** has the elementary functions — `exp`, `ln`, `sin`,
  `sqrt`. Those are what this package is built from, and the line
  between the two is exactly the line between elementary and special.

## Tests

The suite is written against the signatures and is red by
construction: every assertion reaches a `todo()`. Its reference values
are `scipy.special`'s at the arguments its own test suite uses —
`gamma(0.5)`, `gammaln(100)`, `digamma(1)`, `erf(1)`, `erfc(1)`,
`gammainc(1, 1)`, `betainc(2, 3, 0.5)`, `jv(0, 1)`, `yv(0, 1)`,
`iv(0, 1)` and `kv(0, 1)`.

Beside the published values it asserts identities, which constrain the
implementation at every argument rather than at one: `gamma(n)` is
`(n-1)!`, `erf` is odd, `erf + erfc` is 1, `P + Q` is 1, the beta
function is symmetric, `I_x(a, b) = 1 - I_{1-x}(b, a)`, the inverses
round-trip, the three-term Bessel recurrence holds, and the real-order
entry points agree with the integer-order ones at an integral order.

## Implementation status

| Module | Declared | Implemented |
| --- | --- | --- |
| `specfault` | 8 variants, `message` | no |
| `specgamma` | 8 functions | no |
| `specerf` | 6 functions | no |
| `specincomplete` | 9 functions | no |
| `specbessel` | 13 functions | no |

## Licence

Apache-2.0. See [LICENSE](LICENSE).
