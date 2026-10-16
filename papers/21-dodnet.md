# Paper 21 – DoDNet

- **Title:** DoDNet: Learning to segment multi-organ and tumors from multiple partially labeled datasets (Zhang, Xie et al., 2020 arXiv / 2021; CVPR 2021, a venue not printed in the fetched text)
- **Link:** https://arxiv.org/pdf/2011.10217 (matches the model name)
- **One line:** A 3D residual U-Net-like encoder-decoder (about 17.3M parameters) with a "dynamic head". A small controller takes a one-hot task code plus pooled image features and generates the weights (162 numbers) of three 1×1×1 conv layers, so a single network serves many tasks. It is trained with supervision on MOTS, a partially labeled set the authors built: 1,155 abdominal CTs (920 train / 235 test) from LiTS, KiTS and 5 MSD tasks. It is image-only, and the "pretraining" is supervised, not self-supervised.
- **Eval types used:** fine-tuned-segmentation, random-init-baseline, leaderboard, ablation, other:multi-task-partial-label-segmentation, other:efficiency

---

## Eval 1 – Multi-organ and tumor segmentation on the MOTS test set
- **Analogy:** One engine with a quick-change gearbox. You tell it which task (liver, kidney...) and it re-wires its last few gears.
- **How:** Fully trained (no frozen parts). Metrics are Dice and Hausdorff distance (HD) for organ and tumor in 7 tasks (11 categories). Testing uses a sliding window of 64×192×192.
- **Datasets:** MOTS test split (235 scans). It comes from the same sources as training, so this is internal testing, not external.
- **Competitors:** Multi-Nets (7 separate networks), TAL, Multi-Head, Cond-NO (no task condition), Cond-Input, Cond-Dec.
- **Fairness:** Good. All competitors were re-implemented on the **same backbone with the same settings**. The one exception is that Multi-Head's decoder channels were halved to fit GPU memory, a small handicap for it.
- **Result:** DoDNet has the best average: 75.64% Dice and 19.50 HD. For comparison, Multi-Head gets 74.55 / 26.22, TAL 73.35 / 23.56 and Multi-Nets 71.67 / 28.95. Gains are small (about 1 Dice point over the best competitor). It is not best on every task; for example, kidney tumor is 77.59 vs Multi-Head 79.16.

## Eval 2 – Ablations of the dynamic head
- **Analogy:** Trying different gearbox sizes and inputs.
- **How:** Uses a validation split (20% of the training scans). Metrics are Dice and HD.
- **Result:** Head depth 3 and width 8 are best, but the differences are tiny (71.3–71.7 Dice). Task encoding is what matters: image features alone collapse to 51.8 Dice, task code alone gives 71.26, and both together give 71.67.

## Eval 3 – Efficiency
- **How:** Parameter count and an inference speed vs Dice plot.
- **Result:** About 17.3M parameters. Multi-Nets needs about 7× as many and Multi-Head 18.9M. Speed appears only in a figure, with no seconds given.

## Eval 4 – Transfer: MOTS-trained weights fine-tuned on BCV
- **Analogy:** "Engine put in a car and tuned." This is a broken-in engine (trained on MOTS) vs a brand-new one (from scratch), the **random-init baseline**.
- **How:** All weights are fine-tuned on BCV for 13-organ segmentation. Metrics are Dice, surface distance (SD) and HD.
- **Datasets:** BCV (BTCV): 30 train, 20 test (official test server). These organs are mostly not labeled in MOTS, so it is a new-task transfer on new data.
- **Competitors:** Training from scratch, pretraining on a single task (hepatic vessel, validation curves only), and leaderboard methods: Auto Context, DLTK, PaNN and nnU-Net.
- **Fairness:** The scratch and single-task comparisons are fair (same network, run by the authors). The leaderboard methods' numbers were taken from published/leaderboard results and use different architectures.
- **Result:** MOTS pretraining gives 86.44 Dice vs 85.30 from scratch, a gain of about 1.1 points. It also has the best SD (1.17) and HD (15.62). nnU-Net still has the higher Dice (88.10). Validation curves show faster convergence with MOTS pretraining, while single-task pretraining helps only slightly.

## Key takeaways
- This is mainly a **multi-task partial-label segmentation** paper, not a foundation model in the modern sense. Its "pretraining" is supervised on about 920 CTs.
- The main comparison is fair (same backbone, re-implemented) but internal only. There is no external hospital test and no frozen-encoder tests.
- The transfer test is the classic **broken-in vs brand-new engine** comparison: a modest gain of about 1 Dice point, and it still loses to nnU-Net on Dice.
- This is the direct ancestor of the **Universal Model** (paper 22), which replaces DoDNet's one-hot task code with CLIP text embeddings and lists DoDNet as the one-hot baseline (70.42 vs 76.11 Dice there). **SegVol** and **SuPreM** follow the same "combine many partially labeled CT datasets" idea.
