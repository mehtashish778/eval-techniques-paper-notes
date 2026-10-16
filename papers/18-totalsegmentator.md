# Paper 18 – TotalSegmentator

- **Title:** TotalSegmentator: Robust Segmentation of 104 Anatomical Structures in CT Images (Wasserthal et al., 2023, Radiology: Artificial Intelligence)
- **Link:** https://arxiv.org/pdf/2208.05868
- **One line:** Not a pretrained foundation model. It is a supervised nnU-Net (the "car body", trained from scratch) that segments 104 structures (27 organs, 59 bones, 10 muscles, 8 vessels) in CT. Training data: 1,204 CT series from one hospital (University Hospital Basel), annotated with model help and then corrected by hand. Image-only. Two versions: 1.5 mm (main) and 3 mm (faster).
- **Eval types used:** fine-tuned-segmentation, external-validation, robustness/stability, other:clinical-application (aging study), other:runtime

---

## Eval 1 – Internal test set segmentation
- **Analogy:** Engine put in a car and tuned. Here the whole car (nnU-Net) is built and trained from scratch for this one job.
- **How:** Supervised nnU-Net training (the 1.5 mm model is 5 sub-models of about 21 classes each). Metrics: Dice and normalized surface distance (NSD).
- **Datasets:** 65 held-out test patients from the same hospital (1,082 train / 57 val / 65 test). Internal.
- **Competitors:** The 3 mm version of itself.
- **Fairness:** N/A (own model only). The authors trained the final model separately from the annotation-helper models to limit leakage into the test set.
- **Result:** 1.5 mm model: Dice 0.943, NSD 0.966. 3 mm model: Dice 0.840, NSD 0.966.

## Eval 2 – Comparison with an nnU-Net trained on BTCV (internal + external)
- **Analogy:** Two cars of the same model trained on different fuel (large in-house data vs small public BTCV), each driven in both hospitals. The BTCV run is "tuned in hospital A, driven in hospital B".
- **How:** The authors trained an nnU-Net on BTCV themselves and compared on the 13 shared structures (the supplement lists 12). Dice and NSD.
- **Datasets:** The internal test set (65), and BTCV (30 subjects, external, from another continent).
- **Competitors:** nnU-Net trained on BTCV. No other public tools were benchmarked.
- **Fairness:** Re-run by the authors with the same framework, so it is a fair framework comparison. On BTCV, the competitor is tested in its own home domain, which is a tough comparison for TotalSegmentator.
- **Result:** Internal: Dice 0.932 vs 0.871, NSD 0.971 vs 0.921 (p<0.001). On BTCV (external for TotalSegmentator): Dice 0.849 vs 0.839, NSD 0.932 vs 0.915. TotalSegmentator wins even on the other model's home data.

## Eval 3 – Robustness / qualitative
- **Analogy:** Driving the car on rough roads.
- **How:** Visual examples of hard cases, plus per-structure results (Fig. S1 and an online JSON file).
- **Datasets:** The data covers 16 scanners, 8 sites, many contrast phases and kernels, and both pathologic and normal patients. All of it comes from one hospital's PACS, mostly Siemens scanners.
- **Result:** Handles broken bones, displaced bowel, a missing spleen, and a transplanted kidney. Failures: missed parts of the colon and iliac arteries, and wrong rib/vertebra numbering when only part of the spine is visible. Weaker on non-contrast iliac vessels and heart chambers.

## Eval 4 – Clinical application: organ changes with age
- **Analogy:** Using the car for a real delivery job.
- **How:** Ran the model on 4,004 polytrauma whole-body CTs, then measured organ volumes and attenuation (Spearman correlation with age, Kruskal-Wallis test). Not adjusted for confounders.
- **Result:** Example correlations with age: aortic volume rs=0.64, autochthon muscle attenuation −0.74, gluteus minimus attenuation −0.79, hip attenuation −0.61, kidney and pancreas volume −0.49.

## Eval 5 – Runtime
- **How / Result:** On an i9 CPU + RTX 3090: 1.5 mm model takes 1 min 17 s to 3 min 32 s per scan; 3 mm model takes 34 s to 1 min 23 s. It uses up to 11.4 GB of GPU memory.

## Key takeaways
- This is a dataset + tool paper, not a pretraining paper. There are no frozen probes, no transfer, and no foundation-model comparisons.
- It matters for this list because its labels and masks are reused heavily: SuPreM, VISTA3D, CT-FM and others pretrain or evaluate on TotalSegmentator data, and many papers use it as a test set.
- External validation is thin: only BTCV, 30 cases, 13 structures. The training data comes from a single hospital and is mostly Siemens.
- The only competitor is the same framework (nnU-Net) trained on smaller data. This shows the benefit of more labeled data, not a better architecture.
- Related: nnU-Net is the "car body" used in many engine-swap evaluations (e.g. VoCo, SuPreM). VISTA3D and SegVol later compare against TotalSegmentator.
