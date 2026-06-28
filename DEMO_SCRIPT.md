# Demo Video Script (~4 minutes)

**Record:** Screen-share Colab (demo cell output) + briefly scroll README evaluation section.  
**Before recording:** Re-run Colab Sections 1–4 so the model is loaded, then run the demo cell below.

---

## Colab demo cell (paste after Section 4)

```python
demo_posts = [
    "NVDA's revenue grew 72% YoY while gross margins expanded to 78%. Even at a high P/E, that earnings trajectory justifies a premium valuation.",
    "RIP my portfolio after the Fed announcement.",
    "Tesla will easily hit $500 by next year.",
    "Buy SMCI before earnings — this thing is going to rip.",
    "There's no soft landing. The Fed has never pulled this off and won't now.",
]

model.eval()
for i, text in enumerate(demo_posts, 1):
    inputs = tokenizer(text, return_tensors="pt", truncation=True, max_length=256)
    if torch.cuda.is_available():
        inputs = {k: v.cuda() for k, v in inputs.items()}
        model.cuda()
    with torch.no_grad():
        logits = model(**inputs).logits
    probs = torch.softmax(logits, dim=-1)[0].cpu().numpy()
    pred_id = int(probs.argmax())
    print(f"\n{'='*60}")
    print(f"Post {i}: {text}")
    print(f"→ {ID_TO_LABEL[pred_id]}  ({probs[pred_id]:.1%} confidence)")
```

---

## Script

### [0:00 – 0:30] Introduction

> "Hi — this is my TakeMeter project for AI201. I built a text classifier for investing forum posts. The community is r/investing-style discourse — earnings threads, hot takes, and panic posts all mixed together.
>
> I defined three labels: **Evidence-Based Analysis** — posts that argue with specific data; **Speculative Opinion** — bold claims without evidence; and **Market Reaction** — emotional or personal responses to market moves.
>
> I fine-tuned DistilBERT on 211 labeled examples in Google Colab and compared it to a zero-shot Groq baseline using Llama 3.3 70B."

**[Show:** README labels section or planning.md]

---

### [0:30 – 2:00] Live classifications (5 posts)

**[Show:** Colab demo cell output — label + confidence must be visible]

> "Let me run five posts through the fine-tuned model."

**Post 1 — NVDA margins** *(correct)*

> "Post 1: NVDA revenue up 72%, margins at 78%. The model predicts **Evidence-Based Analysis** at high confidence. That makes sense — it cites specific numbers and connects them to valuation. This is exactly the pattern the model learned best: 11 out of 11 analysis posts correct on the test set."

**Post 2 — RIP portfolio** *(correct)*

> "Post 2: 'RIP my portfolio after the Fed announcement.' Predicted **Market Reaction** — also high confidence. First-person, emotional, no argument. The model got every reaction post right on the test set too."

**Post 3 — Tesla $500** *(likely correct)*

> "Post 3: 'Tesla will easily hit $500 by next year.' **Speculative Opinion** — bold prediction, no data. This is one of the harder categories; the model only caught 3 out of 11 speculative posts on the test set."

**Post 4 — Buy SMCI** *(wrong)*

> "Post 4: 'Buy SMCI before earnings — this thing is going to rip.' The model says **Market Reaction** at about 37% confidence — and this is actually wrong. The true label is Speculative Opinion."

**Post 5 — No soft landing** *(wrong)*

> "Post 5: 'There's no soft landing. The Fed has never pulled this off.' Also predicted as **Market Reaction** at 39% confidence — wrong again. It's a macro thesis without evidence, so it's speculative, not a personal reaction."

---

### [2:00 – 2:45] Incorrect prediction deep-dive

**[Show:** Post 4 or Post 5 output + confusion matrix]

> "Let me explain why Post 4 failed. It's a trade recommendation — 'buy before earnings, going to rip.' There's no personal portfolio language, but the hype and urgency sound like excited reaction posts such as 'Up 34% on my MSFT position, I'm shaking.'
>
> The model learned a shortcut: numbers and structure mean analysis, first-person emotion means reaction — and everything urgent in the middle gets labeled reaction. On the confusion matrix, seven of eight errors are speculative posts misclassified as reaction — that cell right here."

**[Point at confusion matrix:** True Speculative → Pred Reaction = 7]

---

### [2:45 – 3:45] Evaluation report walkthrough

**[Show:** README Evaluation Summary or Colab Section 6 output]

> "Overall, the fine-tuned model hit **75% accuracy** on 32 test examples. The Groq zero-shot baseline got **100%** — so fine-tuning actually regressed by 25 points.
>
> Per-class: Analysis F1 was **0.96**, Reaction F1 was **0.74**, but Speculative F1 was only **0.43** — recall of 27%, meaning it missed almost three quarters of hot takes.
>
> That's not deployment-ready by the criteria I set in planning.md, but it's an honest result: the task is learnable — the baseline proves that — but DistilBERT on 211 examples didn't capture the speculative boundary. The model collapsed the middle category into reaction."

---

### [3:45 – 4:00] Closing

> "The takeaway: label design mattered — my hardest boundary was speculative versus reaction — and the evaluation shows exactly where fine-tuning failed. Full write-up, dataset, and results are in the GitHub repo. Thanks."

---

## Recording checklist

- [ ] Label + confidence visible on screen for all 5 posts
- [ ] One correct prediction explained (Post 1 or 2)
- [ ] One incorrect prediction explained (Post 4 or 5)
- [ ] Evaluation numbers stated (0.75 vs 1.00 accuracy, Speculative F1 0.43)
- [ ] Confusion matrix cell pointed out (7 speculative → reaction)
- [ ] Add video link to README Demo Video section after upload
