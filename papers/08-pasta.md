# Paper 08 – PASTA

- **Title:** A Synthetic Data-Driven Radiology Foundation Model for Pan-tumor Clinical Diagnosis (Lei et al., 2025, arXiv preprint 2502.06171; v3 Feb 2026 carries the longer title "A synthetic data–enabled artificial intelligence system for pan-tumour CT screening and diagnosis: a retrospective simulation study"; venue not stated)
- **Link:** https://arxiv.org/pdf/2502.06171 (checked against https://arxiv.org/abs/2502.06171, it is the right paper)
- **One line:** A 3D U-Net (nnU-Net style) with a small MLP head, pretrained only on **synthetic** CT. PASTA-Gen pastes rule-guided fake lesions into healthy organs of 10,767 real CT scans and refines them with a 3D diffusion model. This gives PASTA-Gen-30K: 30,000 synthetic image + lesion mask + structured-report triples covering 15 lesion types in 10 organs. Pretraining has two stages: (1) 25-class segmentation, then (2) lesion-attribute classification with the encoder frozen. It is image-only: the "text" is structured attribute labels, not free-text language.
- **Eval types used:** fine-tuned-segmentation, fine-tuned-classification, engine-swap-framework, random-init-baseline, few-shot/label-efficiency, external-validation, transfer-to-other-modality, human/radiologist-rating, report-generation, ablation, other:synthetic-image-realism-Turing-test, other:reader-study-with-AI-assistance, other:survival-prediction, other:tumor-staging

---

## Eval 1 – Is the synthetic data realistic? (radiologist Turing-style test)
- **Analogy:** Before testing the engine, check the fuel: experts sniff it to see whether the synthetic fuel smells like real fuel.
- **How:** No model test. 4 radiologists scored real and synthetic scans (1:1 mix) on a 1–5 realism scale. They also scored whether each synthetic report matched its image.
- **Datasets:** 1,180 real + synthetic scans for realism. 750 pairs for text–image consistency.
- **Competitors:** none (real scans are the reference).
- **Fairness:** not a competition.
- **Result:** Mean realism scores were 4.23–4.73 out of 5. Synthetic kidney cysts scored slightly *more* realistic than real ones. Report–image consistency scored 4.54–4.93.

## Eval 2 – Tumour detection on non-contrast CT (classification)
- **Analogy:** Engine put in a car and tuned. The no-pretraining version is the random-init baseline (broken-in vs brand-new engine).
- **How:** TotalSegmentator crops each organ, which is resized to 128³. The model is fine-tuned with a 2-class MLP head. Metric: AUC, 5-fold cross-validation.
- **Datasets:** CMU-NC (internal hospital data). Liver 323 tumour / 916 control, pancreas 338/974, kidney 340/973. These are real scans and were not seen in pretraining (pretraining used synthetic lesions on CMU templates).
- **Competitors:** PASTA without pretraining, Models Genesis, SuPreM.
- **Fairness:** The authors re-ran the competitors under the same protocol.
- **Result:** PASTA AUCs were liver 0.965, pancreas 0.987, kidney 0.964. Best competitor was SuPreM: 0.855, 0.944, 0.908. Models Genesis: 0.765, 0.774, 0.705. No pretraining: 0.759, 0.597, 0.781. Clear wins for PASTA.

## Eval 3 – Screening reader study (AI helps radiologists, non-contrast CT)
- **Analogy:** The engine drives a real car with a human co-driver. Does the human get there faster and more safely?
- **How:** 4 readers (2 junior, 2 senior) read for fixed 30-minute sessions, alone or with PASTA-AID. Metrics: cases read, recall, precision. CMH test stratified by reader.
- **Datasets:** 1,080 internal scans (retrospective simulation).
- **Competitors:** the same radiologists without AI.
- **Fairness:** No case overlap between sessions. Retrospective and simulated, not a prospective trial.
- **Result:**
  - Throughput rose: liver +25.1%, pancreas +11.1%, kidney +18.7%.
  - Recall rose: liver 73.5→90.5%, pancreas 61.3→92.7%, kidney 68.2→90.8% (all significant).
  - Precision rose: liver 71.3→96.2% and kidney 81.2→96.3% (significant). Pancreas 80.3→90.8% was not significant (p=0.279).

## Eval 4 – Full-data lesion segmentation
- **Analogy:** Engine swap: the PASTA engine goes into the nnU-Net car body and is tuned.
- **How:** Fine-tuned inside the nnU-Net framework, 5-fold cross-validation, Dice.
- **Datasets:** 15 lesion tasks. Public: MSD (lung, liver, pancreas tumour/cyst, colon), KiTS23 (kidney tumour/cyst), ATLAS. Internal: CMU-LSSR (8 lesion types, e.g. oesophagus, stomach, gallstone, bone metastases).
- **Competitors:** plain nnU-Net (random init), Universal Model, Models Genesis, SuPreM.
- **Fairness:** All re-run by the authors with the same splits and recipe.
- **Result:** PASTA Dice ranged 0.433–0.814. It beat the next-best model on most tasks, e.g. lung +1.9%, liver +2.3%, oesophagus +3.7%, stomach +4.6%, bone metastases +4.4%. Plain nnU-Net was marginally better only on gallstone. With full labels, the margins are small.

## Eval 5 – Few-shot lesion segmentation
- **Analogy:** Tune with only a handful of examples.
- **How:** Same setup as Eval 4 but with only K = 1, 2, 4, 8 or 16 training scans, capped at 2,000 iterations. 5 folds × 5 repeats. Dice.
- **Datasets:** same lesion tasks as Eval 4.
- **Competitors:** nnU-Net, Universal Model, Models Genesis, SuPreM.
- **Fairness:** All re-run with the same budget.
- **Result:** Gaps here are much bigger: +0.025 to +0.463 Dice over baselines. Lung 1-shot: PASTA 0.529 vs SuPreM 0.302, nnU-Net 0.188, Models Genesis 0.179, Universal 0.167. Lung 16-shot: 0.706 vs nnU-Net 0.591.

