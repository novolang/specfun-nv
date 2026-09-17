# Changelog

All notable changes to specfun-nv are recorded here. The format is
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this
package follows [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
with the pre-1.0 rule that a breaking change bumps the MINOR number.

## [0.0.1] — 2026-09-17

**The interface, published before anyone implements it.** Every public
type and function carries its full signature, its effect row and its
doc comment; every body is `todo()`; the release is recorded
`implemented = false`.

### Added

- `specfault` — the load-bearing interface. Every function in the
  package answers `Result<Float, SpecFault>`, and not one of them
  returns a NaN. None of these functions is defined on the whole real
  line, and the domains are not obvious from the names; a NaN carries
  no reason, propagates silently through the arithmetic that follows
  it, and leaves a caller with four special-function calls in one
  expression to bisect by hand. Eight faults, each naming the domain
  rule that was broken and carrying the argument that broke it. An
  overflow is deliberately NOT a fault: `gamma(200.0)` is an infinity
  and that is a representable answer to a well-posed question.
- `specgamma` — `gamma`, `log_gamma`, `log_gamma_sign`, `digamma`,
  `beta`, `log_beta`, `rising_factorial` and `binomial`. The logarithm
  and its sign are two functions because the gamma function is
  negative on half of the intervals between the negative integers, so
  its logarithm is complex there and one `Float` cannot carry both
  parts. Returning a NaN for half the negative line is what every
  library that got this wrong did.
- `specerf` — `erf`, `erfc`, `erfcx`, `erf_inv`, `erfc_inv` and
  `dawson`. `erfc` is a function rather than `1 - erf` because at an
  argument of 6 the subtraction leaves about two correct digits out of
  sixteen, and a tail probability IS that difference.
- `specincomplete` — the regularised incomplete gamma and beta, their
  directly computed complements, their inverses and their logarithms.
  Only the regularised forms are published: the unregularised
  integrals overflow for shape parameters a fitting routine reaches in
  its first few steps.
- `specbessel` — `J`, `Y`, `I` and `K` with an integer-order and a
  real-order entry point each, the scaled modified forms, the zeros of
  `J` and the spherical forms. Two entry points rather than one
  because the integer-order recurrence is exact and cheap while a real
  order needs a series, an asymptotic expansion or a continued
  fraction, and a caller who knows the order is a count should not pay
  for the general case or write `4.0` where they mean `4`.

### Known

- `novo test` is red, and that is the release's expected state: every
  assertion in the API suite reaches `not implemented:
  specfun-nv.<module>.<fn>`.
- **The accuracy claim is a target, not a proof.** The README's
  accuracy section states a relative error against `scipy.special` per
  group, and says that no function here is claimed to be correctly
  rounded and that near a zero of `J` or `Y` only an absolute claim
  can hold.
- **No complex arguments.** The Bessel functions of complex argument,
  the complex error function and the Faddeeva function all need a
  complex type, and no package on the grid declares one.
- **No Airy, elliptic, orthogonal-polynomial, zeta or hypergeometric
  functions.** Each is a body of numerical work of its own and none of
  them is in the way of a distribution function.
