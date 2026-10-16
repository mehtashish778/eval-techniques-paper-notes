# Paper 36 – NiftyNet

- **Title:** NiftyNet: a deep-learning platform for medical imaging (Gibson et al., 2017 arXiv / 2018, Computer Methods and Programs in Biomedicine; the venue is not printed in the fetched text)
- **Link:** https://arxiv.org/pdf/1709.03485
- **One line:** **Not a pretrained model.** NiftyNet is an open-source TensorFlow toolkit (a "car body" / garage) for medical-imaging deep learning. It has modules for data loading, sampling, augmentation, networks, losses, metrics and a model zoo. It supports segmentation, regression, generation and representation learning. No pretraining data.
- **Eval types used:** other:toolkit-demo-cases, fine-tuned-segmentation (trained from scratch in the toolkit), other:image-regression, other:image-generation-qualitative

---

## Eval 1 – Demo: abdominal multi-organ CT segmentation
- **Analogy:** A brand-new engine (dense V-Net) built and trained inside the NiftyNet car body. There is no pretrained engine.
- **How:** Trained from scratch, with 9-fold cross-validation. Network: dense_vnet. Modified Dice + hinge loss (written outside NiftyNet). 3000 iterations, affine augmentation. Metrics: Dice, relative volume difference, mean surface distance, HD95 (medians).
- **Datasets:** 90 abdominal CTs (public BTCV-style data + pancreas CT + in-house labels).
- **Competitors:** None.
- **Fairness:** Not applicable. There is no comparison.
- **Result:** Median Dice: liver 0.95, spleen 0.94, left kidney 0.93, stomach 0.87, gallbladder 0.79, pancreas 0.75, oesophagus 0.68, duodenum 0.62.

## Eval 2 – Demo: MRI-to-CT synthesis (for PET-MR attenuation correction)
- **Analogy:** The same car body used for a different job: turning MRI into CT.
- **How:** HighRes3DNet regression, RMSE loss, 5-fold cross-validation, 15k iterations on 80³ patches. Metrics: mean absolute error and mean error in HU. The dataset size is not stated.
- **Competitors:** pCT (Burgos et al. 2014, multi-atlas) and the Siemens UTE method.
- **Fairness:** Not stated whether the baselines were run on the identical folds. They appear to be comparisons on the same subjects, but details are not stated.
- **Result:** MAE 88 HU vs pCT 121 vs UTE 203. NiftyNet's model was clearly best.

## Eval 3 – Demo: ultrasound simulation with a conditional GAN
- **How:** A GAN trained **outside** NiftyNet (on 26,000 fetal-phantom ultrasound frames) was ported in and its weights loaded. Latent-space interpolation was shown.
- **Metrics:** None. Qualitative only. One example shows a smooth change. Another shows a sudden jump, which hints at mode collapse.

## Eval 4 – The platform itself
- No quantitative benchmark (no speed or memory tests, no comparison with other toolkits).
- The paper describes engineering quality instead: continuous integration, unit and system tests, versioning, pip install and a model zoo.

## Key takeaways
- This is a toolkit paper. Its "evaluation" is three worked demos showing the car body can carry different engines for different jobs (segmentation, regression, generation). It does not show that one engine is better than another.
- There is no pretraining, no frozen features, no label-efficiency test and no external validation. Only one demo has a competitor.
- It is closest in role to MONAI / nnU-Net (frameworks), not to the foundation models in this list. It later appears as a baseline (its model-zoo network) in JSSL's LUNA16 table (94.16 AUC), and as a baseline in Models Genesis-style papers.
- Useful as a contrast: it shows what "evaluating a framework" looks like (demonstration and software quality) versus evaluating a pretrained encoder.