## Eval 6 – Segmentation-assist reader study (contrast CT)
- **Analogy:** The engine drafts the route and the driver corrects it.
- **How:** PASTA-AID (5-model ensemble) drew draft masks and radiologists refined them. Time and Dice were measured with a linear mixed-effects model.
- **Datasets:** 600 internal patients (200 each for liver, pancreas, kidney). 4 readers.
- **Competitors:** manual annotation without AI.
- **Fairness:** Retrospective simulation.
- **Result:** Annotation time fell by up to 74.5% for seniors and 78.2% for juniors. Junior Dice against senior gold standard rose 11.8–12.2%.

## Eval 7 – Structured report generation (lesion attributes)
- **Analogy:** Engine connected to a (very simple) talking dashboard. The dashboard only fills in a fixed form, not free text.
- **How:** Fine-tuned attribute classification on a 96³ lesion-centred patch. Five attributes: shape, density, heterogeneity, surface, invasion. Accuracy and F1, 5-fold cross-validation.
- **Datasets:** 1,535 real internal scans, 15 lesion types (eTable 5 says 2,089, so the paper is inconsistent).
- **Competitors:** Models Genesis, SuPreM.
- **Fairness:** Re-run by the authors.
- **Result:** PASTA was significantly better on shape, density, heterogeneity and surface. It was similar on invasion. Exact numbers are in supplementary eTable 12, which was not in the text I read.
- **Reader-study add-on:** Writing reports with PASTA-AID cut reporting time by 15.7–36.5%. Junior–senior agreement improved, but not significantly.

## Eval 8 – Tumour staging (classification)
- **Analogy:** Engine put in a car and tuned (for a new job: staging).
- **How:** FMCIB-style region-of-interest crop around the tumour centre, encoder adapted, AUC.
- **Datasets:** Internal CMU gastric (stage I–II vs III–IV) and CMU rectal (I–III vs IV). Public TCGA-BLCA bladder (I–II vs III–IV, 120 cases).
- **Competitors:** Models Genesis, SuPreM, FMCIB.
- **Fairness:** Re-run by the authors.
- **Result:** PASTA AUC ranged 0.738–0.855. It beat the next best by +0.029 (gastric), +0.092 (rectal) and +0.166 (bladder).

## Eval 9 – Survival prediction
- **Analogy:** Engine put in a car and tuned. Lung1 is "tuned in hospital A, driven in hospital B" only in the sense that it is a public outside cohort.
- **How:** Binary survival at a cut-off (2 years for lung and gastric, 3 years for rectal and bladder). AUC.
- **Datasets:** Lung1 (public, 422 cases), CMU gastric, CMU rectal, TCGA-BLCA.
- **Competitors:** Models Genesis, SuPreM, FMCIB.
- **Fairness:** Re-run by the authors.
- **Result:** PASTA AUC ranged 0.660–0.878. The biggest gaps over FMCIB were on gastric and rectal.

## Eval 10 – Transfer to MRI (few-shot)
- **Analogy:** An engine built for a petrol car (CT) is put in a different vehicle (MRI) and tuned with a handful of examples.
- **How:** Fine-tuned few-shot segmentation, K = 1–16, Dice.
- **Datasets:** MSD-Brain (484 patients, multi-sequence MRI) and ATLAS liver-tumour MRI (60 patients). Neither was seen in pretraining (CT only).
- **Competitors:** same set as the segmentation evals (SuPreM is the one quoted).
- **Fairness:** Re-run by the authors.
- **Result:** Brain MRI at 16 shots: Dice 0.504, significantly above SuPreM. Liver MRI: Dice 0.603, reported as 13.4% above SuPreM.

## Ablation (small)
- The only ablation in the main text is PASTA with vs without pretraining on non-contrast detection (Eval 2).
- There are **no** ablations of the diffusion refiner, the two-stage pretraining, or synthetic vs real pretraining data.

## Key takeaways
- The headline claim is "best on 45 of 46 tasks". Every competitor (Models Genesis, SuPreM, Universal Model, FMCIB, nnU-Net) was re-run by the authors under the same recipe, so the comparison is fair in that sense. However, nearly every test is "engine put in a car and tuned". There is no frozen-engine (kNN or linear probe) test, so we cannot judge raw embedding quality.
- The big wins are in **few-shot** and **non-contrast detection**. With full labels in nnU-Net, gains are only about 1–5% Dice. This is the usual pattern: pretraining matters most when labels are scarce.
- "Report generation" is really **multi-attribute classification** (fill in a 5-field form). It is not free-text generation like CT2Rep, RadFM or Merlin. The Results section even calls stage 2 "vision–language alignment", but the Methods describe it as attribute classification.
- Unusual: the paper has a strong clinical evaluation, with a radiologist realism test and simulated reader studies measuring time, recall and precision. Most models in this list (VoCo, SuPreM, CT-FM, TAP-CT) do not do this.
- Weaknesses:
  - Most real test data are internal (CMU), and the templates came from the same hospital.
  - The paper has internal inconsistencies: the liver AUC gain is 0.095 in the text vs 0.110 in the table; the dataset size is 1,535 vs 2,089; ATLAS is called CT in one place and MRI in another.
  - Key tables are in supplementary material.
- Links: it uses TotalSegmentator for organ crops, the nnU-Net car body like SuPreM, and an FMCIB-style staging protocol.

