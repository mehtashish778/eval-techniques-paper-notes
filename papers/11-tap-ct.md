# Paper 11 – TAP-CT

- **Title:** TAP-CT: 3D Task-Agnostic Pretraining of CT Foundation Models (Veenboer et al., 2025, arXiv preprint 2512.00872; venue not stated)
- **Link:** https://arxiv.org/pdf/2512.00872 (also read https://arxiv.org/html/2512.00872)
- **One line:** Vision Transformers (ViT-S and ViT-B) in 2D, 2.5D and 3D versions (6 models), pretrained with a 3D adaptation of DINOv2: DINO + iBOT + KoLeo losses, 3D random-resized crops, cube-like masking, gamma augmentation. Pretraining data: 104,405 in-house oncology CT volumes from 19,995 patients at the Netherlands Cancer Institute. Image-only.
- **Eval types used:** linear-probe, other:frozen-ABMIL-probe, ablation, image-image-retrieval (qualitative only), other:model-size-and-2D/2.5D/3D-comparison

---

## Eval 1 – Frozen segmentation with a linear head
- **Analogy:** Engine sealed on a test bench. Only a single linear layer reads the engine's output to paint each voxel.
- **How:**
  - Encoder frozen. Embeddings come from sliding-window inference, then a single linear convolution head is trained.
  - Framework: kaiko's EVA. Metric: macro Dice over non-background classes. Mean ± SD over 3 runs.
- **Datasets (all public, external to the in-house pretraining data):**
  - TotalSegmentator v2 (117 classes merged into 49)
  - AMOS22 (15 organs)
  - LiTS17
  - KiTS23
  - MSD Pancreas
- **Competitors:** Curia, SuPreM, CT-FM, VISTA3D, VoCo, FMCIB.
- **Fairness:**
  - All models go through the same frozen-feature + linear-head protocol in EVA, apparently run by the authors. The paper does not spell out every re-run detail.
  - Each competitor is fed at its own preferred spacing and window. TAP-CT is resized to (z, 224, 224) or (z, 512, 512).
  - FMCIB ran out of memory on TotalSegmentator.
  - SuPreM, CT-FM, VISTA3D and VoCo saw some of these public datasets in their pretraining (marked †), so they had a head start. TAP-CT did not.
  - A linear head is a big handicap for models designed to be used with their own decoder or with fine-tuning (VoCo, CT-FM, VISTA3D).
- **Result:** TAP-B-3D won every dataset, with average Dice 0.582 vs Curia 0.489, SuPreM 0.381, CT-FM 0.326, VISTA3D 0.274, VoCo 0.167, FMCIB 0.146. Per dataset: AMOS 0.724, LiTS 0.626, KiTS 0.480, TotalSeg 0.651, MSD Pancreas 0.429. That is 5–23 points ahead of Curia.

## Eval 2 – Frozen classification with an attention-MIL head
- **Analogy:** Engine sealed on a test bench. A small attention head (ABMIL) pools the engine's readings into one scan-level answer.
- **How:**
  - Encoder frozen. ABMIL head on [CLS] tokens (patch tokens if a model has no CLS).
  - Metrics: AUC; micro-AP for RSNA2022.
- **Datasets (public):**
  - LUNA16 nodule benign/malignant. The authors built a new **patient-level split**, because the usual nodule-level split leaks patients.
  - LUNA25 malignancy
  - RSNA2022 spine fracture
  - RSNA2023 abdominal trauma (liver, spleen and kidney only)
  - FDG-PET-CT tumour presence (from CT)
- **Competitors:** the same six.
- **Fairness:** Same protocol as Eval 1, with each model at its own spacing.
- **Result:** Mixed.
  - TAP-B-3D: LUNA16 0.876 (tie with CT-FM), LUNA25 0.855, RSNA2022 0.420 (best), RSNA2023 0.658, FDG 0.798.
  - The extracted table shows Curia higher on RSNA2023 (0.748) and FDG (0.877).
  - VISTA3D was higher on LUNA25 (0.866).
  - The authors say TAP-B-3D was best on two tasks (one a tie) and second or third elsewhere.

## Eval 3 – Comparing the six TAP-CT variants (size, 2D/2.5D/3D, resolution)
- **Analogy:** Same engine family, different sizes and shapes, all sealed on the same bench.
- **How:** The same frozen protocols as Evals 1 and 2, at 224 and 512 input resolution.
- **Result:**
  - Segmentation: improves with model size and 3D context at 224. At 512, the 2D/2.5D-vs-3D gap narrows (e.g. TAP-B-2.5D KiTS 0.536 vs TAP-B-3D 0.480).
  - Classification: no clear scaling trend. TAP-S-2.5D, a small 2.5D model, was the best variant on 4 of 5 tasks (e.g. FDG 0.889, RSNA2023 0.748).
  - The source table has an inconsistency: TAP-B-3D AMOS is 0.724 in Table 2 vs 0.648 in Table 3's 224 block.

## Eval 4 – Ablations
- **Patch tokens vs [CLS] for classification:** Patch tokens were much worse (e.g. FDG 0.714 vs 0.798, RSNA2023 0.592 vs 0.658).
- **Smaller (6-slice) local crops:** Both segmentation and classification got worse. Not helpful.
- **Longer warmup (12.5k→25k):** Needed to avoid early representation collapse.
- **Compute:** TAP-B-3D cost 3,648 GPU-hours, ~26× its 2D twin.

## Eval 5 – Qualitative
- Patch-embedding retrieval matched a lesion in a baseline scan to the same lesion in a follow-up scan (one patient, figure only).
- AMOS segmentation overlays vs the public models.

## Key takeaways
- A pure "sealed engine" paper. No fine-tuning, no kNN, no few-shot, no external clinical test, no text. It measures raw embedding quality with the thinnest possible heads (linear for segmentation, ABMIL for classification).
- That protocol strongly favours DINOv2-style patch features. It penalises models built to be fine-tuned or used with their own decoder: VoCo, CT-FM and VISTA3D scored very low here, which contrasts with their own papers' fine-tuned results. The authors acknowledge this. Compare with PASTA and Models Genesis, which are judged only after fine-tuning.
- Fairness positives: one framework (EVA) for all, 3 seeds, and a leakage-fixed LUNA16 split. Some competitors (SuPreM, CT-FM, VISTA3D, VoCo) were pretrained on the test datasets.
- Fairness concerns: competitors' preprocessing was set per model (spacing and window), and TAP-CT is resized rather than resampled. Dice is low in absolute terms because of the linear head. Do not compare these numbers with nnU-Net-style fine-tuned Dice.
- The segmentation win is clear. Classification is a split decision: Curia, and even the smaller TAP-S-2.5D, beat TAP-B-3D on several tasks, so "3D is better" holds for dense tasks but not for scan-level ones.
- Competitors in this list: SuPreM, CT-FM, VISTA3D, VoCo, FMCIB.

