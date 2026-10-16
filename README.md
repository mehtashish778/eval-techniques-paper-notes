# Evaluation Techniques – Paper Notes

Running notes on **how papers test their models**, in simple words, using an engine analogy.

## Papers

| # | Paper | Notes |
|---|---|---|
| 1 | SPECTRE (CT foundation model) | [papers/01-spectre.md](papers/01-spectre.md) |
| 2 | CT-RATE / CT-CLIP / CT-CHAT | [papers/02-ct-rate-ct-clip.md](papers/02-ct-rate-ct-clip.md) |
| 3 | CT-FM (Vision Foundation Models for CT) | [papers/03-ct-fm.md](papers/03-ct-fm.md) |
| 4 | FMCIB (Foundation Model for Cancer Imaging Biomarkers) | [papers/04-fmcib.md](papers/04-fmcib.md) |
| 5 | VoCo | [papers/05-voco.md](papers/05-voco.md) |
| 6 | SuPreM | [papers/06-suprem.md](papers/06-suprem.md) |
| 7 | VISTA3D | [papers/07-vista3d.md](papers/07-vista3d.md) |
| 8 | PASTA | [papers/08-pasta.md](papers/08-pasta.md) |
| 9 | MedImageInsight | [papers/09-medimageinsight.md](papers/09-medimageinsight.md) |
| 10 | Models Genesis | [papers/10-models-genesis.md](papers/10-models-genesis.md) |
| 11 | TAP-CT | [papers/11-tap-ct.md](papers/11-tap-ct.md) |
| 12 | MedCLIP | [papers/12-medclip.md](papers/12-medclip.md) |
| 13 | CT-Net | [papers/13-ct-net.md](papers/13-ct-net.md) |
| 14 | Merlin | [papers/14-merlin.md](papers/14-merlin.md) |
| 15 | SAM-Med3D | [papers/15-sam-med3d.md](papers/15-sam-med3d.md) |
| 16 | RadFM | [papers/16-radfm.md](papers/16-radfm.md) |
| 17 | CT2Rep | [papers/17-ct2rep.md](papers/17-ct2rep.md) |
| 18 | TotalSegmentator | [papers/18-totalsegmentator.md](papers/18-totalsegmentator.md) |
| 19 | Med3D / MedicalNet | [papers/19-med3d.md](papers/19-med3d.md) |
| 20 | SegVol | [papers/20-segvol.md](papers/20-segvol.md) |
| 21 | DoDNet | [papers/21-dodnet.md](papers/21-dodnet.md) |
| 22 | Universal Model (CLIP-Driven) | [papers/22-universal-model.md](papers/22-universal-model.md) |
| 23 | MAE3D | [papers/23-mae3d.md](papers/23-mae3d.md) |
| 24 | TransVW (Transferable Visual Words) | [papers/24-transvw.md](papers/24-transvw.md) |
| 25 | 3D Rotation (ROT), within "3D Self-Supervised Methods for Medical Imaging" | [papers/25-rot.md](papers/25-rot.md) |
| 26 | Rubik's Cube (Rubik's cube recovery) | [papers/26-rubiks-cube.md](papers/26-rubiks-cube.md) |
| 27 | Rubik's Cube++ | [papers/27-rubiks-cube-plus-plus.md](papers/27-rubiks-cube-plus-plus.md) |
| 28 | PCRL (Preservational Contrastive Representation Learning) | [papers/28-pcrl.md](papers/28-pcrl.md) |
| 29 | PCRLv2 | [papers/29-pcrlv2.md](papers/29-pcrlv2.md) |
| 30 | Swin UNETR SSL | [papers/30-swin-unetr-ssl.md](papers/30-swin-unetr-ssl.md) |
| 31 | SwinMM | [papers/31-swinmm.md](papers/31-swinmm.md) |
| 32 | GL-MAE | [papers/32-gl-mae.md](papers/32-gl-mae.md) |
| 33 | GVSL | [papers/33-gvsl.md](papers/33-gvsl.md) |
| 34 | UniMiSS | [papers/34-unimiss.md](papers/34-unimiss.md) |
| 35 | JSSL | [papers/35-jssl.md](papers/35-jssl.md) |
| 36 | NiftyNet | [papers/36-niftynet.md](papers/36-niftynet.md) |

