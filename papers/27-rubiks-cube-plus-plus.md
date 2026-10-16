# Paper 27 – Rubik's Cube++

- **Title:** Revisiting Rubik's Cube: Self-supervised Learning with Volume-wise Transformation for 3D Medical Image Segmentation (Tao et al., 2020, MICCAI 2020; the venue is not printed in the arXiv text, so it comes from outside knowledge)
- **Link:** https://arxiv.org/pdf/2007.08826
- **One line:**
  - **Architecture:** a 3D U-Net encoder-decoder (generator) trained GAN-style with a 4-layer discriminator.
  - **Pretext task:** cut the volume into a grid of sub-cubes, rotate random layers by 90° / 180° / 270° around the sagittal, coronal and axial axes (like twisting a real Rubik's cube), then restore the original volume.
  - **Loss:** L1 + adversarial (λ = 10).
  - **Pretraining data:** the target datasets themselves (NIH Pancreas-CT, MRBrainS18), with no extra data.
  - Image-only. The whole encoder + decoder is pretrained.
- **Eval types used:** fine-tuned-segmentation, random-init-baseline, few-shot/label-efficiency, ablation, other:video-pretraining-baseline (UCF101), other:qualitative-restoration/segmentation-figures

---

## Eval 1 – Pancreas segmentation (CT) with label fractions
- **Analogy:**
  - Engine put in a car and tuned.
  - Also "tune with only a handful of examples": 10% of labels.
- **How:**
  - Fine-tuned 3D U-Net (both encoder and decoder pretrained).
  - Adam, lr 1e-4, about 250 epochs.
  - Metric: Dice, with 4-fold cross-validation.
  - Uses 10 / 20 / 50 / 100% of the training labels.
- **Datasets:** NIH Pancreas-CT. The same scans were used for pretraining (without labels). Internal only.
- **Competitors:**
  - Train from scratch.
  - UCF101 video pretraining.
  - Arbitrary puzzles.
  - MedicalNet (Med3D).
  - Rubik's Cube.
  - Models Genesis.
- **Fairness:**
  - Models Genesis and Rubik's Cube were re-pretrained by the authors on the same data, not taken from public weights. This is fair.
  - The Arbitrary puzzles numbers are copied from its paper, which used V-Net.
  - The paper does not state whether MedicalNet used public weights.
  - Competitor backbones are not explicitly stated, apart from the 3D U-Net used for scratch and UCF101.
- **Result (Dice %, at 10 / 20 / 50 / 100% labels):**

  | Method | 10% | 20% | 50% | 100% |
  |---|---|---|---|---|
  | Rubik's Cube++ | 73.30 | 78.10 | 82.80 | 84.08 |
  | Models Genesis | 63.11 | 70.08 | 79.93 | 83.23 |
  | Rubik's Cube | 61.07 | 70.43 | 80.30 | 82.76 |
  | MedicalNet | 64.80 | 71.37 | 77.41 | 80.09 |
  | Arbitrary puzzles (copied) | 70.80 | 76.50 | n/a | 81.68 |
  | Scratch | 58.58 | 70.19 | 79.68 | 82.90 |

  - The big win is at low labels: +14.7 Dice over scratch at 10%.
  - At 100% the gap shrinks to about +1.

## Eval 2 – Brain tissue segmentation (MRI, MRBrainS18)
- **Analogy:** Engine put in a car and tuned, on a different kind of road (MRI).
- **How:** Fine-tuned, leave-one-out evaluation, mean Dice. Per-class Dice is in an appendix table.
- **Datasets:** MRBrainS18, which is very small (leave-one-out). It was also used for pretraining. Internal only.
- **Competitors:** Scratch, UCF101, Rubik's Cube, Models Genesis.
- **Result (mean Dice %):**
  - Rubik's Cube++: 77.56.
  - Models Genesis: 76.19.
  - Scratch: 72.22.
  - UCF101: 71.34.
  - Rubik's Cube: 71.23.
  - A t-test against Models Genesis gives p = 3.42%.

## Eval 3 – Ablations
- **Analogy:** Tuning the break-in recipe.
- **How:**
  - Sub-cube grid size n (main table, pancreas).
  - L1 vs L2 vs adversarial loss (appendix).
  - Number of rotated layers m (appendix).
- **Result:**
  - n = 7 is best at all label fractions. n = 3 is worse than scratch at 10% (47.28 vs 58.58), so a puzzle that is too easy hurts.
  - L1 + adversarial is best (78.10 at 20%).
  - m = 4 is best (78.10 at 20%, versus 68.08 for m = 3 and 73.46 for m = 5).
- **Note:** The reconstruction-error numbers are inconsistent between text and table (0.00826 vs 0.826×10⁻²).

## Eval 4 – Qualitative figures
- Example restorations and segmentation overlays for pancreas and brain. Visual only.

## Key takeaways
- The evaluation is all "engine put in a car and tuned" on two small, single-site datasets that were also the pretraining data. There is no external validation, no frozen probe and no multi-organ test.
- It is fairer than the original Rubik's Cube paper because Models Genesis and Rubik's Cube were re-pretrained on the same data. One row (Arbitrary puzzles) is still copied from a V-Net paper.
- The main message is label efficiency: large gains at 10–20% labels, and small gains with full labels. This pattern recurs across this list (TransVW, PCRL, VoCo).
- The design move is to pretrain the decoder too, by turning the puzzle into restoration. This aligns it with Models Genesis, TransVW and the PCRL family rather than the classification-style original Rubik's Cube.
- Results come from single cross-validation runs, with one t-test on brain only. The pretraining scale is tiny (hundreds of volumes) compared with SuPreM, VoCo and CT-FM.
