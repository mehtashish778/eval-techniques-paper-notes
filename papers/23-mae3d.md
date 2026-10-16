# Paper 23 – MAE3D

- **Title:** Masked Image Modeling Advances 3D Medical Image Analysis (Chen et al., 2022; WACV 2023, a venue not printed on the fetched arXiv page). This is the paper usually cited as "MAE3D".
- **Link:** https://arxiv.org/pdf/2204.11716 (matches; the arXiv title does not say "MAE3D", but the content is 3D masked image modeling: MAE and SimMIM)
- **One line:** A study of masked image modeling (MAE with an 8-layer transformer decoder, and SimMIM with a light linear/conv head) for 3D ViT, Swin3D and VAN3D encoders. Pretraining is small and in-domain: BTCV (24 CTs) plus TCIA-COVID19 (771 unlabeled chest CTs) for CT, and BraTS training (387 MRIs) for MRI. It is image-only.
- **Eval types used:** fine-tuned-segmentation, random-init-baseline, few-shot/label-efficiency, ablation, other:convergence-speed

---

## Eval 1 – BTCV multi-organ CT segmentation (fine-tuned)
- **Analogy:** "Engine put in a car and tuned." The pretrained ViT encoder goes into UNETR (ViT encoder + conv decoder), or into UPerNet for Swin/VAN, and everything is fine-tuned.
- **How:** Full fine-tuning, average Dice over 13 organs. Data handling uses PyTorch + MONAI.
- **Datasets:** BTCV: 24 train / 6 validation. Results are reported on the 6-case validation split, which is very small. The unlabeled BTCV training images were also used for pretraining (same domain).
- **Competitors:** Supervised baseline (no pretraining, i.e. the **random-init baseline**) and SimCLR contrastive pretraining. MAE and SimMIM are compared with each other across ViT-B/L, Swin3D-T/S/B and VAN3D-S/B.
- **Fairness:** One baseline row is copied from the UNETR paper, alongside the authors' own implementation. Whether SimCLR was re-run is not explicitly stated, though it appears to use the same set-up. No other medical SSL methods are compared (e.g., Models Genesis, Swin UNETR SSL).
- **Result:** The MIM methods beat both baselines. ViT-B: supervised 0.708, SimCLR 0.725, MAE 0.753, SimMIM 0.758. ViT-L: SimMIM 0.760. Swin3D and VAN3D also gain about 4 points with SimMIM (e.g., Swin3D-B 0.705 → 0.747).

## Eval 2 – BraTS brain tumor MRI segmentation (fine-tuned)
- **Analogy:** The same engine-in-a-car test on a different kind of road (MRI, 4 input channels).
- **How:** Pretrain on the BraTS training set only, then fine-tune. Dice averaged over TC/WT/ET on 97 validation images.
- **Competitors:** Supervised baseline and SimCLR.
- **Result:** ViT-B: supervised 0.756, SimCLR 0.774, MAE 0.805, SimMIM 0.808. Some table rows are labeled inconsistently.

## Eval 3 – Label efficiency, pretraining data size and resolution
- **Analogy:** "Tune with only a handful of examples." Fine-tuning uses 12 vs 24 labeled scans.
- **How:** MAE, ViT-B/16, 75% masking. Compares pretraining data (BTCV only vs COVID19+BTCV) and input resolution.
- **Result:** More pretraining data and higher resolution both help. With 50% labels (12 scans): 0.655 (BTCV only, low resolution) up to 0.702 (COVID+BTCV, high resolution). With 100% labels: 0.702 up to 0.753. There is no scratch baseline in this table.

## Eval 4 – Ablations of the masking recipe
- **Analogy:** Trying different ways of covering parts of the engine during break-in.
- **Result:** On BTCV-only pretraining, the best settings are MAE with patch 24 / ratio 0.75 (0.724) and SimMIM with patch 32 / ratio 0.15 (0.747). On BraTS, patch 16 with ratio 0.75 is best for both (about 0.80). The paper concludes that predicting raw voxels with a high mask ratio, small patches and a light decoder works well.

## Eval 5 – Convergence speed
- **Analogy:** How fast the broken-in engine gets up to speed vs a brand-new one.
- **Result:** MIM reaches Dice 0.7 with about 1.4× less training time than supervised from scratch. At 1.3k steps, SimMIM models score about 1.76× higher Dice (a figure only).

## Key takeaways
- This is a recipe-study paper, not a large foundation model. Pretraining data is tiny (a few hundred to about 800 volumes) and in-domain, and evaluation is on only two datasets (BTCV with a 6-case validation set, and BraTS).
- Every eval is "engine in a car and tuned" vs a brand-new engine or SimCLR. There are no frozen probes, no external hospital test, and no comparison with other medical SSL methods (**Models Genesis**, **TransVW**, **Swin UNETR SSL**, **PCRL**).
- Results come from very small validation sets with no repeated runs or error bars, so differences of 0.5 points are not reliable.
- It is the reference point for later 3D MAE work such as **GL-MAE** and the MIM-style encoder pretraining in **SegVol** (SimMIM on 96K CTs). There are also text/table number mismatches (e.g., the SimCLR value, weight decay and batch size).
