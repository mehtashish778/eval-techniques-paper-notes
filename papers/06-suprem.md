# Paper 06 – SuPreM

> **Link mismatch:** the given URL (arxiv.org/pdf/2307.07471) is a physics paper ("Spectral Network Principle for Frequency Synchronization in Repulsive Laser Networks"), not SuPreM. The correct paper was found at arXiv 2501.11253 and in the ICLR 2024 PDF on the author's site.

- **Title:** How Well Do Supervised 3D Models Transfer to Medical Imaging Tasks? (Li et al., 2024, ICLR 2024)
- **Link:** https://arxiv.org/html/2501.11253v1 and https://www.cs.jhu.edu/~zongwei/publication/li2023suprem.pdf
- **One line:** U-Net, Swin UNETR and SegResNet backbones pretrained with *supervised* segmentation on AbdomenAtlas 1.1: 9,262 CTs from 88 hospitals with per-voxel labels for 25 structures plus pseudo-labels for 7 tumour types. Benchmarks use 2,100 of these CTs for pretraining. Image-only (labels, no text).
- **Eval types used:** fine-tuned-segmentation, fine-tuned-classification, random-init-baseline, few-shot/label-efficiency, external-validation, cross-task-transfer, human/radiologist-rating, ablation, other:data-and-compute-scaling

---

## Eval 1 – Transfer to TotalSegmentator and a private JHH dataset (main benchmark)
- **Analogy:** "Engine put in a car and tuned", compared against other engines and a brand-new engine (random-init baseline).
- **How:** Full fine-tuning in MONAI (up to 200 epochs, 20 warm-up epochs, 1 GPU). Whether any layers were frozen is not stated. Metric: Dice, mean ± s.d. over 10 runs for SuPreM and scratch.
- **Datasets:**
  - TotalSegmentator v1 (organ, muscle and cardiac groups). Public; some of its organ classes overlap with the pretraining labels.
  - A proprietary JHH dataset (organs, gastro, cardiac).
- **Competitors:**
  - U-Net family: Models Genesis, Med3D, UniMiSS, DoDNet.
  - Swin UNETR family: Swin UNETR SSL (Tang 2022, 5,050 CT), Universal Model (Liu 2023).
  - Scratch for each backbone.
- **Fairness:** The authors fine-tuned the released checkpoints themselves on the same data. Backbones differ, though: older models used plain U-Nets, and the authors note their MONAI U-Net is stronger even from scratch. Competitors were run once, with no s.d., while SuPreM got 10 runs.
- **Result (TotalSegmentator organs):**
  - U-Net: scratch 88.9 → SuPreM 92.1 (Models Genesis 87.8, Med3D 86.9, UniMiSS 88.5, DoDNet 89.3).
  - Swin UNETR: scratch 86.4, SSL 89.3, Universal Model 89.7, SuPreM 91.3.
  - Private cardiac task with U-Net: 38.1 → 70.5.

## Eval 2 – Few-shot transfer (Figure 1)
- **Analogy:** "Tune with only a handful of examples."
- **How:** Fine-tune with N = 5, 10 or 20 CTs; average Dice over 20 organs and 3 tumour types.
- **Datasets:** Proprietary JHH.
- **Competitors:** Public supervised and self-supervised models.
- **Result:** Supervised models beat self-supervised ones, and SuPreM was best. Exact numbers are not stated in the text (figure only).

## Eval 3 – Data, annotation and compute efficiency (scaling)
- **Analogy:** How long must the engine be broken in? Labelled break-in versus long unlabelled break-in.
- **How:** Vary the number of pretraining CTs and the fine-tuning label budget.
- **Datasets:** TotalSegmentator.
- **Competitors:** Swin UNETR SSL (Tang 2022).
- **Result:**
  - 21 labelled CTs (40 GPU-hours) gave 90.4 Dice, vs 89.7–89.9 for SSL pretrained on 1,000–5,050 CTs (1,152 GPU-hours).
  - Scaling supervised pretraining to 1,575 CTs gave 91.3.
  - SuPreM fine-tuned on 512 volumes ≈ SSL fine-tuned on 1,024 (about 50% fewer labels).
  - Fine-tuning time on a 10% subset fell from 60 to 20 GPU-hours.

## Eval 4 – Direct inference on external data (no fine-tuning)
- **Analogy:** "Tuned in hospital A, driven in hospital B", with no extra tuning at all.
- **How:** SuPreM trained on all 9,262 CTs, applied as-is. Dice compared with models trained in-distribution on each target set.
- **Datasets:**
  - TotalSegmentator (Switzerland).
  - DAP Atlas (Germany).
  - Proprietary JHH (US).
- **Competitors:** In-distribution models: Liu et al., Jaus et al., Wang et al.
- **Fairness:** Numbers appear to come from those models. How they were obtained is not stated.
- **Result:**
  - TotalSegmentator: 87.7 vs 87.6 (not significant).
  - DAP Atlas: 93.3 vs 93.6 (slightly worse, and significant).
  - JHH: 86.7 vs 86.1 (not significant).

## Eval 5 – New classes and tumours (Table 4)
- **Analogy:** Engine tuned for parts it never saw during break-in ("cross-task transfer").
- **How:** Fine-tune on 66 novel classes (the text says 63): 19 muscles, 15 cardiac, 5 organs, 24 vertebrae, 3 pancreatic tumour types. Dice.
- **Competitors:** Self-supervised (Swin UNETR SSL) vs supervised.
- **Result:**
  - Supervised won everywhere (P ≤ 0.0001 for most).
  - Vertebrae: 89.2 vs 86.4.
  - Tumours: 49.7 vs 43.4.
  - Cyst: 49.4 vs 41.5.
  - PanNET: 46.0 vs 35.5.
  - PDAC: 53.6 vs 53.3 (nearly tied).

## Eval 6 – Pancreatic tumour subtype classification (private)
- **Analogy:** Engine tuned in a classification car, then checked against radiologists.
- **How:** Fine-tuned classification of PDAC, cyst or PanNET. Data: 1,869 train / 1,073 test CTs, 3,577 tumours. Metrics: ROC/AUC, sensitivity and specificity.
- **Competitors:** Swin UNETR SSL. Radiologist numbers were taken from another paper (Cao et al. 2023).
- **Fairness:** The radiologist comparison is copied from a different study, not a reader study on the same cases.
- **Result:**
  - PDAC sensitivity 86.1% and specificity 95.4%, which the authors claim is +27.6 / +4.4 points over average radiologists.
  - AUC was higher than SSL for all 3 subtypes. Exact AUCs are not stated (figure only).

## Key takeaways
- The paper is mainly an argument: supervised pretraining on a large labelled CT atlas transfers better, and more cheaply, than self-supervised pretraining (Swin UNETR SSL, Models Genesis, UniMiSS).
- Every test fine-tunes the engine. There is no frozen or linear-probe test, so feature quality alone is never measured.
- The authors re-fine-tuned competitor checkpoints, which is better than copying numbers. But backbones and run counts differ, and the gains over strong baselines are about 1.5–2 Dice points.
- Much of the evidence uses private JHH data, which others cannot reproduce. TotalSegmentator overlaps AbdomenAtlas classes, so "transfer" there is partly to familiar classes.
- "Direct inference" is a true external test and shows SuPreM matching in-distribution models without any tuning.
- It sits in the line from Universal Model and DoDNet (same JHH group). Compare it with VoCo and Swin UNETR SSL (self-supervised) and with VISTA3D and TotalSegmentator (other big supervised CT segmenters).
