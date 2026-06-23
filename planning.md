# TakeMeter — Planning Document

## Milestone 1: Community + Label Design

### Community Description

r/nba is Reddit's primary NBA community with millions of members who discuss all matters regarding the NBA — players, coaches, stats, trades, and GOAT debates. The discourse varies wildly in quality: some posts bring real evidence and structured arguments, others are bold claims stated with total confidence and nothing behind them. The ANALYSIS vs HOT_TAKE distinction matters here because it's genuinely hard to tell at times whether a user is backing up their claim with stats and reasoning or just asserting an opinion as fact.

---

### Label Taxonomy

#### Label 1: ANALYSIS

**Definition:** Makes a structured argument backed by statistics, historical comparison, or tactical observation. Evidence is specific and verifiable. The post argues rather than asserts.

**Clear example:**

> "They had 2 of the 5 20 point 4th quarter comebacks in playoff HISTORY this year including the greatest comeback in finals history with a 29 point comeback when the old record was 24. They also came back from 20+ in the 2nd half twice against the Celtics in 2025. They pulled off over half of the biggest 4th quarter comebacks in playoff history in a 2 year span."

Why this is ANALYSIS: specific stats, historical comparisons, verifiable numbers that build toward a conclusion.

**Edge case:**

Same post as above, but the conclusion is "best comeback team ever."

Why this is hard: the stats are real and verifiable, but the conclusion overshoots the evidence with an absolute superlative. Decision: label as HOT_TAKE — if a post uses real stats but concludes with an absolute superlative ("best ever," "greatest of all time"), the conclusion is no longer proportional to the evidence. If they had said "one of the best" it would remain ANALYSIS.

---

#### Label 2: HOT_TAKE

**Definition:** A bold, confident opinion stated without supporting evidence — often uses extreme or absolute language. May be deliberately provocative or genuinely held, but the post asserts rather than argues.

**Clear example:**

> "NBA is a rigged league. Tonight could be my last NBA game and I'd be totally fine. This game is no longer fun to watch once you know what's really going on."

Why this is HOT_TAKE: makes a broad unsupported claim ("rigged league"), no evidence provided, asserts rather than argues.

**Edge case:**

A post with real stats that concludes with "the best ever" or "not even close."

Why this is hard: the presence of stats makes it look like ANALYSIS, but the leap to an absolute superlative is unsupported by the evidence. Decision: label as HOT_TAKE — stats alone don't make something ANALYSIS if the conclusion overreaches the evidence.

---

### Labeling Rules

These rules apply when a post is ambiguous:

1. **The core test:** Does this post provide specific, verifiable evidence for its claim? Yes → ANALYSIS. No → HOT_TAKE.

2. **Stats + superlative rule:** If a post uses real stats but concludes with an absolute superlative ("best ever," "GOAT," "not even close," "nobody does it better") — label it HOT_TAKE. If the conclusion is proportional to the evidence ("one of the best," "historically significant") — label it ANALYSIS.

3. **Assertion vs. argument rule:** If you can ask "what's the evidence?" and there's a real answer in the post — ANALYSIS. If the post just states the claim confidently with nothing behind it — HOT_TAKE.

4. **Emotion + claim rule:** When a post has both reactive emotion AND a broad unsupported claim, label based on the core assertion, not the emotion.

---

### Mutual Exclusivity Check

| Potential Overlap | Resolution |
|---|---|
| ANALYSIS vs HOT_TAKE | Ask: does the post provide specific, verifiable evidence that is proportional to its conclusion? If yes → ANALYSIS. If no, or if the conclusion overshoots the evidence with a superlative → HOT_TAKE. |

---

## Milestone 2: Data Collection

### What Actually Happened

Originally planned to use PRAW (Reddit API) for automated scraping. In practice, data was collected manually by copying comment threads from r/nba directly, which gave better control over thread quality and comment relevance.

**Threads scraped:**
1. KD "underdog" controversy thread (~70 comments)
2. "Best player in the league" debate thread (~65 comments)
3. Overrated/underrated player poll thread (~100 comments)
4. Bam Adebayo 83-point game / "ethical basketball" thread (~47 comments)
5. MJ vs LeBron GOAT debate thread (~31 comments)
6. Pacers vs OKC film breakdown thread (~19 comments)
7. NBA analytics / "bad players who look good under analytics" thread (~16 comments)

**Final dataset:** 200 rows after trimming hot_takes for balance  
**Split:** 70/15/15 (train/validation/test) — handled automatically by the Colab notebook  
**Actual split sizes:** ~140 train / ~30 validation / ~30 test

**Final label distribution:**
- HOT_TAKE: 141 (70.5%)
- ANALYSIS: 59 (29.5%)
- Total: 200

**Original target was 50/50 balance.** This was not achievable because r/nba comment threads are naturally dominated by hot takes. Analytical comments with specific verifiable evidence are a minority of the discourse. The 70/30 split reflects the real distribution of the community and stays within the project requirement of no label exceeding 70%.

