# Paper 1 – SPECTRE (CT foundation model)

- **Title:** Scaling Self-Supervised and Cross-Modal Pretraining for Volumetric CT Transformers
- **Link:** https://arxiv.org/abs/2511.17209
- **One line:** A 3D vision transformer for CT, pretrained with DINO (self-supervised) + SigLIP (matching scans to reports), using only public data.

---

## Eval 1 – Cancer biomarker prediction (frozen + kNN)

- **Analogy:** engine sealed on a test bench; record its output, sort it.
- **How:** frozen encoder → one vector per tumor crop → kNN → AUC. Protocol from Pai et al.
- **Datasets:** malignancy – LUNA16, DLCS. 2-year survival – NSCLC-Radiomics, NSCLC-Radiogenomics, C4KC-KiTS, Colorectal-Liver-Metastases.
- **Competitors:** 11 CT foundation models (FMCIB, CT-FM, CT-CLIP, PASTA, VISTA3D, VoCo, SuPreM, Merlin, MedImageInsight, Models Genesis, SPECTRE).
- **Fairness:** authors ran **all** models on the same bench → cleanest comparison in the paper.
- **Result:** SPECTRE best on 4 of 6 datasets.

```
Tumor crop → [SPECTRE 🔒 frozen] → one vector ● → kNN vote → malignant? → AUC
```

## Eval 2 – Segmentation (fine-tuned)

- **Analogy:** engine swap. Only the network was replaced; the nnU-Net car body (prep, augmentation, loss, folds, evaluation) stayed the same.
- **Network (SEoMT):** SPECTRE encoder + one learnable query per class + small mask head. No U-Net decoder. Masks at 1/4 size, then upsampled.
- **Training:** whole encoder fine-tuned. AdamW, LR 1e-5, Dice + CE, deep supervision, crop 320×320×128. (SGD diverged / was worse.)

```
        ORIGINAL nnU-Net               SPECTRE version
   Prepare data     ✅ same        Prepare data
   ╔═══════════╗                  ╔══════════════════════╗
   ║  U-Net    ║  🔄 SWAPPED ───► ║ SPECTRE encoder      ║
   ╚═══════════╝                  ║ + small mask head    ║
                                  ╚══════════════════════╝
   Loss + training  ✅ same        Loss + training
   Evaluate         ✅ same        Evaluate
```

### Table 1 – KiTS23, LiTS, WORD (abdominal organs + tumors)

- Competitors: nnU-Net, ResEnc L, CoTr, nnFormer, SwinUNETRv2, UNETR, WaveFormer, Primus.
- Their numbers were **copied** from the Primus paper (Wald et al.), same splits → fair engine-only comparison.
- SPECTRE: 86.6 / 80.1 / 83.3 Dice; ResEnc L still a bit higher.

### Table 9 – TotalSegmentator (100+ body parts across the whole body)

- Competitors: SuPreM, CT-FM, Merlin, nnU-Net, SAM-Med3D.
- Numbers seem copied from each model's own paper with their own setups → rougher comparison.
- SPECTRE competitive; CT-FM a bit higher.

**Same SPECTRE setup in both tables** – only the dataset and competitors change.
**Note:** SPECTRE trained 150–300 epochs vs 1000 for nnU-Net ResEnc L.

## Eval 3 – Zero-shot text-to-image retrieval

- **Analogy:** two sealed engines (SPECTRE for images, Qwen3-0.6B for text). Do their readings match?
- **How:** give a report → find the closest CT scan by cosine similarity → is the right scan in the top K? (Recall@K). No extra training.
- **Datasets:** CT-RATE (validation split); Merlin (Findings→Image, Impressions→Image, both combined).
- **Competitors:** CT-CLIP, Merlin, OpenCLIP, BiomedCLIP.
- **Not reasoning** – it is matching words to images.
- *To check:* were these competitors re-run or copied?

## Key takeaways

- Bench test (kNN) = fairest proof the features are good.
- Segmentation = proves the features adapt well, but mixes in training effects.
- Retrieval = proves image and text live in the same space.
