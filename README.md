# Evaluation Techniques – Paper Notes

Running notes on **how papers test their models**, in simple words, using an engine analogy.

## Papers

| # | Paper | Notes |
|---|---|---|
| 1 | SPECTRE – Scaling Self-Supervised and Cross-Modal Pretraining for Volumetric CT Transformers | [papers/01-spectre.md](papers/01-spectre.md) |
| 2 | CT-RATE / CT-CLIP / CT-CHAT – Generalist Foundation Models from a Multimodal Dataset for 3D CT | [papers/02-ct-rate-ct-clip.md](papers/02-ct-rate-ct-clip.md) |
| 3 | CT-FM – Vision Foundation Models for Computed Tomography | [papers/03-ct-fm.md](papers/03-ct-fm.md) |

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
