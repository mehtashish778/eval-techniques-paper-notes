# Paper 33 – GVSL

- **Title:** Geometric Visual Similarity Learning in 3D Medical Image Self-supervised Pre-training (He et al., 2023, CVPR 2023; the venue is not printed in the fetched text)
- **Link:** https://arxiv.org/pdf/2303.00874
- **One line:** A 3D U-Net (GroupNorm) pretrained by learning to **register** (align) two different patients' scans. A "Z-Matching" head (affine + deformable) predicts a deformation field, trained with NCC + smoothness loss. Self-restoration (Models Genesis-style) runs alongside as a warm-up. Pretraining data: 302 unlabelled cardiac CT angiography scans (single scanner). Image-only.
- **Eval types used:** linear-probe, fine-tuned-segmentation, fine-tuned-classification, random-init-baseline, transfer-to-other-modality, cross-task-transfer, ablation, embedding-visualization, other:convergence-curves

---

## Eval 1 – Linear evaluation (frozen backbone) on 4 tasks
- **Analogy:** Engine sealed on a test bench. Only a tiny reading layer is trained.
- **How:** The backbone is frozen. Segmentation uses a conv + softmax on the decoder's local features. Classification uses a linear layer + sigmoid on the encoder's global features. Note that the frozen part includes the U-Net **decoder**, not only the encoder. Metrics: Dice (segmentation), AUC (classification).
- **Datasets (all different from pretraining data, none from the same scanner set):**
  - SHC: MM-WHS 2017 cardiac CT, 7 heart structures (15/5/40). The test set is scored by the official challenge.
  - SAC: ASOCA 2020 coronary arteries, small vessels (15/5/20).
  - CCC: STOIC chest CT, COVID-19 diagnosis (1000/400/600).
  - SBM: CANDI brain T1 MRI, 28 tissues (40/20/43). This is a new body region and a new modality.
- **Competitors:** Denoising, In-painting, Models Genesis, Rotation, DeepCluster, SimSiam, BYOL, SimCLR, plus scratch (random init).
- **Fairness:** All methods use the same 3D U-Net and "the same basic training setting". This implies they were re-trained by the authors, though it is not stated outright. Good apples-to-apples. The only handicap is that all were pretrained on the small 302-scan set.
- **Result:** GVSL got Dice/AUC 68.4 / 28.7 / 60.8 / 79.9 (SHC / SAC / CCC / SBM). Best competitor per column: 56.5 (SimSiam) / 22.5 (Models Genesis) / 62.1 (Rotation) / 67.5 (DeepCluster). Scratch: 21.9 / 10.0 / 52.7 / 56.4. GVSL wins big on segmentation (+11.9 on SHC) but **loses** to Rotation on COVID classification.

## Eval 2 – Fine-tuning on the same 4 tasks
- **Analogy:** Engine put in a car and tuned, compared against a brand-new engine (scratch).
- **How:** The whole network is fine-tuned with the same tiny heads. 4×10⁴ iterations. The best checkpoint on the validation set is kept.
- **Result:** GVSL 91.2 / 81.3 / 82.2 / 90.0. Best competitor 90.6 / 80.7 / 80.7 / 89.8. Scratch 87.8 / 80.4 / 74.4 / 89.7. The gaps shrink a lot once tuned (under 1 point on SAC and SBM). On brain MRI everyone is roughly equal to scratch.

## Eval 3 – Ablations (same table)
- Removing Z-Matching, the self-restoration warm-up, or the affine head.
- Without self-restoration, frozen SAC drops to 0.0 Dice, and the NCC loss does not converge (Fig. 8). This shows the registration task alone cannot train from scratch.
- Removing the affine head hurts COVID classification (about 3 AUC frozen). Oddly, it gives the best fine-tuned SAC score (82.3).

## Eval 4 – Convergence and t-SNE
- **Analogy:** Engine output readings plotted on a map (t-SNE).
- **Convergence curves (SAC, SBM):** GVSL learns faster than scratch and Models Genesis. The paper claims over 30% faster convergence. Scratch converges faster on SBM but overfits.
- **t-SNE** of local features for 7 heart structures: GVSL forms the cleanest clusters.

## Key takeaways
- It is one of the few SSL papers in this list that reports **both** frozen (test-bench) and fine-tuned (in-car) results side by side, with all methods re-trained on one backbone. That makes it a fairly fair comparison.
- The frozen-probe gains are large, but fine-tuning gains are small. This pattern repeats across SSL papers: pretraining matters most when the engine is sealed or labels are few.
- The pretraining set is tiny and narrow (302 cardiac CTAs). SHC and SAC are the same anatomy (heart CT), so "in-scene" tasks are close to the pretraining domain. Brain MRI is the real transfer test, and there fine-tuning gains vanish.
- There is no label-fraction sweep, no external hospital test and no comparison with big models (e.g. Swin UNETR SSL or SuPreM). The baselines are older generic SSL methods plus Models Genesis.
- It is listed as a competitor ("GLSV") in GL-MAE. Its registration idea differs from the reconstruction family (Models Genesis, TransVW, MAE3D) and the contrastive family (PCRL).
