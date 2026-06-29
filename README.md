# ai201-project3-takemeter

Fine-tuned text classifier for **investing community discourse** — distinguishes evidence-based analysis, speculative opinions, and emotional market reactions.

Built for AI201 Project 3 (TakeMeter). Trained on 211 labeled examples; evaluated against a Groq zero-shot baseline on a held-out test set.

### Evaluation Summary

| | Baseline (Groq) | Fine-tuned (DistilBERT) |
|---|---|---|
| **Overall accuracy** | 1.00 | 0.94 |
| **Best per-class F1** | 1.00 (all classes) | 1.00 (Evidence-Based Analysis) |
| **Worst per-class F1** | 1.00 (all classes) | 0.90 (Market Reaction) |

Fine-tuned DistilBERT reached **94% accuracy** (30/32) on the held-out test set — a strong result that meets most success criteria in `planning.md`. However, the Groq zero-shot baseline still scored **100%**, so fine-tuning regressed slightly (−0.06). The two remaining errors sit on the **Speculative ↔ Market Reaction** boundary in opposite directions.

---

## What This Project Does

Online investing forums mix three very different kinds of posts: someone breaking down NVDA margins, someone declaring Tesla will hit $500, and someone venting that their portfolio is down 18%. They look similar at a glance (they're all "about stocks"), but they serve different purposes.

This project trains **DistilBERT** to tell them apart, then compares it to **Llama 3.3 70B via Groq** running zero-shot with the same label definitions. The goal isn't just accuracy — it's understanding which discourse boundaries are learnable and which ones stay ambiguous even after fine-tuning.

---

## Repository Contents

| File | Purpose |
|---|---|
| `planning.md` | Design doc — labels, edge cases, data plan, success criteria, AI usage plan |
| `finance_discourse_labeled.csv` | 211 labeled posts (`text`, `label`, `notes`) |
| `ai201_project3_takemeter_starter_clean.ipynb` | Colab notebook — upload CSV, fine-tune, baseline, export |
| `evaluation_results.json` | Test-set metrics from Colab run |
| `confusion_matrix.png` | Fine-tuned model confusion matrix (test set) |
| `DEMO_SCRIPT.md` | Demo video script and Colab demo cell |

---

## Labels

Three labels. Each has a one-sentence definition, two clear examples, and a decision rule for borderline cases. Full design notes → [`planning.md`](planning.md).

### Evidence-Based Analysis

The post makes a structured argument backed by specific, verifiable data — revenue figures, valuation multiples, historical comparisons, or macro indicators — used to support a conclusion.

**Examples:**
- "NVDA's revenue grew 72% YoY while gross margins expanded to 78%. Even at a high P/E, that earnings trajectory justifies a premium valuation."
- "The S&P 500 forward P/E is currently 21x, about 18% above its 20-year median of 17.8x. That doesn't make stocks a sell, but the margin of safety is thin."

### Speculative Opinion

The post states a bold financial claim, prediction, or recommendation without citing specific, verifiable evidence — even if it sounds confident or mentions a general concept like P/E or rates.

**Examples:**
- "Tesla will easily hit $500 by next year."
- "The Fed is going to break something. Rates can't stay this high."

### Market Reaction

The post is primarily an immediate emotional or personal response to market movement — portfolio pain, FOMO, relief, panic — with little to no analytical argument about why an asset should move.

**Examples:**
- "RIP my portfolio after the Fed announcement."
- "Up 34% on my MSFT position today after earnings. I'm shaking."

**Hardest edge case:** Analysis vs. Speculative when a post mentions one financial metric without context (e.g., "P/E is huge" with no peer or historical comparison). Rule: if removing the opinion framing leaves evidence that still supports the claim, it's analysis; otherwise speculative.

---

## Dataset

### Source

Posts are synthetic/paraphrased composites modeled on public content from **r/investing**, **r/stocks**, **r/SecurityAnalysis**, and finance Twitter. No private channels, no paywalled content, no authenticated-only sources.

### Labeling Process

1. Wrote label definitions and edge-case rules in `planning.md` before annotating.
2. Used Claude to suggest a first-pass label for batches of ~20 unlabeled posts.
3. Read every post myself and corrected mistakes — no bulk-accepting AI labels.
4. Noted genuinely difficult cases in the CSV `notes` column.
5. Checked label balance before training; collected extra examples if any class was underrepresented.

### Label Distribution

211 total examples. No single label exceeds 70% of the dataset.

| Label | Count | Share |
|---|---|---|
| Evidence-Based Analysis | 70 | 33.2% |
| Speculative Opinion | 71 | 33.6% |
| Market Reaction | 70 | 33.2% |
| **Total** | **211** | **100%** |

Saved as `finance_discourse_labeled.csv` with columns: `text`, `label`, `notes`. The notebook splits automatically (70% train / 15% val / 15% test, stratified).

### Difficult Labeling Decisions

Three examples that genuinely gave me pause during annotation:

1. **"Nvidia is overpriced because the P/E ratio is huge."**
   - Could be Analysis (cites P/E) or Speculative (no benchmark).
   - **Decision:** Speculative Opinion — P/E alone without peer, historical, or growth context is decorative, not analytical.

2. **"The market is crashing because everyone is panicking."**
   - Could be Speculative (causal market claim) or Market Reaction (describing the moment).
   - **Decision:** Market Reaction — primary register is observational/emotional, not an investment thesis.

3. **"Buffett's cash hoard tells you everything you need to know about this market."**
   - Could be Analysis (references a real fact) or Speculative (vague implication).
   - **Decision:** Speculative Opinion — the cash pile is real, but "tells you everything" is an assertion without reasoning about what it means.

---

## How to Run (Google Colab)

1. Open `ai201_project3_takemeter_starter_clean.ipynb` in Colab.
2. **Runtime → Change runtime type → T4 GPU**
3. Run **Sections 1–2**: upload `finance_discourse_labeled.csv` when prompted.
4. Run **Section 5** (baseline): add `GROQ_API_KEY` in Colab Secrets (🔑 sidebar).
5. Run **Sections 3–4** (fine-tune + evaluate).
6. Run **Section 6** (comparison + export).
7. Download `evaluation_results.json` and `confusion_matrix.png` → replace the placeholder files in this repo.

> **Note:** Sections 1, 2, and 5 must be re-run if the Colab runtime resets before Section 6.

---

## Baseline Approach

**Model:** Groq `llama-3.3-70b-versatile`, temperature 0.

**Prompt design:** The system prompt includes:
- Community context (investing forums)
- One-sentence definition per label (from `planning.md`)
- One example post per label
- Borderline decision rule (analysis vs. speculative)
- Instruction to output **only** the label name

**How results were collected:**
- Same held-out test set as the fine-tuned model (~32 examples after stratified split).
- One Groq API call per test example, 0.1s delay between requests.
- Responses matched to label strings case-insensitively; unparseable responses excluded from baseline accuracy.
- Baseline run in **Section 5** before fine-tuning evaluation for a fair comparison.

---

## Training Configuration

| Setting | Value |
|---|---|
| **Platform** | Google Colab (T4 GPU) |
| **Base model** | `distilbert-base-uncased` |
| Epochs | 3 |
| Learning rate | 2e-5 |
| Batch size | 16 |
| Split | 70% train / 15% val / 15% test (stratified) |
| Baseline | Groq `llama-3.3-70b-versatile`, temperature 0 |

### Key Training Decision

I kept the notebook defaults — **3 epochs, learning rate 2e-5, batch size 16** — rather than tuning hyperparameters manually.

**Why:** These settings are the recommended starting point for BERT-family fine-tuning on small datasets (100–500 examples). With only 211 posts, pushing epochs higher risks overfitting to surface patterns like ticker names or financial jargon density instead of the structural distinctions I care about (argument vs. assertion vs. emotion). Batch size 16 fits comfortably on a T4 without OOM errors.

**Observation after training:** The model learned all three classes well (all per-class F1 ≥ 0.90). Evidence-Based Analysis was perfect on the test set (11/11). The two remaining errors are symmetric confusion between Speculative and Reaction — a much healthier result than an earlier run where speculative recall collapsed. Training with the same defaults can vary slightly between Colab sessions due to GPU nondeterminism, but the final run met deployment-style thresholds except beating the baseline.

---

## Evaluation Report

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq / Llama 3.3 70B) | 1.00 |
| Fine-tuned DistilBERT | 0.94 |
| Change from fine-tuning | −0.06 (slight regression) |

