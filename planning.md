# TakeMeter Project Plan — Investing Discourse Classifier

*Working notes written before and during the project. This is the messy, honest version — README.md is the polished report.*

---

## Community

I chose **online investing and personal-finance communities** — the kind of posts you see on r/investing, r/stocks, r/SecurityAnalysis, and finance Twitter threads. Not WSB meme chaos (mostly), but the everyday mix of people trying to figure out markets.

Why this community works for classification:

- People actually argue about discourse quality here. "Is this analysis or just a hot take?" is a real question regulars ask.
- The text is varied: earnings breakdowns sit next to panic posts and bold predictions with no numbers.
- It's text-heavy and public — easy to collect 200+ examples without scraping behind a login.

The distinctions matter because bad signal drowns out good signal in these spaces. A feed-sorting tool that surfaces evidence-based posts and deprioritizes pure reaction would be genuinely useful to someone trying to learn investing, not just feel something about the market.

---

## Labels

Three labels. Each one maps to a recognizable way people talk about money online.

### 1. Evidence-Based Analysis

**Definition:** The post makes a structured argument backed by specific, verifiable data — revenue figures, historical comparisons, valuation multiples, macro indicators, or company fundamentals — and uses that evidence to support a conclusion.

**Clear examples:**
- "NVDA's revenue grew 72% YoY while gross margins expanded to 78%. Even at a high P/E, that earnings trajectory justifies a premium valuation."
- "The S&P 500 forward P/E is currently 21x, about 18% above its 20-year median of 17.8x. That doesn't make stocks a sell, but the margin of safety is thin."

**Uncertain example:**
- "Nvidia is overpriced because the P/E ratio is huge." — mentions P/E but doesn't compare to peers, history, or growth. **Decision: Speculative Opinion** (the stat is decorative, not part of an argument).

---

### 2. Speculative Opinion

**Definition:** The post states a bold financial claim, prediction, or recommendation without citing specific, verifiable evidence — even if it sounds confident or references a general concept (like "P/E is high" or "rates are too high").

**Clear examples:**
- "Tesla will easily hit $500 by next year."
- "The Fed is going to break something. Rates can't stay this high."

**Uncertain example:**
- "Commercial real estate is in a slow-motion collapse that the market isn't pricing." — plausible concern, but no specific data (delinquency rates, loan maturities, etc.). **Decision: Speculative Opinion**.

---

### 3. Market Reaction

**Definition:** The post is primarily an immediate emotional or personal response to market movement — portfolio pain, FOMO, relief, panic — with little to no analytical argument about why an asset should move.

**Clear examples:**
- "RIP my portfolio after the Fed announcement."
- "Up 34% on my MSFT position today after earnings. I'm shaking."

**Uncertain example:**
- "Honestly can't tell if this market is a gift or a trap right now." — could sound speculative, but the primary register is personal uncertainty/emotion, not a market thesis. **Decision: Market Reaction**.

---

## Hard Edge Cases

### The main boundary: Analysis vs. Speculative Opinion

This is where most of my annotation time went. The rule I settled on:

> **If you removed the opinion framing and the specific evidence would still support the claim on its own, label it Evidence-Based Analysis. If the evidence is vague, cherry-picked, or just enough to sound credible without actually reasoning, label it Speculative Opinion.**

**Example that forced this rule:**

*"ASML is 50% overvalued. Semi cycles don't support these multiples."*

Could be analysis (mentions valuation and cycles) or speculation (no actual numbers, no peer comparison). I labeled it **Speculative Opinion** because "these multiples" is hand-wavy — there's no P/E, no cycle data, no historical comparison.

### Secondary boundary: Speculative Opinion vs. Market Reaction

Some posts mix a vague thesis with personal emotion. Rule:

> **If the post is mostly about the author's portfolio experience or feelings in the moment, label Market Reaction — even if there's a hint of market commentary.**

*"This rally feels fake to me. Something doesn't add up."* → **Market Reaction** (gut feeling, not an argument).

---

### Difficult cases I actually labeled (documented during annotation)

1. **"Nvidia is overpriced because the P/E ratio is huge."**
   - Could be: Analysis (cites P/E) or Speculative (no benchmark)
   - **Decided:** Speculative Opinion — P/E alone without context isn't analysis in this community's standards.

2. **"The market is crashing because everyone is panicking."**
   - Could be: Speculative (causal claim) or Market Reaction (observational/emotional)
   - **Decided:** Market Reaction — primary register is describing the moment, not building an investment thesis.