**Start here for revision:** [CONSOLIDATED.md](CONSOLIDATED.md) – all eval techniques grouped, paper × eval matrix, checklist.

New paper? Copy [TEMPLATE.md](TEMPLATE.md) into `papers/`.

---

## Analogy key (use for every paper)

- **Encoder = the engine.** The model that turns a scan (or text) into numbers.
- **Embedding = the engine's output reading.** A list of numbers (a vector) that summarises the input.
- **Frozen = engine sealed.** Weights never change during the test.
- **Fine-tuned = engine tuned.** Weights are adjusted for the new task.
- **Framework / pipeline (e.g. nnU-Net) = the car body.** Data prep, training loop, loss, evaluation.

## Evaluation types – cheat sheet

| Type | Engine analogy | What it tells you | Seen in |
|---|---|---|---|
| Frozen features + kNN | Engine sealed on a test bench. Record its output and sort it. | How good the engine is out of the box (purest test). | 1 |
| Fine-tuned segmentation | Engine swapped into a car and tuned while driving. | How well the engine adapts to a detailed task. | 1 |
| Zero-shot retrieval (text → image) | Two sealed engines (image + text). Check if their readings match. | Whether the model links words to images. | 1, 2 |
| Zero-shot classification with prompts | Two sealed engines. Is the scan closer to "X is present" or "X is not present"? | Can it spot diseases with no labels or training? | 2 |
| Linear-probe style finetune (ClassFine) | Engine in a car with a new fixed dashboard (one new output layer). | Best on preset labels, but loses flexibility. | 2 |
| Open-vocabulary finetune (VocabFine) | Tune both engines but keep them speaking the same language. | Better accuracy and still works with any text prompt. | 2 |
| Image → image retrieval | One sealed engine. Find scans whose readings are closest to a query scan. | Does it group similar cases together? | 2 |
| Chat / VQA / report generation | Engine connected to a talking dashboard (LLM). Grade what it says. | Can it explain findings in words? | 2 |
| Random-init baseline | Two identical engines: one broken in (pretrained), one brand new. | Pure value of pretraining. | 3 |
| Few-shot fine-tuning | Tune the engine with only 5–20 examples. | Does pretraining help when labels are scarce? | 3 |
| External test ("zero-shot transfer") | Tune in hospital A, drive in hospital B without re-tuning. | Does it generalise to new data? | 2, 3 |
| Embedding inspection (t-SNE, PCA, patch search, saliency, test-retest) | Open the sealed engine's readings and check they make sense. | Is the representation meaningful and stable? | 3 |

> ⚠️ "Zero-shot" means different things in different papers: text prompts (CT-CLIP), frozen embeddings, or testing on a new hospital without retraining (CT-FM). Always check.

### kNN in simple words

- kNN is **not trained**. It just stores the vectors of known patients with their labels.
- New patient → get vector → find the k closest known patients → they vote → prediction.
- The true label is only used at the end to score the vote (AUC).
- It compares the new patient to **other patients**, not to the expected output.
- Because kNN cannot learn, a good score can only come from a good engine.

```
Engine (frozen) → record output (vectors) → sort by similarity (kNN) → check if sorting was right (AUC)
```

### nnU-Net in simple words

- A standard 3D U-Net that sets itself up for each dataset (patch size, depth, spacing).
- U-Net = shrink the image (learn *what*), grow it back (learn *where*), skip links keep edges sharp.
- Two meanings: nnU-Net **the framework** (car body) vs nnU-Net **the U-Net model** (one engine).

```
Input CT
   │
[Conv] ───────────skip───────────► [Conv] → mask
   │ down                             ▲ up
  [Conv] ──────skip──────► [Conv] ────┘
     │ down                   ▲ up
       [Conv] (bottleneck) ───┘
```

### Fairness checklist (ask for every paper)

- Did the authors run all competitors themselves, or copy numbers from other papers?
- Same data splits, same pipeline, same training budget?
- Is the encoder frozen or fine-tuned?
- Is there an external test set (data from another hospital)?
- Were competitors handicapped (e.g. 2D models given 3D tasks)?
