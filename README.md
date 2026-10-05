# 3d-computer-vision

Reconstructing geometry from images and point clouds - depth, surfaces, normals,
registration, decimation and novel views. Classical methods and learned ones, held to
the same standard: every number is measured against ground truth that is either known
in closed form or planted on purpose.

| project | kind | result |
|---|---|---|
| [Marching cubes](projects/marching-cubes/) | classical | error is exactly second order: fitted 2.00-2.05 on both shapes and both variants |
| [ICP registration](projects/icp-registration/) | classical | point-to-plane tolerates twice the misalignment (60 deg vs 30) and is 5.4x worse under noise |
| [Surface reconstruction](projects/surface-reconstruction/) | classical | the convex hull wins on a sphere and cannot represent a hole: 16x more points change its torus error by 4.7% |
| [Normal estimation](projects/normal-estimation/) | classical | angular error falls as k^-1, not the k^-0.5 that averaging k neighbours implies |
| [Point-cloud decimation](projects/point-cloud-decimation/) | classical | Chamfer separates four methods by 1.24x; covering radius separates them by 3.28x |
| [Two-view triangulation](projects/two-view-triangulation/) | classical | a longer baseline buys depth accuracy and nothing else: depth error is 513x lateral at 2 cm |
| [Monocular depth](projects/monocular-depth/) | learned | the alignment protocol moves delta<1.25 from 0.461 to 0.911 on identical predictions |
| [Point-cloud part segmentation](projects/point-cloud-segmentation/) | learned | rare categories average 0.7016 IoU against 0.8249 for common ones |
| [Novel-view synthesis](projects/mesh-or-novel-view/) | learned | 16.78M of 16.79M parameters are the hash table; train views beat held-out by 5.357 dB |
| [Volumetric segmentation](projects/volumetric-segmentation/) | learned | whole-tumour Dice 0.8734 hides enhancing Dice of exactly 0.0000 on 6 of 55 patients |

## Why both kinds live here

A sphere's volume is `4/3 pi r^3` whether you extract it with marching cubes or
predict it with a network, so the two families can be judged the same way. Splitting
them by technique would have put the classical work at the end of a 63-project
classical repo and the learned work inside a recognition repo, where neither is
findable by anyone looking for 3D.

## Method

- **Ground truth is exact or planted.** Analytic surfaces for the classical projects,
  a known rigid transform for registration, Kinect depth and official held-out splits
  for the learned ones. Nothing is scored against another algorithm's output.
- **Every number in a README is checked against that project's `results.json`** by
  `tools/check_readme_numbers.py` before it ships. Derived figures are declared in a
  `readme_numbers_allow.txt` so the exemptions are reviewable rather than invisible.
- **Classical projects: no neural network, no training, no GPU.** Learned projects run
  on Kaggle's GPU and say what they do and do not establish.

```
pytest projects/ -q                             # 59 tests across the classical six
python tools/check_readme_numbers.py projects/<name>
```

Part of a portfolio: [vision-portfolio-woad.vercel.app](https://vision-portfolio-woad.vercel.app)
 · [notebooks](https://www.kaggle.com/muhammadhammas13/code)
 · [github.com/hammasbuilds](https://github.com/hammasbuilds)
