# Paper 29 – PCRLv2

- **Title:** A Unified Visual Information Preservation Framework for Self-supervised Pre-training in Medical Image Analysis (Hong-Yu Zhou et al., 2023, IEEE TPAMI; journal not named in the arXiv text, which is in IEEE journal format)
- **Link:** https://arxiv.org/pdf/2301.00772 (matches the model; no mismatch)
- **One line:** Non-skip U-Net ("nsUNet", ResNet-18 encoder in 2D, 3D U-Net-style encoder in 3D) pretrained with multi-scale pixel restoration + SimSiam-style feature comparison + "sub-crop" global/local views; pretrained on NIH ChestX-ray (112k), CheXpert (224k) and LUNA (888 CTs); image-only.
- **Eval types used:** fine-tuned-classification, fine-tuned-segmentation, random-init-baseline, few-shot/label-efficiency, transfer-to-other-modality, ablation, saliency, other:qualitative-segmentation-and-failure-cases

---

## Eval 1 – Semi-supervised chest X-ray classification (NIH ChestX-ray)
- **Analogy:** Engine put in a car and tuned, with only a handful of labeled examples (5%–50%, plus 100%).
- **How:** Fine-tuned full encoder; pretrain on unlabeled NIH part, fine-tune on labeled fraction. Metric: mean AUROC.
- **Datasets:** NIH ChestX-ray14 (internal; unlabeled part seen during pretraining).
- **Competitors:** Train from scratch (TS), ImageNet supervised (IN), Models Genesis (MG), TransVW, C2L, SimSiam, PCRLv1.
- **Fairness:** Not stated whether competitors were re-run or copied (many numbers match PCRLv1's paper, by the same authors, suggesting re-use of their own earlier runs).
- **Result:** At 5% labels PCRLv2 76.6 vs PCRLv1 74.1, SimSiam 71.7, TS 61.8. At 10%: 78.2 vs 76.2 (PCRLv1).

## Eval 2 – 3D lung nodule detection (LUNA, semi-supervised)
- **Analogy:** Engine put in a car and tuned with 10%–40% labels.
- **How:** Fine-tuned 3D encoder; metric AUROC.
- **Datasets:** LUNA (internal; seen unlabeled in pretraining).
- **Competitors:** TS, MG, TransVW, Rubik's Cube++, 3D-CPC, PCRLv1.
- **Fairness:** Not stated.
- **Result:** 10% labels: PCRLv2 85.5 vs PCRLv1 84.4, TS 78.4. 40%: 93.1 vs 92.2.

## Eval 3 – Transfer CheXpert → NIH ChestX-ray
- **Analogy:** Engine broken in on CheXpert, then put in a car and tuned on NIH.
- **How:** Fine-tuned; 10%–100% labels; AUROC.
- **Datasets:** Pretrain CheXpert; downstream NIH (not seen in pretraining).
- **Competitors:** TS, IN, MG, TransVW, C2L, SimSiam, PCRLv1.
- **Result:** 10%: 77.2 vs PCRLv1 75.8, TS 68.1. 100%: 84.0 vs 83.0, TS 80.9.

## Eval 4 – 3D segmentation transfer: LUNA (CT) → BraTS 2018 (MRI) and → LiTS (CT)
- **Analogy:** Engine put in a car and tuned for segmentation; BraTS = broken in on CT, driven on MRI.
- **How:** Fine-tuned encoder + decoder; 10%–40% and 100% labels; 70/10/20 split; Dice (BraTS: WT, TC, ET).
- **Datasets:** BraTS 2018 (351 MRI), LiTS (131 CT); neither seen in pretraining.
- **Competitors:** TS, MG, TransVW, Cube++, 3D-CPC, PCRLv1.
- **Result:** BraTS 10%: mean Dice 73.0 vs PCRLv1 71.6, TS 66.6 (ET 67.2 vs 65.2). LiTS 10%: 79.0 vs 77.3, TS 71.1; 100%: 94.5 vs 93.7, TS 90.7.

## Eval 5 – Ablations
- **Analogy:** Swap engine parts in and out.
- **Items:** Removing U-Net skip connections during pretraining helps (+1.2 NIH, +1.5 BraTS, +1.4 LiTS); pairwise vs crossed comparison (76.6 vs 76.0); restoration-only vs comparison-only vs both; number of scales (5 best); sub-crop vs multi-crop (multi-crop fails to train in 3D BraTS; sub-crop only marginally better in 2D). Also efficiency: dropping mixup roughly halves training time vs PCRLv1.

## Eval 6 – Visualizations
- **Analogy:** Heat-map of where the engine "looks".
- CAM saliency maps on chest X-rays; qualitative BraTS/LiTS segmentations; failure cases where high-confidence CAM falls outside the lung. No numbers.

## Key takeaways
- Same evaluation menu as PCRL (paper 28): full fine-tuning with label fractions and a from-scratch baseline on NIH, LUNA, LiTS, BraTS. No frozen probe, no external hospital test, no zero-shot.
- Gains over PCRLv1 are steady but small (about 1–2.5 points); gains over training from scratch are large at low labels.
- Not stated whether competitor numbers were re-run or reused; many baseline numbers equal those in PCRLv1, by the same authors.
- Cross-modality claim (CT pretraining → MRI BraTS) again relies on fine-tuning.
- Competitors are the "puzzle/restoration" family: Models Genesis, TransVW, Rubik's Cube++, PCRL. Later models like Swin UNETR SSL, SwinMM, GL-MAE and VoCo move to transformer backbones and bigger pretraining sets.
