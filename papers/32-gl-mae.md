# Paper 32 – GL-MAE

- **Title:** Advancing Volumetric Medical Image Segmentation via Global-Local Masked Autoencoder (Zhuang et al., 2023 arXiv / 2024, AAAI per the copyright line)
- **Link:** https://arxiv.org/pdf/2306.08913
- **One line:** A 3D ViT encoder (ViT-B and ViT-T) pretrained with masked autoencoding (MAE) on 3D CT. It adds big "global" views next to small "local" crops, plus a consistency loss between the masked views and an unmasked global view (momentum teacher). Pretraining data: BTCV abdomen CT + TCIA COVID-19 CT (number of volumes not stated). Image-only.
- **Eval types used:** fine-tuned-segmentation, linear-probe, random-init-baseline, few-shot/label-efficiency, transfer-to-other-modality, external-validation, ablation, other:convergence-speed, other:qualitative-masks

---

## Eval 1 – Main segmentation fine-tuning (BTCV, MSD Spleen, MM-WHS)
- **Analogy:** Engine put in a car and tuned. The car body is UNETR.
- **How:** The ViT-B encoder is fine-tuned end-to-end with a UNETR decoder. Metric is Dice (%). Each dataset is run with 25%, 50% and 100% of the labels.
- **Datasets:** BTCV (abdomen CT, validation split). BTCV was **also in the pretraining data** (no labels used). MSD Spleen (CT). MM-WHS (heart). MM-WHS was not seen in pretraining.
- **Competitors:** Supervised 3D U-Net, SegResNet, UNETR (from scratch). SSL methods: Models Genesis, VicRegL, Swin UNETR SSL, JSSL, GVSL ("GLSV" in table), MAE3D.
- **Fairness:** The caption says checkpoints came from the official code and all methods used the same split. It does not clearly say whether every competitor was re-run. Some cells are "-", which hints that some numbers were copied or missing. The architectures differ (Swin vs ViT vs U-Net), so this is not one engine type compared fairly.
- **Result:** GL-MAE was best almost everywhere. At 100% labels: BTCV 82.33 vs MAE3D 81.74 vs UNETR scratch 79.61. MSD Spleen 95.72 vs 95.20 vs 94.20. MM-WHS 88.88 vs 86.03 vs 83.85. It was also best at 25% labels on every dataset. The gains over MAE3D are small (about 0.5 to 3 Dice).

## Eval 2 – Small model (ViT-T): "linear" vs end-to-end on BTCV
- **Analogy:** "Linear" here means the engine is sealed but a new decoder is trained on top. This is half test-bench, half car. End-to-end means the engine is tuned in the car.
- **How:** ViT-T. In the "linear" setting the encoder is frozen and the decoder is trained. In the other setting everything is fine-tuned. Metrics: Dice, NSD, HD95.
- **Datasets:** BTCV validation (seen during pretraining, no labels).
- **Competitors:** Supervised baseline (scratch) and MAE3D.
- **Fairness:** Same backbone and setup for all three. It looks like everything was re-run.
- **Result:** Frozen: Dice 80.22 vs MAE3D 78.36. End-to-end: Dice 81.02 vs 80.21 vs scratch 78.01. NSD is also higher. HD95: GL-MAE wins when frozen (13.03 vs 16.41). When end-to-end, the table shows MAE3D better (5.46 vs 7.53), even though the text claims GL-MAE wins. That is a mismatch in the paper.
- **Note:** This is not a true linear probe. A full decoder is trained on top of the sealed engine.

## Eval 3 – Unseen dataset (MM-WHS per-class)
- **Analogy:** The engine is tested on roads it never saw in pretraining (heart instead of abdomen). This is not a true hospital-A to hospital-B test, because it is still fine-tuned on MM-WHS.
- **How:** The fine-tuned model from Eval 1. Per-class Dice for 7 heart structures (shown as a figure).
- **Result:** Average Dice 88.88 vs MAE3D 86.03. The biggest gains were on aorta, LV and RV.

## Eval 4 – Cross-modality: CT-pretrained, fine-tuned on MRI (BraTS)
- **Analogy:** The engine was broken in on CT roads and then tuned for MRI roads.
- **How:** ViT-T, fine-tuned on BraTS brain tumour MRI. Dice for TC, WT and ET. The exact recipe is not described in detail.
- **Competitors:** Scratch baseline and MAE3D.
- **Result:** Average Dice 78.31 vs MAE3D 77.29 vs scratch 75.59. The gain is small but consistent.

## Eval 5 – COVID-19 lesion segmentation (supplement)
- **How:** Fine-tuned. Details are only in the supplement, which was not read.
- **Result:** Dice 49.88 vs baseline 47.92.

## Eval 6 – Label efficiency
- **Analogy:** Tune with only a handful of examples.
- **How:** BTCV with 25%, 50% and 100% of labels, under the "linear" (frozen encoder) setting (Fig. 4). The 25% and 50% columns of Table 1 also count here.
- **Result:** The text claims gains of 4.4% to 8.0% with limited labels. Exact numbers are only in a figure. The wording "semi-supervised" in the text conflicts with "linear" in the figure.

## Eval 7 – Ablation, convergence, mask ratio, scaling
- **Ablation (Table 4):** Each loss part is added one at a time, with frozen and end-to-end runs on BTCV and MM-WHS for ViT-B and ViT-T. The full model is best in most columns (e.g. MM-WHS ViT-B 87.89 vs plain MAE 86.58). The gains are **not monotonic**: BTCV ViT-T end-to-end drops to 78.70 when global reconstruction is added, then recovers. Single runs, with no error bars.
- **Convergence:** GL-MAE reaches good Dice in fewer pretraining epochs than MAE3D (figure only).
- **Mask ratio (supplement):** 0.6 was best.
- **Scaling (supplement):** BTCV Dice rose from 79.70 to 81.41 with more pretraining. "Larger data" is not clearly defined.
- **Qualitative:** BTCV masks compared against MAE3D and Swin UNETR.

## Key takeaways
- This is a classic SSL paper: almost everything is "engine put in a car (UNETR) and tuned", plus a frozen-encoder/trained-decoder variant. There is no kNN, no true linear probe, no text and no external hospital test.
- BTCV is both pretraining data and the main test set, so it is not an unseen benchmark. MM-WHS and BraTS are the "new road" tests.
- Margins over the closest relative MAE3D are small (often under 1 Dice on BTCV and Spleen). There are no seeds or significance tests, and the ablation is noisy.
- Competitors include Models Genesis, Swin UNETR SSL, JSSL, GVSL and MAE3D from this list. Mixed architectures make it a loose comparison, and it is unclear which numbers were re-run.
- It is a direct successor to MAE3D. It is close in spirit to SwinMM and Swin UNETR SSL (all BTCV + UNETR-style decoders).