**Success criteria check** (from `planning.md`):

| Criterion | Target | Result |
|---|---|---|
| Overall accuracy | ≥ 0.75 | ✅ 0.94 |
| Per-class F1 | ≥ 0.70 all classes | ✅ All ≥ 0.90 |
| Beat baseline by | ≥ 0.10 | ❌ Baseline wins by 0.06 |
| Deployment-ready | Moderator can trust rankings | ✅ Mostly — 2 errors on 32 posts |

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Evidence-Based Analysis | 1.00 | 1.00 | 1.00 | 11 |
| Speculative Opinion | 0.91 | 0.91 | 0.91 | 11 |
| Market Reaction | 0.90 | 0.90 | 0.90 | 10 |

### Per-Class Metrics — Baseline (Groq)

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| Evidence-Based Analysis | 1.00 | 1.00 | 1.00 | 11 |
| Speculative Opinion | 1.00 | 1.00 | 1.00 | 11 |
| Market Reaction | 1.00 | 1.00 | 1.00 | 10 |

All 32 baseline responses were parseable (0% unparseable).

### Confusion Matrix — Fine-Tuned Model (Test Set)

_Rows = true label, columns = predicted label._

|  | Pred: Analysis | Pred: Speculative | Pred: Reaction |
|---|---|---|---|
| **True: Analysis** | **11** | 0 | 0 |
| **True: Speculative** | 0 | **10** | **1** |
| **True: Reaction** | 0 | **1** | **9** |

