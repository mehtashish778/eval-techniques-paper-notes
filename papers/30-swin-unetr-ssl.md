# Paper 30 – Swin UNETR SSL

- **Title:** Self-Supervised Pre-Training of Swin Transformers for 3D Medical Image Analysis (Yucheng Tang et al., 2022, CVPR 2022; venue not printed in the arXiv text)
- **Link:** https://arxiv.org/pdf/2111.14791 (matches the model; no mismatch)
- **One line:** Swin UNETR (3D Swin Transformer encoder + CNN decoder with skip connections); encoder pretrained with three proxy tasks (masked volume inpainting, rotation prediction, contrastive coding) on 5,050 unlabeled public CT scans (LUNA16, TCIA Covid19, LIDC, HNSCC, TCIA Colon); image-only.
- **Eval types used:** fine-tuned-segmentation, leaderboard, random-init-baseline, few-shot/label-efficiency, ablation, other:pretraining-data-size-scaling, other:qualitative-segmentation-figures

---

## Eval 1 – BTCV multi-organ segmentation (leaderboard)
- **Analogy:** Engine put in a car and tuned, then entered into a public race (leaderboard).
- **How:** Fine-tuned whole Swin UNETR (encoder pretrained, decoder trained). 5-fold cross-validation; best model per fold ensembled for the final test prediction. Metric: Dice (13 abdominal organs), also HD.
- **Datasets:** BTCV (30 labeled training CTs; hidden test set). Not named among pretraining sources (overlap not explicitly ruled out).
- **Competitors:** SETR variants, ASPP, TransUNet, CoTr, RandomPatch, PaNN, nnU-Net, UNETR.
- **Fairness:** Competitor numbers taken from the BTCV leaderboard (not re-run). Swin UNETR uses a 5-model ensemble; other entries' setups differ. Paper says it wins both "free" and "standard" competitions but does not say which result goes in which.
- **Result:** Average Dice 0.918 in the table (the text says 0.908 — inconsistency in the paper). Ranked first.

## Eval 2 – Medical Segmentation Decathlon (MSD, 10 tasks, leaderboard)
- **Analogy:** Same tuned car entered in a 10-event race.
- **How:** Fine-tuned; ensembled; metrics Dice and NSD. Pretrained weights used only for CT tasks; MRI tasks (brain, heart, hippocampus, prostate) trained from scratch "because of the domain gap".
- **Datasets:** MSD public test leaderboard.
- **Competitors:** DiNTS, nnU-Net, Models Genesis, TransVW, C2FNAS, Kim et al.
- **Fairness:** Numbers "directly obtained from the MSD leaderboard". Note MRI tasks get no benefit from pretraining at all, so the overall MSD rank is not purely a pretraining result.
- **Result:** Average Dice 78.68% / NSD 89.28%, ahead of DiNTS 77.93, nnU-Net 77.89, Models Genesis 76.97, TransVW 76.96. Supplement: MRI tasks average Dice 82.14%, NSD 94.66%, "comparable" to Models Genesis / nnU-Net on heart, hippocampus, prostate.

## Eval 3 – Pretrained vs from scratch (MSD CT tasks)
- **Analogy:** Random-init baseline: broken-in vs brand-new engine, same car.
- **How:** Same Swin UNETR, fine-tuned with and without pretrained weights; Dice.
- **Result:** Liver 75.27 → 77.77, hepatic vessel 64.63 → 68.52, colon 34.83 → 43.38, spleen 96.05 → 97.32, pancreas 67.12 → 67.82.

## Eval 4 – Label efficiency on BTCV
- **Analogy:** Tune with only a handful of examples.
- **How:** Fine-tune with fractions of BTCV labels (fractions shown in a figure; how subsets were drawn not stated). Dice on BTCV test.
- **Result:** With 10% labels, about 10% Dice gain from pretraining; with all labels +1.3% average Dice. Pretrained with 60% of data matches scratch with 100%, so authors claim "at least 40%" less annotation needed.

## Eval 5 – Ablations: proxy tasks and pretraining data size
- **Analogy:** Swap engine-building steps; break in with more or fewer miles.
- **How:** BTCV Dice/HD.
- **Result:** Scratch 83.43 / 42.36; rotation only 83.56; contrastive only 83.67; inpainting only 83.85; all three 84.72 / HD 20.03 (text gives scratch as 83.13 — another small inconsistency). Pretraining with 100 vs 3,000 vs 5,000 scans: more scans = better per organ (figure only, no averaged number).

## Eval 6 – Visual results
- Qualitative BTCV (pancreas, portal vein, adrenals) and MSD comparisons vs DiNTS. No t-SNE/feature plots.

## Key takeaways
- Everything is "engine put in a car and tuned" for segmentation; no frozen probe, no classification task, no external-hospital test, no text.
- Headline claims rely on public leaderboards with copied competitor numbers and a 5-fold ensemble — strong showing, but not a controlled head-to-head of pretraining methods. The cleaner evidence is the scratch-vs-pretrained comparison (same architecture).
- Pretraining did not help MRI here (MRI tasks were trained from scratch), unlike PCRL/PCRLv2 which claim CT→MRI transfer.
- Small numerical inconsistencies in the paper (0.918 vs 0.908 BTCV; 83.43 vs 83.13 scratch).
- Very influential: the Swin UNETR backbone and weights are reused or compared against by SwinMM (paper 31), VoCo, SuPreM, GL-MAE, Universal Model, and many CT foundation-model papers; it is effectively the "standard engine" baseline for 3D CT SSL.
