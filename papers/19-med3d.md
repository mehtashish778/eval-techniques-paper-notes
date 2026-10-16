# Paper 19 – Med3D / MedicalNet

- **Title:** Med3D: Transfer Learning for 3D Medical Image Analysis (Chen et al., 2019, arXiv preprint)
- **Link:** https://arxiv.org/pdf/1904.00625
- **One line:** 3D ResNet encoders (ResNet-10 to 200) pretrained with supervised segmentation on "3DSeg-8": 8 public CT/MRI segmentation datasets, about 1,638 volumes in total. A separate small decoder branch per dataset handles the fact that each dataset labels different organs. Image-only. The decoders are thrown away after pretraining.
- **Eval types used:** fine-tuned-segmentation, fine-tuned-classification, random-init-baseline, other:video-pretrained-baseline (Kinetics), ablation, other:convergence-speed

---

## Eval 1 – Pretraining ablations (data amount and number of domains)
- **Analogy:** Checking how well the engine breaks in when given more or fewer kinds of fuel.
- **How:** ResNet-152, segmentation Dice on the 3DSeg-8 test splits (90/10). They varied the fraction of training data (10–100%) and the number of datasets used (1, 2, 4, 8).
- **Datasets:** 3DSeg-8 (seen during pretraining; internal splits).
- **Competitors:** Itself.
- **Result:** More data and more domains help. Heart Dice 86.73% → 91.05% going from 1 to 8 domains. Brain and vessel gained little (about 0.5 points). The data-fraction trend is shown only in a figure.

## Eval 2 – Lung segmentation (Visceral)
- **Analogy:** Engine put in a car and tuned; also random-init baseline (broken-in vs brand-new engine); also an engine broken in on video (Kinetics).
- **How:** The pretrained encoder initializes a segmentation network, then everything is fine-tuned (no freezing mentioned). Metric: Dice.
- **Datasets:** Visceral lung, 72 train / 8 test volumes. Not in 3DSeg-8 (a new task).
- **Competitors:** Train from scratch (TFS) and Kinetics-pretrained 3D ResNets (Hara et al.), at each depth.
- **Fairness:** Same architecture for all, but the optimizer settings are described inconsistently (Adam 0.001 for pretrained vs 0.01 for scratch in one place, SGD 0.1 elsewhere). Single runs. The test set is tiny (8 volumes).
- **Result:** Med3D 87.16–93.82% Dice across depths. Kinetics up to 88.61%. Scratch 70.66–76.82%.

## Eval 3 – Pulmonary nodule classification (LIDC)
- **Analogy:** Engine put in a car and tuned (classification); random-init baseline.
- **How:** Fine-tuned encoder + classifier. Metric: accuracy.
- **Datasets:** LIDC, 1,050 train / 99 test nodules. Not in pretraining.
- **Competitors:** Scratch and Kinetics-pretrained, at each depth.
- **Fairness:** Same caveats. With 99 test nodules, one error moves accuracy by about 1%.
- **Result:** Med3D 86.87–91.92%. Best Kinetics 83.84%, best scratch 84.85%. Deeper scratch and Kinetics models got worse, while Med3D did not.

## Eval 4 – Liver segmentation (LiTS)
- **Analogy:** Engine put in a car (a DenseASPP head) and tuned, then compared with leaderboard entries.
- **How:** ResNet-152 + DenseASPP, fine-tuned, single model, no post-processing. Metrics: Dice and ASSD.
- **Datasets:** LiTS. Overlaps with pretraining: the "Liver" dataset in 3DSeg-8 is the same LiTS data, so this is not unseen.
- **Competitors:** Scratch, Kinetics, and LiTS challenge entries plus 3D AH-Net and H-DenseNet (numbers copied from papers or the leaderboard).
- **Fairness:** The leaderboard numbers are copied, and those methods use their own pipelines and post-processing.
- **Result:** Med3D Dice 94.6% (ASSD 1.9), Kinetics 89.6% (5.0), scratch 66.0% (30.4). It is below the top challenge and published methods (96.0–96.6% Dice), so it does not win the leaderboard.

## Eval 5 – Convergence speed
- **How / Result:** The abstract claims 2x faster than Kinetics and 10x faster than scratch, with 3–20% accuracy gains. The body only shows training-loss curves, with no numeric speed measurement.

## Key takeaways
- This is a classic "broken-in vs brand-new engine" paper: supervised pretrained init vs scratch vs video-pretrained init, all fully fine-tuned. There are no frozen or linear-probe tests, so the engine is never tested sealed.
- The evaluations are small (8 lung test volumes, 99 nodules), use single runs, and report no variance. The LiTS test overlaps with pretraining data.
- The baselines are fair in architecture (same ResNet), but the optimizer recipes are unclear. Leaderboard comparisons use copied numbers, and Med3D loses them.
- It is an early, supervised alternative to the self-supervised methods in this list (Models Genesis, TransVW, Rubik's Cube, PCRL, Swin UNETR SSL, VoCo). Those papers often use Med3D/MedicalNet as a pretrained baseline. SuPreM later revisits supervised segmentation pretraining at much larger scale.