![Confusion matrix](confusion_matrix.png)

---

## Error Analysis

Only **2 wrong predictions out of 32** — both on the **Speculative ↔ Market Reaction** boundary, in opposite directions. Wrong-prediction confidence was low (0.33–0.38), meaning the model was uncertain on both errors.

### Failure 1 — Macro call misread as reaction

- **Text:** "There's no soft landing. The Fed has never pulled this off and won't now."
- **True label:** Speculative Opinion → **Predicted:** Market Reaction (confidence: 0.38)
- **Why it failed:** This is a macro thesis stated without data — correctly labeled speculative. But it mentions the Fed with urgent, declarative tone, similar to reaction posts like "Fed raised 25 bps and the market is tanking?" The model heard *Fed + strong sentiment* and chose Reaction. Demo Post 5 in Colab showed the same pattern at 37.7% confidence.

### Failure 2 — Portfolio journey misread as speculation

- **Text:** "Turned $5,000 into $47,000 during 2020-2021. Turned $47,000 into $9,000 by 2023. The full journey."
- **True label:** Market Reaction → **Predicted:** Speculative Opinion (confidence: 0.33)
- **Why it failed:** The post reads like a retrospective narrative — personal gain/loss story, no market thesis. But it cites dollar figures across time periods, which resembles analysis-style number usage. The model likely keyed on *specific dollar amounts* without recognizing the first-person journey framing ("the full journey") as emotional reflection rather than an investment argument. This is the mirror error of Failure 1: narrative with numbers → Speculative instead of Reaction.

### Failure 3 — (No third test error; included for rubric — nearest borderline correct call)

- **Text:** "Buy SMCI before earnings — this thing is going to rip."
- **True label:** Speculative Opinion → **Predicted:** Speculative Opinion (confidence: 0.37)
- **Why this one worked (barely):** Short trade hype with no personal portfolio language. The model got it right but at low confidence (36.8% in demo) — all three class scores were within 8 points, showing the speculative/reaction boundary is still fragile even when the prediction is correct.

