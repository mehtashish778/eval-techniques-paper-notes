# Paper 16 – RadFM

- **Title:** Towards Generalist Foundation Model for Radiology by Leveraging Web-scale 2D&3D Medical Data (Wu et al., 2023, arXiv preprint; v5 Nov 2023 read, no venue listed on arXiv)
- **Link:** https://arxiv.org/pdf/2308.02463 (plus https://arxiv.org/html/2308.02463 for the transfer section)
- **One line:** Image+text generative model (about 14B params): a 12-layer 3D ViT encoder (2D images are padded to 4 slices so one encoder handles 2D and 3D) plus a 6-layer Perceiver (32 latent tokens) feeding MedLLaMA-13B; pretrained on MedMD (about 16M 2D/3D image-text pairs: PubMed figures, Radiopaedia, MedPix, MIMIC-CXR, VinDr, etc.) then instruction-tuned on RadMD (about 3M radiology-only samples). The paper does not say the ViT starts from pretrained weights.
- **Eval types used:** report-generation, VQA, other:modality-recognition-as-text, other:yes/no-disease-diagnosis-as-text, other:rationale-diagnosis, human/radiologist-rating, fine-tuned-classification, cross-task-transfer, other:unseen-class-zero-shot-diagnosis

---

## Eval 1 – RadBench: modality recognition and disease diagnosis (closed questions)
- **Analogy:** Engine connected to a talking dashboard. The dashboard is asked "what kind of scan is this?" or "does this scan show disease X? yes/no".
- **How:** No extra training. The model answers in text, and the answer is scored. Metrics: accuracy (ACC) and F1. Disease diagnosis covers more than 5,000 diseases, framed as yes/no questions.
- **Datasets:** RadBench test splits built from the same sources as training (PMC papers split by paper; Radiopaedia (RP3D) split by date, pre-2023 train vs later test; plus VinDr and others). The data sources are in-distribution. The test cases are held out, but this is not an external test.
- **Competitors:** OpenFlamingo (9B), MedVInT, Med-Flamingo, GPT-4V (run by hand through the chat website).
- **Fairness:** The authors ran the competitors themselves using the released checkpoints. Big handicap: the competitors are 2D-only. For 3D scans the three open models got only the central slice, and GPT-4V got 4 slices picked by radiologists. RadFM sees the full volume and was trained on data from the same sources as the test set. The other models never saw these sources. The main RadFM numbers are "zero-shot" (no examples in the prompt), and a few-shot prompting setting was reported only for OpenFlamingo and Med-Flamingo.
- **Result:** Modality ACC: RadFM 92.95% vs the best baseline (MedVInT) 84.25%. Disease diagnosis ACC: RadFM 80.62% vs about 50% (chance) for the baselines. Note: OpenFlamingo's modality number is reported inconsistently (49.41 / 49.91 / 49.47).

## Eval 2 – RadBench: VQA, report generation, rationale diagnosis (open-ended text)
- **Analogy:** Engine connected to a talking dashboard. The dashboard answers free-text questions, writes a report, or explains a diagnosis.
- **How:** No extra training. Generated text is scored with BLEU, ROUGE, BERT-Sim, and two new metrics the authors made, UMLS_Precision and UMLS_Recall (overlap of medical terms from the UMLS vocabulary).
- **Datasets:** VQA: VQA-RAD, SLAKE, PMC-VQA, PMC-CaseReport, RP3D-VQA. Reports: MIMIC-CXR, RP3D-Caption, MPx-Single, MPx-Multi. Rationale: RP3D-Rationale. All are in-distribution (VQA-RAD, SLAKE, PMC-VQA and MIMIC-CXR are also in the training mix).
- **Competitors:** Same as Eval 1.
- **Fairness:** Same handicaps as Eval 1 (2D-only competitors get 1 or 4 slices, and RadFM was trained on these sources). The authors also introduced the UMLS metrics themselves.
- **Result:** VQA UMLS precision: 31.77% vs 14.55% for MedVInT. Report generation UMLS precision: 22.49% vs 11.16% for the best baseline. Rationale diagnosis: only RadFM gives usable answers (BERT-Sim 68.47%). Some gap numbers in the text do not match the table.

