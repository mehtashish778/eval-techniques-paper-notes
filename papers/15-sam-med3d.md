# Paper 15 – SAM-Med3D

- **Title:** SAM-Med3D: Towards General-purpose Segmentation Models for Volumetric Medical Images (Wang et al., 2023, arXiv preprint; v3 Sept 2024; venue not stated in the paper)
- **Link:** https://arxiv.org/pdf/2310.15161
- **One line:** A fully 3D version of Segment Anything (SAM): a 3D ViT image encoder, a 3D point-prompt encoder, and a 3D mask decoder. It is trained from scratch, not from SAM weights, with supervised promptable segmentation. Training data is SA-Med3D-140K: about 22K 3D images and 143K masks, 245 categories, 70 public + 24 private datasets, CT, ultrasound, and 26 MR sequences. Training has two stages: 131K masks, then about 75K "high-quality" masks. Image only.
- **Eval types used:** interactive-segmentation, external-validation (unseen sources and modality), engine-swap-framework, random-init-baseline, ablation, other:task-specific-vs-generalist, other:inference-speed

---

## Eval 1 – Promptable segmentation vs 2D SAMs (main benchmark)
- **Analogy:** driver points at the target and the engine follows.
- **How:**
  - No task tuning.
  - Simulated clicks: the first point is random inside the target, and later points are placed in the error region.
  - Settings: 1, 3, 5, and 10 points per volume for SAM-Med3D. The 2D models get N points (one per slice, 10 ≤ N ≤ 200), 3N, or 5N.
  - Metric: Dice.
- **Datasets:**
  - Validation split, about 12K masks from 16 public benchmarks.
  - Results are split into "seen" and "unseen" targets. About 58% of cases come from sources not used in training, and all ultrasound is held out.
  - Some validation datasets have their official training sets in SAM-Med3D's training data.
- **Competitors:** SAM and SAM-Med2D, both run slice by slice.
- **Fairness:**
  - The 2D models get many more clicks (one per slice) but work slice-wise.
  - It is not stated whether SAM-Med2D's training data overlaps the test sets.
  - Comparing a 3D model trained on this data against 2D models is not like-for-like.
- **Result:**
  - SAM-Med3D with 1 point: Dice 76.3 overall (seen 82.0, unseen 37.0). With 10 points: 80.7 (seen 85.2, unseen 49.9).
  - SAM with N points: 16.2. SAM-Med2D with N points: 36.8.
  - There is a big drop on unseen targets.
  - SAM-Med3D runs in 1–26% of SAM's inference time.

## Eval 2 – Breakdowns by anatomy, lesion, modality, and organ
- **Analogy:** same "driver points" test, broken down by road type.
- **Result:**
  - SAM-Med3D (1 point) beats SAM-Med2D (N points) on most targets, including 45 of 49 organ and lesion targets, with gains up to 68%. It loses on unseen lesions.
  - Strongest on CT and MR. On ultrasound (never seen in training) it is competitive only with enough clicks.
  - Brain structures and lesions are hardest.

## Eval 3 – Generalist vs task-specific models
- **Analogy:** a "driver points" engine vs cars built and tuned for one route.
- **How:** 6 public benchmarks: TotalSegmentator, KiTS21, BTCV, TDSC-ABUS23 (ultrasound), AMOS-CT, and AMOS-MR. Metric: Dice.
- **Competitors:**
  - UNETR and nnU-Net, trained by the authors on each benchmark's train set (no prompts).
  - SAM-Med2D, SegVol (point + text), and SAM, run with official weights. SegVol has only one number reported (BTCV 73.8).
- **Fairness:**
  - The interactive model gets 10 clicks, which is extra information the automatic models do not get.
  - Several of these datasets' training sets are inside SA-Med3D-140K.
- **Result:**
  - With 10 points, SAM-Med3D beats nnU-Net on TotalSegmentator (87.6 vs 85.2), KiTS21 (75.4 vs 75.3), BTCV (83.0 vs 81.9), and ultrasound TDSC-ABUS23 (54.4 vs 45.1).
  - It loses on AMOS-CT (84.0 vs 88.9) and AMOS-MR (81.1 vs 86.9).

## Eval 4 – Encoder transfer to semantic segmentation
- **Analogy:** engine swap. The SAM-Med3D image encoder is put into a UNETR car body and tuned. It is compared with a random-init baseline (broken-in vs brand-new engine).
- **How:** UNETR is fine-tuned with the SAM-Med3D encoder vs UNETR from scratch. Metric: Dice. It is not stated whether the encoder is fully unfrozen.
- **Datasets:** AMOS, TotalSegmentator, CAS2023, and SEG.A.2023. AMOS and TotalSegmentator overlap with the pretraining data.
- **Competitors:** UNETR from scratch only. There is no nnU-Net and no other pretrained encoder (e.g. Swin UNETR SSL, SuPreM).
- **Fairness:** Same architecture and recipe, so this is a clean init comparison, but there is only one baseline.
- **Result:**
  - Average Dice 83.1 → 85.8.
  - Per dataset: AMOS +5.6 (76.3 → 81.9), TotalSegmentator +2.5, CAS2023 +2.1, SEG.A +0.5.

## Eval 5 – Ablations
- **Analogy:** testing which engine build works best.
- **How and results:**
  - **Build choice (on AMOS):**
    - 3D adapter on frozen SAM: 43.8 Dice overall.
    - SAM weights inflated to 3D and fine-tuned: 43.8 overall (best on seen targets).
    - Training from scratch: 45.2 overall, which is why the final model uses it.
  - **Stage-2 fine-tuning data:** Fine-tuning on one dataset helps that dataset but hurts others. The curated high-quality mix from 44 datasets is best overall.
  - **Number of clicks:** More points help, especially for lesions and ultrasound.
  - **Qualitative figures:** Better consistency between slices.

## Key takeaways
- This is mainly an interactive ("driver points") segmentation model. The headline evals measure clicks → Dice, not frozen-feature quality. Compare with SegVol and VISTA3D (also promptable 3D), and with SAM / SAM-Med2D (2D).
- The "unseen" story is weaker than it sounds:
  - Some validation sets have their training splits inside SA-Med3D-140K.
  - AMOS and TotalSegmentator were used both for design choices and for evaluation.
  - Stage-2 data was selected on the same validation set.
  - There is no separate external test set.
  - Unseen-target Dice (37–50) is much lower than seen (82–85).
- The comparison with nnU-Net and UNETR mixes an interactive model with fully automatic models. The interactive model gets 10 clicks of extra information.
- The engine-swap result (UNETR +2.7 Dice on average) is a useful "is it a good pretrained encoder?" check. But it has only a random-init UNETR baseline: no nnU-Net, and no SSL encoders such as Swin UNETR SSL, VoCo, or SuPreM.
- Interesting design finding: training from scratch beat starting from 2D SAM weights. This suggests 2D natural-image SAM weights do not transfer well to 3D medical volumes.
