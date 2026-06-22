# TakeMeter — Planning Document

## Milestone 1: Community + Label Design

---

### Community Description

r/nba is Reddit's primary NBA community with millions of members who discuss all matters
regarding the NBA — players, coaches, stats, trades, and GOAT debates. The discourse
varies wildly in quality: some posts bring real evidence and structured arguments, others
are bold claims stated with total confidence and nothing behind them. The ANALYSIS vs
HOT_TAKE distinction matters here because it's genuinely hard to tell at times whether a
user is backing up their claim with stats and reasoning or just asserting an opinion as
fact.

---

### Label Taxonomy

#### Label 1: `ANALYSIS`

**Definition:** Makes a structured argument backed by statistics, historical comparison,
or tactical observation. Evidence is specific and verifiable. The post argues rather than
asserts.

**Clear example:**
> "They had 2 of the 5 20 point 4th quarter comebacks in playoff HISTORY this year
> including the greatest comeback in finals history with a 29 point comeback when the old
> record was 24. They also came back from 20+ in the 2nd half twice against the Celtics
> in 2025. They pulled off over half of the biggest 4th quarter comebacks in playoff
> history in a 2 year span."

Why this is ANALYSIS: specific stats, historical comparisons, verifiable numbers that
build toward a conclusion.

**Edge case:**
> Same post as above, but the conclusion is "best comeback team ever."

Why this is hard: the stats are real and verifiable, but the conclusion overshoots the
evidence with an absolute superlative. Decision: **label as HOT_TAKE** — if a post uses
real stats but concludes with an absolute superlative ("best ever," "greatest of all
time"), the conclusion is no longer proportional to the evidence. If they had said "one
of the best" it would remain ANALYSIS.

---

#### Label 2: `HOT_TAKE`

**Definition:** A bold, confident opinion stated without supporting evidence — often uses
extreme or absolute language. May be deliberately provocative or genuinely held, but the
post asserts rather than argues.

**Clear example:**
> "NBA is a rigged league. Tonight could be my last NBA game and I'd be totally fine.
> This game is no longer fun to watch once you know what's really going on."

Why this is HOT_TAKE: makes a broad unsupported claim ("rigged league"), no evidence
provided, asserts rather than argues.

**Edge case:**
> A post with real stats that concludes with "the best ever" or "not even close."

Why this is hard: the presence of stats makes it look like ANALYSIS, but the leap to an
absolute superlative is unsupported by the evidence. Decision: **label as HOT_TAKE** —
stats alone don't make something ANALYSIS if the conclusion overreaches the evidence.

---

### Labeling Rules

These rules apply when a post is ambiguous:

1. **The core test:** Does this post provide specific, verifiable evidence for its claim?
   Yes → ANALYSIS. No → HOT_TAKE.

2. **Stats + superlative rule:** If a post uses real stats but concludes with an absolute
   superlative ("best ever," "GOAT," "not even close," "nobody does it better") — label
   it HOT_TAKE. If the conclusion is proportional to the evidence ("one of the best,"
   "historically significant") — label it ANALYSIS.

3. **Assertion vs. argument rule:** If you can ask "what's the evidence?" and there's a
   real answer in the post — ANALYSIS. If the post just states the claim confidently with
   nothing behind it — HOT_TAKE.

4. **Emotion + claim rule:** When a post has both reactive emotion AND a broad unsupported
   claim, label based on the core assertion, not the emotion.

---

### Mutual Exclusivity Check

| Potential Overlap | Resolution |
|---|---|
| ANALYSIS vs HOT_TAKE | Ask: does the post provide specific, verifiable evidence that is proportional to its conclusion? If yes → ANALYSIS. If no, or if the conclusion overshoots the evidence with a superlative → HOT_TAKE. |

---

### Data Collection Plan

*(To be filled in — Milestone 2)*

- **Source:** r/nba via PRAW (Reddit API)
- **Target:** ~300 posts (buffer for filtering down to 200 labeled examples)
- **Planned split:** 140 train / 30 validation / 30 test (70/15/15)
- **Label distribution target:** ~100 examples per label (50/50 balance)
- **If a label is underrepresented:** pull additional posts from specific thread types
  (e.g. game threads for HOT_TAKE, stats/discussion threads for ANALYSIS) until balance
  is restored.

---

### Evaluation Metrics

Accuracy alone isn't enough because you need to see how the model handles each class
specifically and where it is getting confused. F1 score is the right metric because it
combines both precision and recall evenly — since both types of errors (missing a
HOT_TAKE or mislabeling an ANALYSIS) are equally costly for this task. The confusion
matrix shows exactly where the model is getting confused between the two labels.

**Metrics to report:**
- Overall accuracy (both fine-tuned model and Groq zero-shot baseline)
- F1 score per label (ANALYSIS and HOT_TAKE)
- Confusion matrix
- At least 3 specific wrong predictions with analysis of why

---

### Definition of Success

My definition of success is an F1 score of 80% or above. This is high enough to be
genuinely useful — wrong only 2 in 10 times — while remaining realistic given only 200
labeled examples. For a subjective task like classifying r/nba posts, a score above 90%
would actually be suspicious, suggesting the model memorized the training data rather
than learning the pattern.

---

### AI Tool Plan

**Label stress-testing:**
I will use an AI tool to stress-test my labels by generating posts that sit on the
boundary between ANALYSIS and HOT_TAKE. If I can't classify them cleanly, I will tighten
the label definitions and add a new labeling rule before annotating 200 examples.

**Annotation assistance:**
I will use Groq to pre-label my 200 posts by feeding each post along with my label
definitions and labeling rules, then I will review every prediction myself to catch
mistakes and fix edge cases. I will track which examples were pre-labeled by Groq for
disclosure in my AI usage section.

**Failure analysis:**
After training, I will feed my wrong predictions to an AI tool to identify patterns, and
then I will verify those patterns myself by double checking and seeing if I can agree with
the patterns that the AI tool noticed.

---

### Stretch Features Being Considered

- [ ] Inter-annotator reliability (get a friend to label 30 examples)
- [ ] Deployed interface (simple Gradio app)
- [ ] Error pattern analysis
