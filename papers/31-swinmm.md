# Paper 31 – SwinMM

- **Title:** SwinMM: Masked Multi-view with Swin Transformers for 3D Medical Image Segmentation (Yiqing Wang et al., 2023, MICCAI 2023; venue not printed in the arXiv text)
- **Link:** https://arxiv.org/pdf/2307.12591 (matches the model; no mismatch)
- **One line:** Swin Transformer encoder (Swin UNETR style) pretrained on masked multi-view inputs (axial/coronal/sagittal views x 4 rotations) with four proxy tasks — masked reconstruction, rotation prediction, cross-view contrastive learning and cross-view "mutual learning" — then fine-tuned with a cross-view decoder (cross-attention at the bottleneck) and a KL multi-view consistency loss; pretrained on 5,833 CT volumes from 8 public datasets (AbdomenCT-1K, BTCV, MSD, TCIA-Covid19, WORD, TCIA-Colon, LIDC, HNSCC); image-only.
- **Eval types used:** fine-tuned-segmentation, few-shot/label-efficiency, random-init-baseline, ablation, transfer-to-other-modality, other:qualitative-segmentation-figures

---

## Eval 1 – WORD abdominal organ segmentation
- **Analogy:** Engine put in a car and tuned.
- **How:** Fine-tuned encoder + new cross-view decoder; loss Dice + CE + KL consistency; official WORD split, results on test set. Metrics: Dice and HD. Multi-view predictions aggregated at test time.
- **Datasets:** WORD (CT). Note: WORD is listed among the pretraining datasets; the paper does not say the test set was excluded (possible leakage).
- **Competitors:** Supervised: U-Net, Swin UNet, VT-UNet, UNETR, DeepLab V3+, ESPNet, DMFNet, LCOVNet, CoTr. Self-supervised: Swin UNETR (paper 30).
- **Fairness:** Not stated whether competitor numbers were re-run or copied (many look like the WORD benchmark paper's list). Not stated whether Swin UNETR was re-pretrained on the same 5,833 volumes. SwinMM also uses multi-view test-time aggregation, an extra advantage; authors partly address this with a "Swin UNETR-Fuse" ensemble (85.50%).
- **Result:** SwinMM 86.18% Dice / HD 9.35 vs Swin UNETR 84.34% / 14.24. Single-view Swin UNETR variants 84.32–84.97%.

## Eval 2 – ACDC cardiac segmentation (MRI)
- **Analogy:** Engine broken in on CT, put in a car and tuned on cardiac MRI.
- **How:** Fine-tuned; official split; Dice and HD.
- **Datasets:** ACDC (cardiac MRI; not in pretraining).
- **Competitors:** Same family as WORD (incl. Swin UNETR).
- **Fairness:** Same caveats as Eval 1.
- **Result:** SwinMM 90.80% Dice / HD 6.37 vs Swin UNETR 89.49% / 7.42.

## Eval 3 – Label efficiency on WORD (with semi-supervised pseudo-labels)
- **Analogy:** Tune with only a handful of examples; unlabeled scans get "pseudo-labels" agreed on by multiple views.
- **How:** Train with a fraction of WORD labels; multi-view aggregated pseudo-labels for unlabeled data. Dice.
- **Competitors:** Swin UNETR.
- **Result:** 10% labels: SwinMM 67.83% vs Swin UNETR 56.14% (about +11.7). 100%: 84.78% vs 83.13%. No label-fraction study on ACDC.

## Eval 4 – Ablations (smaller 2,639-volume pretraining set)
- **Analogy:** Swap engine-building steps and car-body parts.
- **Proxy tasks:** no pretraining 84.78% / HD 11.77 (this is the random-init baseline) → all four tasks 85.74% / 9.56; mutual learning gives the largest single gain (about 1 Dice point).
- **Masking ratio:** 0.25 → 85.54, 0.50 → 85.74 (best), 0.75 → 85.43.
- **Fine-tuning parts:** without KL consistency and cross-attention 83.72% / 16.24 → with both 84.78% / 11.77.
- Ablation numbers differ from headline numbers because of the smaller pretraining set and different test settings.

## Eval 5 – Visual results
- Qualitative segmentation figures; no numbers.

## Key takeaways
- Narrow eval set: two segmentation datasets (WORD, ACDC), all fine-tuned; no classification, no frozen probe, no external-hospital test, no robustness test.
- Main comparison target is Swin UNETR SSL (paper 30); the gain mixes three things — better pretraining, a new decoder, and multi-view test-time aggregation — so the pretraining contribution alone is mostly shown only by the ablation (+~1 Dice).
- Possible leakage: WORD (and BTCV) are in the pretraining pool and WORD is a test set; the paper does not discuss test-set exclusion.
- Fairness of competitor numbers (re-run vs copied) and whether Swin UNETR got the same pretraining data are not stated.
- Strongest benefit is at 10% labels (+~12 Dice vs Swin UNETR), the usual pattern also seen in PCRL, PCRLv2 and Swin UNETR SSL.
