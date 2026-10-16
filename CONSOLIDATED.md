# Consolidated Notes – How 36 CT / Medical Foundation-Model Papers Evaluate Their Models

One place to revise everything. Simple words + the engine analogy.
Per-paper details live in [`papers/`](papers/).

---

## 0. The analogy key

| Term | Analogy |
|---|---|
| Encoder | the **engine** |
| Embedding | the engine's **output reading** (a vector) |
| Frozen | engine **sealed** – weights don't change |
| Fine-tuned | engine **tuned** for the new job |
| Framework (nnU-Net, Auto3DSeg, MONAI) | the **car body** (data prep, training loop, loss, evaluation) |

**The one question every eval answers:** *is this engine good, and good for what?*

---

## 1. The big picture – 4 families of evaluation

```
                   ┌─ A. Sealed engine (frozen)        → "is the engine good out of the box?"
How much is        ├─ B. Tuned engine (fine-tuned)     → "can it be adapted to a real task?"
the engine         ├─ C. Two engines (image + text)    → "does it link pictures and words?"
changed?           └─ D. Driver-guided (interactive)   → "does it segment what a user points at?"
```

Plus **E. "Fairness levers"** (how the comparison is set up) and **F. Minor / sanity checks** (grouped at the end).

---

## A. Sealed-engine tests (frozen encoder) – *test bench*

| Technique | What happens | Metric | Used by |
|---|---|---|---|
| **kNN on embeddings** | Record each scan's vector; a new scan takes the vote of its k nearest neighbours. Nothing is trained. | AUC | SPECTRE, MedImageInsight |
| **Linear probe** | Freeze engine, train only one small linear layer on top. | AUC / Dice | FMCIB, Merlin, MedCLIP, TAP-CT, GVSL, PCRL, TransVW, GL-MAE*, CT-CLIP (ClassFine-ish) |
| **Frozen + light head** (attention-pooling / small decoder) | Same idea, slightly bigger head. | AUC / Dice | TAP-CT (ABMIL, linear seg head), GL-MAE* (*trains a full decoder – not a true linear probe*) |

- **Why it matters:** the purest test – a weak engine can't hide behind training.
- **Watch out:** it can **handicap** models built to be fine-tuned (TAP-CT beat VoCo, CT-FM, VISTA3D this way).
- **Lesson:** frozen gains are often **large**, fine-tuned gains **small** (GVSL: 68.4 vs 56.5 frozen; small when fine-tuned).

---

## B. Tuned-engine tests (fine-tuning) – *engine in a car*

This is **by far the most common** family (27 of 36 papers fine-tune for segmentation).

| Technique | What happens | Metric | Typical datasets |
|---|---|---|---|
| **Fine-tuned segmentation** | Add/keep a decoder, train everything on labelled masks. | Dice, HD95, NSD, ASD | BTCV, TotalSegmentator, MSD, LiTS, KiTS, WORD, AMOS, MM-WHS, BraTS |
| **Fine-tuned classification** | Add a pooling + MLP head, train end-to-end. | AUC, F1, accuracy | LUNA16 nodules, CC-CCII COVID, SinoCT/CQ500 head CT, ChestX-ray14 |
| **Engine swap into a framework** | Put the pretrained encoder into nnU-Net / Auto3DSeg / UNETR; keep everything else fixed. | Dice | SPECTRE (nnU-Net), CT-FM (Auto3DSeg), PASTA (nnU-Net), TransVW (nnU-Net), SAM-Med3D (UNETR), Merlin |

- **Why it matters:** shows real-world usefulness.
- **Watch out:** gains mix *pretraining* + *training recipe* + *architecture*. Engine swap is the cleanest way to isolate the engine.

---

## C. Two-engine tests (image + text) – *do the readings match?*

| Technique | What happens | Metric | Used by |
|---|---|---|---|
| **Zero-shot classification by prompt** | Is the scan closer to "X is present" or "X is not present"? No labels used. | AUROC, F1 | CT-CLIP, Merlin, MedCLIP, MedImageInsight* |
| **Image ↔ text retrieval** | Give a report → find its scan (or vice versa). | Recall@K | SPECTRE, CT-CLIP, Merlin, MedCLIP |
| **Report generation / VQA** | Engine + LLM "talking dashboard" writes text. | BLEU/ROUGE/METEOR/CIDEr, clinical-efficacy F1, LLM-judge, radiologist score | CT-CHAT, RadFM, CT2Rep, Merlin, MedImageInsight, PASTA (form-filling) |

