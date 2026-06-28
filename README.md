# ai201-project3-takemeter

Fine-tuned text classifier for **investing community discourse** — distinguishes evidence-based analysis, speculative opinions, and emotional market reactions.

Built for AI201 Project 3 (TakeMeter). Trained on 211 labeled examples; evaluated against a Groq zero-shot baseline on a held-out test set.

### Evaluation Summary

<!-- Fill in after Colab run -->

| | Baseline (Groq) | Fine-tuned (DistilBERT) |
|---|---|---|
| **Overall accuracy** | _TBD_ | _TBD_ |
| **Best per-class F1** | _TBD_ | _TBD_ |
| **Worst per-class F1** | _TBD_ | _TBD_ |

_Fine-tuning improved accuracy by _TBD_ on the held-out test set. The main confusion pattern was _TBD_ → _TBD_. Full breakdown below._

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
| `evaluation_results.json` | Test-set metrics (update after Colab run) |
| `confusion_matrix.png` | Fine-tuned model confusion matrix (update after Colab run) |

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

**Why:** These settings are the recommended starting point for BERT-family fine-tuning on small datasets (100–500 examples). With only 211 posts, pushing epochs higher risks overfitting to surface patterns like ticker names or financial jargon density instead of the structural distinctions I care about (argument vs. assertion vs. emotion). Batch size 16 fits comfortably on a T4 without OOM errors. If validation accuracy plateaued early or dropped on later epochs, that would be the signal to reduce epochs — I'll note here if the Colab run showed that.

---

## Evaluation Report

<!-- ↓↓↓ Fill in after Colab run ↓↓↓ -->

> **Status:** Pending Colab run.

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq / Llama 3.3 70B) | _TBD_ |
| Fine-tuned DistilBERT | _TBD_ |
| Improvement from fine-tuning | _TBD_ |

**Success criteria** (from `planning.md`): accuracy ≥ 0.75, all per-class F1 ≥ 0.70, fine-tuned beats baseline by ≥ 0.10.

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 |
|---|---|---|---|
| Evidence-Based Analysis | _TBD_ | _TBD_ | _TBD_ |
| Speculative Opinion | _TBD_ | _TBD_ | _TBD_ |
| Market Reaction | _TBD_ | _TBD_ | _TBD_ |

### Per-Class Metrics — Baseline (Groq)

| Label | Precision | Recall | F1 |
|---|---|---|---|
| Evidence-Based Analysis | _TBD_ | _TBD_ | _TBD_ |
| Speculative Opinion | _TBD_ | _TBD_ | _TBD_ |
| Market Reaction | _TBD_ | _TBD_ | _TBD_ |

### Confusion Matrix — Fine-Tuned Model (Test Set)

_Rows = true label, columns = predicted label. Copy values from Colab Section 4._

|  | Pred: Analysis | Pred: Speculative | Pred: Reaction |
|---|---|---|---|
| **True: Analysis** | _TBD_ | _TBD_ | _TBD_ |
| **True: Speculative** | _TBD_ | _TBD_ | _TBD_ |
| **True: Reaction** | _TBD_ | _TBD_ | _TBD_ |

![Confusion matrix](confusion_matrix.png)

---

## Error Analysis

<!-- Pick 3 misclassified test examples from Colab Section 4 output -->

### Failure 1 — _[short title, e.g. "One-stat P/E post"]_

- **Text:** _TBD_
- **True label:** _TBD_ → **Predicted:** _TBD_ (confidence: _TBD_)
- **Why it failed:** _TBD — which label boundary? Did the model latch onto a keyword, post length, or topic? Is this a labeling issue or a training data gap?_

### Failure 2 — _[short title]_

- **Text:** _TBD_
- **True label:** _TBD_ → **Predicted:** _TBD_ (confidence: _TBD_)
- **Why it failed:** _TBD_

### Failure 3 — _[short title]_

- **Text:** _TBD_
- **True label:** _TBD_ → **Predicted:** _TBD_ (confidence: _TBD_)
- **Why it failed:** _TBD_

**Dominant confusion pattern:** _TBD — e.g. "Analysis → Speculative on posts mentioning one metric without comparative reasoning." Verify against confusion matrix above._

---

## Sample Classifications

_Run 3–5 posts through the fine-tuned model in Colab (demo cell after Section 4). Include at least one correct prediction with reasoning._

| Post (truncated) | Predicted Label | Confidence | Notes |
|---|---|---|---|
| _TBD_ | _TBD_ | _TBD_ | _Why this correct prediction is reasonable_ |
| _TBD_ | _TBD_ | _TBD_ | |
| _TBD_ | _TBD_ | _TBD_ | |
| _TBD_ | _TBD_ | _TBD_ | |
| _TBD_ | _TBD_ | _TBD_ | |

---

## Reflection: Intended vs. Learned Behavior

<!-- Expand with specific patterns after Colab — avoid generic "needs more data" -->

**What I intended the model to learn:** Whether a post *argues with evidence* vs. *asserts without evidence* vs. *expresses emotion* — structural distinctions, not topic (mentioning "Fed" or "NVDA" shouldn't determine the label).

**What the model likely learned:** _TBD — e.g. surface cues like financial jargon density, post length, ticker mentions._

**Specific failure pattern:** _TBD — e.g. "The model treats any post mentioning P/E or revenue as Analysis, even when the stat isn't used in an argument. This matches the Analysis → Speculative off-diagonal in the confusion matrix."_

**What would fix it:** _TBD — e.g. more borderline one-stat examples in training, or tighter label definitions._

---

## Spec Reflection

**How planning.md helped:** Writing the "remove the opinion framing" decision rule before annotating kept my labels consistent across 211 examples. Without it, I would have drifted — some one-stat posts labeled analysis, others speculative.

**Where implementation diverged:** I used synthetic/paraphrased posts modeled on public forum style instead of raw scraped Reddit text. Cleaner for annotation, but the model may not see typos, slang, or meme formatting that real r/wallstreetbets posts have. If performance is high on our CSV but feels brittle on messy real posts, that's probably why.

---

## AI Usage Disclosure

1. **Label stress-testing:** Asked Claude to generate borderline posts between Evidence-Based Analysis and Speculative Opinion before finalizing definitions. Surfaced the "mentions P/E without benchmark" case — added explicit rule in `planning.md`.

2. **Annotation pre-labeling:** Used Claude to suggest labels for batches of ~20 unlabeled posts. Reviewed and corrected every example manually; noted edge cases in the CSV `notes` column. Did not bulk-accept AI labels.

3. **Failure analysis (post-Colab):** _TBD — paste misclassified examples into Claude, identify pattern themes, verify manually._

4. **Project setup:** Cursor assisted with `planning.md` structure, README template, and notebook label-map configuration. All label decisions and dataset content were reviewed by me.

---

## Demo Video

**Link:** _TBD_

3–5 minute walkthrough covering:

- [ ] 3–5 live classifications with label + confidence visible
- [ ] One correct prediction narrated (why the label fits)
- [ ] One incorrect prediction narrated (what boundary failed and why)
- [ ] Brief evaluation report walkthrough (accuracy, one F1, one confusion-matrix cell)

---

## Author

AI201 Project 3 — TakeMeter