---

### Annotation Decisions

**3-label → 2-label collapse (key decision):**  
Originally designed with 3 labels: ANALYSIS, HOT_TAKE, and REACTION. During early labeling, the REACTION/HOT_TAKE boundary proved too blurry to apply consistently — short emotional responses ("Lmao get a grip") and confident assertions ("Softest superstar of all time") were functionally identical in terms of discourse quality. Both lack evidence and assert rather than argue. Decision: collapse REACTION into HOT_TAKE. This made the taxonomy more consistent and the boundary cleaner to learn.

**Taxonomy loosening (Option A):**  
The strict taxonomy (requiring hard verifiable stats) produced only 22 ANALYSIS examples out of 235 rows — an 89/11 split that violated the project requirement. After review, the ANALYSIS definition was loosened to include posts that make a reasoned argument with specific observable evidence even without hard statistics. For example, "The Bulls' three first round losses were against the 59-win Bucks in Jordan's rookie year, and back-to-back against the Celtics dynasty" — this cites specific teams and verifiable outcomes rather than raw numbers, but qualifies as ANALYSIS under the looser standard because the evidence is specific and the conclusion is proportional.

**Borderline candidates reviewed manually:**  
Approximately 27 borderline candidates were identified and reviewed one by one. Only those where the evidence was specific and the conclusion proportional were flipped to ANALYSIS. Posts with reasoning but no verifiable evidence stayed HOT_TAKE.

---

### Evaluation Metrics

Accuracy alone isn't enough because you need to see how the model handles each class specifically and where it is getting confused. F1 score is the right metric because it combines both precision and recall evenly — since both types of errors (missing a HOT_TAKE or mislabeling an ANALYSIS) are equally costly for this task. The confusion matrix shows exactly where the model is getting confused between the two labels.

**Metrics reported:**
- Overall accuracy (both fine-tuned model and Groq zero-shot baseline)
- F1 score per label (ANALYSIS and HOT_TAKE)
- Confusion matrix
- At least 3 specific wrong predictions with analysis of why

**Definition of Success:**  
My definition of success is an F1 score of 80% or above. This is high enough to be genuinely useful — wrong only 2 in 10 times — while remaining realistic given only 200 labeled examples. For a subjective task like classifying r/nba posts, a score above 90% would actually be suspicious, suggesting the model memorized the training data rather than learning the pattern.

---

## Milestone 3: AI Tool Plan

**Label stress-testing:** Used Claude to stress-test labels by reviewing borderline candidates one by one, applying taxonomy rules, and identifying cases where the boundary was genuinely ambiguous. This led to the Option A taxonomy loosening decision.

**Annotation assistance:** Originally planned to use Groq for pre-labeling. In practice, Claude was used to apply labels to all 238 raw rows based on the taxonomy rules. Every prediction was then reviewed manually and corrected where needed. Multiple rounds of corrections were applied as the taxonomy was refined. Claude did not make final labeling decisions — all labels were reviewed and approved by the annotator.

**Failure analysis:** After training, the 9 wrong predictions were analyzed with Claude's assistance to identify common patterns. Findings were then verified manually against the actual examples.

---

## Milestones 4–5: Baseline and Fine-Tuning Results

### Baseline (Groq llama-3.3-70b-versatile, zero-shot)

| Class | Precision | Recall | F1 |
|---|---|---|---|
| analysis | 0.91 | 0.83 | 0.87 |
| hot_take | 0.91 | 0.95 | 0.93 |
| **Overall accuracy** | | | **0.909** |

### Fine-Tuned Model (DistilBERT, 3 epochs)

Training args: 3 epochs, lr=2e-5, batch size 16, warmup 50 steps, best checkpoint saved by validation accuracy.

| Epoch | Train Loss | Val Loss | Val Accuracy |
|---|---|---|---|
| 1 | 0.707 | 0.696 | 0.455 |
| 2 | 0.690 | 0.672 | 0.758 |
| 3 | 0.651 | 0.632 | 0.697 |

Best checkpoint: Epoch 2 (loaded via `load_best_model_at_end=True`)

| Class | Precision | Recall | F1 |
|---|---|---|---|
| analysis | 1.00 | 0.25 | 0.40 |
| hot_take | 0.70 | 1.00 | 0.82 |
| **Overall accuracy** | | | **0.727** |

### Key Finding

The zero-shot baseline (91%) outperformed the fine-tuned DistilBERT (73%). The fine-tuned model defaulted to predicting HOT_TAKE for almost everything — analysis recall was 0.25, meaning it only caught 3 of 12 analysis examples in the test set. This is consistent with class imbalance in the training data (70/30 split) and the small dataset size (200 examples). DistilBERT learned surface-level cues rather than the semantic distinction between assertion and argument.

---

## Stretch Features

- [ ] Inter-annotator reliability (get a friend to label 30 examples)
- [ ] Deployed interface (simple Gradio app)
- [x] Error pattern analysis
