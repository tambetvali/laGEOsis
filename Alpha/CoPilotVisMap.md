As it's not yet even implemented, this goes to Alpha stage:

# Laegna Visualization Mapping  
### (GitHub‑flavored Markdown with inline KaTeX)

This document defines the **first practical Laegna coordinate system** suitable for visualization.  
It is based on the double‑unit ball, linear/exponent dual bands, and the interlace rule.

---

## 1. Double‑Unit Ball

Laegna uses a *double‑unit ball*:

- The **unit** is not the radius.  
- The unit is **one quarter of the diameter**.

Thus the full diameter is:

- from south pole to north pole  
- measured as  
  $4$ units.

We define a vertical coordinate:

- $v \in [0,4]$  
  - $v = 0$ → south pole  
  - $v = 2$ → equator  
  - $v = 4$ → north pole

This axis is where **linear unfolding** and **exponential folding** meet.

---

## 2. Linear vs Exponential Coordinates

Every triangle (sphere) and every cell (chessfield) receives **two coordinates**:

### Linear band (local)
This grows at constant speed:

- $T_{\text{lin}} = v$

### Exponential band (global)
This grows rapidly from south to equator,  
then collapses symmetrically from equator to north:

- For $v \in [0,2]$:  
  $R_{\text{exp}} = 2v$

- For $v \in [2,4]$:  
  $R_{\text{exp}} = 8 - 2v$

This expresses the Laegna rule:

> What is linear in one direction is exponential in the other,  
> and the flip happens at the equator.

---

## 3. The $1/4$ Dimension

The $1/4$ dimension is the Laegna base‑2 cut:

- First cut: $0 \rightarrow 2 \rightarrow 4$  
- Second cut: $0 \rightarrow 1 \rightarrow 2$ and $2 \rightarrow 3 \rightarrow 4$

This mirrors:

- “$1$ between $0$ and $2$” in the first octave  
- “$0$ between $0$ and \infty$” in the second octave

Precision bands switch roles:

- linear becomes exponential  
- exponential becomes linear

This is why the equator ($v=2$) is the **flip point**.

---

## 4. Interlace Anchors

Interlacing happens where:

- $T_{\text{lin}}$ and $R_{\text{exp}}$  
- fall into the same Laegna band  
- or mirror each other across the equator.

We check:

- $T_{\text{lin}} = R_{\text{exp}}$  
- or both in $\{0,2,4\}$  
- or symmetric under the equator flip.

These are the **true Laegna points**.

Everything else is scaffolding.

---

## 5. How This Drives Visualization

For each triangle on the sphere:

1. Compute its average vertical position $v$.
2. Assign:
   - $T_{\text{lin}} = v$
   - $R_{\text{exp}}$ using the piecewise rule.
3. Highlight triangles where linear and exponent coordinates **meet or mirror**.

For each cell on the chessfield:

1. Assign $(X_{\text{lin}}, Y_{\text{exp}})$ using the same $0\!-\!2\!-\!4$ structure.
2. Highlight cells where $X_{\text{lin}}$ and $Y_{\text{exp}}$ **agree**.

This gives us a **first Laegna‑consistent interlacing rule** we can implement.

---

## 6. Summary of the Mapping

- Vertical coordinate:  
  $v \in [0,4]$

- Linear band:  
  $T_{\text{lin}} = v$

- Exponential band:  
  $R_{\text{exp}} = 2v$ for $v \le 2$  
  $R_{\text{exp}} = 8 - 2v$ for $v \ge 2$

- Interlace anchors:  
  $T_{\text{lin}} = R_{\text{exp}}$  
  or both in $\{0,2,4\}$  
  or symmetric under equator flip.

This is the first usable Laegna coordinate system for visualization.
