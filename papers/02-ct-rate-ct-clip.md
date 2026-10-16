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

## Eval 2 – Finetuned detection (ClassFine vs VocabFine)

- **ClassFine:** add one classification layer to the image engine (like a linear probe). Better scores, but fixed to preset labels.
- **VocabFine:** finetune using positive/negative text prompts. Beats ClassFine AND keeps open-vocabulary ability.
- **Note:** training all weights mattered more than training only the last layer.

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
