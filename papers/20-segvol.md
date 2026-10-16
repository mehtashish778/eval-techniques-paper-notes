# Paper 20 – SegVol

- **Title:** SegVol: Universal and Interactive Volumetric Medical Image Segmentation (Du et al., 2023 arXiv / 2024; NeurIPS 2024 judging by the NeurIPS checklist in the paper, but the fetched page does not name the venue)
- **Link:** https://arxiv.org/html/2311.13385 (HTML version of the given PDF link; the paper matches the model name)
- **One line:** A 3D ViT image encoder (patch 4×16×16, ViT-Base size) with a SAM-style prompt encoder and mask decoder plus a frozen CLIP text encoder. The encoder is first pretrained with SimMIM masked image modeling on about 96K CT volumes (about 90K unlabeled from Radiopaedia plus about 6K labeled). Then the whole model is trained with supervision on 25 public CT segmentation datasets (about 150K masks, 200+ categories) using point, box and text prompts, plus pseudo-masks. It uses image + text, but text only as category-name prompts.
- **Eval types used:** interactive-segmentation, leaderboard, fine-tuned-segmentation, transfer-to-other-modality, few-shot/label-efficiency, ablation, other:efficiency, other:qualitative-semantic-disambiguation

---

## Eval 1 – Main comparison with other SAM-style interactive models
- **Analogy:** "Driver points at the target and the engine follows." The driver gives a box plus a text name, and each competing engine gets its own preferred pointing style.
- **How:** No extra tuning. SegVol uses bbox + text prompts. The 3D volume goes through "zoom-out-zoom-in": a coarse mask from the whole resized volume, then a sliding window inside the region of interest. Metric is Dice, reported as the median with quartiles.
- **Datasets:** 1,778 cases. These are 120 AMOS22 validation cases, 1,618 ULS23 lesion cases (DeepLesion3D 750, Radboudumc Bone 744, Radboudumc Pancreas 124) and 40 SegTHOR cases. The paper calls these "external," but AMOS22, ULS23 and SegTHOR also appear in its training-dataset list (Table 4). They are most likely held-out splits of datasets seen in training, not truly unseen hospitals.
- **Competitors:** SAM (point, with 5-click correction), SAM (bbox), SAM-Med2D (points, 5-click), SAM-Med3D (points, 5-click, input limited to 128³), MedSAM (bbox).
- **Fairness:** The competitor settings are described, but the paper does not clearly say whether it re-ran them or copied numbers (it most likely ran their public checkpoints). Competitors get different prompt types, and SegVol alone gets text. The 2D models (SAM, MedSAM, SAM-Med2D) work slice by slice. SAM-Med3D is handicapped by its crop size. Competitors were not trained on SegVol's 25-dataset collection.
- **Result:** SegVol wins clearly on organs. AMOS22 average median Dice is 0.859 vs 0.667 for the next best (SAM-Med2D), and SegTHOR is 0.816 vs 0.773 (SAM-Med3D). On ULS23 lesions it only ties MedSAM: the listed average is 0.7046 for both, although SegVol's own subset values average about 0.708. The abstract claims SegVol wins 19 of 22 tasks.

## Eval 2 – Against task-specific models trained per dataset (appendix)
- **Analogy:** The "one universal engine" against specialist cars, each built and tuned for a single dataset.
- **How:** nnU-Net, 3D UX-Net and Swin UNETR are each trained per dataset. SegVol is the universal model with prompts. Metric is Dice.
- **Datasets:** The 20% internal test splits of several training datasets (MSD lung, colon, liver tumor; organ tasks). The appendix inconsistently says 10 vs 8 tasks.
- **Competitors:** nnU-Net, 3D UX-Net, Swin UNETR.
- **Fairness:** Unequal by design. The specialists get no prompts, while SegVol gets box/point/text hints from the ground truth plus far more training data.
- **Result:** SegVol wins big on lesions: lung tumor 0.712 vs nnU-Net 0.596, colon cancer 0.697 vs 0.377, and liver tumor 0.783 vs 0.361. Organs are mixed: nnU-Net is comparable or better on e.g. the spleen and left adrenal gland, while SegVol leads on liver (0.956), pancreas (0.846) and stomach (0.918).

## Eval 3 – Zero-shot MRI (CT-trained model on MRI)
- **Analogy:** "Tuned in hospital A, driven in hospital B." Here it goes further: an engine tuned on CT is driven on MRI.
- **How:** No MRI training. Prompts are used, and the metric is median Dice.
- **Datasets:** CHAOS MRI, 60 scans, 4 organs. Unseen modality.
- **Competitors:** None stated.
- **Fairness:** No baseline, so the numbers are hard to put in context.
- **Result:** Liver 85.7%, spleen 80.1%, left kidney 80.0%, right kidney 81.5%.

## Eval 4 – Few-shot fine-tuning
- **Analogy:** "Tune with only a handful of examples."
- **How:** SegVol is fine-tuned on one small dataset only, then compared with SegVol trained on all data.
- **Datasets:** FLARE22 (40 training cases) and MSD-spleen (32 training cases).
- **Competitors:** The full-data SegVol only. There is no other few-shot baseline and no random-init model.
- **Result:** FLARE22 reaches 0.557 vs 0.882 for full SegVol. MSD-spleen reaches 0.947 vs 0.960. Small data is fine for an easy single organ but weak for multi-organ.

## Eval 5 – Ablations
- **Analogy:** Taking engine parts out to see which ones matter.
- **Zoom-out-zoom-in (48 AMOS22 cases):** Resize only gives 0.451 Dice at 65 ms, sliding window 0.653 at 3,331 ms, and zoom-out-zoom-in 0.730 at 190 ms.
- **Data scaling:** Trained on 1, 2, 8 and 25 datasets, tested on the BTCV 20% split. More data helps, most of all for text prompts. The numbers appear only in a figure.
- **Prompt combinations (19 tasks):** bbox+text beats text alone by 5.85 Dice points, and point+text beats point alone by 4.62 points.

## Eval 6 – Efficiency and qualitative checks (grouped)
- **Efficiency:** 181M parameters and about 0.33 s per AMOS22 case. SAM takes 2.18 s and SAM-Med2D 3.55 s. SAM-Med3D is faster (0.18 s) but works on crops.
- **Semantic disambiguation / category identification:** Figures only. For example, a point on the liver returns "liver" with 0.997 confidence. These are qualitative only.

## Key takeaways
- The core eval is **interactive/prompted segmentation**. The engine is never tested sealed (no linear probe or kNN), so we learn little about the SimMIM-pretrained encoder alone, and there is no random-init ablation of the 96K-CT pretraining.
- The "external" test sets (AMOS22, ULS23, SegTHOR) are also in its training list, so this is closer to internal held-out testing. The truly unseen test is the CT→MRI CHAOS run, which has no baseline.
- Comparisons with specialists (nnU-Net etc.) are not apples to apples, because SegVol gets ground-truth-derived box/point prompts and the specialists do not.
- It has the same competitor family as **SAM-Med3D**. The CLIP-text-prompt idea echoes the **Universal Model** (paper 22), and the partial-label multi-dataset training goes back to **DoDNet** (paper 21). **VISTA3D** is its closest later rival (auto + interactive).
- Small numeric inconsistencies exist (the ULS23 average, the "19.25%" gain being absolute rather than relative, the 10 vs 8 tasks).
