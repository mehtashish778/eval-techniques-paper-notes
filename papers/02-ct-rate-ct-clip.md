# Paper 2 – CT-RATE / CT-CLIP / CT-CHAT

- **Title:** Generalist Foundation Models from a Multimodal Dataset for 3D Computed Tomography (Hamamci et al.)
- **Link:** https://arxiv.org/abs/2403.17834
- **One line:** Three things in one paper: a dataset (CT-RATE: 25,692 chest CTs + reports), a CLIP model for CT (CT-CLIP), and a chat model (CT-CHAT = CT-CLIP's image engine + an LLM).

---

## Eval 1 – Zero-shot multi-abnormality detection

- **Analogy:** two sealed engines; compare the scan's reading to "present" and "not present" text readings.
- **How:** no training. 7 prompts tried; best was "{Abnormality} is / is not present." Metrics: AUROC, F1, accuracy, precision.
- **Datasets:** CT-RATE internal validation + 2 external sets (RAD-ChestCT, UPMC) → tests generalisation to new hospitals.
- **Competitor:** CT-Net (fully supervised). Authors **retrained it themselves** on CT-RATE with the same labels → fair.
- **Result:** zero-shot CT-CLIP beat supervised CT-Net. AUROC +0.10 (internal), +0.085 (RAD-ChestCT), +0.087 (UPMC). Significance tested (permutation test, p < 0.05).

```
"Nodule is present"     → [text engine 🔒]  → ▲
"Nodule is not present" → [text engine 🔒]  → △
CT scan                 → [image engine 🔒] → ●   closer to ▲ or △ ?
```

### Eval 1 – clarifications

- **Same training data for both:** CT-RATE training scans. Same test scans and same metrics.
- **Different way of learning:** CT-CLIP learned from scan + **report text** (never saw labels). CT-Net learned from scan + **18 yes/no labels** (labels were extracted from the same reports).
- **Why CT-Net uses labels, not reports:** CT-Net is a plain image classifier — it has no text engine, so it *can't* read reports. It stands for the traditional "label everything by hand" approach.
- **Who trained what:** the authors trained **both**. CT-CLIP = their own model, trained once for general scan–report matching (not for detection). CT-Net = **retrained** on CT-RATE because the original wasn't available; its scores matched the original paper, so it wasn't a weak competitor.
- **Same answer format:** both give yes/no for the same 18 diseases → fair comparison. Only *how* they decide differs (trained output slot vs comparing to sentences).

```
        CT-RATE training scans (same for both)
              │                       │
   CT-CLIP learns from          CT-Net learns from
   scan + report text           scan + yes/no labels
              │                       │
        Same test scans (CT-RATE val, RAD-ChestCT, UPMC)
              │                       │
       yes/no via sentences     yes/no via classifier
              └──── compare AUROC / F1 ────┘
```

**Zero-shot eval step by step**

1. Write two sentences per disease: "Nodule is present." / "Nodule is not present." → text engine → ▲ △
2. Test scan → image engine → ●
3. Which sentence is ● closer to? → probability (e.g. 80% present)
4. Repeat for every scan × 18 diseases
5. Compare with true labels — labels used **only for grading**, never for training CT-CLIP
6. Score: AUROC (ranking sick above healthy; 1.0 perfect, 0.5 guessing), F1/accuracy/precision; per disease, then averaged
7. Same for CT-Net, and repeat on outside hospitals (RAD-ChestCT, UPMC)

## Eval 2 – Finetuned detection (ClassFine vs VocabFine)

- **Question:** zero-shot already works — if we *do* use labels, how much better can it get?
- **ClassFine:** add one classification layer to the image engine (like a linear probe). Better scores, but fixed to preset labels.
- **VocabFine:** finetune using positive/negative text prompts. Beats ClassFine AND keeps open-vocabulary ability.
- **Note:** training all weights mattered more than training only the last layer.

```
ClassFine:  scan → [image engine 🔧] → ● → [18 fixed slots] → yes/no
            (text engine thrown away)

VocabFine:  "X is present"     → [text engine 🔧]  → ▲
            "X is not present" → [text engine 🔧]  → △
            scan               → [image engine 🔧] → ●   closer to ▲ or △ ?
```

**Same labels, different meaning**

| | Uses labels? | What the label means | Input | Text engine | Output classes | New disease? | Score |
|---|---|---|---|---|---|---|---|
| Zero-shot | ❌ | – | scan + sentences | kept, frozen | any (you write them) | ✅ | Good |
| ClassFine | ✅ | "this slot should light up" | scan only | removed | fixed 18 | ❌ | Better |
| VocabFine | ✅ | "this sentence is the correct one" | scan + sentences (train **and** test) | kept, tuned | not fixed — new sentences = new class | ✅ (best on the 18 it was tuned on) | Best |

- **Analogy:** ClassFine = fixed panel of 18 buttons. VocabFine = keeps the talking dashboard and tunes both engines to pick the right sentence.

**What the text engine does (and doesn't)**

- It only **reads** text and turns it into a vector. It **never writes** text.
- Like a **question box**: put in any question, the only answer is "how well does this scan match?"
- Lets you ask about new diseases, use different wording, or search with a full report.
- Writing answers/reports needs an LLM → that's CT-CHAT.

| Part | Reads text? | Writes text? |
|---|---|---|
| Text engine (CT-CLIP) | ✅ | ❌ (only a vector) |
| LLM (CT-CHAT) | ✅ | ✅ |

**Why VocabFine matters**

- You don't have to choose between accuracy and flexibility — training with labels *through sentences* gives both.
- Practical: improve with some labeled data, still query rare/new findings without new labels or retraining; one model for detection, search and chat.
- **Caveat:** not free — VocabFine got worse at full report → scan retrieval (tuned on short sentences, so long reports suffer).

## Eval 3 – Case retrieval

- **Image → image:** query scan → find similar scans (cosine similarity). Metric: MAP@K. Internal + RAD-ChestCT. All methods similar; all far above random (~6× internal, ~3× external).
- **Report → image:** metric Recall@K, internal set only (RAD-ChestCT has no reports). CT-CLIP well above random.
- **Key lesson:** VocabFine was much **worse** at report→image retrieval → finetuning for one task can hurt another.
- **Tip:** they compare "fold change vs random" because raw scores depend on the dataset.

## Eval 4 – CT-CHAT (chat, VQA, report generation)

- **Analogy:** CT-CLIP's image engine plugged into an LLM dashboard that talks.
- **Trained on:** 2.7M question–answer pairs made from CT-RATE.
- **Competitors (2D):** LLaVA 1.6 (Mistral 7B, Vicuna 13B), LLaVA-Med, CXR-LLaVA. They can't read 3D, so they were given DRRs (2D X-ray-like images made from the CT).
- **Competitors (3D reports):** RadFM, CT2Rep.
- **Metrics:** text overlap (BLEU, METEOR, ROUGE-L, CIDEr) + "Llama score" (an LLM judges clinical accuracy) + **3 radiologists** scoring 150 reports each (0–10).
- **Result:** CT-CHAT best everywhere. Radiologist means ≈ 3.4–4.5 vs CT2Rep ≈ 2.5–2.7 vs RadFM ≈ 0.5–0.9 (out of 10 → all still far from perfect).
- **Fairness:** 2D models were handicapped (DRR instead of 3D), so the win partly reflects 3D vs 2D input.

## Key takeaways

- Zero-shot (no labels) beat a fully supervised model → strong sign of good image–text alignment.
- External test sets (RAD-ChestCT, UPMC) are a big plus for trust.
- Finetuning trade-off: better on one task, worse on others.
- Chat evaluation mixes automatic text scores, an LLM judge, and human experts — human scores are the most trustworthy.
- Link to Paper 1: SPECTRE later used CT-RATE and compared against CT-CLIP.
