# Mixed Dimensional Sensor Fusion

Processed evidence from four sensing modalities for wall inspection, reference
labels, and the results of the proposed method and three comparison methods.
The data are provided so that others can apply their own methods and compare
results on the same experiment.

## Download

Select **Code > Download ZIP** and extract the archive, or download individual
CSV files from the folders below. The download contains the data and numerical
outputs. Plot previews are displayed on this page using hosted images.

## Sensor evidence

The inputs are **processed, dimensionless evidence scores on a percentage
scale**, not raw sensor measurements or calibrated class probabilities.
All sensors share a **76 row x 48 column** wall grid (3,648 pixels).

| Sensor | Evidence CSV |
| --- | --- |
| XM125 radar | [3D evidence](data/evidence/xm125_absdiff_wall_twin_76x48.csv) |
| FDC2114 capacitance | [2D evidence](data/evidence/fdc_pct_wall_twin_76x48.csv) |
| RAK12029 inductive | [2D evidence](data/evidence/rak_pct_wall_twin_76x48.csv) |
| EM | [2D evidence](data/evidence/em_pct_wall_twin_76x48.csv) |

### Evidence previews

<img src="https://raw.githubusercontent.com/Muxipi/mixed_dimensional_sensor_fusion/plot-previews/xm125_absdiff_3d_pct_plot.png" alt="Radar depth evidence" width="46%">

Radar display threshold: **20%**, retaining points where `value_pct >= 20`.

<img src="https://raw.githubusercontent.com/Muxipi/mixed_dimensional_sensor_fusion/plot-previews/fdc_pct.png" alt="Capacitance evidence" width="46%">

<img src="https://raw.githubusercontent.com/Muxipi/mixed_dimensional_sensor_fusion/plot-previews/rak_pct.png" alt="Inductive evidence" width="46%">

<img src="https://raw.githubusercontent.com/Muxipi/mixed_dimensional_sensor_fusion/plot-previews/em_pct.png" alt="Electromagnetic evidence" width="46%">


FDC, RAK, and EM CSVs are headerless 76 x 48 matrices. Rows increase from the
wall's top to bottom. Columns increase from left to right. Keep this alignment
when combining sensors or comparing predictions with the reference.

The radar CSV has headers `row,col,depth_idx,depth_mm,value_pct`:

| Column | Meaning |
| --- | --- |
| `row` | Wall grid row, 0 to 75 |
| `col` | Wall grid column, 0 to 47 |
| `depth_idx` | Depth bin, 0 to 64 |
| `depth_mm` | Depth coordinate in millimeters |
| `value_pct` | Processed radar evidence score |

It contains all **237,120 voxels** of a 76 x 48 x 65 volume. Depth ranges from
80 to 240 mm, inclusive, at 2.5 mm intervals. Low and zero evidence values are
included. **To reproduce the radar point cloud, display only voxels with
`value_pct >= 20` (a 20% normalized evidence score threshold).** The threshold
filters the preview only. The evidence CSV retains all voxels, so you can choose
a different threshold for your own display or method. It is not 20% of total
signal energy or an additional normalization step.

The wall is 120 cm wide and 190 cm high. Acquisition used 5 cm windows and a
nominal 2.5 cm step. Existing 3D figures map columns with `linspace(0,120,48)`
and rows with `linspace(190,0,76)`. 2D figures use the same wall extent with row
zero at the top. Align arrays by row and column rather than inferring indices
from figure coordinates.

The evidence fields are already normalized inputs to fusion. Radar, FDC, and
RAK evidence are derived relative to a baseline. EM evidence is based on the
ratio of energy in the first five 50 Hz harmonics to total signal energy,
followed by normalization. Original waveforms and baseline recordings are not
part of this release. A score of 80 should not be interpreted as an 80%
probability of a material.

## Reference labels

- [Reference matrix](data/reference/reference_label_map.csv): headerless 76 x 48 integer map.
- [Reference table](data/reference/reference_labels_table.csv): `row,col,label_name,state_id` records.

| ID | State | Pixels |
| ---: | --- | ---: |
| 0 | Void | 3,235 |
| 1 | Moisture | 217 |
| 2 | Metal | 76 |
| 3 | Electric | 120 |

These labels describe the reference target layout for this wall experiment.
They are supplied for evaluating your own predictions. No separate train/test
split or independently annotated 3D reference volume is provided.

## Proposed method outputs

- [2D label matrix](results/proposed/method_2d_label_map.csv): 76 x 48, with the same class IDs as the reference.
- [Numerical 3D labels](results/proposed/method_3d_labels.npz): a compressed NumPy archive.

The archive contains `labels` with shape `(76,48,65)`, `depth_mm`,
`state_names`, `wall_width_cm`, and `wall_height_cm`. It can be loaded with
NumPy using `allow_pickle=False`. Labels are indexed by row, column, and depth.
Moisture is excluded from the 3D result because it lacks radar depth support.
The final 2D result includes all four classes.

<img src="https://raw.githubusercontent.com/Muxipi/mixed_dimensional_sensor_fusion/plot-previews/method_3d_no_moisture.png" alt="Proposed 3D reconstruction" width="65%">

## Comparison methods

| Method | Label matrix |
| --- | --- |
| Weighted sum | [CSV](results/baselines/weighted_sum_label_map.csv) |
| Dempster-Shafer | [CSV](results/baselines/dempster_shafer_label_map.csv) |
| Direct 2D Bayesian | [CSV](results/baselines/direct_2d_bayesian_label_map.csv) |

These are the comparison results used for this experiment. All output CSVs use
the reference grid orientation and class IDs.

![Reference and reconstructed maps](https://raw.githubusercontent.com/Muxipi/mixed_dimensional_sensor_fusion/plot-previews/comparison_2d_maps.png)

## Evaluation metrics

| Method | Accuracy (%) | Macro precision (%) | Macro recall (%) | Macro F1 (%) |
| --- | ---: | ---: | ---: | ---: |
| Weighted sum | 54.25 | 50.50 | 82.91 | 47.09 |
| Dempster-Shafer | 74.67 | 56.52 | 81.43 | 54.05 |
| Direct 2D Bayesian | 70.20 | 57.09 | 75.91 | 47.11 |
| Proposed method | 91.28 | 66.33 | 96.15 | 75.44 |

[Download the metrics CSV](results/metrics.csv). Values in that CSV are
fractions in `[0,1]`. The table above displays percentages.

Metrics use all 3,648 reference pixels. Macro precision, recall, and F1 average
over the four classes with equal weight. To benchmark a new method, produce a
76 x 48 label map using the same IDs and orientation, then compare every pixel
with the reference. This is one wall experiment with a dominant Void class.
Macro metrics complement accuracy when comparing methods.

## License

Data and results are provided under [CC BY 4.0](DATA_LICENSE.md), with
attribution to Muxipi. No paper citation is requested as a condition of use.
The existing [MIT license](LICENSE) applies to any software material.
