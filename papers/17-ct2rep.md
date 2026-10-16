# Paper 17 – CT2Rep

- **Title:** CT2Rep: Automated Radiology Report Generation for 3D Medical Imaging (Hamamci et al., 2024, MICCAI 2024 per later citations; the arXiv v2 I read gives no venue)
- **Link:** https://arxiv.org/pdf/2403.06801
- **One line:** Image+text report generator trained from scratch for 3D chest CT. A 3D vision feature extractor (patches → spatial transformer → causal transformer) feeds a transformer encoder-decoder with relational memory and memory-driven conditional layer norm. It is trained on CT-RATE (about 25.7k non-contrast chest CTs from 21.3k patients, about 49k reconstructions). CT2RepLong adds the patient's previous scan and report through cross-attention.
- **Eval types used:** report-generation, other:clinical-efficacy-via-report-classifier, ablation, other:qualitative-examples

---

## Eval 1 – Report generation on CT-RATE (CT2Rep vs baseline)
- **Analogy:** Engine connected to a talking dashboard. The CT goes in, a written "findings" report comes out.
- **How:** The whole model is trained end to end (not frozen). There are two kinds of metric:
  - Text-overlap metrics: BLEU-1..4, METEOR, ROUGE-L.
  - "Clinical efficacy" (CE): precision, recall and F1 over 18 abnormalities. The labels are pulled out of the generated and real reports by a CXR-BERT text classifier fine-tuned for this.
- **Datasets:** CT-RATE (one hospital). 20,000 patients for training and 1,314 for validation, with no patient overlap. Only the findings section is used. This is an internal split from the same source. There is no external test. The paper says "test set" but describes only train/validation.
- **Competitors:** Only one baseline, built by the authors: CT-Net features fed into the same transformer. The paper says no earlier 3D report-generation method existed to compare against.
- **Fairness:** Self-made baseline with the same data and split. No 2D report methods were adapted, and no general models (RadFM, etc.) were compared.
- **Result:** CT2Rep beats the CT-Net baseline. BLEU-4 0.369 vs 0.354. METEOR 0.295 vs 0.286. ROUGE-L 0.459 vs 0.442. CE F1 0.534 vs 0.456, the largest gap.

## Eval 2 – Longitudinal reports (CT2RepLong) + ablation
- **Analogy:** Same talking dashboard, also shown last visit's scan and report.
- **How:** Trained end to end on patients with multiple visits (28,441 train / 1,689 val pairs). Same metrics. The ablation adds pieces step by step: no prior → previous report → previous volume → both (simple fusion) → full cross-attention fusion.
- **Datasets:** Longitudinal subset of CT-RATE (internal).
- **Competitors:** Its own ablation variants and CT2Rep.
- **Fairness:** Everything was trained by the authors on the same data.
- **Result:** Small gain. CE F1 0.536 vs 0.525 for the no-prior version, and about the same as CT2Rep. ROUGE-L dropped, which the authors blame on report length. The table has an apparent typo (BLEU-4 0.401 > BLEU-3 0.304).

## Eval 3 – Qualitative examples
- **Analogy:** Reading sample printouts.
- **How / Result:** Two single-case examples (figures), plus per-abnormality CE tables in the supplement. No radiologist rating.

## Key takeaways
- This is a task model, not a reusable foundation encoder. Its engine is trained from scratch only for report writing and is never tested alone (no frozen probe, no segmentation, no classification transfer).
- Evaluation is internal only (one dataset, train/validation split), against a single self-built baseline. There is no external validation and no human rating.
- The "clinical efficacy" score depends on a text classifier reading the reports. This is an automatic proxy for correctness, not a direct check.
- Some dataset counts in the paper are inconsistent (25,692 vs 25,701 volumes).
- Same group and dataset as CT-CLIP (CT-RATE). RadFM and Merlin also do report generation, and Merlin later compares on CT-RATE-style data. CT-Net is the baseline engine here.
