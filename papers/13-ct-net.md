# Paper 13 – CT-Net

- **Title:** Machine-Learning-Based Multiple Abnormality Prediction with Large-Scale Chest Computed Tomography Volumes (Draelos et al., 2020/2021, Medical Image Analysis)
- **Link:** https://arxiv.org/pdf/2002.04752
- **One line:** A supervised 3D chest CT classifier, not a self-supervised foundation model. An ImageNet-pretrained 2D ResNet-18 reads stacks of 3 adjacent axial slices. 3D convolutions then combine the features along the body, and fully connected layers predict 83 abnormalities at once. It is trained on the new RAD-ChestCT dataset (36,316 unenhanced chest CTs from 19,993 Duke patients). Labels are extracted automatically from reports by the SARLE tool. Image only (text is used only to make labels).
- **Eval types used:** fine-tuned-classification, random-init-baseline, ablation, other:label-extraction-accuracy, other:multi-label-vs-few-label-training

---

## Eval 1 – Label extraction quality (SARLE)
- **Analogy:** checking the answer key before grading the engine.
- **How:** SARLE is a rule-based labeler that reads radiology reports and outputs 83 abnormality labels. It was compared with manual labels on 427 test reports, for 9 labels. Metric: F-score.
- **Competitors:** The CheXpert labeler, on the labels both tools cover.
- **Fairness:** Same 427 reports for both tools.
- **Result:**
  - SARLE-Rules averages F = 0.976 and SARLE-Hybrid averages 0.930.
  - SARLE-Rules beats the CheXpert labeler on the overlapping labels except pleural effusion.

## Eval 2 – Main test: CT-Net-83 vs CT-Net-9 (multi-label benefit)
- **Analogy:** engine put in a car and tuned. Here the whole network is trained end-to-end for the task; there is no separate pretraining stage apart from the ImageNet start.
- **How:**
  - Training: the full network is trained on 25,355 training CTs with multilabel cross-entropy and SGD. This took about 15 days on 2 Titan XP GPUs.
  - Test: 7,209 CTs, split by patient, used only once after all development was finished.
  - Metrics: AUROC with 95% CIs and DeLong tests (Benjamini-Hochberg correction), plus average precision per label.
- **Datasets:** RAD-ChestCT, internal only (single institution, Duke).
- **Competitors:** The same architecture trained on only 9 labels (CT-Net-9).
- **Fairness:** Same architecture, data, and recipe. Only the number of labels differs.
- **Result:**
  - On the 9 shared labels, mean AUROC is 0.802 for CT-Net-83 vs 0.726 for CT-Net-9. CT-Net-83 wins on every label (e.g. pneumothorax 0.904 vs 0.816).
  - Over all 83 labels, mean AUROC is 0.773, and 18 abnormalities are above 0.90 (e.g. pacemaker/defibrillator 0.975, honeycombing 0.972).
  - The weakest labels include infiltrate (0.526) and scattered nodules (0.559).
  - Label frequency alone does not predict performance (heart failure at 0.2% prevalence reaches 0.952).
  - No mean AP over all 83 labels is reported.

## Eval 3 – Architecture comparison and ablations (small subset)
- **Analogy:** a dyno test of different engine designs, including a random-init baseline (broken-in vs brand-new engine).
- **How:** All models are trained on 83 labels with a random subset of 2,000 training and 1,000 validation scans. Validation AUROCs are shown as boxplots.
- **Competitors:**
  - BodyConv and 3DConv (alternative 3D architectures).
  - CT-Net-83 (Rand): the ResNet is randomly initialized instead of ImageNet-pretrained.
  - CT-Net-83 (Pool): max pooling replaces the 3D conv aggregation.
  - Single-label binary classifiers.
- **Fairness:**
  - Same data subset and all models were trained by the authors.
  - It is not stated whether BodyConv and 3DConv are faithful re-implementations of prior work.
  - Parameter counts are not reported.
  - The comparison is on validation data, not the test set.
- **Result:**
  - CT-Net-83 beats all the alternatives. ImageNet init beats random init.
  - The Pool variant did not converge.
  - Single-label binary classifiers also did not converge (AUROC about 0.5).
  - Exact numbers are only in a figure (Fig. 3).

## Not done
- No external validation, no saliency or localization maps, no segmentation, no few-shot experiments.
- 1,667 scans are held back "for future studies".

## Key takeaways
- This is a supervised baseline paper plus a dataset paper (RAD-ChestCT). It is not a self-supervised "engine" built for reuse. Its main lessons:
  - Training on many labels at once helps a lot, even for the labels you care about.
  - ImageNet 2D init helps 3D CT.
- Every evaluation is on Duke data. Nothing is "driven in hospital B". Labels come from automatic report mining, so they contain some noise (SARLE F about 0.93–0.98 on 9 checked labels).
- The test set was used only once, which is good practice. But the ablations sit on a small 2,000-scan subset and are reported only as boxplots.
- The RAD-ChestCT and 83-label idea later shaped CT foundation-model benchmarks. CT-RATE / CT-CLIP and Merlin's chest probes use similar multi-abnormality chest CT classification with report-mined labels. Merlin also uses an ImageNet-inflated ResNet, which is close in spirit to CT-Net's ImageNet init.
- Unusual point: single-label models failed to train at all. This suggests that 3D CT with weak volume-level labels needs the extra signal from many labels.
