# Paper 05 – VoCo

- **Title:** VoCo: A Simple-yet-Effective Volume Contrastive Learning Framework for 3D Medical Image Analysis (Wu et al., 2024, CVPR 2024; the venue is not printed in the arXiv text read)
- **Link:** https://arxiv.org/pdf/2402.17300
- **One line:** Swin-UNETR (also a 3D U-Net variant) pretrained with self-supervised "contextual position prediction". Random crops are matched to a grid of base crops, and the model predicts their overlap. Pretrained on about 1.6k unlabelled CT scans (BTCV, TCIA Covid-19, LUNA); image-only.
- **Eval types used:** fine-tuned-segmentation, fine-tuned-classification, random-init-baseline, transfer-to-other-modality, leaderboard, ablation, other:2D-chest-X-ray-pretrain-and-finetune

---

## Eval 1 – BTCV abdominal organ segmentation (13 organs)
- **Analogy:** "Engine put in a car and tuned", plus a random-init baseline (broken-in vs brand-new engine).
- **How:** Full fine-tuning of Swin-UNETR in a MONAI/PyTorch setup, sliding-window inference. Metric: average Dice.
- **Datasets:** BTCV. Seen during pretraining: for this task the model was pretrained on BTCV + TCIA Covid-19 (~0.8k scans), so the images, but not the labels, were in pretraining.
- **Competitors:** Scratch UNETR and Swin-UNETR; MAE3D, SimCLR, SimMIM, MoCo v3, Jigsaw, PositionLabel, Models Genesis, ROT, VicRegL, Rubik's Cube++, PCRL, PCRLv2, Swin UNETR SSL, SwinMM, GL-MAE, GVSL.
- **Fairness:** The caption says most competitor numbers were copied from earlier papers, and a few (marked †) were re-implemented. Backbones differ across rows.
- **Result:** VoCo 83.85% vs scratch Swin-UNETR 80.53% (+3.32) and the best prior, GL-MAE, 82.01% (+1.84).

## Eval 2 – LiTS liver, MSD Spleen, MM-WHS heart (segmentation)
- **Analogy:** Engine put in a car and tuned, on more roads.
- **How:** Full fine-tuning, Dice. Pretraining used all three datasets (~1.6k CT).
- **Datasets:** LiTS, MSD Spleen and MM-WHS. None were in pretraining.
- **Competitors:** Similar lists, adding TransVW, Rubik's Cube, UniMiSS and JSSL; also a VoCo-pretrained 3D U-Net.
- **Fairness:** Mostly copied numbers, with some re-implemented (†).
- **Result:**
  - LiTS: 96.52 vs scratch Swin 93.42; best prior Rubik++† 95.72.
  - Spleen: 96.34 vs GVSL 95.47.
  - MM-WHS: 90.54 vs GVSL 88.27.

## Eval 3 – BraTS 21 brain tumour on MRI (cross-modality)
- **Analogy:** An engine broken in on CT, swapped into an MRI car and tuned ("transfer to another modality").
- **How:** Full fine-tuning, Dice (TC, WT, ET, average).
- **Competitors:** Scratch, MAE3D, SimCLR, SimMIM, MoCo v3, Jigsaw, PositionLabel, PCRL, PCRLv2, Rubik's Cube++, Swin UNETR SSL, SwinMM.
- **Fairness:** Mostly copied numbers.
- **Result:** 78.53 average vs scratch Swin 75.89 and the best prior, SimMIM, 77.85 (a small gap).

## Eval 4 – CC-CCII COVID-19 classification
- **Analogy:** Engine put in a car and tuned, for a classification car.
- **How:** Fine-tuned encoder with a classification head, accuracy.
- **Competitors:** Scratch UNETR and Swin, MAE3D, MoCo v3, Jigsaw, PositionLabel, PCRL, PCRLv2, Rubik's Cube++, Swin UNETR SSL, SwinMM.
- **Fairness:** Here the authors re-ran all baselines themselves, because no earlier paper reported this dataset.
- **Result:** 90.83% vs the best prior, SwinMM, 89.61%, and scratch Swin 88.04%.

## Eval 5 – NIH ChestX-ray (2D, appendix)
- **Analogy:** Same break-in recipe on a different (2D) engine.
- **How:** 3D U-Net pretrained and fine-tuned on NIH ChestX-ray (60% used for pretraining).
- **Competitors:** PCRLv2, PCRL, SimSiam, C2L, TransVW, Models Genesis, scratch.
- **Result:** 82.02 vs PCRLv2 81.5 and scratch 75.4.

## Eval 6 – Leaderboards and MSD (appendix)
- **Analogy:** Race on public tracks.
- **How:** Online test leaderboards, no ensembling; offline MSD validation for 10 tasks.
- **Result:**
  - BTCV test: 86.44 vs SwinUNETR 84.72.
  - FLARE23: 90.07 vs 87.84.
  - AMOS22: 89.06 / 78.86 (DSC/NSD) vs 88.00 / 76.15.
  - VoCo was ahead of SwinUNETR on all 10 MSD tasks (offline).

## Eval 7 – Ablations
- **Analogy:** Change one part of the break-in recipe at a time.
- **How:** BTCV and MM-WHS Dice.
- **Result:**
  - Losses: baseline 80.53, prediction loss only 82.96, plus regularisation loss 83.85 (BTCV).
  - Number of bases: 4×4×1 was best (83.85); 5×5×1 gave 83.60.
  - λ = 1.0 was best.

## Key takeaways
- Every downstream evaluation fine-tunes the engine. There is no frozen or linear-probe test, so we do not learn how good the raw features are.
- Most competitor numbers are copied from other papers, which differ in backbone, budget and recipe. CC-CCII is the only task where all baselines were re-run, so the gaps of about 1–2 Dice points should be read with care.
- The pretraining data is small (~1.6k CT) and overlaps a downstream set: BTCV images were used for pretraining before the BTCV fine-tuning.
- There is no label-efficiency or few-shot study (the authors list semi-supervised learning as future work), and no external-hospital test.
- It is a direct successor to Swin UNETR SSL, SwinMM, GL-MAE, GVSL, PCRLv2 and Rubik's Cube++, compared on the same BTCV and MSD-style tables. SuPreM (paper 06) argues that supervised pretraining beats this kind of self-supervised approach.
