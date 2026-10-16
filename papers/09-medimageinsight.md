# Paper 09 – MedImageInsight

- **Title:** MedImageInsight: An Open-Source Embedding Model for General Domain Medical Imaging (Codella et al., 2024, arXiv preprint 2410.06542; venue not stated)
- **Link:** https://arxiv.org/pdf/2410.06542
- **One line:** A two-tower image+text model like CLIP, from Microsoft and derived from Florence. The image encoder is DaViT (~360M params) and the text encoder is ~252M params. Trained with UniCL (unified image–text–label contrastive learning) on ~3.79M images from 14 domains: chest X-ray, other X-ray, CT, MRI, mammography, ultrasound, echo, dermatology, pathology, OCT, fundus and more. Data mix PMC-15M, ROCO and proprietary data, with ~1,000 text labels. Mostly 2D; CT is handled slice by slice.
- **Eval types used:** zero-shot-text-classification (see caveat), frozen-kNN, image-image-retrieval, fine-tuned-classification, report-generation, random-init-baseline, external-validation, few-shot/label-efficiency (weak), robustness/stability (fairness subgroups), other:regression-bone-age, other:3D-retrieval-via-slice-median, other:ROC-threshold-tuning

---

## Eval 1 – Classification with the text head, across 14 domains
- **Analogy:** Two sealed engines: do the readings match? The image reading is compared against text readings of class names.
- **How:** Frozen. The text encoder turns class prompts into a classifier head and the image is matched to the closest one. Metric: mean AUC (mAUC), sometimes accuracy or balanced accuracy.
- **Caveat:** This is **not true zero-shot**. Those same label sets and datasets were part of UniCL training (image–label pairs). The test images are held-out splits from domains and labels the model was trained on.
- **Datasets (16):** LT-CXR, MGB CXR, IRMA, RSNA Mammography, LIDC-IDRI CT, exam-parameters, BUSI, HMC-QU, Bing echo, ISIC2019, SD-198, PAD-UFES-20, PCAM, OCT2017, OCT2018, ODIR-5K. Mostly test splits of training domains, not external.
- **Competitors:** "Reference SOTA" numbers for each dataset.
- **Fairness:** **Copied from other papers**, not re-run. The one exception is ISIC, where the authors recreated the test selection. Some references use extra inputs (e.g. PAD-UFES-20 uses clinical criteria).
- **Result:** Very high mAUCs on most sets (e.g. IRMA 0.989, BUSI 0.985, OCT2017 1.000, MGB CXR 0.937). LIDC-IDRI CT was weaker at 0.767. It lost to the reference on PCAM (0.963 vs 0.975) and PAD-UFES-20 (0.954 vs 0.973). It beat the reference on ISIC2019 (0.966 vs 0.954) and SD-198 accuracy (0.696 vs 0.578).

## Eval 2 – kNN classification and bone-age regression (frozen)
- **Analogy:** Engine sealed on a test bench (frozen kNN).
- **How:** Frozen image embeddings, k=20, similarity-weighted vote. Bone age is predicted by kNN regression (mean absolute error in months).
- **Datasets:** same 16 as Eval 1, plus RSNA and MGB bone age.
- **Competitors:** the same copied references, plus a human expert for bone age (7.32 months).
- **Fairness:** Copied numbers.
- **Result:** kNN is a bit below the text head on most sets (e.g. MGB CXR 0.839 vs 0.937, LIDC 0.718). Bone-age error was 6.19 months (RSNA) and 6.57 (MGB), better than the human reference of 7.32.

## Eval 3 – Fine-tuning on ChestX-ray14
- **Analogy:** Engine put in a car and tuned (lightly: early layers sealed).
- **How:** Early layers frozen, then fine-tuned for 800 iterations. AUC on edema, consolidation and effusion.
- **Datasets:** ChestX-ray14.
- **Competitors:** CXR Foundation, ELIXR.
- **Fairness:** Numbers copied from published work. ELIXR values were **read off a graph**.
- **Result:** About a tie with ELIXR, a chest-only model: 0.860 / 0.744 / 0.832 vs 0.856 / 0.749 / 0.835. It beat CXR Foundation (0.827 / 0.727 / 0.813).

## Eval 4 – Chest X-ray report generation (MIMIC-CXR)
- **Analogy:** Engine connected to a talking dashboard. Random-init version = brand-new vs broken-in engine.
- **How:**
  - Stage 1: a 6-layer text decoder is trained with the image encoder frozen (sealed).
  - Stage 2: the full model is fine-tuned.
  - Metrics: CheXbert micro-F1, RadGraph F1, BLEU-4, ROUGE-L.
