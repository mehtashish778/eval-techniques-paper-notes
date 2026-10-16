# Paper 10 – Models Genesis

- **Title:** Models Genesis (Zhou et al., 2021 journal version / arXiv v4 Dec 2020, Medical Image Analysis; earlier MICCAI 2019 conference version)
- **Link:** https://arxiv.org/pdf/2004.07882v4
- **One line:** A 3D U-Net (~16.3M params; 2D variants too) pretrained with self-supervised **image restoration**. The model must undo four distortions: non-linear intensity curves, local pixel shuffling, outer-cutout and inner-cutout. Loss: MSE on 64×64×32 sub-volumes. Pretraining data: 623 LUNA16 chest CT scans ("Genesis Chest CT"), plus 2D versions from LUNA16 slices and ChestX-ray14. Image-only, no labels.
- **Eval types used:** fine-tuned-classification, fine-tuned-segmentation, random-init-baseline, few-shot/label-efficiency, transfer-to-other-modality, cross-task-transfer, ablation, leaderboard, other:convergence-speed, other:2D-vs-3D-comparison

---

## Eval 1 – Five 3D target tasks, fine-tuned (main table)
- **Analogy:** Engine put in a car and tuned. Random init = brand-new vs broken-in engine.
- **How:**
  - Classification tasks use the encoder plus a head; segmentation tasks use the full encoder-decoder.
  - All layers are fine-tuned. A frozen setting is described as possible but not reported.
  - Each method was run 10 times; reported as mean ± s.d. with t-tests (p=0.05).
  - Metrics: AUC for classification, IoU (and Dice) for segmentation.
- **Datasets:**
  - NCC: lung nodule false-positive reduction (LUNA16, AUC). Same modality and organ as pretraining, but the test scans were held out from pretraining.
  - NCS: lung nodule segmentation (LIDC-IDRI, IoU). Same organ.
  - ECC: pulmonary embolism false-positive reduction (121 CTPA scans, AUC). Different disease, contrast CT.
  - LCS: liver segmentation (LiTS 2017, IoU). Different organ: cross-task transfer.
  - BMS: brain tumour segmentation (BraTS 2018 Flair MRI, 3-fold CV, IoU). Different modality: transfer to MRI.
- **Competitors:**
  - 3 random inits (uniform, Xavier, MSRA/He).
  - Supervised 3D pretrained: I3D, NiftyNet, MedicalNet (= Med3D).
  - Self-supervised, re-implemented in 3D: de-noising, in-painting, Jigsaw, DeepCluster, patch shuffling, Rubik's Cube.
- **Fairness:** The SSL methods were re-implemented in 3D by the authors on the same data and splits. I3D, NiftyNet and MedicalNet were used via their released weights, fine-tuned on the same tasks. The architectures differ (MedicalNet ResNet-101 is 85.8M params, I3D 25.4M, NiftyNet 2.6M, Genesis 16.3M), so this is not a pure same-backbone test. MedicalNet was supervised **on LiTS itself**, which gives it an advantage on LCS.
- **Result:** Genesis was best or tied on 4 of 5 tasks, mostly by small margins:
  - NCC: 98.34 vs I3D 98.26.
  - NCS: 77.62 vs MedicalNet 75.68.
  - ECC: 87.20 vs MedicalNet 86.43.
  - BMS: 67.96 vs I3D 67.83.
  - LCS: 85.10 vs MedicalNet 85.52 (statistically comparable).
  - Table 4 gives a different ECC number (88.04), which is an inconsistency.

## Eval 2 – Label efficiency
- **Analogy:** Tune with only a handful of examples.
- **How:** Fine-tune with growing fractions of labels and compare against training from scratch.
- **Datasets:** the same 5 tasks.
- **Competitors:** training from scratch.
- **Fairness:** same setup.
- **Result:** Genesis matched full-data scratch performance with fewer labels. Reported annotation savings: NCC 30%, NCS 50%, ECC 57%, LCS 84%, BMS 44% ("at least 30%"). Some of the text's worked examples do not exactly match these numbers.

## Eval 3 – 3D Genesis vs 2D ImageNet transfer
- **Analogy:** A purpose-built 3D engine vs a borrowed 2D engine (ImageNet) on the same roads.
- **How:** 2D slice or 2.5D ImageNet fine-tuning vs 2D Genesis vs 3D Genesis, all fine-tuned.
- **Datasets:** the same tasks.
- **Competitors:** ImageNet-pretrained 2D models, 2D/3D scratch.
- **Fairness:** Re-run by the authors.
- **Result:** 3D Genesis beat 2D/2.5D ImageNet. 3D from scratch did *not* reliably beat 2D, so 3D pretraining is what makes 3D worth it.

## Eval 4 – Ablations
- **Distortions:** Each of the four alone vs combined. Combined was best on all 5 tasks. Plain identity (auto-encoder style) was worse than random init on 3 tasks.
- **Init stability:** Genesis gave higher and more stable results than the 3 random inits.
- **Loss:** MSE vs MAE vs SSIM gave mixed results, with no clear winner.
- **Convergence:** Genesis fine-tunes over 13% faster.

## Eval 5 – Leaderboard / literature comparisons (footnotes)
- **Analogy:** Comparing the tuned car's lap time with published race records.
- **How:** Numbers copied from challenge winners or prior papers.
- **Result:**
  - LUNA16 NCC: AUC 0.971 vs winner 0.968.
  - LiTS: Dice 93.19 without post-processing vs winner 96.60 with post-processing (Genesis loses).
  - ECC: 87.20 vs 87.06 prior.
  - NCS: Dice 75.86 vs 74.05 prior.
  - BraTS: not submitted, since the paper used Flair only.

## Key takeaways
- This is the "grandfather" of restoration-based SSL in this list. Later models (TransVW, Rubik's Cube++, PCRL/PCRLv2, Swin UNETR SSL, VoCo, SuPreM, PASTA) all use it as a baseline. In the PASTA paper, Models Genesis is a fine-tuned baseline and is clearly beaten.
- Every evaluation is "engine put in a car and tuned". There is no frozen-engine test (kNN or linear probe), unlike newer papers such as TAP-CT, CT-FM or MedImageInsight.
- Strong statistics for its era: 10 runs per method, t-tests, mean ± s.d. Competitor SSL methods were re-implemented fairly. The supervised baselines (MedicalNet/Med3D, NiftyNet, I3D) differ in size and pretraining data; MedicalNet even saw LiTS labels.
- Margins over the best baseline are small (often under 1 point). The real story is label efficiency and transfer to MRI and new organs from only 623 chest CTs.
- Pretraining data is tiny by today's standards (hundreds of scans vs 100K+ for TAP-CT, CT-FM or SPECTRE).

