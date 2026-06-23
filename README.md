# TakeMeter — NBA Discourse Classifier

A fine-tuned text classifier that distinguishes **analytical** NBA Reddit comments from **hot takes** — trained on r/nba data using DistilBERT.

---

## What It Does

TakeMeter classifies r/nba comments into two categories:

- **ANALYSIS** — structured argument backed by specific, verifiable evidence (stats, historical comparisons, tactical observations) where the conclusion is proportional to the evidence
- **HOT_TAKE** — bold opinion stated without supporting evidence, or stats used to reach an absolute conclusion that overshoots the evidence ("best ever," "GOAT," "not even close")

The goal is to capture a real distinction in online sports discourse: the difference between a post that argues something and one that just asserts it.

---

## Community

**r/nba** — Reddit's primary NBA discussion community with millions of members. Chosen because the ANALYSIS vs HOT_TAKE distinction is both meaningful and genuinely difficult here. The community produces a natural mix of evidence-based arguments (film breakdowns, stat comparisons, historical context) and confident assertions (player rankings, legacy debates, game reactions), making it a rich and challenging classification target.

---

## Dataset

**Source:** 7 r/nba comment threads scraped manually  
**Threads used:**
1. KD "underdog" controversy
2. "Best player in the league" debate
3. Overrated/underrated player poll
4. Bam Adebayo 83-point game / "ethical basketball" debate
5. MJ vs LeBron GOAT debate
6. Pacers vs OKC film breakdown
7. NBA analytics ("bad players who look good under analytics")

**Final dataset:** 200 labeled examples  
**Split:** 70% train / 15% validation / 15% test (handled by Colab notebook)

| Label | Count | % |
|---|---|---|
| hot_take | 141 | 70.5% |
| analysis | 59 | 29.5% |
| **Total** | **200** | |

**Why not 50/50?** r/nba naturally skews toward hot takes. Analytical comments with specific verifiable evidence are a minority of the discourse. The 70/30 split reflects the real community distribution while staying within the project requirement of no label exceeding 70%.

---

## Label Design

### ANALYSIS
Makes a structured argument backed by statistics, historical comparison, or tactical observation. Evidence is specific and verifiable. The conclusion is proportional to the evidence.

### HOT_TAKE
A bold, confident opinion stated without supporting evidence. May use extreme or absolute language. May include stats, but if the conclusion overshoots the evidence with an absolute superlative ("best ever," "GOAT," "not even close"), it is HOT_TAKE regardless of the stats present.

### Key Labeling Rules

**Stats + superlative rule:** Stats alone don't make something ANALYSIS. If the conclusion is "best ever" or "not even close," it's HOT_TAKE even with real numbers behind it.

**Core test:** Can you ask "what's the evidence?" and find a real answer in the post? Yes → ANALYSIS. No → HOT_TAKE.

### Annotation Decisions

**3-label → 2-label collapse:** Originally designed with a REACTION label for short emotional responses. During labeling, the REACTION/HOT_TAKE boundary proved too blurry to apply consistently — both lack evidence and assert rather than argue. Collapsed REACTION into HOT_TAKE for a cleaner, more learnable boundary.

**Taxonomy loosening:** The strict taxonomy (requiring hard verifiable stats) produced only 22 ANALYSIS examples — an 89/11 split. Loosened to include posts making reasoned arguments with specific observable evidence even without raw numbers. Example: citing specific teams and historical outcomes ("59-win Bucks in Jordan's rookie year, Celtics dynasty of Bird-McHale-Parish") qualifies if the conclusion is proportional.

---

## Model

**Base model:** `distilbert-base-uncased`  
**Fine-tuning:** Google Colab T4 GPU, 3 epochs  
**Training args:** lr=2e-5, batch size 16, warmup 50 steps, best checkpoint by validation accuracy

| Epoch | Train Loss | Val Loss | Val Accuracy |
|---|---|---|---|
| 1 | 0.707 | 0.696 | 0.455 |
| 2 | 0.690 | 0.672 | **0.758** |
| 3 | 0.651 | 0.632 | 0.697 |

Best checkpoint: Epoch 2 (saved via `load_best_model_at_end=True`).

---

## Evaluation Results

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq llama-3.3-70b-versatile) | **0.909** |
| Fine-tuned DistilBERT | 0.727 |
| Difference | -0.182 |

### Per-Class Metrics — Zero-Shot Baseline

| Class | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 0.91 | 0.83 | 0.87 | 12 |
| hot_take | 0.91 | 0.95 | 0.93 | 21 |
| **macro avg** | **0.91** | **0.89** | **0.90** | 33 |

### Per-Class Metrics — Fine-Tuned DistilBERT

| Class | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| analysis | 1.00 | 0.25 | 0.40 | 12 |
| hot_take | 0.70 | 1.00 | 0.82 | 21 |
| **macro avg** | **0.85** | **0.62** | **0.61** | 33 |

### Confusion Matrix — Fine-Tuned Model

|  | Predicted: analysis | Predicted: hot_take |
|---|---|---|
| **True: analysis** | 3 | 9 |
| **True: hot_take** | 0 | 21 |

The model correctly identified 3 of 12 analysis examples and all 21 hot_take examples. It never falsely predicted analysis for a true hot_take, but missed 9 of 12 true analysis examples — sending them to hot_take instead.

See `confusion_matrix.png` for the visual version.

---

## Wrong Prediction Analysis

The fine-tuned model made 9 errors, all in the same direction: true ANALYSIS predicted as HOT_TAKE. Here are three analyzed in depth.

**Error #1**
> *"No one's ever had to do this by the way. Of all of the 70+ games in the recent past, no player has had to explain themselves as to why they scored that much instead of getting subbed out."*  
> True: analysis | Predicted: hot_take | Confidence: 0.52

This is ANALYSIS because it makes a specific historical claim (no player in any 70+ point game has ever been asked to justify their scoring). The model likely flagged it as HOT_TAKE because the phrasing is assertive and
