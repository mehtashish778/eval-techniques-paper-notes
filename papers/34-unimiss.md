# Paper 34 – UniMiSS

- **Title:** UniMiSS: Universal Medical Self-Supervised Learning via Breaking Dimensionality Barrier (Xie et al., 2021 arXiv v1 / v2 Aug 2022, ECCV 2022; the venue is not printed in the fetched text)
- **Link:** https://arxiv.org/pdf/2112.09356
- **One line:** One Transformer ("MiT": pyramid U-shaped encoder + decoder) that takes **both 2D and 3D** images through a switchable 2D/3D patch embedding. Pretrained with DINO-style self-distillation plus a volume-vs-slice consistency loss. Pretraining data: 5,022 3D CT scans (MOTS, LIDC-IDRI, Tianchi, RibFrac, TCIA) + 108,948 2D chest X-rays (ChestX-ray8). Image-only.
- **Eval types used:** fine-tuned-segmentation, fine-tuned-classification, random-init-baseline, few-shot/label-efficiency, leaderboard, transfer-to-other-modality, ablation, other:qualitative-masks

---

## Eval 1 – 3D tasks: BCV segmentation + RICORD COVID classification
- **Analogy:** Engine put in a car and tuned, with less and less fuel (20%, 40%, 100% labels).
- **How:** Fine-tuned. Segmentation keeps the pretrained encoder + decoder and adds a segmentation head. Classification adds an FC layer on the encoder. Metrics: Dice and AUC. Each task has its own recipe (e.g. BCV batch 2, 25k iterations).
- **Datasets:** BCV abdomen CT (24 train / 6 offline test). RICORD COVID CT (182 / 45). Not stated as part of pretraining. MOTS contains abdominal CT but BCV is not listed.
- **Competitors:** CNN group (3D ResNet-50): random init, MoCo v2, PGL, PCRL. Transformer group (same U-like PVT): random init, MoCo v3, DINO.
- **Fairness:** Good. Competitors were **pretrained by the authors on the same 5,022 CTs**. However, the CNN competitors use a different backbone, so CNN vs Transformer is not fully apples-to-apples.
- **Result:** UniMiSS was best everywhere. BCV 77.96 / 80.97 / 84.99 Dice (20/40/100%) vs DINO 75.33 / 78.88 / 82.61 vs Transformer scratch 70.09 / 74.60 / 79.97. RICORD AUC 89.06 at 100% vs DINO 85.91. With 40% labels it beats scratch at 100% on BCV.

## Eval 2 – BCV online leaderboard
- **Analogy:** Leaderboard race against published cars.
- **How:** Fine-tuned model submitted to the BCV online test set (20 cases). Dice, HD, SD.
- **Competitors:** PaNN, UNETR, nnU-Net (10-model ensemble), DoDNet.
- **Fairness:** Competitor numbers were **copied from their papers**, and ensemble sizes differ (1 vs 5 vs 10 models).
- **Result:** A single UniMiSS got Dice 87.05 (best HD 13.92 and SD 1.02). The nnU-Net ensemble of 10 was higher at 87.62. A UniMiSS ensemble of 10 reached 88.11, the best.

## Eval 3 – 2D tasks: JSRT X-ray segmentation + ChestXR pneumonia classification
- **Analogy:** The same engine is put into a 2D car without re-building it.
- **How:** Fine-tuned, at 20/40/100% labels. The same UniMiSS weights are used directly.
- **Competitors:** Random init (CNN and Transformer), ImageNet pretraining, MoCo v2, PGL, PCRL, MoCo v3, DINO. All were pretrained on the same 2D X-rays (ResNet-50 for CNNs).
- **Result:** JSRT Dice 91.88 / 93.15 / 94.08 vs DINO 90.40 / 92.16 / 93.03. ChestXR AUC 97.09 / 98.14 / 99.07 vs DINO 96.44 / 97.69 / 98.70. The largest gains are at low labels. ChestXR is near the ceiling (~99 AUC), so the gaps are tiny.

## Eval 4 – Unseen modalities: ISIC skin (2D) + CHAOS MRI (3D)
- **Analogy:** An engine broken in on CT/X-ray roads, tuned for skin photos and MRI.
- **How:** Fine-tuned segmentation at 20/40/100%.
- **Competitors:** Random init, MoCo v3, DINO.
- **Result:** ISIC 86.67 vs DINO 86.21 vs scratch 85.07 (100%). CHAOS 91.36 vs 90.52 vs 88.38. Gains are small but consistent, and bigger at 20% labels (CHAOS 80.50 vs scratch 73.28).
- **Note:** The CHAOS split sizes differ between the paper's tables (48/12 vs 16/4).

## Eval 5 – Ablations
- **Volume-slice consistency:** +1.4 to +2.5 points on BCV and RICORD validation.
- **Switch interval between 2D and 3D batches:** every 2 steps is best.
- **Depth:** MiT-7 vs MiT-22. Deeper is better, and the gap between UniMiSS and single-dimension pretraining grows with depth.
- **Need for the switchable embedding:** a naive interpolation variant reached 76.65 vs 84.99 Dice with pretraining.
- These ablations are done on held-out validation splits, which is good practice.
- **Qualitative:** masks on BCV, CHAOS, JSRT and ISIC.

## Key takeaways
- Everything is "engine in a car and tuned" with label fractions. There is no frozen probe, no kNN, no external hospital test and no text.
- Fairness is above average: the authors re-pretrained MoCo, PGL, PCRL and DINO on the same data. The exception is the leaderboard table, which uses copied numbers and mixed ensemble sizes.
- The main claim is that adding cheap 2D X-rays helps 3D CT (and vice versa). This is shown mostly by beating 3D-only DINO, not by a direct "same model, with vs without 2D data" table in the main results (the depth figure partly covers it).
- It is a direct competitor or predecessor to JSSL (which copies UniMiSS's JSRT numbers as "USST") and to PCRL/PCRLv2. It uses DoDNet and nnU-Net as leaderboard rivals.
