# Paper 25 – 3D Rotation (ROT), within "3D Self-Supervised Methods for Medical Imaging"

- **Title:** 3D Self-Supervised Methods for Medical Imaging (Taleb et al., 2020, NeurIPS 2020; the venue is not printed in the arXiv text, so it comes from outside knowledge)
- **Link:** https://arxiv.org/pdf/2006.03829
- **Note:** This paper is not "a ROT model". It proposes and benchmarks five 3D pretext tasks:
  - 3D Contrastive Predictive Coding (CPC)
  - Relative Patch Location (RPL)
  - 3D Jigsaw
  - **3D Rotation prediction (ROT)**, which predicts multiples of 90°
  - 3D Exemplar networks

  "ROT" is one of these five. When other papers (e.g. Swin UNETR, PCRL) list "ROT" as a baseline, they usually cite this paper or the 2D rotation paper.
- **One line:**
  - **Encoder:** a 3D U-Net-style residual encoder (5 levels, 32–512 filters), pretrained with each pretext task separately.
  - **3D pretraining data:** about 22K UK Biobank brain MRIs (T1 / T2-FLAIR) and unlabelled pancreas CT from the Decathlon.
  - **2D:** DenseNet-121 on 170K UK Biobank fundus images.
  - Image-only.
- **Eval types used:** fine-tuned-segmentation, fine-tuned-classification, random-init-baseline, few-shot/label-efficiency, cross-task-transfer, other:convergence-speed, other:2D-vs-3D-comparison, other:supervised-pretraining-baseline, leaderboard (comparison with BraTS challenge entries)

---

## Eval 1 – BraTS 2018 brain tumour segmentation (full data)
- **Analogy:**
  - Engine put in a car and tuned.
  - Brand-new engine as the random-init baseline.
- **How:**
  - Fine-tuned, with a hybrid schedule: the encoder is frozen for 25 warm-up epochs, then the whole model is trained.
  - U-Net decoder.
  - Metric: Dice for enhancing tumour (ET), whole tumour (WT) and tumour core (TC).
  - The authors say the hybrid schedule beat full fine-tuning and frozen-only training, but they give no numbers for those alternatives.
- **Datasets:** BraTS 2018 (285 train / 66 val). Pretraining used UK Biobank brain MRI, so the modality is the same but the dataset is different.
- **Competitors:**
  - From scratch.
  - 3D supervised pretraining on FSL-FAST tissue labels.
  - The 2D versions of all five tasks.
  - BraTS challenge entries: Popli, Baid, Chandra, Isensee.
- **Fairness:**
  - Scratch, supervised and 2D baselines appear to be run by the authors with the same recipe.
  - The BraTS challenge numbers seem copied from those papers. They use different architectures and recipes, and Isensee is an nnU-Net-style ensemble pipeline.
  - The paper does not state that all methods used identical splits.
- **Result (Dice %, ET / WT / TC):**

  | Method | ET | WT | TC |
  |---|---|---|---|
  | **3D-Rotation** | 80.21 | 89.63 | 84.75 |
  | 3D-RPL (best overall) | 81.28 | 90.71 | 86.12 |
  | 3D-CPC | 80.83 | 89.88 | 85.11 |
  | 3D-Exemplar | 79.46 | 90.80 | 83.87 |
  | 3D-Jigsaw | 79.66 | 89.20 | 82.52 |
  | Scratch | 76.38 | 87.82 | 83.11 |
  | 3D supervised | 78.88 | 90.11 | 84.92 |
  | Isensee | 80.36 | 90.80 | 84.32 |
  | 2D-Rotation | 76.60 | 88.78 | 82.41 |

  - ROT is mid-pack among the 3D methods. It is clearly better than scratch and than its 2D version.

## Eval 2 – Label efficiency (BraTS and pancreas)
- **Analogy:** Tune with only a handful of examples.
- **How:**
  - Fine-tune using random subsets of the labels: BraTS 10 / 25 / 50 / 100%, pancreas 5 / 10 / 50 / 100%.
  - Metric: Dice.
  - Pancreas task: Decathlon pancreas + tumour segmentation (420 cases). The pretraining used pancreas CT from the same source, without labels.
- **Result:**
  - Figures only; no pancreas Dice numbers are given in the text.
  - The 3D methods give their biggest gains at low label fractions and beat their 2D counterparts and scratch.

## Eval 3 – Convergence speed (pancreas, fundus)
- **Analogy:** How fast the tuned engine gets up to speed.
- **Result:** Pretrained models reach much higher Dice within the first ~20 epochs. Figures only.

## Eval 4 – 2D diabetic retinopathy (fundus)
- **Analogy:** The same break-in routines on a different (2D) engine and car.
- **How:**
  - DenseNet-121 pretrained on UK Biobank fundus images.
  - Fine-tuned on the Kaggle 2019 diabetic retinopathy set (~5.6K images, 5-fold cross-validation).
  - Metric: Quadratic Weighted Kappa.
  - Includes low-label runs, and compares pretraining on UK Biobank with pretraining on the Kaggle set itself.
- **Result:**
  - Figures only; no kappa values are stated in the text.
  - Pretraining on the larger UK Biobank set beat pretraining on the target set.

## Key takeaways
- This is a benchmark of pretext tasks, not a single model. "ROT" is just one row, and here it is not the winner (RPL and CPC do better).
- The evaluation is all "engine put in a car and tuned". There are no frozen/linear-probe numbers, no external validation and no multi-organ CT. Only BraTS has numeric tables; pancreas and fundus results are shown only as plots.
- Comparing against BraTS challenge entries mixes in copied numbers from very different pipelines, so it is a loose leaderboard-style comparison.
- Rubik's Cube and Models Genesis are discussed but not benchmarked numerically. Later papers (TransVW, PCRL, Swin UNETR SSL) use "Rotation" as a 3D baseline in their own tables.
- An unusual detail is the hybrid schedule: the engine is sealed for the first epochs and then tuned.
