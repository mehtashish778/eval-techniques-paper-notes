# Paper 04 – FMCIB (Foundation Model for Cancer Imaging Biomarkers)

- **Title:** Foundation model for cancer imaging biomarkers (Pai et al., 2024, Nature Machine Intelligence 6(3):354–367)
- **Link:** https://pmc.ncbi.nlm.nih.gov/articles/PMC10957482/ (the given PDF link only returned a "Preparing to download" page, so the PMC HTML version of the same article was read)
- **One line:** 3D ResNet50 encoder pretrained with a modified SimCLR (contrastive) method on 11,467 unlabelled CT lesions (from 5,513 scans of 2,312 patients in DeepLesion), using 50×50×50 mm patches centred on a lesion; image-only.
- **Eval types used:** linear-probe, fine-tuned-classification, random-init-baseline, few-shot/label-efficiency, external-validation, cross-task-transfer, robustness/stability, saliency, embedding-visualization, ablation, other:biological-gene-association

---

## Eval 1 – Choosing the pretraining method (pretext ablation)
- **Analogy:** Build several engines with different break-in methods, seal them, and test each on the same bench.
- **How:** Frozen features + linear model on use case 1 (lesion site). Metrics: balanced accuracy, mAP.
- **Datasets:** DeepLesion labelled lesions (held out from pretraining).
- **Competitors:** Standard SimCLR, SwAV, NNCLR, an autoencoder.
- **Fairness:** All trained by the authors on the same data and split.
- **Result:** Modified SimCLR won: balanced accuracy 0.779, mAP 0.847, vs plain SimCLR 0.696 / 0.779.

## Eval 2 – Use case 1: lesion anatomical site (8-way classification, in-distribution)
- **Analogy:** Two ways: (a) "engine sealed on a test bench" (frozen features + logistic regression), and (b) "engine put in a car and tuned" (full fine-tuning).
- **How:** Both frozen-feature logistic regression and full fine-tuning. Metrics: balanced accuracy, mAP, with bootstrap 95% CIs and permutation tests.
- **Datasets:** DeepLesion labelled lesions, 5,051 total (2,610 train / 1,220 tune / 1,221 test). Same source dataset as pretraining, but these labelled lesions were removed from pretraining (internal test).
- **Competitors:** Supervised ResNet50 from random init; Med3D and Models Genesis (both as frozen features and fine-tuned).
- **Fairness:** Authors re-ran all competitors themselves from public weights, same splits, matched configs. Note FMCIB's encoder (~200M parameters per the paper) differs in size/architecture from the baselines.
- **Result:** Fine-tuned FMCIB best: mAP 0.857, balanced accuracy 0.804. Significant on mAP vs all; on balanced accuracy not significant vs fine-tuned Med3D (P = 0.059). Frozen FMCIB beat frozen Med3D and Models Genesis, and the fine-tuned baselines did not significantly beat frozen FMCIB.

## Eval 3 – Label efficiency (use cases 1 and 2)
- **Analogy:** "Tune with only a handful of examples."
- **How:** Retrain with 50%, 20%, 10% of training data, frozen and fine-tuned.
- **Datasets:** Use case 1 (down to 505 lesions), use case 2 (down to 51 nodules).
- **Competitors:** Same as above.
- **Fairness:** Same splits and recipes for all.
- **Result:** Frozen FMCIB significantly beat every baseline at all reduced sizes. Fine-tuned FMCIB's advantage lost significance at 20% and below. From 100% to 10% data, FMCIB dropped only 9% (balanced accuracy) and 12% (mAP) on use case 1.

## Eval 4 – Use case 2: lung nodule malignancy (out-of-distribution task)
- **Analogy:** Same engine, new road; both sealed-bench and tuned-in-car.
- **How:** Frozen features + linear model, and fine-tuning. Metrics: AUC, mAP.
- **Datasets:** LUNA16/LIDC: 507 nodules train/tune, 170 test. Not seen in pretraining.
- **Competitors:** Supervised from scratch, Med3D, transferred use-case-1 supervised model. Models Genesis dropped because it was pretrained on overlapping LUNA16 data.
- **Fairness:** Re-run by the authors, same splits. Excluding Models Genesis for leakage is a careful choice.
- **Result:** Fine-tuned FMCIB AUC 0.944 vs fine-tuned Med3D 0.917, not significant (P = 0.134). Frozen FMCIB significantly beat the frozen baselines (P < 0.001).

## Eval 5 – Use case 3: 2-year survival in lung cancer (NSCLC), external cohorts
- **Analogy:** "Tuned in hospital A, driven in hospital B."
- **How:** Mainly frozen features + linear model (also fine-tuned baselines). Metrics: AUC, Kaplan–Meier log-rank, Cox.
- **Datasets:** Train/tune on HarvardRT (n = 291); test on LUNG1 (n = 420) and RADIO (n = 133), both external.
- **Competitors:** Supervised from scratch, transferred use-case-1 model, Med3D, Models Genesis (frozen and fine-tuned).
- **Fairness:** Same splits, re-run by the authors. No reduced-data runs, because the cohorts were already small.
- **Result:** Frozen FMCIB was best on LUNG1 (AUC 0.638), significant vs all except fine-tuned Med3D (P = 0.053). On RADIO, AUC 0.653, but only significant vs the supervised model (0.520). Risk groups separated with KM P < 0.001 (LUNG1) and P = 0.009 (RADIO). These are modest AUCs.

## Eval 6 – Stability (test–retest and reader variation)
- **Analogy:** Read the engine twice on the same car, or nudge where the driver points: does the reading stay the same?
- **How:** RIDER test–retest (26 patients, scans 15 minutes apart): ICC of features. Inter-reader simulation on LUNG1: 50 random shifts of the lesion seed point.
- **Competitors:** Supervised fine-tuned model.
- **Result:** ICC 0.984 (FMCIB) vs 0.966 (supervised). FMCIB features and predictions were significantly more stable under seed shifts (P < 0.05).

## Eval 7 – Saliency, embedding plots, efficiency (small)
- **Analogy:** Look inside the engine: which parts light up, and how readings cluster.
- **How:** Smooth guided backprop on fine-tuned models (qualitative). t-SNE and PCA of features. Time/memory of frozen vs fine-tuned.
- **Result:** Saliency focuses on the lesion and nearby tissue. Clusters look clearer for FMCIB (qualitative only). The frozen-feature route is cheaper than fine-tuning.

## Eval 8 – Biology link (gene expression)
- **Analogy:** Does the engine reading track something real under the hood?
- **How:** Correlate model predictions with RNA-seq (RADIO, GSE103584, n = 130), then pathway enrichment.
- **Competitors:** Supervised model.
- **Result:** FMCIB predictions correlated more with genes (P = 0.008), with immune pathways enriched (interferon, MHC-II, PD-1). The authors call this preliminary.

## Key takeaways
- The main story is the sealed engine: frozen features + a simple linear model often match or beat fully fine-tuned baselines, especially with little data.
- The baselines are fair: Med3D, Models Genesis and a random-init ResNet50 were all re-run on the same splits, and the authors excluded Models Genesis where its pretraining data leaked.
- It is a lesion-patch model, not a whole-scan model, so it needs a seed point at the lesion. Compare whole-volume encoders such as CT-FM, Merlin, SPECTRE or VoCo.
- There is no segmentation and no text. The tasks are classification and prognosis only.
- Gains over fine-tuned Med3D are often not significant (P = 0.059, 0.134, 0.053), and survival AUCs are only around 0.64–0.65.
- The stability and biology analyses are unusual extras that most foundation-model papers do not include.
