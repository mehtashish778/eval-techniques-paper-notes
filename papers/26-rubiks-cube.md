# Paper 26 – Rubik's Cube (Rubik's cube recovery)

- **Title:** Self-supervised Feature Learning for 3D Medical Images by Playing a Rubik's Cube (Zhuang et al., 2019, MICCAI 2019; the venue is not printed in the arXiv text, so it comes from outside knowledge)
- **Link:** https://arxiv.org/pdf/1910.02241
- **One line:**
  - **Architecture:** a 3D VGG-style encoder in a "Siamese-Octad" set-up, with 8 weight-sharing branches, one per sub-cube.
  - **Pretext task:** cut the volume into a 2×2×2 grid of cubes, shuffle and flip them, then predict the order (classify among K permutations) and each cube's 180° rotation.
  - **Pretraining data:** the target datasets themselves (1,486 brain CTs; BraTS-2018 MRI), with no extra data.
  - Image-only.
- **Eval types used:** fine-tuned-classification, fine-tuned-segmentation, random-init-baseline, ablation, other:pretext-task-accuracy, other:video-pretraining-baseline (UCF101)

---

## Eval 1 – Pretext-task accuracy
- **Analogy:** Checking that the break-in routine itself was learned, before the engine goes in any car.
- **How:** Test accuracy on solving the cube.
- **Result:**
  - Ordering alone: about 99.5–99.7%.
  - Ordering + orientation: ordering 92.0–92.9%, orientation 82.1–93.1%.

## Eval 2 – Brain hemorrhage classification (CT)
- **Analogy:**
  - Engine put in a car and tuned.
  - Brand-new engine as the random-init baseline.
- **How:**
  - Fine-tuned encoder with a classification head.
  - 4-class classification (cause of hemorrhage).
  - Metric: accuracy. 80:20 random train/test split.
  - Fine-tuning hyperparameters: not stated.
- **Datasets:** A private/in-house dataset of 1,486 brain CTs. The same scans were used for pretraining, so the data was seen during pretraining (without labels). Internal only.
- **Competitors:**
  - Train from scratch.
  - Pretraining on UCF101 action-recognition videos.
  - Cube ordering only (ablation).
- **Fairness:**
  - All rows appear to be run by the authors on the same split.
  - There are no other medical SSL competitors (no Models Genesis, no Med3D) and no variance or repeat runs.
- **Result (accuracy):**
  - Rubik's Cube: 83.8%.
  - Ordering only: 81.1%.
  - UCF101: 75.3%.
  - Scratch: 72.6%.
  - That is +11.2 points over scratch.

## Eval 3 – BraTS-2018 brain tumour segmentation (MRI)
- **Analogy:** Engine put in a car and tuned, tried in two car bodies (two decoder designs).
- **How:**
  - The pretrained encoder is fine-tuned with either a standard 3D U-Net decoder or a "3D DUC" decoder (dense upsampling convolution). DUC has fewer random-init decoder parameters.
  - Metric: mIoU. 80:20 split of 285 volumes.
- **Datasets:** BraTS-2018. It was also the pretraining data (without labels). Internal only.
- **Competitors:** Same as Eval 2.
- **Result (mIoU %):**

  | Method | U-Net decoder | DUC decoder |
  |---|---|---|
  | Rubik's Cube | 76.2 | 77.3 |
  | UCF101 | 75.2 | 76.8 |
  | Ordering only | 73.9 | 75.0 |
  | Scratch | 73.3 | 74.0 |

  - The gains are modest: +2.9 (U-Net) and +3.3 (DUC) over scratch, and only +0.5 to +1.0 over UCF101 video pretraining.

## Eval 4 – Ablation (ordering vs ordering + orientation; decoder choice)
- **Analogy:** Which parts of the break-in routine matter.
- **Result:**
  - Adding the orientation task gives +2.3 to +2.7 points over ordering-only.
  - The DUC decoder adds about +1 point over the U-Net decoder.
  - There is no ablation on grid size or K in this paper.

## Key takeaways
- This is a very small-scale evaluation: two tasks, both on data the encoder was pretrained on, one random split each, and no error bars.
- The only competitors are scratch and video (UCF101) pretraining. There is no comparison with Models Genesis or Med3D in this paper.
- When later papers re-run it on their own setups, Rubik's Cube often lands close to scratch:
  - TransVW: LCS 75.59 vs scratch 79.76.
  - Rubik's Cube++: pancreas 61.07 vs scratch 58.58 at 10% labels.
  - This suggests its gains do not transfer well beyond its home datasets.
- Its successor is Rubik's Cube++ (same group), which turns the puzzle into a restoration (encoder-decoder) task so the decoder is pretrained too. TransVW, PCRL and Swin UNETR SSL papers use Rubik's Cube as a baseline.