- **Datasets:** MIMIC-CXR (public).
- **Competitors:** GPT-4o fine-tuned on MIMIC-CXR (re-run by the authors), plus copied numbers for LLaVA-Rad, MAIRA-1, Med-PaLM M and MAIRA-2. MAIRA-2 uses multiple images, so it is not like-for-like.
- **Fairness:** Mixed: one re-run competitor, the rest copied. A random-init ablation was done.
- **Result:** CheXbert F1 53.6 vs fine-tuned GPT-4o 48.9. GPT-4o was slightly better on RadGraph (26.4 vs 25.5), BLEU-4 and ROUGE-L. Random init dropped CheXbert F1 to 49.1, so pretraining helps. The model is ~0.4B params, under 10% the size of 7B+ competitors.

## Eval 5 – Independent clinical test at UW-Madison (external)
- **Analogy:** Tuned in hospital A, driven in hospital B, with the engine still sealed (kNN on the test bench).
- **How:** Frozen kNN, 64/16/20 split, mAUC and normal-vs-abnormal AUC. ROC curves shown.
- **Datasets:** 8,842 private X-rays from UW-Madison, not seen in training.
- **Competitors:** RAD-DINO, BiomedCLIP, Med-Flamingo.
- **Fairness:** **Re-run by the authors under the same protocol.** This is the fairest comparison in the paper.
- **Result:** mAUC 91.2 vs RAD-DINO 83.9, BiomedCLIP 80.0, Med-Flamingo 72.9. Abnormal-vs-normal AUC 96.7 vs 90.5 / 88.5 / 83.4. The abstract says "over 6 points" better; the table gap is 7.3.

## Eval 6 – Fairness by sex and age
- **Analogy:** Checking that the engine runs the same for every kind of driver.
- **How:** Subgroup mAUC or bone-age error.
- **Datasets:** the clinical set, LT-CXR, RSNA bone age.
- **Competitors:** none (within-model comparison).
- **Fairness:** n/a.
- **Result:** Small gaps. Clinical set: female 92.5 vs male 90.4. The text claims a 0.3-point gap, which does not match the table. Age bands ranged 89.8–94.1.

## Eval 7 – 3D CT retrieval (3D-MIR benchmark)
- **Analogy:** Image-image retrieval with a sealed engine. A 3D scan's reading is the median of its slice readings.
- **How:** Frozen. Median of slice embeddings, dot-product search. Precision@3/5/10 and AP.
- **Datasets:** 3D-MIR (public).
- **Competitors:** BiomedGPT, BiomedCLIP, Med-Flamingo.
- **Fairness:** Evaluated by the authors.
- **Result:** Tumour presence (macro): P@3 95.2, AP 95.4 vs best other 85.5 / 91.1. Tumour staging: AP 66.9 vs BiomedCLIP 64.6. It does not win every organ (BiomedGPT is better on liver staging AP, 55.4 vs 54.6).

## Smaller items
- **Data efficiency:** No proper few-shot curve. They only note some domains had about 100 training images (Bing echo, mAUC 0.941).
- **Qualitative image search:** ISIC2019 retrieval examples (figure only).
- **ROC thresholds:** They claim scores can be tuned for sensitivity or specificity. No operating-point numbers are given.

## Key takeaways
- A broad 2D generalist. CT appears only as slices (LIDC-IDRI classification, 3D-MIR via the median of slice embeddings). It is the weakest CT model in this list for true 3D work, compared with CT-specific models like CT-CLIP, Merlin, CT-FM, SPECTRE or TAP-CT.
- Watch the word "zero-shot": the "classification" uses the text head, but on label sets that were in training. It is closer to a trained classifier than CT-CLIP-style zero-shot.
- Most competitor numbers are **copied from other papers** (one read from a graph). The re-run comparisons are the UW-Madison external X-ray test (vs RAD-DINO, BiomedCLIP, Med-Flamingo), 3D-MIR and GPT-4o report generation. The UW-Madison test is the strongest evidence.
- Good mix of frozen (kNN / text head) and tuned evaluations, plus a random-init ablation, external validation and fairness subgroups. Small text-vs-table inconsistencies (gender gap, "over 6 points").
- No segmentation at all, unlike most CT models in this list (SuPreM, VoCo, PASTA, TAP-CT).