- **Watch out:** text-overlap scores (BLEU…) reward wording, not correctness → look for **clinical F1** or **radiologist ratings**.
- *MedImageInsight's text-head "zero-shot" used labels seen in training → not truly zero-shot.*

---

## D. Driver-guided tests (interactive / promptable segmentation)

| Technique | What happens | Metric | Used by |
|---|---|---|---|
| **Click / box / text prompts** | User points at a target; model segments it. Count clicks needed. | Dice vs #clicks | SAM-Med3D, SegVol, VISTA3D |
| **Novel class with clicks** | Segment a class never trained on, guided by clicks. | Dice | VISTA3D ("zero-shot"), SAM-Med3D |

- **Watch out:** comparing a model **given 10 clicks** to fully automatic nnU-Net is not apples-to-apples (SAM-Med3D, SegVol). SegVol's prompts came from the true masks.

---

## E. Fairness levers – how the comparison is set up (check these in every paper)

| Lever | Analogy | Why it matters | Strong examples |
|---|---|---|---|
| **Random-init (scratch) baseline** – same architecture, no pretraining | broken-in vs brand-new engine | Isolates the value of pretraining | CT-FM, Models Genesis, TransVW, PCRL, Med3D, SuPreM |
| **Few-shot / label efficiency** – fine-tune with 1–20 scans or 10–50% labels | tune with a handful of examples | Pretraining helps **most when labels are scarce** | VISTA3D (1 scan: 0.795 vs 0.185), PASTA, CT-FM, Rubik++, SuPreM |
| **External validation** – train at hospital A, test at B without retraining | tuned in A, driven in B | Tests generalisation | CT-CLIP (RAD-ChestCT, UPMC), CT-FM (CQ500), Merlin, Universal Model |
| **Transfer to another modality** – CT-pretrained → MRI / X-ray / skin | engine moved to a different vehicle | Tests generality | VoCo (BraTS), PCRL, UniMiSS (ISIC, CHAOS), Models Genesis |
| **Re-run vs copied competitors** | same track vs times from another track | Copied numbers = different recipes/splits | Re-run: CT-CLIP (CT-Net), FMCIB, GVSL, UniMiSS, Rubik++. Copied: VoCo, Swin UNETR SSL, SPECTRE seg table |
| **Leakage check** – was the test set in pretraining? | testing on the practice track | Inflates results | BTCV used in pretraining *and* testing (VoCo, GL-MAE); WORD (SwinMM); SegVol "external" sets in training |

---

## F. Minor / sanity checks (grouped – lighter evidence)

These rarely decide "who wins" but support the story. Read quickly.

| Group | What it checks | Examples |
|---|---|---|
| **Look inside the engine** – t-SNE / UMAP clusters, PCA colours, patch-concept search | Are the readings organised by anatomy/disease? | CT-FM, FMCIB, Universal Model, MedCLIP, GVSL |
| **Where does it look?** – saliency, occlusion, CAM | Does it focus on sensible regions? | CT-FM, FMCIB, PCRLv2 |
| **Is it stable?** – test-retest, reader variation, subgroup fairness (sex/age) | Same patient → same reading? | CT-FM (RIDER), FMCIB, MedImageInsight |
| **Ablations** – remove/alter one pretraining ingredient | Which design choice matters? | Almost every paper |
| **Speed & cost** – convergence curves, runtime, compute scaling | Faster training / inference? | Models Genesis, TransVW, Med3D, VISTA3D, SegVol, SuPreM |
| **Public leaderboards** – BTCV, MSD, FLARE, AMOS | Rank vs the world (often with ensembles) | Swin UNETR SSL, VoCo, Universal Model, UniMiSS |
| **Odd baselines** – video-pretrained (Kinetics/UCF101), natural-image transfer | Is medical pretraining better than generic? | Med3D, Rubik's Cube, ROT, PCRL |
| **Qualitative figures** – example masks, failure cases | Visual sanity only | Most segmentation papers |
| **Clinical add-ons** – survival, staging, aging study, reader study with AI help, gene association | Does it matter clinically? | PASTA, FMCIB, TotalSegmentator, Merlin |
| **Toolkit demos** – no competitors, just "it works" | – | NiftyNet |