3. **"Buffett's cash hoard tells you everything you need to know about this market."**
   - Could be: Analysis (references real fact) or Speculative (vague implication)
   - **Decided:** Speculative Opinion — the cash pile is real, but "tells you everything" is an assertion without reasoning about what it means.

---

## Data Collection Plan

**Sources:** Posts and comments styled after public content from r/investing, r/stocks, r/SecurityAnalysis, and finance Twitter. All examples are synthetic or paraphrased composites — no private messages, no paywalled content.

**Target:** 200+ examples, roughly balanced across labels.

**Per-label target:** ~70 examples each (33% per class).

**If a label is underrepresented after 200 examples:** Collect 15–20 more from that category before training. I won't proceed if any single label exceeds 70% of the dataset.

**Actual distribution after labeling (211 total):**

| Label | Count | Share |
|---|---|---|
| Evidence-Based Analysis | 70 | 33.2% |
| Speculative Opinion | 71 | 33.6% |
| Market Reaction | 70 | 33.2% |

Saved as `finance_discourse_labeled.csv` with columns: `text`, `label`, `notes`.

---

## Evaluation Metrics

**Why accuracy alone isn't enough:** On a 3-class balanced dataset, a model could hit ~33% by always guessing the majority class. Accuracy tells me the headline number; it doesn't tell me which boundaries failed.

**Metrics I'll use:**

| Metric | Why it matters for this task |
|---|---|
| **Overall accuracy** | Simple sanity check — is the model learning anything at all? |
| **Per-class precision** | Of posts the model calls "analysis," how many actually are? High precision = fewer false positives when surfacing quality content. |
| **Per-class recall** | Of all real analysis posts, how many did we catch? High recall = we're not hiding good posts. |
| **Per-class F1** | Harmonic mean — the single best number per label for comparing boundaries. |
| **Confusion matrix** | Shows directional errors (e.g., analysis → speculative) which map directly to label-boundary problems. |
| **Baseline comparison (Groq zero-shot)** | Proves fine-tuning added value vs. a general LLM with no training on my labels. |

---

## Definition of Success

For a real community moderation or feed-ranking tool, I'd want:

- **Overall accuracy ≥ 0.75** on the held-out test set
- **Per-class F1 ≥ 0.70** for all three labels — if one class collapses, the tool would systematically mis-rank that type of post
- **Fine-tuned model beats Groq baseline by ≥ 0.10 accuracy** — otherwise fine-tuning wasn't worth the effort
- **Unparseable baseline responses < 10%** — if the Groq prompt can't get clean outputs, the baseline comparison is unreliable

"Good enough for deployment" means a moderator could trust the top-ranked "Evidence-Based Analysis" bucket without manually checking every post. If analysis recall is below 0.60, we'd be hiding too much good content. If speculative precision is below 0.65, we'd still be surfacing hot takes as analysis.

I'll know I hit or missed these thresholds objectively once I have test-set numbers from Colab.

---

## AI Tool Plan

### Label stress-testing (before annotation)

I'll give an AI my three label definitions and the analysis-vs-speculation edge case rule, and ask it to generate 8–10 posts sitting right on that boundary. If I can't classify them cleanly with my rules, I'll tighten the definitions before labeling 200 examples.

**Decision:** Do this. Already surfaced cases like "mentions P/E but no benchmark" — added explicit rule for that.

### Annotation assistance

**Decision:** Use Claude to draft a first-pass label for batches of ~20 unlabeled posts, then read every single one myself and correct mistakes. Track pre-labeled rows in the `notes` column (e.g., "AI suggested Speculative, confirmed after review").

I will not bulk-accept AI labels without reading. Skimming defeats the purpose.

### Failure analysis (after training)

After Colab evaluation, I'll paste misclassified test examples into an AI tool and ask: "What patterns do you see?" I'll look for:

- Consistent confusion between two labels (especially Analysis ↔ Speculative)
- Length effects (very short posts)
- Posts where topic words (e.g., "Fed", "P/E") trigger wrong labels

I'll verify every pattern manually by re-reading the examples — AI suggestions are hypotheses, not conclusions.

---

## Spec Reflection (working notes — expanded in README after results)

**How the spec helped:** Writing label decision rules before annotating prevented me from drifting mid-dataset. The "remove the opinion framing" test for analysis vs. speculation came directly from the milestone guidance and saved me from inconsistent labels.

**Where I diverged:** I used synthetic/paraphrased posts instead of raw Reddit scrapes — faster and no API/scraping setup, but the language might be cleaner than real forum text. I'll note in README if the model struggles on messy grammar or slang we didn't capture.

---

*Last updated: before Colab training run. Evaluation numbers and failure analysis will be added to README after Milestones 4–6.*
