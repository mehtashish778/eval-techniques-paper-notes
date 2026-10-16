# Paper 3 – CT-FM (Vision Foundation Models for CT)

- **Title:** Vision Foundation Models for Computed Tomography (Pai, Hadzic et al., Mass General Brigham / Maastricht)
- **Link:** https://arxiv.org/abs/2501.09001
- **One line:** An **image-only** 3D model (SegResNet encoder) pretrained on 148,000 CT scans from the Imaging Data Commons with contrastive learning (a modified SimCLR). No reports, no labels.

**Two families of tests:**

- **Adaptation** (engine put in a car and tuned): whole-body segmentation, tumor segmentation, head CT triage.
- **Zero-shot** (engine sealed, just read its output): retrieval, clustering, semantic concept search, PCA colours, saliency, test-retest stability.

**Key competitor style here: "random init" baseline** = the *same* architecture trained from scratch, no pretraining.
> Analogy: two identical engines — one already broken in (pretrained), one brand new (random). Any difference = value of pretraining. This is the cleanest test of pretraining in this paper.

---

## Eval 1 – Whole-body segmentation (TotalSegmentator, 117 structures)

- **Analogy:** engine in a car, tuned.
- **How:** fine-tune CT-FM's SegResNet for 117 structures (1,228 scans). Dice.
- **Competitors:** random-init baseline and SuPreM — **run by the authors**. Auto3DSeg, VISTA3D, nnU-Net — **copied** from the VISTA3D paper (same split). Merlin — copied number.
- **Result:** CT-FM 0.898 Dice vs baseline 0.896 vs SuPreM 0.870; also above Auto3DSeg (0.882) and VISTA3D (0.893). Small gain over scratch, but better on 73.5% of structures (91.5% vs SuPreM). Baseline was better on the **organ** group.
- **Few-shot:** with only 5, 10, 20 training scans, CT-FM clearly better → pretraining helps most when labels are scarce.

## Eval 2 – Tumor segmentation (Medical Segmentation Decathlon)

- **Analogy:** engine swap into an existing car (**Auto3DSeg** framework) — same idea as SPECTRE in nnU-Net.
- **How:** Auto3DSeg with CT-FM weights vs Auto3DSeg default (same architecture, random init). Dice + Average Surface Distance (ASD).
- **Tasks:** liver tumor, liver tumor near vessels, pancreas tumor, lung tumor.
- **Result:** small Dice gains on liver (0.696 vs 0.681); pancreas Dice no difference but ASD much better (7.8 vs 13.8 mm). Gains are modest and confidence intervals overlap.
- **Note:** CT-FM used LR 5e-4, baseline the default 2e-4.

## Eval 3 – Head CT triage (normal vs abnormal)

- **Analogy:** engine + small dashboard (pool → 2 dense layers → 1 output), whole thing trained.
- **Train/test:** train on **SinoCT** (~9,000 scans), test on SinoCT test set, then test on **CQ500** (another hospital) **without retraining**.
- ⚠️ **"Zero-shot" here means something different:** not text prompts — it means "trained on hospital A, tested on hospital B with no extra training" (external validation).
- **Competitors:** random-init baseline, SuPreM — run by authors.
- **Result:** SinoCT – CT-FM AUC 0.836 vs scratch 0.802 vs **SuPreM 0.868 (best)**. CQ500 – CT-FM 0.794 ≈ SuPreM 0.793 > scratch 0.767. So CT-FM does **not** win here; it ties on the external set.

## Eval 4 – Content-based image retrieval (zero-shot)

- **Analogy:** one sealed engine; find scans with the closest readings.
- **How:** test-set embeddings search the train set; check if the retrieved scans share the label. Metrics: average precision, hit rate, F1 / precision at k = 3, 5, 10.
- **Datasets:** OrganMNIST3D (same body region?) and 3D-MIR (same lesion site? same lesion type?).
- **Result:** CT-FM slightly better than SuPreM (AP 0.932 vs 0.923) and the 3D-MIR baseline (lesion group precision 0.595 vs 0.534).

## Eval 5 – Understanding the embeddings (zero-shot, mostly qualitative)

- **Anatomical clustering:** t-SNE of embeddings, coloured by body region → both CT-FM and SuPreM form clear region clusters.
- **Semantic concept search (patch level):** take a patch (e.g. heart) in one scan, find the closest patch in another scan. Metric **organ centroid distance**: CT-FM 5.6 cm vs SuPreM 23.4 cm; 97.7% of matches landed in the heart vs 81.9%.
- **PCA colours:** first 3 principal components as RGB → heart red, lungs blue, bones green consistently across scans.
- **Saliency:** occlude regions, see which change the embedding most → lungs, heart, spine, pelvis.
- **Test-retest stability (RIDER):** same patient scanned twice → embeddings stay very similar.
- **Analogy:** opening the sealed engine's readings and checking they make sense (consistent, stable, pointing at real anatomy).

## Ablations (pretraining choices)

- Their "intra-sample" tweak (contrast patches within the same scan) improved SimCLR, SimSiam, VicReg.
- More crops per scan helped (micro Dice 0.654 → 0.732).
- Longer training isn't always better (best at epoch 449).

## Key takeaways

- **New fair-test idea:** compare against the *same architecture from scratch* → isolates the benefit of pretraining.
- Gains over scratch are **small with full data, bigger with few labels** (few-shot).
- **Doesn't win everything:** SuPreM beats CT-FM on head CT triage (internal test).
- **Watch the word "zero-shot":** in CT-CLIP it meant text prompts; here it means external testing or using frozen embeddings.
- Many "understanding" evals are **qualitative** (pictures) — nice for intuition, weaker as proof.
- Image-only model → no text tasks (no report retrieval, no zero-shot disease detection by prompt).
- Link: CT-FM is one of the competitors in SPECTRE's kNN biomarker test and its TotalSegmentator table.
