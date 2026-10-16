# Paper 07 – VISTA3D

- **Title:** VISTA3D: A Unified Segmentation Foundation Model For 3D Medical Imaging (He et al., 2024, arXiv v3; the arXiv text read does not state a venue, though it was later published at CVPR 2025)
- **Link:** https://arxiv.org/html/2406.05285v3 (Table 3 and the supplement were read from https://arxiv.org/pdf/2406.05285v3)
- **One line:**
  - A SegResNet encoder (MONAI) shared by two heads: an automatic head that takes a class index (127 classes) and an interactive head that takes 3D clicks (SAM-style).
  - Trained with supervision on 11,454 CT volumes (public and in-house), with TotalSegmentator pseudo-labels for 117 classes.
  - Supervoxels distilled from 2D SAM features, and MAISI synthetic tumours, are added for training.
  - Image-only.
- **Eval types used:** fine-tuned-segmentation, interactive-segmentation, few-shot/label-efficiency, transfer-to-other-modality, external-validation, random-init-baseline, ablation, other:inference-speed, other:zero-shot-novel-class-with-clicks

---

## Eval 1 – Automatic and interactive segmentation on 14 test sets (Table 1)
- **Analogy:**
  - Automatic mode: the finished car on the road, compared with cars built with other bodies (nnU-Net, Auto3DSeg).
  - Click mode: "the driver points at the target and the engine follows."
- **How:**
  - VISTA3D is one model for all datasets. It is not tuned per dataset.
  - Clicks are simulated: one positive click at the foreground centre ("point"). In "auto+point", the auto result is corrected with one click per 128³ patch, placed in the larger error region.
  - Metric: Dice.
- **Datasets:**
  - 14 test sets: MSD 03/06/07/08/09/10, Airway, a private bone lesion set, BTCV-Abdomen, BTCV-Cervix, VerSe, AbdomenCT-1K, AMOS22 and TotalSegV2.
  - These are held-out 20% test splits of the same sources used in training (internal, not external hospitals).
- **Competitors:**
  - nnU-Net and Auto3DSeg, trained by the authors on each dataset separately (specialist models, same splits).
  - TotalSegmentator run out of the box (only where its classes apply).
- **Fairness:**
  - Baselines were re-run on identical splits, which is good.
  - But this compares one generalist model against per-dataset specialists.
  - The authors flag the TotalSegV2 numbers as biased: its ground truth came from TotalSegmentator, and its training data may overlap the test set.
- **Result (average Dice):**
  - Automatic mode: VISTA3D 0.711 vs nnU-Net 0.718 vs Auto3DSeg 0.706, so roughly tied, slightly below nnU-Net.
  - Weak spots in automatic mode: pancreatic tumour 0.324 vs 0.488.
  - With clicks: one click gives 0.760, and auto+clicks gives 0.792.
  - Big gains from clicks on tumours, e.g. bone lesion 0.585 vs nnU-Net 0.396.

## Eval 2 – Inference speed (Table 2)
- **Analogy:** How fast the car finishes the lap.
- **How:** Whole-body segmentation time on a V100 16 GB (plus a 12 GB GPU in the supplement).
- **Competitors:** TotalSegmentator.
- **Result:**
  - About 3–4× faster: 512³ took 2m09s vs 12m01s, and 333×333×603 took 1m07s vs 4m34s.
  - One interactive click takes about 3.2 s.

## Eval 3 – "Zero-shot" segmentation of new classes with clicks (Figure 5)
- **Analogy:** "The driver points at the target and the engine follows", for targets it was never trained on.
- **How:**
  - "Zero-shot" here means segmenting classes outside the 127 using clicks only. It has nothing to do with text.
  - Iterative clicks; Dice plotted against the number of clicks.
- **Datasets:** External and unseen: murine micro-CT (140 scans, 4 organs, a non-human transfer), C4KC-KITS (210), adrenocortical carcinoma (53) and hepatocellular carcinoma (105).
- **Competitors:**
  - MedSAM: run slice by slice with boxes, each box counted as 2 clicks.
  - SegVol: 7 points, with zoom-out-zoom-in.
  - VISTA3D without supervoxels (ablation).
- **Fairness:** The prompt types differ between methods. The authors tried to equalise them by counting clicks.
- **Result:**
  - VISTA3D is described as better than the competitors, but numbers are shown only in a figure; no numbers are given in the text.
  - The authors claim supervoxels improved zero-shot performance by about 50%.

## Eval 4 – Fine-tuning with few examples (Table 3)
- **Analogy:**
  - "Tune with only a handful of examples."
  - nnU-Net and Auto3DSeg are trained from scratch (brand-new engines).
  - TotalSegmentator is fine-tuned (another broken-in engine).
- **How:** Fine-tune with 1, 5, 10, 20, 40 or all cases. Metric: Dice.
- **Datasets:** WORD (100 training cases) and mouse micro-CT (89 training cases). Both are external to training.
- **Competitors:**
  - nnU-Net and Auto3DSeg from scratch.
  - The TotalSegmentator checkpoint fine-tuned in the nnU-Net pipeline.
- **Fairness:** Same splits, all run by the authors.
- **Result:**
  - WORD one-shot: VISTA3D 0.795 vs TotalSegmentator 0.779, Auto3DSeg 0.214 and nnU-Net 0.185.
  - WORD five-shot: 0.839 vs 0.823.
  - WORD full data: everyone is about 0.873–0.875.
  - Mouse one-shot: 0.926 vs 0.820 for the best baseline. With full data all methods are about 0.95.
  - The pretraining advantage disappears once enough labels exist.

## Eval 5 – Comparison with SAM2 and small ablations (supplement/text)
- **Analogy:** Comparison with another driver-points engine.
- **How:** Single click on MSD Task07 (pancreas).
- **Competitors:** Fine-tuned SAM2.
- **Result:**
  - VISTA3D: pancreas 0.802, tumour 0.603. SAM2: 0.557 and 0.308.
  - Other design choices are ablated only qualitatively, with no numbers. For example, CLIP class embeddings were "slightly worse" than random ones, and the merge algorithm, synthetic tumours and ambiguity embeddings are described without numbers.

## Key takeaways
- This is a segmentation tool, not a feature extractor. There are no frozen or linear-probe tests and no classification. The encoder is judged only through its own two heads.
- In automatic mode the honest headline is that it ties per-dataset nnU-Net with one model. The real gains come from clicks and from very low-label fine-tuning (one-shot or five-shot).
- "Zero-shot" means new classes segmented through clicks, which is different from the CLIP-style text zero-shot in CT-CLIP or Merlin.
- The main tests are internal held-out splits. The external tests (WORD, mouse micro-CT and the zero-shot sets) are smaller, and the zero-shot results are shown only in figures.
- It is closely related to TotalSegmentator (pseudo-labels and baseline), SegVol and SAM-Med3D (interactive rivals), and SuPreM (another big supervised CT segmentation pretraining). It is built inside the MONAI/Auto3DSeg "car body".
