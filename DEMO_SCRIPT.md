# Demo Video Script — Final Results (~4–5 min)

Use after your **second Colab run** (94% accuracy, 2 errors). Every rubric-required moment is marked **[REQUIRED]**.

---

## Before you record

1. Colab Sections 1–4 already run — model loaded.
2. Run the **demo cell** below.
3. Have open: demo output, `confusion_matrix.png`, README Evaluation section, Section 6 comparison.
4. Record screen + mic.

---

## Colab demo cell

```python
demo_posts = [
    "NVDA's revenue grew 72% YoY while gross margins expanded to 78%. Even at a high P/E, that earnings trajectory justifies a premium valuation.",
    "RIP my portfolio after the Fed announcement.",
    "Tesla will easily hit $500 by next year.",
    "Buy SMCI before earnings — this thing is going to rip.",
    "There's no soft landing. The Fed has never pulled this off and won't now.",
]

model.eval()
print("FINE-TUNED MODEL — LIVE CLASSIFICATIONS\n")

for i, text in enumerate(demo_posts, 1):
    inputs = tokenizer(text, return_tensors="pt", truncation=True, max_length=256)
    if torch.cuda.is_available():
        inputs = {k: v.cuda() for k, v in inputs.items()}
        model.cuda()
    with torch.no_grad():
        logits = model(**inputs).logits
    probs = torch.softmax(logits, dim=-1)[0].cpu().numpy()
    pred_id = int(probs.argmax())
    print("=" * 70)
    print(f"POST {i}")
    print(f"Text:       {text}")
    print(f"Predicted:  {ID_TO_LABEL[pred_id]}")
    print(f"Confidence: {probs[pred_id]:.1%}")
    print(f"All scores: ", end="")
    print(", ".join(f"{ID_TO_LABEL[j]}: {probs[j]:.1%}" for j in range(NUM_LABELS)))
```

**Expected demo output (your run):**

