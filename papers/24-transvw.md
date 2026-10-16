# Paper 24 – TransVW (Transferable Visual Words)

- **Title:** Transferable Visual Words: Exploiting the Semantics of Anatomical Patterns for Self-supervised Learning (Haghighi et al., 2021, IEEE Transactions on Medical Imaging; the venue is not printed in the arXiv text, so it comes from outside knowledge)
- **Link:** https://arxiv.org/pdf/2102.10680
- **One line:** A self-supervised 3D U-Net (about 23M params) for CT, plus a 2D U-Net with a ResNet-18 encoder for X-ray. Pretraining jointly does two things: (1) classifies "visual words", which are anatomically consistent patches found across similar patients by nearest-neighbour search in an autoencoder's latent space, and (2) restores those patches after Models-Genesis-style corruptions. Pretrained on 623 LUNA16 chest CTs (3D) and 76K ChestX-ray14 images (2D). Image-only.
- **Eval types used:** fine-tuned-classification, fine-tuned-segmentation, random-init-baseline, few-shot/label-efficiency, linear-probe, engine-swap-framework, transfer-to-other-modality, cross-task-transfer, ablation, leaderboard, other:convergence-speed, other:add-on-to-other-SSL-methods, embedding-visualization (visual-word examples)

---

## Eval 1 – Fine-tuning on five 3D target tasks (main table)
- **Analogy:** Engine put in a car and tuned, five different cars. Also compared with a brand-new engine (random init) and with other broken-in engines.
- **How:** Fine-tuned (all weights). The tasks are 3D classification (AUC) and 3D segmentation (IoU/Dice). Each method was run 10 times, and results are mean ± SD with a t-test (p = 0.05). Early stopping on validation.
- **Datasets:**
  - **NCC:** lung nodule false-positive reduction, LUNA16, AUC.
  - **NCS:** lung nodule segmentation, LIDC-IDRI, IoU.
  - **ECC:** pulmonary embolism false-positive reduction, PE-CAD CT, AUC.
  - **LCS:** liver segmentation, LiTS-2017, IoU.
  - **BMS:** brain tumour segmentation, BraTS2018 FLAIR MRI, IoU.
  - **Seen vs unseen:** LUNA16 is the pretraining source, but target val/test images were excluded from pretraining. The other datasets (and MRI as a modality) were not seen.
- **Competitors:**
  - Random init.
  - Self-supervised: Models Genesis, Rubik's Cube, and 3D versions of Inpainting, Context Restoration and Rotation.
  - Supervised pretrained: NiftyNet, MedicalNet (Med3D), I3D.
- **Fairness:**
  - Every row has an SD over 10 runs, which suggests the authors re-ran the baselines with the same splits. Some footnotes refer to published numbers, and the paper does not clearly say for every baseline whether it was re-run.
  - Architectures differ. MedicalNet is ResNet-101 (about 86M params) and I3D's encoder about 25M, versus TransVW's 23M 3D U-Net.
- **Result (IoU/AUC, %):**

  | Task | TransVW | Models Genesis | Random init |
  |---|---|---|---|
  | NCC | 98.46 | 98.07 | 94.25 |
  | NCS | 77.33 | 77.41 | 74.05 |
  | ECC | 87.07 | 87.20 | 80.36 |
  | LCS | 86.53 | 85.10 | 79.76 |
  | BMS | 68.82 | 67.96 | 59.87 |

  - TransVW is best or tied (statistically) on all five tasks.
  - Models Genesis is a hair higher on NCS and ECC, but the difference is not significant.
  - TransVW beats all supervised pretrained models (NiftyNet, MedicalNet, I3D) on all five.
  - Rubik's Cube is clearly weaker, for example LCS 75.59 and BMS 62.75.

## Eval 2 – Ablation of pretext parts (rows in the main table)
- **Analogy:** Testing which parts of the break-in routine matter.
- **How:** Same fine-tuning setup, using restoration only, classification only, or both.
- **Result:**
  - Restoration alone beats classification alone on NCC and BMS. The two are similar on the other tasks.
  - The combination is best on 4 of 5 tasks.
  - Restoration-only is numerically top on NCS (77.70).