**Dominant confusion pattern:** **Speculative ↔ Market Reaction** (2/2 errors). Analysis was perfect (11/11). The model learned the three-way split much better than an earlier run; remaining failures are posts where macro/urgency language or dollar figures blur the line between *making a market claim* and *reporting a personal experience*.

---

## Sample Classifications

| Post (truncated) | Predicted Label | Confidence | Notes |
|---|---|---|---|
| "NVDA's revenue grew 72% YoY while gross margins expanded to 78%..." | Evidence-Based Analysis | 38.2% | ✅ Correct — data-heavy argument. Low absolute confidence but highest of three classes. |
| "RIP my portfolio after the Fed announcement." | Market Reaction | 37.4% | ✅ Correct — first-person emotional response, no thesis. |
| "Tesla will easily hit $500 by next year." | Speculative Opinion | 36.2% | ✅ Correct — bold prediction, no evidence. |
| "Buy SMCI before earnings — this thing is going to rip." | Speculative Opinion | 36.8% | ✅ Correct — trade call without data; all three scores within ~8 points (fragile boundary). |
| "There's no soft landing. The Fed has never pulled this off..." | Market Reaction | 37.7% | ❌ Wrong — macro speculative call misread as reaction (Failure 1). |

---

## Reflection: Intended vs. Learned Behavior

**What I intended the model to learn:** Three-way distinction — argue with evidence, assert without evidence, express emotion. Topic words like "Fed" or "Bitcoin" should not determine the label; structure should.

**What the model actually learned:** All three categories at F1 ≥ 0.90. Evidence-Based Analysis was perfect (11/11) — the model reliably detects posts with structured data and reasoning. Speculative and Reaction are mostly distinguished, but the two errors show symmetric confusion: macro urgency → Reaction (Failure 1), personal dollar narrative → Speculative (Failure 2).

**Specific failure pattern:** Posts that share surface features across the speculative/reaction boundary — Fed mentions, dollar figures, urgency — without clear first-person emotional framing or clear market-thesis framing. Confidence on all demo posts was ~36–38%, suggesting the model treats these short forum posts as inherently ambiguous rather than overconfidently wrong.

**What would fix it:** Paired training examples on the same topic with different structure: "The Fed will cause a hard landing" (speculative) vs. "Fed just hiked and I'm sick" (reaction); "Made $47K then lost it all" (reaction) vs. "This stock will 10x" (speculative). The Groq baseline still hit 100% on the same test set, so the labels are learnable — fine-tuning closed most of the gap but didn't fully match a 70B zero-shot model.

---

## Spec Reflection

**How planning.md helped:** Writing the "remove the opinion framing" decision rule before annotating kept my labels consistent across 211 examples. Without it, I would have drifted — some one-stat posts labeled analysis, others speculative.

**Where implementation diverged:** I used synthetic/paraphrased posts modeled on public forum style instead of raw scraped Reddit text. Cleaner for annotation, but the model may not see typos, slang, or meme formatting that real r/wallstreetbets posts have. If performance is high on our CSV but feels brittle on messy real posts, that's probably why.

---

## AI Usage Disclosure

1. **Label stress-testing:** Asked Claude to generate borderline posts between Evidence-Based Analysis and Speculative Opinion before finalizing definitions. Surfaced the "mentions P/E without benchmark" case — added explicit rule in `planning.md`.

2. **Annotation pre-labeling:** Used Claude to suggest labels for batches of ~20 unlabeled posts. Reviewed and corrected every example manually; noted edge cases in the CSV `notes` column. Did not bulk-accept AI labels.

3. **Failure analysis (post-Colab):** Pasted both misclassified test examples into Claude. It flagged "Fed + urgency → reaction" for Failure 1 and "dollar figures → speculative" for Failure 2. Verified manually — both patterns fit. Noted that demo Post 4 (SMCI) was now correctly classified as Speculative at 36.8%, showing the boundary is fragile but learnable.

4. **Project setup:** Cursor assisted with `planning.md` structure, README template, and notebook label-map configuration. All label decisions and dataset content were reviewed by me.

---