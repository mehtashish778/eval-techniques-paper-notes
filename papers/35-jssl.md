# Paper 35 – JSSL

- **Title:** Joint Self-Supervised Image-Volume Representation Learning with Intra-Inter Contrastive Clustering (Nguyen et al., 2022 arXiv, AAAI 2023)
- **Link:** https://arxiv.org/pdf/2212.01893
- **One line:** A 2D ResNet-50 learns slice features (SwAV or DeepCluster-V2 clustering). A deformable-attention Transformer stacks the slice features into one volume feature. Trained with 2D clustering, 3D clustering and masked slice-feature prediction, in three stages. Pretraining data: "Universal" setting = LUNA16, LiTS, BraTS, MSD Heart, MOTS (about 2,460 CT/MRI volumes). "Unified" setting = LIDC-IDRI, RibFrac, TCIA-CT + 108,948 ChestX-ray8 X-rays. Image-only.
- **Eval types used:** fine-tuned-segmentation, fine-tuned-classification, random-init-baseline, few-shot/label-efficiency, transfer-to-other-modality, cross-task-transfer, ablation, other:detection

---

## Eval 1 – BraTS 2018 brain tumour segmentation (seen domain)
- **Analogy:** Engine put in a car and tuned. The car is a 2D U-Net run slice by slice.
- **How:** Fine-tuned. A 2D U-Net with the ResNet-50 slice engine. Slice predictions are stacked back into 3D. Metric: IoU. Averaged over 5 runs.
- **Datasets:** BraTS 2018 MRI (133/57/95). **Seen during pretraining** (Universal setting).
- **Competitors:** 3D: Models Genesis, TransVW, Universal Model, SwinViT3D (Swin UNETR SSL), 3D-SwAV, 3D-DeepCluster, scratch 3D, V-Net. 2D: ImageNet, MoCo-v2, SimCLR, SwAV, DC-V2.
- **Fairness:** Mixed. Older numbers were **copied** from earlier papers (Zhang et al. 2020, Xie et al. 2021). Newer ones (2D methods via VISSL, TransVW, SwinViT3D) were **re-run** by the authors using public weights. The architectures differ a lot (2D vs 3D, CNN vs Swin).
- **Result:** JSSL (SwAV) 73.03 IoU vs Universal Model 72.10 vs ImageNet 71.24 vs SwinViT3D 70.58 vs Models Genesis 67.96 vs scratch 3D 58.51. It wins by about 1 point over the best rival.

## Eval 2 – LUNA16 lung nodule classification (seen domain)
- **Analogy:** Engine put in a car and tuned, with a classification head.
- **How:** Fine-tuned. Two FC layers on the volume transformer output. Metric: AUC. Batch sizes 8/16/32, two trials each.
- **Competitors:** 3D-Rotation, V-Net, NiftyNet, scratch 3D, TransVW, Models Genesis, I3D, Med3D, 3D-JigSaw, 3D-DeepCluster, 3D-SwAV, SwinViT3D, 3D-Transformer.
- **Result:** JSSL **did not win**. 94.22 AUC, third place, behind 3D-Rotation 95.91 and V-Net 95.85. Scratch 3D got 94.15, about equal to JSSL. Med3D 91.32 and SwinViT3D 88.68 were lower. The authors admit the gains here are small.

## Eval 3 – MM-WHS left atrium segmentation (unseen domain, CT and MRI)
- **Analogy:** The engine is tuned on a new organ, on both CT and MRI roads.
- **How:** Fine-tuned. IoU. 13/3/4 train/val/test per modality. These are very small splits.
- **Competitors:** TransVW, Universal Model, SwinViT3D, 2D SwAV/DC-V2, scratch 3D.
- **Result:** CT 84.89 vs SwAV-2D 82.15 vs TransVW 79.74 vs scratch 68.29. MRI 78.73 vs SwinViT3D 78.25 (very close) vs scratch 67.04. A 4-case test set makes these numbers fragile.

## Eval 4 – VinDr-CXR abnormality detection (unseen, 2D X-ray)
- **Analogy:** The engine is put into a different kind of car (a detector: Faster R-CNN) and tuned.
- **How:** Fine-tuned Faster R-CNN with the ResNet-50. mAP@0.5. Three trials. The authors had to change the learning rate because their weights diverged at lr 0.001.
- **Competitors:** DC-V2, ImageNet, SwAV, MoCo-v2, Barlow Twins, SimCLR, scratch 2D.
- **Result:** JSSL (DC-V2) 28.47 vs DC-V2 alone 28.03 vs ImageNet 27.82 vs scratch 24.35. The gains are tiny (under 1 mAP). Label sweeps from 10% to 100%: 80% labels is about as good as 100%.

## Eval 5 – JSRT X-ray organ segmentation (Unified setting, label fractions)
- **Analogy:** Tune with only a handful of examples (20%, 40%, 100% labels).
- **How:** Fine-tuned. Dice.
- **Competitors:** USST (= UniMiSS), PCRL, DINO, MoCo-v3, PGL, MoCo-v2, ImageNet, scratch. These numbers match UniMiSS's table, so they were almost certainly **copied** from the UniMiSS paper.
- **Result:** At 100% JSSL wins (94.45 vs UniMiSS 94.08). At 20% and 40% **UniMiSS wins** (91.88 vs 90.60; 93.15 vs 93.03).

## Eval 6 – Ablation and model size
- **Ablation on MM-WHS:** Removing masked prediction, 3D clustering or multi-level features each costs about 1 to 3 IoU (full model CT 84.89 → 81.97 without 3D clustering).
- **Parameter count:** JSSL 31.2M vs SwinViT3D 62.2M vs UniMiSS 47.8M vs TransVW / Universal Model 19.7M.

## Key takeaways
- Every test is "engine put in a car and tuned". There is no frozen probe, no kNN, no external hospital test and no text.
- The comparison table mixes copied and re-run numbers and very different architectures (2D vs 3D, CNN vs Transformer). Treat the rankings as loose.
- It is honest about weak spots: it loses on LUNA16, is only marginal on VinDr-CXR, and loses to UniMiSS at low labels on JSRT.
- Some test sets are tiny (MM-WHS test = 4 cases per modality). BraTS and LUNA16 were also in the pretraining data.
- "Universal Model" here means Zhang et al.'s 2020 SSL model (the Models Genesis family lineage), **not** the CLIP-driven Universal Model. Competitors from this list: Models Genesis, TransVW, Swin UNETR SSL (SwinViT3D), Med3D, NiftyNet, UniMiSS. JSSL itself is later a GL-MAE competitor.
