# adam-optimizer-alternatives
# Adam Optimizer Alternatives

This repository explores **alternative variants of the Adam optimizer** in scenarios where Adam behaves poorly in directions with very small curvature. The work is based on a homework project for the *Collective Learning* course and focuses on understanding Adam's weaknesses and proposing simple modifications to improve its behavior in flat directions.

---

## 1. Project Motivation

Adam is a widely used adaptive optimizer, but it can behave badly when the gradient becomes very small:

- As the gradient norm shrinks, the second-moment estimate \\(\hat{v}_t\\) in the denominator also becomes very small.
- This makes the **effective step size artificially large**, causing the parameters to jump too much even though the gradient is tiny.
- As a result, update steps can become inconsistent (one step too large, next step too small), and the direction information can be corrupted by bad rescaling.
- In practice, this may lead to **oscillation, instability or slow convergence**, especially in flat directions. 

The goal of this project is to **analyze this behavior and propose simple, interpretable fixes**.

---

## 2. Proposed Adam Variants

Two alternative variants of Adam are implemented and evaluated:

### 2.1 Adam_S1 – Clipped Second Moment

**Idea:** Prevent the denominator \\(\hat{v}_t\\) from becoming too small.

- Define a minimum threshold \\( \hat{v}_{t,\text{min}} \\).
- During the update, instead of using \\(\hat{v}_t\\) directly, use:

  \\[
  \hat{v}_t^\* = \max(\hat{v}_t, \hat{v}_{t,\text{min}})
  \\]

- This ensures that the denominator never collapses to extremely small values, so the step size cannot explode artificially.
- Target: **More stable and consistent steps** in directions where the curvature is very small.

### 2.2 Adam_S2 – Fallback to SGD in Flat Directions

**Idea:** If \\(\hat{v}_t\\) becomes too small, stop using Adam and behave like vanilla SGD for that dimension.

- Define a threshold \\( \hat{v}_{t,\text{thresh}} \\).
- For each dimension:
  - If \\(\hat{v}_t > \hat{v}_{t,\text{thresh}}\\), use standard Adam update.
  - If \\(\hat{v}_t \le \hat{v}_{t,\text{thresh}}\\), switch to **SGD-style update** in that coordinate.
- This avoids the artificial growth of the step size in very flat regions and keeps updates more controlled. 

---

## 3. Test Function and Experimental Setup

To compare **Adam**, **Adam_S1** and **Adam_S2**, we use a simple 2D convex function with very different curvature in each direction:
\\[
f(w) = \frac{1}{2} \left( w_1^2 + 10^{-6} w_2^2 \right)
\\]

- In the \\(w_1\\) direction, curvature is 1 (normal).
- In the \\(w_2\\) direction, curvature is **extremely small** (almost flat).
- This highlights exactly the regime where Adam can become unstable or behave oddly.

**Default experimental parameters:**

- Initial point: \\( w_0 = (1, 1) \\)
- Number of steps: 200
- Learning rate: \\( \alpha = 0.01 \\)
- For Adam_S1: \\( \hat{v}_{t,\text{min}} = 10^{-3} \\)
- For Adam_S2: \\( \hat{v}_{t,\text{thresh}} = 10^{-6} \\) 

The same setup is repeated with much smaller thresholds (e.g. \\( \hat{v}_{t,\text{min}} = 10^{-13} \\), \\( \hat{v}_{t,\text{thresh}} = 10^{-16} \\)) to study the sensitivity to these hyperparameters. 
---

## 4. Summary of Results

For the default threshold values, the final parameters and function values are approximately: 

| Method    | w₁ (final)   | w₂ (final)   | f(w) [×10⁻⁴] |
|----------|-------------:|-------------:|-------------:|
| AdamBase | 0.01557249   | 0.01767338   | 1.212513     |
| Adam_S1  | 0.01557249   | 0.99800182   | 1.217492     |
| Adam_S2  | 0.01557249   | 0.99999800   | 1.217512     |

**Observations:**

- All three methods behave similarly in the **w₁ direction**; there is no clear advantage there.
- **AdamBase** drives \\(w_2\\) very close to 0, taking large and inconsistent steps in the “flat” direction.
- **Adam_S1** and **Adam_S2** keep \\(w_2\\) closer to 1, avoiding the unstable step explosion in the flat direction and achieving **more consistent updates**.
- However, the **loss value is not significantly improved**; the alternatives mainly change the trajectory in flat directions, not the final objective value in this simple setting.

When the thresholds are made extremely small (e.g. \\(10^{-13}, 10^{-16}\\)), all three methods become almost identical to standard Adam, thus no improvement is observed. The alternatives only matter in the parameter regimes where Adam is already known to be weak.  

---