| Post | Predicted | Confidence | Correct? |
|---|---|---|---|
| 1 NVDA margins | Evidence-Based Analysis | 38.2% | ✅ |
| 2 RIP portfolio | Market Reaction | 37.4% | ✅ |
| 3 Tesla $500 | Speculative Opinion | 36.2% | ✅ |
| 4 Buy SMCI | Speculative Opinion | 36.8% | ✅ |
| 5 No soft landing | Market Reaction | 37.7% | ❌ (test error #1) |

---

## Rubric checklist

| Requirement | Where |
|---|---|
| **[REQUIRED]** 5 posts, label + confidence visible | Section 2 |
| **[REQUIRED]** One correct prediction explained | Section 3 — Post 1 |
| **[REQUIRED]** One incorrect prediction explained | Section 4 — Post 5 |
| **[REQUIRED]** Eval walkthrough with key metrics | Section 5 |

---

## Full script

### SECTION 1 — Introduction [0:00 – 0:40]

**[Show:** README Labels section]

> "This is my AI201 TakeMeter project — a classifier for **investing forum posts** on communities like r/investing and r/stocks.
>
> Three labels: **Evidence-Based Analysis** — arguments with verifiable data; **Speculative Opinion** — bold claims without evidence; **Market Reaction** — emotional or personal responses to the market.
>
> I labeled **211 examples**, fine-tuned **DistilBERT on Google Colab**, and compared against a **Groq zero-shot baseline** on 32 held-out test posts."

---

### SECTION 2 — Live classifications [0:40 – 2:10]

**[REQUIRED] Show demo cell output — label + confidence for all 5 posts.**

> "Five posts through the fine-tuned model."

**Post 1 — NVDA** *(✅ Analysis, 38.2%)*
> "Revenue up 72%, margins 78% — predicted **Evidence-Based Analysis**. Specific metrics supporting a valuation claim."

**Post 2 — RIP portfolio** *(✅ Reaction, 37.4%)*
> "'RIP my portfolio after the Fed announcement' — **Market Reaction**. First-person, emotional, no argument."

**Post 3 — Tesla $500** *(✅ Speculative, 36.2%)*
> "Bold price target, no data — **Speculative Opinion**."

**Post 4 — Buy SMCI** *(✅ Speculative, 36.8%)*
> "Trade call before earnings — **Speculative Opinion**. Got it right, but confidence is only 37% — all three scores are close."

**Post 5 — No soft landing** *(❌ Reaction, 37.7%)*
> "'No soft landing, Fed never pulled this off' — model says **Market Reaction**. **Wrong** — true label is **Speculative Opinion**. This was one of only **2 errors** on the entire test set."

---

### SECTION 3 — Correct prediction deep-dive [2:10 – 2:50]

**[REQUIRED] Explain why a correct prediction is reasonable.**

**[Show:** Post 1 output]

> "Post 1 is a strong correct prediction. The model chose **Evidence-Based Analysis** because the post cites **72% revenue growth** and **78% gross margins** and connects them to valuation — exactly my definition in planning.md.
>
> On the test set, analysis was **perfect: 11 out of 11**, F1 **1.00**. When the structure is clearly data → conclusion, the model is reliable.
>
> Note confidence is only 38% — the model spreads probability across classes on short posts, but Analysis still wins."

---

### SECTION 4 — Incorrect prediction deep-dive [2:50 – 3:40]

**[REQUIRED] Explain what went wrong and why — not just 'it got it wrong'.**

**[Show:** Post 5 output + confusion matrix**

> "Post 5 — my test-set error. True label: **Speculative Opinion**. Predicted: **Market Reaction** at 38%.
>
> **Why it's wrong:** This is a macro claim — 'no soft landing, Fed can't pull it off' — with no personal portfolio language and no data. That's a market thesis, not someone venting about their account.
>
> **Why the model confused it:** 'Fed' plus urgent tone resembles reaction posts like 'Fed raised rates and the market is tanking.' The model picked sentiment register over claim type.
>
> **The other error** (not in demo cell) went the opposite way: a personal portfolio journey with dollar figures — 'turned $5K into $47K then lost it' — was labeled Speculative because numbers sounded analytical. So the boundary fails **both directions**: macro urgency → Reaction, personal numbers → Speculative.
>
> **Confusion matrix:** only **2 off-diagonal cells** — Speculative→Reaction (1), Reaction→Speculative (1). Everything else correct."

**[Point at:** True Speculative → Pred Reaction = 1, True Reaction → Pred Speculative = 1]

---

### SECTION 5 — Evaluation walkthrough [3:40 – 4:40]

**[REQUIRED] Key metrics: accuracy, F1, baseline comparison.**

**[Show:** Section 6 comparison + README eval table]

> "**Results on 32 test examples:**
> - Groq baseline: **100%** accuracy
> - Fine-tuned DistilBERT: **94%** — 30 correct, 2 wrong
> - Fine-tuning regressed by **6 points** — baseline still wins, but much closer than it could have been
>
> **Per-class F1:** Analysis **1.00**, Speculative **0.91**, Reaction **0.90** — all above my 0.70 threshold.
>
> **Success criteria:** accuracy ✅, per-class F1 ✅, beat baseline ❌ (missed by 0.06).
>
> **Takeaway:** Fine-tuning works well for this task — 94% with balanced classes — but a 70B zero-shot model with explicit label definitions in the prompt still edges it out. The remaining errors are both on the speculative/reaction boundary, which is exactly where I predicted the hardest cases would live in planning.md."

---

### SECTION 6 — Closing [4:40 – 5:00]

**[Show:** GitHub repo]

> "Repo has planning.md, the dataset, notebook, evaluation results, and full error analysis. Thanks."

---

## Quick reference — final numbers

| Metric | Value |
|---|---|
| Test set | 32 |
| Baseline accuracy | **1.00** |
| Fine-tuned accuracy | **0.94** |
| Change | **−0.06** |
| Analysis F1 | **1.00** (11/11) |
| Speculative F1 | **0.91** (10/11) |
| Reaction F1 | **0.90** (9/10) |
| Errors | **2 / 32** |

### Both test errors

| Text | True | Predicted | Conf |
|---|---|---|---|
| No soft landing / Fed never pulled this off | Speculative | Reaction | 0.38 |
| $5K→$47K→$9K portfolio journey | Reaction | Speculative | 0.33 |

---

## Recording checklist

- [ ] 5 posts with label + confidence on screen
- [ ] Post 1 correct — explained with data/structure reasoning
- [ ] Post 5 wrong — explained with boundary + confusion matrix
- [ ] Stated 0.94 vs 1.00, F1 scores, 2 errors
- [ ] Add video link to README
