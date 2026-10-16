# Paper 22 – Universal Model (CLIP-Driven)

- **Title:** CLIP-Driven Universal Model for Organ Segmentation and Tumor Detection (Liu et al., 2023; ICCV 2023, a venue not printed on the fetched page)
- **Link:** https://arxiv.org/html/2301.00785 (HTML version of the given PDF; matches the model name)
- **One line:** A Swin UNETR (or U-Net) segmentation network whose output head is generated from **CLIP text embeddings** of class names (frozen CLIP ViT-B/32 text encoder, prompt "A computerized tomography of a [CLS]"). It is trained with supervision on 14 public CT datasets (3,410 scans, 25 organs + 6 tumor types) with a masked loss for missing labels plus pseudo-labels. It uses image + text, where text serves as class codes, not reports.
- **Eval types used:** fine-tuned-segmentation, leaderboard, external-validation, cross-task-transfer, random-init-baseline, ablation, embedding-visualization, human/radiologist-rating, other:tumor-detection-sensitivity-specificity, other:efficiency

---

## Eval 1 – MSD public leaderboard (official test server)
- **Analogy:** A public race with an official judge. The engine is in a car and tuned, and the organizers time it.
- **How:** The Universal Model predicts the 6 CT tasks. The other 4 MRI tasks were filled in with nnU-Net predictions. Metric is Dice from the official server.
- **Datasets:** MSD hidden test sets (CT: liver, pancreas, hepatic vessel, lung, spleen, colon). MSD training data was part of its 14-dataset training.
- **Competitors:** Leaderboard entries such as nnU-Net, Swin UNETR and Models Genesis.
- **Fairness:** Competitor numbers come from the leaderboard (copied, not re-run). The Universal Model has far more training data (3,410 CTs from 14 datasets) than the single-task entries.
- **Result:** Claims 1st place overall. Liver 87.39 vs Models Genesis 86.61. Pancreas 72.59 vs Swin UNETR 70.71. Lung 80.01 vs 76.60. Colon 63.14 vs 59.45. On spleen it is slightly behind nnU-Net (97.27 vs 97.43).

## Eval 2 – BTCV 5-fold and MSD tumor 5-fold (internal)
- **Analogy:** "Engine put in a car and tuned" against specialist cars.
- **How:** 5-fold cross-validation with Dice, no ensembling.
- **Datasets:** BTCV (13 organs), plus MSD liver, pancreas, colon and lung tumors. All of these are inside the training collection.
- **Competitors:** Swin UNETR and nnU-Net (BTCV). Swin UNETR (MSD tumors).
- **Fairness:** The authors say the BTCV runs used the same splits and conditions, but it is unclear whether the baselines were re-run. The MSD 5-fold Swin UNETR runs were re-run by the authors. The Universal Model sees extra datasets, while the baselines are trained per dataset.
- **Result:** BTCV average 86.13 vs Swin UNETR 82.06 and nnU-Net 82.01. Tumors: liver 71.94 vs 57.86, pancreas 60.82 vs 52.53, colon 62.14 vs 50.45, but lung is lower (67.15 vs 68.90).

## Eval 3 – Tumor detection (yes/no per patient)
- **Analogy:** Reading the engine's warning light: does it fire only when there really is a tumor?
- **How:** Patient-level sensitivity and specificity from the segmentation masks. Specificity is tested on normal scans from CHAOS and Pancreas-CT.
- **Result:** Liver tumor sensitivity 88.89 / specificity 95.00. Kidney 91.67 / 95.00. Pancreas 93.98 / 91.25. No competitor is reported.

## Eval 4 – External generalization without fine-tuning
- **Analogy:** "Tuned in hospital A, driven in hospital B," with no retuning.
- **How:** Direct inference, Dice.
- **Datasets:** 3D-IRCADb (20 scans, public) and JHH (private, 5,038 scans). Neither is in training.
- **Competitors:** Dataset-specific models (details not clearly stated on the page).
- **Fairness:** How the baselines were obtained is not stated.
- **Result:** 3D-IRCADb mDSC 92.86. The text claims about 5% Dice gain on 3D-IRCADb and about 4% on JHH over baselines.

## Eval 5 – Transfer to new tasks (fine-tuning)
- **Analogy:** Swap in the broken-in engine and tune it for a new route. This also compares broken-in vs brand-new engines (random-init).
- **How:** Fine-tune on new classes, Dice.
- **Datasets:** TotalSegmentator (vertebrae, cardiac, muscles, organs) and JHH (cardiac, organs). These are new label sets.
- **Competitors:** From scratch, and Swin UNETR pretrained weights.
- **Fairness:** How the baselines were obtained is not stated.
- **Result:** Small but consistent gains. TotalSeg vertebrae 86.49 vs Swin UNETR 86.23 vs scratch 81.06. Other scores: cardiac 89.57, muscles 94.43, organs 88.95, JHH cardiac 72.06, JHH organs 89.37.

## Eval 6 – Human comparison
- **Analogy:** Comparing the engine's readings with two human mechanics.
- **How:** 17 BTCV scans with two independent expert annotations. Compares AI-vs-human Dice with human-vs-human Dice.
- **Result:** AI-vs-human agreement is slightly higher than human-vs-human for 6 organs (spleen, liver, kidneys, stomach, gallbladder, pancreas).

## Eval 7 – Ablations, embeddings, efficiency (grouped)
- **Text embedding ablation:** One-hot (the DoDNet style) 70.42, BioBERT 71.55, and CLIP prompts 73.49–76.11 average Dice. Prompt wording matters.
- **Backbone:** U-Net 76.73 vs Swin UNETR 76.11, so the method works with either.
- **Embedding visualization:** t-SNE shows that CLIP embeddings group related organs (e.g., left/right kidney) better than one-hot codes.
- **Efficiency:** By FLOPs, about 6× cheaper than Swin UNETR and 19× cheaper than nnU-Net when running all tasks, because one model replaces many.

## Key takeaways
- This is a supervised, multi-dataset universal segmenter. The engine is never sealed (no linear/kNN probe). Every eval is "engine in a car."
- The strongest evidence is the official MSD leaderboard (an outside judge) and the external 3D-IRCADb/JHH tests. Many other tables compare against per-dataset models trained on much less data, so part of the gain is simply "more data."
- "CLIP" here means text embeddings used as class codes. There is no zero-shot classification or retrieval, and the model can't segment truly new classes without fine-tuning.
- It builds directly on **DoDNet** (paper 21, the one-hot baseline). It is followed by **SuPreM** (same group: AbdomenAtlas pretraining) and resembles **SegVol**'s use of CLIP text prompts. **Models Genesis** and **Swin UNETR** appear as leaderboard competitors.
- Minor inconsistencies: the external scan count (6,162 in the abstract vs 6,082 summed), the tumor-gain percentages, and which prompt template is best.
