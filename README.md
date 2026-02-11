# Foundations

[![Build](https://github.com/RoyGBivens137/Foundations/actions/workflows/build.yml/badge.svg)](https://github.com/RoyGBivens137/Foundations/actions/workflows/build.yml)

A formal proof of **Bochner's theorem**, the **Fejér-Riesz factorization**, and the **Riesz representation theorem** on the circle group U(1), written in [Lean 4](https://lean-lang.org/) using [Mathlib](https://leanprover-community.github.io/mathlib4_docs/).

The entire `FourierBochner` library compiles with **zero `sorry`**.

## Main Results

### Bochner's Theorem on U(1)

A continuous, periodic, positive-definite function on the circle admits a non-negative spectral measure:

```lean
theorem bochner_spectral_measure (f : ℝ → ℂ) (hf : Continuous f)
    (hf_pd : IsPositiveDefinite f) (hf_per : ∀ θ, f (θ + 2 * π) = f θ) :
    ∃ (μ : Measure ℤ), IsFiniteMeasure μ ∧
      ∀ θ, f θ = ∑' k : ℤ, ↑((μ {k}).toReal) * exp (I * ↑k * ↑θ)
```

This factors through an explicit Fourier coefficient construction:

```lean
theorem constructive_bochner_via_sheaf (f : ℝ → ℂ) (hf : Continuous f)
    (hf_pd : IsPositiveDefinite f) (hf_per : ∀ θ, f (θ + 2 * π) = f θ) :
    ∃ (μ : ℤ → ℝ), (∀ k, 0 ≤ μ k) ∧ (Summable μ) ∧
      ∀ θ, f θ = ∑' k : ℤ, ↑(μ k) * exp (I * ↑k * ↑θ)
```

### Fejér-Riesz Factorization

A non-negative real-valued trigonometric polynomial is the squared modulus of an analytic trigonometric polynomial:

```lean
theorem fejer_riesz (R : TrigPolyℤ)
    (hR_real : ∀ θ : 𝕋, (R.toCircle θ).im = 0)
    (hR_nonneg : ∀ θ : 𝕋, 0 ≤ (R.toCircle θ).re) :
    ∃ (P : TrigPolyℤ), R = TrigPolyℤ.normSq P
```

### Finite Bochner Theorem

On finite cyclic groups, positive-definite is equivalent to having strictly positive Fourier coefficients:

```lean
theorem bochner_finite (n : ℕ) [NeZero n] (f : ZMod n → ℂ) :
    IsPositiveDefiniteFinite n f ↔
    ∃ μ : ZMod n → ℝ, (∀ k, 0 < μ k) ∧
      ∀ m, f m = ∑ k : ZMod n, μ k * character n k m
```

### Argument Principle for Polynomials

Discrete winding numbers on the profinite lattice converge to the true root count:

```lean
theorem argument_principle_polynomial (Q : Polynomial ℂ) (hQ : Q ≠ 0)
    (r : ℝ) (hr : 0 < r)
    (h_no_roots_on_circle : ∀ α ∈ Q.roots, ‖α‖ ≠ r) :
    ∃ (N₀ : ℕ), ∀ N ≥ N₀,
      |discrete_winding_number Q N r -
        (Q.roots.filter (fun α => ‖α‖ < r)).card| < 1/2
```

## Proof Architecture

The proof of Bochner's theorem follows a **point-sampling** approach that avoids the analytic difficulties of the traditional measure-theoretic proof:

1. **Point samples are weak-PD.** Evaluating a continuous positive-definite function at evenly-spaced points on the circle produces a weak positive-definite function on the finite cyclic group Z/NZ.

2. **Weak PD implies non-negative DFT.** Testing the PD condition against characters shows all discrete Fourier coefficients have non-negative real part.

3. **Riemann sums converge.** The discrete DFT sums are exactly Riemann sums for the Fourier coefficient integral, and Mathlib's `riemann_sum_converges_to_integral` gives convergence.

4. **Limits preserve non-negativity.** The Fourier coefficients, as limits of non-negative quantities, are themselves non-negative.

The Fejér-Riesz factorization is proved via **spiral symmetry**: the conjugate-reciprocal pairing of roots forces a |P(z)|² structure. Root detection uses discrete winding numbers on a profinite lattice that converge to the argument principle. The factorization is extracted via Mahler measure bounds and Bolzano-Weierstrass compactness.

## Module Structure

| File | Lines | Description |
|------|-------|-------------|
| `Defs.lean` | 277 | Positive-definite functions, characters on ZMod n |
| `Character.lean` | 2,142 | Character orthogonality, Parseval, Fourier inversion |
| `TrigPoly.lean` | 3,529 | Trigonometric polynomials, Riemann sums, Fejér/Dirichlet kernels |
| `FejerRiesz.lean` | 10,472 | Spiral symmetry, argument principle, spectral factorization |
| `Converse.lean` | 1,466 | Riesz-Markov functional extension |
| `FiniteBochner.lean` | 1,656 | Bochner's theorem on finite cyclic groups, profinite tower |
| `Bochner.lean` | 810 | Full Bochner theorem via point samples |

Dependency graph:
```
Defs ← Character ← TrigPoly ← FejerRiesz ← Converse
                                    ↑
                              FiniteBochner

Bochner ← Character + FejerRiesz
```

## Building

Requires [elan](https://github.com/leanprover/elan) (the Lean version manager).

```bash
lake exe cache get   # download prebuilt Mathlib oleans
lake build           # build FourierBochner
```

Toolchain: `leanprover/lean4:v4.28.0-rc1`

## Authors

Zachary Mullaghy, Gianfranco Romaelle
