# Paper 12 – MedCLIP

- **Title:** MedCLIP: Contrastive Learning from Unpaired Medical Images and Text (Wang et al., 2022, EMNLP 2022)
- **Link:** https://arxiv.org/pdf/2210.10163
- **One line:** 2D chest X-ray image-text model. Image engine is a Swin Transformer (ImageNet init; a ResNet-50 version is also reported); text engine is BioClinicalBERT. Both map to a 512-d shared space. It is trained with a "semantic matching" loss that does not need paired image-report data. Pretraining data: MIMIC-CXR (377K images, 201K reports) plus CheXpert (223K images with labels, no reports). Image + text.
- **Eval types used:** zero-shot-text-classification, linear-probe, image-text-retrieval, few-shot/label-efficiency (pretraining-data efficiency), embedding-visualization, external-validation (COVID, RSNA not used in pretraining)

---

## How the pretraining works (needed to understand the evals)
- Report sentences and image labels go through MetaMap/UMLS. Each becomes a 14-finding multi-hot label vector.
- The "target" similarity between any image and any sentence is the similarity of their label vectors. Images and texts do not need to come from the same patient (they are "decoupled").
- This means unlabeled-report-free images (CheXpert) and image-free reports can also be used. It also means two scans with the same finding are not pushed apart as false negatives.
- Training is cheap: about 8 hours on one RTX 3090, 10 epochs.

## Eval 1 – Zero-shot classification
- **Analogy:** two sealed engines, do readings match? The image reading is compared to readings of text prompts like "pleural effusion…". The class with the closest prompt wins.
- **How:** Frozen. No training on the target task. Prompts are generated per class, and a prompt-ensemble (ENS) version is also reported. Metric: accuracy, averaged over 5 runs with different prompt samples.
- **Datasets:**
  - CheXpert-5x200 and MIMIC-5x200: 5 classes × 200 images each, from the same sources as pretraining (held-out split).
  - COVID (Rahman et al.): 3,000 test images. Not seen in pretraining, and COVID has no pretraining examples. COVID prompts were hand-written from a radiology description paper.
  - RSNA Pneumonia: 3,538 test images. Not seen in pretraining.
- **Competitors:** CLIP (general web model), ConVIRT, GLoRIA.
- **Fairness:**
  - ConVIRT was re-implemented by the authors (ResNet-50 + BioClinicalBERT).
  - GLoRIA used the official code and released weights, so it was pretrained on different data (about CheXpert-sized).
  - CLIP is off-the-shelf.
  - The backbones are not matched: MedCLIP-ViT is a Swin, while the baselines are ResNet-50.
- **Result:**
  - MedCLIP wins clearly.
  - CheXpert-5x200: 0.594 vs GLoRIA 0.433 and ConVIRT 0.419.
  - MIMIC-5x200: 0.501 (ResNet+ENS 0.543) vs ConVIRT 0.402 and GLoRIA 0.331.
  - COVID: 0.847 (ResNet) vs GLoRIA 0.709.
  - RSNA: 0.768 (ViT+ENS) vs GLoRIA 0.581.
  - CLIP is about chance.

## Eval 2 – Supervised classification ("fine-tune" in the heading, but the text says linear probe)
- **Analogy:** engine sealed on a test bench. The paper's text says the image encoder is frozen and only a new linear head is trained. The section is called "fine-tune", so the protocol is a bit ambiguous.
- **How:** Linear head with cross-entropy, trained on each task's training split. Metric: accuracy.
- **Datasets:** the same 4 as Eval 1 (CheXpert-5x200, MIMIC-5x200, COVID, RSNA).
- **Competitors:** Random-init ResNet-50, ImageNet ResNet-50, CLIP, ConVIRT, GLoRIA.
- **Fairness:**
  - Same head and same data for all models.
  - Backbones differ (Swin vs ResNet-50).
  - The random-init baseline (broken-in vs brand-new engine) is a ResNet-50, not a matching Swin.
- **Result:**
  - MedCLIP is best everywhere.
  - CheXpert 0.596 vs GLoRIA 0.537.
  - MIMIC 0.565 vs ConVIRT 0.404.
  - COVID 0.789 vs GLoRIA 0.762.
  - RSNA 0.808 vs GLoRIA 0.798.
  - Gaps are big on in-domain 5x200 sets and small on RSNA.
  - Oddly, the zero-shot COVID accuracy (0.80–0.85) beats the supervised one.

## Eval 3 – Image-text retrieval
- **Analogy:** two sealed engines, do readings match?
- **How:** Frozen. 1,000 CheXpert-5x200 images are the queries. The pool is 1,000 MIMIC-CXR sentences (200 per class). A hit means the retrieved sentence has the same class, not the exact paired report. Metric: Precision@1/2/5/10.
- **Datasets:** CheXpert-5x200 images and MIMIC sentences. Both sources were seen in pretraining (held-out samples).
- **Competitors:** CLIP, ConVIRT, GLoRIA.
- **Fairness:** Same as Eval 1.
- **Result:**
  - MedCLIP: 0.45 / 0.49 / 0.48 / 0.50.
  - GLoRIA: 0.47 / 0.47 / 0.46 / 0.46.
  - CLIP and ConVIRT: about 0.20.
  - GLoRIA actually wins at K=1. MedCLIP wins at K≥2.

## Eval 4 – Pretraining-data efficiency
- **Analogy:** how much running-in the engine needs before it is good. This is not few-shot on the downstream task; it is less pretraining data.
- **How:** MedCLIP is pretrained on 20K, 50K, and 200K subsets (and the full set). Zero-shot accuracy is measured on CheXpert-5x200.
- **Competitors:** GLoRIA (about 200K pairs) and ConVIRT (369K pairs), at their full data.
- **Fairness:** Only MedCLIP is subsampled. The baselines use their own full data.
- **Result:**
  - MedCLIP with 20K pairs already beats both baselines.
  - Accuracy keeps rising with more data.
  - Exact values are only in a figure (Fig. 1).
  - The headline claim: "10× less data".

## Eval 5 – Embedding visualization
- **Analogy:** plotting the engine's output readings on a map.
- **How:** t-SNE of CheXpert-5x200 image embeddings, MedCLIP vs CLIP.
- **Result:** Qualitative only. MedCLIP forms clusters by disease and CLIP does not.

## Key takeaways
- This is a 2D chest X-ray model, not CT. It is the "two sealed engines" style of evaluation, like CT-CLIP and Merlin's zero-shot evals but in 2D.
- The main idea is a loss that turns label agreement into soft targets. This avoids false negatives and allows unpaired data. The evaluation tests the benefit indirectly. There is no clean ablation that swaps the semantic matching loss for plain InfoNCE on the same backbone and data.
- The "zero-shot" here is the standard CLIP meaning: prompt matching with no task training. But the 5x200 test sets come from the same hospitals and sources as pretraining, so only COVID and RSNA are truly "driven in hospital B".
- Fairness is mixed:
  - ConVIRT was re-run by the authors.
  - GLoRIA was used from its released weights.
  - Backbones are not matched (Swin vs ResNet-50).
- The test sets are small (1,000 images for the 5x200 sets).
- The retrieval eval is class-level matching rather than exact report retrieval. GLoRIA wins Precision@1, so the paper's "best across all K" claim is overstated.
- There are no segmentation, no fine-tuning of the whole encoder, and no 3D experiments. Compare with Merlin (paper 14), which uses a similar image-text contrastive idea but on 3D CT plus EHR codes.