## Eval 3 – Add-on to other SSL methods
- **Analogy:** Bolting the visual-word part onto other engines' break-in routines to see whether it helps them too.
- **How:** Visual-word self-discovery and self-classification were added to Rotation, Context Restoration, Inpainting and Models Genesis. Each was fine-tuned on the 5 tasks.
- **Result (figure only, approximate):**
  - Rotation: +1.5 to +5%.
  - Context Restoration: +1 to +8%.
  - Inpainting: +1.5 to +6%.
  - Models Genesis: +1 to +1.5% on some tasks.
  - The gains are reported as significant (p < 0.05).

## Eval 4 – Convergence speed
- **Analogy:** How quickly the tuned engine reaches top speed.
- **How:** Learning curves during fine-tuning.
- **Result:** TransVW converges faster than both random init and Models Genesis on all 5 tasks. The comparison is shown as plots; no exact epoch counts are given.

## Eval 5 – Annotation cost (label efficiency)
- **Analogy:** Tune with fewer labelled examples.
- **How:** Fine-tune with growing fractions of the target labels and find how much data matches the from-scratch model trained on all of it.
- **Result:**
  - Versus random init, TransVW saves about 50% of labels (NCC), 50% (NCS), 57% (ECC), 60% (LCS) and 80% (BMS).
  - Versus Models Genesis, it saves roughly 17–50% on some tasks and is equal on others. The paper's text lists the task names for these savings inconsistently.

## Eval 6 – Engine swap into nnU-Net (appendix)
- **Analogy:** Engine swap: the TransVW engine goes into the nnU-Net car body.
- **How:** nnU-Net framework on the Decathlon liver tumour task, 5-fold cross-validation, Dice.
- **Result:**
  - Cross-validation Dice: TransVW 66.06, Models Genesis 65.03, nnU-Net from scratch 63.52.
  - Official Decathlon leaderboard: TransVW 0.77 versus 0.76 for nnU-Net from scratch.

## Eval 7 – 2D version: linear probe + fine-tuning (X-ray)
- **Analogy:**
  - Linear probe: engine sealed on a test bench.
  - Fine-tuning: engine put in a car and tuned.
- **How:**
  - TransVW 2D (ResNet-18) pretrained on ChestX-ray14.
  - Linear evaluation of frozen features from layers res1 to res4 on DXC (ChestX-ray14 14-disease classification) and NCC.
  - Full fine-tuning on DXC, NCC and PXS (SIIM-ACR pneumothorax segmentation).
  - Metrics: AUC and Dice.
- **Competitors:** Random init, Models Genesis 2D, ImageNet supervised.
- **Fairness:** Same backbone for all methods. ImageNet saw far more (natural) images.
- **Result (figures only):**
  - **Linear probe:** TransVW 2D beats Models Genesis 2D and ImageNet at every layer.
  - **Fine-tuning:** TransVW beats scratch and Models Genesis 2D. It ties ImageNet on NCC and PXS, but ImageNet wins on DXC.
  - The authors argue that linear-probe scores correlate poorly with fine-tuning scores.

## Eval 8 – Hyperparameter ablation + visualization
- **Analogy:** Tuning the break-in recipe, and looking at what the engine learned.
- **How:**
  - Ablation: vary the number of visual-word classes C.
  - Visualization: show example visual words (each with 10 instances) and the self-discovery process.
- **Result:**
  - C = 45 is best across the 3D tasks.
  - The visual-word figures are qualitative only; they show that the same anatomy recurs across patients.

## Key takeaways
- The evaluation is almost entirely "engine put in a car and tuned" on small, single-organ, mostly chest-CT tasks. It is strong on statistics (10 runs, t-tests), which is rare in this list.
- The gain over its predecessor Models Genesis (same lab, same backbone, same LUNA16 pretraining) is small and often not significant. The bigger wins are against random init and against Rubik's Cube / supervised Med3D (MedicalNet), NiftyNet and I3D.
- The pretraining set is tiny (623 CTs) compared with later models (SuPreM, VoCo, CT-FM). There is only one MRI task (BraTS) and no external-hospital test.
- The nnU-Net engine swap and the Decathlon leaderboard entry are a nice "real framework" check, but the gain is only +0.01 Dice on the leaderboard.
- The 2D linear-probe vs fine-tune disagreement is a useful lesson: test-bench readings (frozen probes) do not always predict on-the-road performance. This is relevant when comparing with frozen-eval models like SPECTRE and CT-FM.
- The direct successors are PCRL and PCRLv2. Rubik's Cube and ROT-style tasks appear here as baselines.
