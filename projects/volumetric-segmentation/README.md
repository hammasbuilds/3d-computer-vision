# Volumetric segmentation: what a whole-tumour Dice of 0.87 hides

BraTS 2020 is 368 multi-modal brain MRI volumes with three annotated tumour subregions.
A 3D U-Net (5.6 M parameters, base 16, depth 4) is **trained from scratch** on four
channels — FLAIR, T1ce, T2, T1 — for 60 epochs. Split by patient, 70/15/15, seed 0, so
no patient's voxels appear on two sides. The epoch was chosen on validation Dice; the
test patients were untouched until the end.

## Result

Reported on all three splits, because the gap between them is the only way to see what
the selection cost.

| split | patients | whole tumour | tumour core | enhancing |
|---|---|---|---|---|
| train | 258 | 0.8802 | 0.8605 | 0.6943 |
| val | 55 | 0.8385 | 0.7922 | 0.6605 |
| **test** | **55** | **0.8734** | **0.7993** | **0.6658** |

Bar was whole-tumour Dice ≥ 0.80 on test. With 95% intervals over patients:

| test region | Dice | 95% CI |
|---|---|---|
| whole tumour | **0.8734** | [0.8537, 0.8931] |
| tumour core | 0.7993 | [0.7447, 0.8539] |
| enhancing tumour | 0.6658 | [0.5869, 0.7447] |

The bar is cleared, and the interval sits entirely above it.

## The finding: the headline metric is blind to the failure that matters

On **6 of the 55 test patients the enhancing-tumour Dice is exactly 0.0000** — the model
and the annotation share no enhancing voxel at all. The whole-tumour score does not
notice:

| | whole tumour | tumour core |
|---|---|---|
| the 6 patients scoring 0.0000 on enhancing | **0.8550** | **0.4400** |
| the other 49 | 0.8756 | 0.8433 |

Whole tumour falls by 0.02. Tumour core falls by 0.40. **Five of those six patients
still clear the 0.80 whole-tumour bar.**

| case | whole tumour | tumour core | enhancing |
|---|---|---|---|
| 266 | 0.8960 | 0.7205 | 0.0000 |
| 324 | 0.8771 | 0.3781 | 0.0000 |
| 289 | 0.8757 | 0.5777 | 0.0000 |
| **361** | **0.8735** | **0.0322** | 0.0000 |
| 286 | 0.8178 | 0.7153 | 0.0000 |
| 278 | 0.7896 | 0.2159 | 0.0000 |

Case 361 is the clearest: a whole-tumour Dice of 0.8735, above the bar, with a tumour
core of 0.0322. Whole tumour is the union of all three labels, and that union is
dominated by peritumoural oedema — the largest and most separable region. A model can own
the oedema and miss the enhancing rim entirely while the headline barely moves.

Had this project reported one number, it would have reported 0.8734 and been wrong about
what the model does.

## Per class, and where the floor is

| region | train | val | test | test scored |
|---|---|---|---|---|
| necrotic / non-enhancing core | 0.6667 | 0.5509 | 0.6265 | 55 |
| peritumoural oedema | 0.7419 | 0.7012 | **0.7413** | 55 |
| enhancing tumour | 0.6943 | 0.6605 | 0.6658 | 55 |

Oedema is the best-segmented class on every split, which is the mechanism above. An
empty-and-correctly-predicted-empty class is recorded as `None` and **excluded** from the
average rather than scored 1.0 — scoring it 1.0 would inflate exactly the rare classes
that are sometimes absent. So a 0.0000 is a real disagreement, not an absent region.

Whole tumour clears 0.80 on **49 of 55** patients; its floor is case 089 at 0.5953.

## Input / Output

Input: BraTS 2020 on Kaggle, attached by reference. Nothing is uploaded.
Output: `results.json` — per-split Dice per class and per composite region, per-patient
rows for all 55 test cases, the per-epoch history, and `run.log`.

Notebook: [3D UNet BraTS Dice](https://www.kaggle.com/code/muhammadhammas13/3d-unet-brats-dice)

## What this does not show

- **These numbers are not comparable to the BraTS leaderboard.** Dice is measured on a
  128³ resample of the brain-cropped volume, not the native 240×240×155 grid. Resampling
  changes the denominator, and small regions are affected most — which is the enhancing
  class. The comparison would be apples to oranges in the direction that flatters this
  run, so no leaderboard figure is quoted here.
- **The direction of the six failures is not established.** A Dice of 0.0000 means a
  non-empty union with an empty intersection; `results.json` does not record ground-truth
  voxel counts, so it cannot be said whether the model missed a region that was present
  or predicted one that was absent. Those are different faults and this run does not
  separate them.
- One architecture, one seed, 60 epochs, 5.6 M parameters. A larger model, a longer
  schedule or an ensemble would move every number, and the oedema-dominance effect is a
  property of the whole-tumour metric rather than of this particular network.
- The 15% test split is 55 patients. The enhancing interval spans 0.16; it supports
  "roughly two thirds", not a comparison against another method within a few points.