## Eval 3 – Radiologist rating
- **Analogy:** Human driver test-drives each talking dashboard and scores it.
- **How:** Three radiologists scored 1,200 cases (400 per open-ended task) on a 0–5 scale. They also judged pairwise win/tie/lose.
- **Datasets:** Subsets of the RadBench open-ended tasks.
- **Competitors:** GPT-4V, OpenFlamingo, Med-Flamingo, MedVInT.
- **Fairness:** Same input handicaps as above. 22 GPT-4V cases were excluded because GPT-4V refused for safety reasons.
- **Result:** Average score: RadFM 2.17, GPT-4V 1.99, OpenFlamingo 1.40, Med-Flamingo 1.21, MedVInT 1.14. In head-to-head pairs RadFM was preferred over GPT-4V. Note that even the best score is low on a 0–5 scale.

## Eval 4 – Zero-shot diagnosis on unseen PadChest classes
- **Analogy:** Engine connected to a talking dashboard, asked about diseases it never heard of in training.
- **How:** Yes/no diagnosis questions for 163 PadChest classes not seen in training.
- **Datasets:** PadChest. External, and the classes are unseen.
- **Competitors:** Only RadFM's numbers are shown. The baselines are said to be at about chance (50%).
- **Fairness:** Thin comparison.
- **Result:** Reported as better than chance. Exact numbers were not captured in my read.

## Eval 5 – Fine-tuning on public benchmarks (Table 8)
- **Analogy:** Engine put in a car and tuned (diagnosis), or the whole talking dashboard tuned (VQA and reports).
- **How:** Diagnosis: the RadFM image encoder initializes a classifier. The paper does not say whether the rest is fully trained or a linear probe. VQA and report generation: the whole model is fine-tuned. Metrics: Macro-AUC/F1 for diagnosis, BLEU/ROUGE/F1 for VQA, BLEU-1..4 and ROUGE-L for reports.
- **Datasets:** VinDr-Mammo, CXR14, LDCT, BraTS2019 (the authors made a 7:3 split), VQA-RAD, SLAKE, PMC-VQA, IU-Xray. VinDr-Mammo, VQA-RAD, SLAKE and PMC-VQA are in RadFM's training data. CXR14, LDCT, BraTS2019 and IU-Xray are not named in training, but "CXR-Mix" is not detailed, so overlap there is unclear.
- **Competitors:** "Task-specific SOTA" numbers, one per dataset, from cited papers.
- **Fairness:** Mostly copied numbers from other papers. Exceptions: PMC-VQA (MedVInT re-tested on a cleaned test set) and BraTS2019 (the SOTA method was re-run on the authors' own split). There is no random-init baseline and no other foundation model fine-tuned under the same recipe.
- **Result:** Small wins on most benchmarks. VinDr-Mammo AUC 64.76 vs 64.5. CXR14 81.13 vs 80.1. LDCT 83.23 vs 82.1. BraTS2019 AUC 90.61 vs 88.06. VQA-RAD BLEU 73.44 vs 71.03. SLAKE BLEU 83.16 vs 78.6. PMC-VQA F1 48.50 vs 43.93 (but ROUGE was lower, 25.64 vs 27.20). IU-Xray is mixed: BLEU-1 and BLEU-4 are slightly below SOTA, and the others are slightly above.

## Eval 6 – Qualitative examples
- **Analogy:** Reading sample printouts from the dashboard.
- **How / Result:** Example VQA answers, reports and rationales. The authors note failures such as confusing tumor subtypes and missing laterality (left/right) in reports.

## Key takeaways
- This is a "talking dashboard" paper. Almost every test asks the whole model to produce text. The 3D encoder is never tested alone (no frozen kNN, no linear probe, no segmentation), so the engine's own quality is not isolated.
- The competitors were heavily handicapped. They are 2D models given 1 slice (or 4 for GPT-4V), while RadFM sees the full 3D volume and was trained on the same data sources as RadBench. Most of the RadBench win is in-distribution.
- "Zero-shot" here only means no examples in the prompt. It does not mean unseen data: RadBench comes from RadFM's own training sources. Only the PadChest unseen-class test is truly zero-shot.
- There are no ablations: no study of PMC-Inline, model size, or the 2D/3D mixing.
- The fine-tuning table compares mostly against SOTA numbers copied from other papers, and several of those datasets were in pretraining.
- Related papers in this list: CT2Rep (3D CT report generation, much narrower), Merlin and CT-CLIP (CT image-text), and MedImageInsight.