---

## G. Paper × evaluation matrix

| # | Model | Frozen probe | FT seg | FT cls | Engine swap | Scratch baseline | Few-shot | External | New modality | Zero-shot text | Img↔text retr. | Img↔img retr. | Report/VQA | Interactive seg | Human rating | Leaderboard |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| 01 | [SPECTRE](papers/01-spectre.md) | ✓ | ✓ |  | ✓ |  |  |  |  |  | ✓ |  |  |  |  |  |
| 02 | [CT-RATE / CT-CLIP / CT-CHAT](papers/02-ct-rate-ct-clip.md) | ✓ |  | ✓ |  |  |  | ✓ |  | ✓ | ✓ | ✓ | ✓ |  | ✓ |  |
| 03 | [CT-FM](papers/03-ct-fm.md) |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |  | ✓ |  |  |  |  |
| 04 | [FMCIB](papers/04-fmcib.md) | ✓ |  | ✓ |  | ✓ | ✓ | ✓ |  |  |  |  |  |  |  |  |
| 05 | [VoCo](papers/05-voco.md) |  | ✓ | ✓ |  | ✓ |  |  | ✓ |  |  |  |  |  |  | ✓ |
| 06 | [SuPreM](papers/06-suprem.md) |  | ✓ | ✓ |  | ✓ | ✓ | ✓ |  |  |  |  |  |  | ✓ |  |
| 07 | [VISTA3D](papers/07-vista3d.md) |  | ✓ |  |  | ✓ | ✓ | ✓ | ✓ |  |  |  |  | ✓ |  |  |
| 08 | [PASTA](papers/08-pasta.md) |  | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  |  |  | ✓ |  | ✓ |  |
| 09 | [MedImageInsight](papers/09-medimageinsight.md) | ✓ |  | ✓ |  | ✓ | ✓ | ✓ |  | ✓ |  | ✓ | ✓ |  |  |  |
| 10 | [Models Genesis](papers/10-models-genesis.md) |  | ✓ | ✓ |  | ✓ | ✓ |  | ✓ |  |  |  |  |  |  | ✓ |
| 11 | [TAP-CT](papers/11-tap-ct.md) | ✓ |  |  |  |  |  |  |  |  |  | ✓ |  |  |  |  |
| 12 | [MedCLIP](papers/12-medclip.md) | ✓ |  |  |  |  | ✓ | ✓ |  | ✓ | ✓ |  |  |  |  |  |
| 13 | [CT-Net](papers/13-ct-net.md) |  |  | ✓ |  | ✓ |  |  |  |  |  |  |  |  |  |  |
| 14 | [Merlin](papers/14-merlin.md) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  | ✓ |  |  |  |
| 15 | [SAM-Med3D](papers/15-sam-med3d.md) |  |  |  | ✓ | ✓ |  | ✓ |  |  |  |  |  | ✓ |  |  |
| 16 | [RadFM](papers/16-radfm.md) |  |  | ✓ |  |  |  |  |  |  |  |  | ✓ |  | ✓ |  |
| 17 | [CT2Rep](papers/17-ct2rep.md) |  |  |  |  |  |  |  |  |  |  |  | ✓ |  |  |  |
| 18 | [TotalSegmentator](papers/18-totalsegmentator.md) |  | ✓ |  |  |  |  | ✓ |  |  |  |  |  |  |  |  |
| 19 | [Med3D / MedicalNet](papers/19-med3d.md) |  | ✓ | ✓ |  | ✓ |  |  |  |  |  |  |  |  |  |  |
| 20 | [SegVol](papers/20-segvol.md) |  | ✓ |  |  |  | ✓ |  | ✓ |  |  |  |  | ✓ |  | ✓ |
| 21 | [DoDNet](papers/21-dodnet.md) |  | ✓ |  |  | ✓ |  |  |  |  |  |  |  |  |  | ✓ |
| 22 | [Universal Model](papers/22-universal-model.md) |  | ✓ |  |  | ✓ |  | ✓ |  |  |  |  |  |  | ✓ | ✓ |
| 23 | [MAE3D](papers/23-mae3d.md) |  | ✓ |  |  | ✓ | ✓ |  |  |  |  |  |  |  |  |  |
| 24 | [TransVW](papers/24-transvw.md) | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |  | ✓ |  |  |  |  |  |  | ✓ |
| 25 | [3D Rotation](papers/25-rot.md) |  | ✓ | ✓ |  | ✓ | ✓ |  |  |  |  |  |  |  |  | ✓ |
| 26 | [Rubik's Cube](papers/26-rubiks-cube.md) |  | ✓ | ✓ |  | ✓ |  |  |  |  |  |  |  |  |  |  |
| 27 | [Rubik's Cube++](papers/27-rubiks-cube-plus-plus.md) |  | ✓ |  |  | ✓ | ✓ |  |  |  |  |  |  |  |  |  |
| 28 | [PCRL](papers/28-pcrl.md) | ✓ | ✓ | ✓ |  | ✓ | ✓ |  | ✓ |  |  |  |  |  |  |  |
| 29 | [PCRLv2](papers/29-pcrlv2.md) |  | ✓ | ✓ |  | ✓ | ✓ |  | ✓ |  |  |  |  |  |  |  |
| 30 | [Swin UNETR SSL](papers/30-swin-unetr-ssl.md) |  | ✓ |  |  | ✓ | ✓ |  |  |  |  |  |  |  |  | ✓ |
| 31 | [SwinMM](papers/31-swinmm.md) |  | ✓ |  |  | ✓ | ✓ |  | ✓ |  |  |  |  |  |  |  |
| 32 | [GL-MAE](papers/32-gl-mae.md) | ✓ | ✓ |  |  | ✓ | ✓ | ✓ | ✓ |  |  |  |  |  |  |  |
| 33 | [GVSL](papers/33-gvsl.md) | ✓ | ✓ | ✓ |  | ✓ |  |  | ✓ |  |  |  |  |  |  |  |
| 34 | [UniMiSS](papers/34-unimiss.md) |  | ✓ | ✓ |  | ✓ | ✓ |  | ✓ |  |  |  |  |  |  | ✓ |
| 35 | [JSSL](papers/35-jssl.md) |  | ✓ | ✓ |  | ✓ | ✓ |  | ✓ |  |  |  |  |  |  |  |
| 36 | [NiftyNet](papers/36-niftynet.md) |  | ✓ |  |  |  |  |  |  |  |  |  |  |  |  |  |
| | **Papers using it** | **11** | **27** | **20** | **6** | **27** | **21** | **13** | **14** | **4** | **4** | **4** | **6** | **3** | **5** | **9** |

---

## H. "Zero-shot" means different things – always check!

| Paper | What "zero-shot" meant |
|---|---|
| CT-CLIP, Merlin, MedCLIP | Classify with **text prompts**, no labels |
| CT-FM | Tested at **another hospital** without retraining |
| VISTA3D | Segment **new classes with clicks** |
| RadFM | **No examples in the prompt** – but test data came from its own training sources |
| MedImageInsight | Text-head classification on label sets **seen in training** |

---

## I. Quick checklist for any new paper

1. Frozen or fine-tuned? (A vs B)
2. Is there a **same-architecture scratch baseline**?
3. Few-shot / low-label results?
4. **External** test set (other hospital)?
5. Were competitors **re-run** on the same split, or **copied**?
6. Was the test data **seen in pretraining**?
7. For text models: clinical metrics or radiologist ratings, not just BLEU?
8. What does *their* "zero-shot" mean?

---

## J. Big lessons across all 36 papers

- **Fine-tuned segmentation is the default test**; frozen tests are rarer but more revealing.
- **Pretraining gains shrink with full labels** (often < 1–2 Dice) and **grow with few labels** – few-shot is where foundation models shine.
- **Many comparisons are copied numbers** with different recipes → small margins may not be real.
- **Leakage is common** (BTCV, WORD, AMOS appear in both pretraining and testing).
- **Text models need clinical metrics**; BLEU-type scores alone are weak.
- **Interactive models** should be compared at equal effort (clicks) to automatic ones.
