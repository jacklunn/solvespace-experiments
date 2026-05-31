# Print-in-Place Hinge — Design Approach

## Concept

A knuckle hinge where both leaves and the pin are printed as a single piece.
The alternating barrel segments interlock around a shared pin with a small
clearance gap so the joint can move freely after printing without supports
or assembly.

```
Leaf A         Leaf B
  |    [===]    |
  |  [=]   [=] |
  |    [===]    |
       ^ pin (shared)
```

Leaf A has 3 barrel segments, Leaf B has 2, interleaved along the pin axis.

---

## Parameters (adjust per printer)

| Parameter       | Value    | Notes                              |
|-----------------|----------|------------------------------------|
| `gap`           | 0.25 mm  | Clearance between pin and barrel; tune ±0.05mm per printer |
| `pin_r`         | 2.0 mm   | Pin radius                         |
| `barrel_r`      | pin_r + gap + wall | Outer barrel radius (wall ~1.2mm) |
| `wall`          | 1.2 mm   | Barrel wall thickness              |
| `leaf_w`        | 20 mm    | Width of each hinge leaf           |
| `leaf_t`        | 3.0 mm   | Thickness of each leaf             |
| `knuckle_len`   | 6.0 mm   | Length of each barrel segment      |
| `knuckle_gap`   | 0.25 mm  | Axial gap between barrel segments  |
| `total_len`     | 5 × knuckle_len + 4 × knuckle_gap | Total hinge length |

---

## SolveSpace Group Structure

Each group is a separate solid. Use `New Group > Extrude` for each.

```
Group 1 — Construction geometry
  - Hinge axis line (Z-axis, length = total_len)
  - Pin circle (radius = pin_r, centered on axis)
  - Barrel circle (radius = barrel_r)
  - Construction lines for leaf attachment faces

Group 2 — Pin
  - Extrude pin circle along full hinge axis length
  - This is a single cylinder spanning all knuckles

Group 3 — Leaf A body + barrel segments (3 knuckles)
  - Sketch the leaf profile (rectangle, leaf_w × leaf_t)
    attached to the barrel on one side
  - Sketch the barrel annulus (barrel_r circle minus pin_r circle + gap)
  - Extrude knuckle_len, skip knuckle_gap, repeat ×3
  - Positions along Z: [0, k], [2k+g, 3k+g], [4k+2g, 5k+2g]
    where k = knuckle_len, g = knuckle_gap

Group 4 — Leaf B body + barrel segments (2 knuckles)
  - Same profile as Leaf A, mirrored to opposite side of pin
  - Extrude at the gaps left by Leaf A
  - Positions along Z: [k+g, 2k+g], [3k+2g, 4k+2g]
```

---

## Modeling Steps

### 1. Set up construction geometry (Group 1)

1. Open a new sketch on the XY plane.
2. Draw the hinge axis as a **construction line** along Z of length `total_len`.
3. Draw the **pin circle** (r = 2.0 mm) centered at origin — mark as construction.
4. Draw the **barrel circle** (r = pin_r + gap + wall) — mark as construction.
5. Add a distance constraint: pin circle to barrel circle inner face = `gap`.
   This makes the gap a single number to change later.

### 2. Model the pin (Group 2)

1. `New Group > Extrude`, sketch on XY plane.
2. Draw a solid circle at r = `pin_r`.
3. Extrude `total_len` along +Z.

### 3. Model Leaf A (Group 3)

Sketch on XY plane. You will use **three separate extrusions** (one per knuckle)
or a single sketch extruded with step-extrude if your SolveSpace version supports it.
Simplest approach: three groups, each a thin extrusion.

For each knuckle segment:
1. `New Group > Extrude`, sketch on the face at the correct Z offset.
2. Draw the **annular barrel cross-section**: outer circle (r = barrel_r) with
   the inner hole circle (r = pin_r + gap). Use "Difference" or draw both
   circles and let the solid fill the annulus.
3. Attach a **leaf tab**: rectangle extending from the barrel edge, `leaf_w` wide
   and `leaf_t` thick, on the +X side.
4. Extrude `knuckle_len` in +Z.
5. Repeat for the other two knuckle positions.

> Tip: Constrain the barrel circles to be **concentric with the construction
> axis point** from Group 1. This keeps all knuckles aligned.

### 4. Model Leaf B (Group 4)

Same as Leaf A but:
- Leaf tab extends in the **−X direction** (opposite side of pin).
- Extrusions sit at the gaps between Leaf A's knuckles.
- Two knuckle segments.

> The leaves can extend the same direction if you want a 180° flat-open hinge,
> or opposite directions for a 90° default-closed hinge.

### 5. Verify clearance

Before exporting:
- Zoom into the barrel cross-section and confirm the annular gap is visible.
- Measure: inner barrel face to pin outer face should read `gap` (0.25 mm).
- If SolveSpace merges the solids, the gap has collapsed — increase `gap` slightly.

---

## Print Settings

| Setting              | Recommendation |
|----------------------|----------------|
| Orientation          | Pin axis horizontal on bed |
| Layer height         | 0.15–0.2 mm    |
| Perimeters           | 3+             |
| Supports             | None (by design) |
| First-layer gap tuning | Adjust `gap` if joint fuses or is too loose |

**After printing:** flex the hinge back and forth a few times to break any
light layer adhesion. If it won't move, the gap is too small. If it falls apart,
the gap is too large or `wall` is too thin.

---

## Files

| File | Description |
|------|-------------|
| `hinge-pin.slvs` | Pin solid |
| `hinge-leaf-a.slvs` | Leaf A with 3 knuckles |
| `hinge-leaf-b.slvs` | Leaf B with 2 knuckles |
| `hinge-assembly.slvs` | Linked assembly for visualization |

Or model all groups in a single `print-in-place-hinge.slvs` file —
simpler but harder to isolate issues per part.

---

## Iteration Log

| Version | gap (mm) | Result |
|---------|----------|--------|
| v1      | 0.25     | TBD    |
