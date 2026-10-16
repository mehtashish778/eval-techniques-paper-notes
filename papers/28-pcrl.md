# Paper 28 – PCRL (Preservational Contrastive Representation Learning)

- **Title:** Preservational Learning Improves Self-supervised Medical Image Models by Reconstructing Diverse Contexts (Hong-Yu Zhou, Chixiang Lu et al., 2021, ICCV 2021; venue not printed in the arXiv text)
- **Link:** https://arxiv.org/pdf/2109.04379 (matches the model; no mismatch)
- **One line:** U-Net with ResNet-18 encoder (2D) or 3D U-Net (3D), pretrained with contrastive learning (MoCo-style, momentum encoder + queue) PLUS image reconstruction (with "TransAtt" transform-conditioned attention and "CrossMix" feature mixup); pretrained on ChestX-ray14 (112k X-rays), CheXpert (224k X-rays) and LUNA (888 CTs); image-only.
- **Eval types used:** fine-tuned-classification, fine-tuned-segmentation, random-init-baseline, few-shot/label-efficiency, transfer-to-other-modality, linear-probe, ablation, other:natural-image-transfer (ImageNet→Cityscapes/COCO), other:qualitative-segmentation-figures

---

## Eval 1 – Pretext probes: does the engine "remember" the input? (ablation)
- **Analogy:** Engine sealed on a test bench; we only read a small gauge (one linear layer) to see if the engine output still knows rotation/position.
- **How:** Frozen encoder + a final FC layer trained to predict rotation and relative patch position on Chest14 features. Metric: accuracy.
- **Datasets:** ChestX-ray14 (same data as pretraining).
- **Competitors:** Variants of their own method only: plain contrastive loss, contrastive + self-reconstruction, + TransAtt, + CrossMix, full PCRL.
- **Fairness:** Same setup for all variants (internal ablation).
- **Result:** Full PCRL 93.2 / 91.6 (rotation / position acc) vs plain contrastive 85.5 / 82.3. Shows reconstruction keeps more information in the features.

## Eval 2 – 2D chest X-ray classification, semi-supervised (same dataset)
- **Analogy:** Engine put in a car and tuned, using only part of the labeled road (label efficiency).
- **How:** Fine-tuned (full encoder). Pretrain on the unlabeled part of ChestX-ray14, then fine-tune on labeled part; labeled:unlabeled ratios from 9.5:0.5 down to 6:4. Metric: mean AUC (14 diseases). Each setting run 3 times and averaged; p-values vs second-best method. Exact classification head not clearly stated.
- **Datasets:** ChestX-ray14 (internal; unlabeled part was seen in pretraining).
- **Competitors:** Train from scratch (TS), ImageNet pretraining (IN), Models Genesis (MG), Semantic Genesis (SG), Comparing to Learn (C2L).
- **Fairness:** Baselines use hyperparameters "following their official papers", which suggests they were re-implemented on the same splits; not stated explicitly whether any numbers were copied.
- **Result:** At 9:1, PCRL 76.2 AUC vs C2L 74.1, IN 73.6, TS 68.1. At 8:2, PCRL 78.8 vs C2L 76.4, TS 71.5. PCRL wins every ratio.

## Eval 3 – 2D transfer CheXpert → ChestX-ray14
- **Analogy:** Engine broken in on one road (CheXpert), then put in a car and tuned on another road (ChestX-ray14); a mild "tuned in A, driven in B" setup but for pretraining→downstream, not an external test.
- **How:** Fine-tuned with 10%–100% of ChestX-ray14 labels. Metric: AUC.
- **Datasets:** Pretrain CheXpert; fine-tune/test ChestX-ray14 (not seen in pretraining).
- **Competitors:** TS, IN, MG, SG, C2L.
- **Fairness:** Same as Eval 2.
- **Result:** At 100% labels, PCRL 83.1 vs C2L 81.5, IN 81.0, TS 80.9. Gains are larger at low label fractions.

## Eval 4 – 3D lung nodule detection/classification (LUNA → LUNA)
- **Analogy:** Engine put in a car and tuned, with limited labels.
- **How:** Fine-tuned 3D encoder; labeled:unlabeled ratios 9:1 down to 6:4. Metric: AUC.
- **Datasets:** LUNA (internal; unlabeled part seen in pretraining).
- **Competitors:** TS, MG, SG, 3D-CPC, Rubik's Cube++ (Cube++).
- **Fairness:** Same recipe described for all; reimplementation implied, not fully stated.
- **Result:** At 9:1 PCRL 84.4 vs 3D-CPC 80.2, TS 78.4; at 8:2 PCRL 87.5 vs 3D-CPC 85.2, TS 83.0.

## Eval 5 – 3D segmentation transfer: LUNA (CT) → LiTS (liver CT) and → BraTS (brain MRI)
- **Analogy:** Engine put in a car and tuned for segmentation; for BraTS the engine was broken in on CT and then driven on MRI ("transfer to other modality").
- **How:** Fine-tuned 3D U-Net (encoder + decoder); 10%–100% of labels. Metric: Dice.
- **Datasets:** LiTS (liver), BraTS (brain tumor MRI); neither seen in pretraining.
- **Competitors:** TS, MG, SG, 3D-CPC, Cube++.
- **Fairness:** Same as above.
- **Result:** LiTS: 10% labels PCRL 77.3 vs 3D-CPC 74.8, TS 71.1; 100% PCRL 93.7 vs 91.9 / 90.7. BraTS: 10% PCRL 71.6 vs 70.1 / 66.6; 100% PCRL 85.0 vs 82.9 / 81.5. Biggest gains at small label fractions.

## Eval 6 – Ablation of modules on Chest14
- **Analogy:** Swap engine parts in and out and see which part helps.
- **How:** Fine-tuned at 9:1 and 8:2 ratios; AUC. Compares contrastive only, + TransAtt, + CrossMix (alpha 0.5 vs 1), RotNet, self-reconstruction variants.
- **Result:** Full PCRL 76.2 / 78.8 vs contrastive only 71.7 / 74.8 (about 4 points gain).

## Eval 7 – Natural images (bonus)
- **Analogy:** Same engine-building recipe used for a regular road car (ImageNet), then tuned for city scenes.
- **How:** ResNet-50 pretrained on ImageNet-1k; fine-tuned on Cityscapes (segmentation, mIoU) and COCO (detection, mAP, Detectron2).
- **Competitors:** SimCLR, SwAV, MoCo v2.
- **Result:** Cityscapes 77.3 mIoU vs MoCo v2 76.3; COCO 41.3 mAP vs 40.5. Small gains.

## Eval 8 – Qualitative segmentation figures
- Random LiTS and BraTS cases shown visually; no numbers.

## Key takeaways
- Every medical result is "engine put in a car and tuned" (full fine-tuning) with a random-init baseline (TS) and label fractions; the only frozen readout is the pretext probe ablation (rotation/position), not a real disease task.
- Pretraining data is small by today's standards (888 LUNA CTs for 3D); the "semi-supervised" setting pretrains on the unlabeled part of the same dataset it is tested on, so it is not an external test.
- No external hospital test, no zero-shot, no frozen downstream probe; repeats (3 runs) and p-values are a plus.
- Competitor list is the classic self-supervised family: Models Genesis, Semantic Genesis, Rubik's Cube++, 3D-CPC, C2L — the same family that TransVW / ROT / PCRLv2 later compare against. PCRLv2 (paper 29) is its direct follow-up and reuses the same benchmarks (LUNA, LiTS, BraTS, NIH/CheXpert).
- CT→MRI transfer (LUNA→BraTS) shows some cross-modality transfer, but only with fine-tuning.
