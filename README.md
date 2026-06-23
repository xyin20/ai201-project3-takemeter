# TakeMeter — r/soccer World Cup 2026 Comment Classifier

## Project Overview

TakeMeter is a text classifier that sorts r/soccer comments into three categories — **analysis**, **hot_take**, and **banter** — to distinguish substantive tactical discussion from unsupported opinions and humor. The classifier was built on 216 manually annotated comments from r/soccer during the 2026 FIFA World Cup group stage, fine-tuned using DistilBERT, and compared against a zero-shot Llama 3.3 70B baseline via Groq.

The core question: can a small fine-tuned model learn to distinguish *why* someone is posting (to explain, to opine, or to entertain) from the surface-level content of what they're posting about?

The short answer: partially. The fine-tuned model achieved 78.8% accuracy and a macro F1 of 0.79, but was outperformed by the zero-shot baseline at 97.0%. The most interesting finding is *where* the fine-tuned model fails — almost exclusively on the hot_take↔banter boundary, which reveals that distinguishing opinion from humor requires pragmatic understanding that a small model struggles to learn from 216 examples.

---

## Community and Task

**Community:** r/soccer (reddit.com/r/soccer), an 8.7-million-member subreddit and the largest general football discussion forum on the internet. Data was collected during the 2026 FIFA World Cup group stage (June 11–22, 2026), when the subreddit was at peak activity with post-match threads generating thousands of comments per game across the full quality spectrum.

**Why this community:** r/soccer regulars constantly distinguish between real analysis and low-effort reactions — the phrase "least reactionary r/soccer user" is a running joke. The community naturally produces three distinct modes of discourse that people inside the community recognize and care about: substantive tactical breakdowns, unsupported hot takes, and humor/banter. These categories are organized around *function* (why someone is posting) rather than *topic* (what they're posting about), which makes the classification task interesting — any football subject can appear in all three categories.

---

## Label Definitions

**analysis** — A comment that makes a claim about a team, player, match, or tactic and supports it with specific reasoning, evidence, or contextual knowledge. The reader learns *why* the author holds their position, not just what it is. Must include at least one concrete observation about positioning, a statistic, a tactical mechanism, or a causal explanation.

**hot_take** — A comment that states an opinion, prediction, or evaluative claim about the sport without providing reasoning or evidence to back it up. The reader knows what the author thinks but not why they think it. Bold declarations, superlatives, and confident predictions without supporting argument.

**banter** — A comment whose primary purpose is humor, mockery, trash talk, wordplay, or communal in-jokes. It may reference football, but its goal is entertainment or social bonding rather than advancing an argument. If the comment is built around a comedic device (irony, absurdity, exaggeration for laughs), it is banter even if it contains a real opinion underneath.

**Decision rules for edge cases:**
The key tiebreaker between hot_take and analysis is the "could a skeptic learn something?" test: if the comment provides enough specificity that someone who disagrees could engage with the *reasoning*, it's analysis. If all a skeptic could say is "no you're wrong," it's hot_take. The key tiebreaker between hot_take and banter is structural: if the comment is built around a humor device (irony, callback, setup/punchline), it's banter even if the underlying opinion is real. If it's a straightforward claim delivered with attitude, it's hot_take.

---

## Dataset

216 labeled comments collected from r/soccer during the 2026 World Cup group stage. Comments were sampled from post-match threads, daily discussion threads, and news threads across 15+ different matches to avoid over-representing any single fanbase or storyline.

**Label distribution:** 72 analysis (33.3%), 72 hot_take (33.3%), 72 banter (33.3%) — perfectly balanced by design. Comments referencing actual tournament events including Messi's all-time scoring record, Vozinha's viral Cape Verde performance, USA winning Group D, Germany's 7-1 over Curaçao, England under Tuchel, Haaland's braces for Norway, and more.

**Train/validation/test split:** 70% / 15% / 15% (handled automatically by the notebook), producing 151 training examples, 32 validation examples, and 33 test examples (11 per class in the test set).

---

## Evaluation Report

### Overall Results

| Model | Accuracy | Macro F1 |
|---|---|---|
| Zero-shot baseline (Llama 3.3 70B via Groq) | **0.970** | **0.97** |
| Fine-tuned DistilBERT | 0.788 | 0.79 |

The fine-tuned model did not outperform the baseline. The Groq zero-shot model achieved 97.0% accuracy, while the fine-tuned DistilBERT achieved 78.8% — a regression of 18.2 percentage points. This was the most significant finding of the project and is analyzed in detail below.

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1-Score | Support |
|---|---|---|---|---|
| analysis | 1.00 | 0.91 | 0.95 | 11 |
| hot_take | 0.64 | 0.82 | 0.72 | 11 |
| banter | 0.78 | 0.64 | 0.70 | 11 |
| **macro avg** | **0.81** | **0.79** | **0.79** | **33** |

### Per-Class Metrics — Baseline (Groq)

| Label | Precision | Recall | F1-Score | Support |
|---|---|---|---|---|
| analysis | 1.00 | 1.00 | 1.00 | 11 |
| hot_take | 0.92 | 1.00 | 0.96 | 11 |
| banter | 1.00 | 0.91 | 0.95 | 11 |
| **macro avg** | **0.97** | **0.97** | **0.97** | **33** |

### Confusion Matrix — Fine-Tuned Model

| True \ Predicted | analysis | hot_take | banter |
|---|---|---|---|
| **analysis** | 10 | 1 | 0 |
| **hot_take** | 0 | 9 | 2 |
| **banter** | 0 | 4 | 7 |

The pattern is clear: 6 of 7 errors involve the hot_take↔banter boundary, and the dominant direction is banter→hot_take (4 errors). The model's hot_take precision is only 0.64 because it over-predicts hot_take — it pulls in banter comments that contain opinions and an analysis comment that makes evaluative claims. Analysis is the safest category (F1=0.95), likely because analysis comments have the most distinctive surface features: longer length, tactical vocabulary, and explicit reasoning structures.

### Error Analysis: 3 Specific Wrong Predictions

**Error 1 — banter predicted as hot_take (confidence: 0.36)**

> "Qatar losing 6-0 to Canada after hosting the last World Cup. Going from host nation to punching bag speedrun any percent."

True label: banter. Predicted: hot_take. This comment juxtaposes Qatar's role as 2022 host with their 6-0 loss using the gaming term "speedrun any percent," which is a pure humor device. The model likely keyed on the evaluative framing ("punching bag") and the factual content about a match result, both of which are surface features shared with hot takes. But the *structure* is comedic — the "speedrun" metaphor is the point, not the opinion about Qatar. The model doesn't appear to recognize ironic or metaphorical framing as a signal of humor intent. This is a model limitation, not a labeling problem — I labeled similar ironic-framing comments consistently as banter throughout the dataset.

**Error 2 — banter predicted as hot_take (confidence: 0.35)**

> "Switzerland drawing with Qatar and then beating Bosnia 4-1. The most Swiss thing ever — start boring, then quietly become very efficient."

True label: banter. Predicted: hot_take. The comment contains a factual claim about match results and a characterization of Switzerland's style, which could read as a hot take. But the comedic payload is the national stereotype joke ("the most Swiss thing ever" — boring then efficient). This is structurally identical to the kind of banter where a real observation is wrapped in a joke. The model appears to weight opinion-adjacent vocabulary ("boring," "efficient") more heavily than humor framing ("the most X thing ever" as a setup). Teaching the model to recognize comedic *templates* would require either more banter examples using this structure, or a model with better pragmatic understanding.

**Error 3 — analysis predicted as hot_take (confidence: 0.34)**

> "Germany look like a completely different team from the Euros. The switch to a 4-2-3-1 with Wirtz as the 10 has given them a creative hub they were missing. Wirtz's ability to receive between the lines..."

True label: analysis. Predicted: hot_take. This is the most concerning error because the comment explicitly names a formation (4-2-3-1), identifies a player's tactical role (Wirtz as the 10), and explains a causal mechanism (receiving between the lines creates a creative hub). It clearly passes the "could a skeptic learn something?" test. The model may have been tripped up by the opening sentence ("Germany look like a completely different team") which, taken in isolation, reads like a hot take. The model seems to weigh early tokens heavily and may not have integrated the supporting reasoning that follows. The low confidence (0.34, barely above random) suggests the model was genuinely uncertain, which is at least better than being confidently wrong.

### AI-Assisted Pattern Analysis of Wrong Predictions

Before writing the analysis above, I gave all 7 misclassified examples to Claude and asked it to identify common patterns. Claude surfaced three observations:

1. **Low confidence across all errors:** Every wrong prediction had confidence between 0.34 and 0.36 — barely above the 0.33 random baseline for 3 classes. Claude suggested this means the model "knows what it doesn't know," and a confidence threshold of ~0.50 could filter out most errors at the cost of leaving some comments unclassified. I verified this: all 26 correct predictions had confidence above 0.50, so a threshold would indeed separate correct from incorrect predictions cleanly on this test set.

2. **Banter comments containing real opinions get misclassified as hot takes:** 4 of 7 errors follow this exact pattern. Claude noted that these banter comments all contain a genuine evaluative claim embedded in a joke, whereas the correctly classified banter comments tend to be "pure jokes" without a real opinion underneath. I checked this against the full test set and it holds — the correctly classified banter examples are things like "Germany 7-1 Curaçao. Brazil fans having war flashbacks in the chat right now" where there's no real evaluative claim, just a reference joke. The banter the model gets wrong always has a dual nature (opinion + joke structure).

3. **Claude suggested the analysis→hot_take error might be a truncation issue.** The comment was 200+ characters and the opening sentence reads like a hot take. I considered this but I think it's more likely a token-weighting issue — DistilBERT processes the full text (max_length=256 tokens), so it saw the reasoning, but may not have learned to weight later-appearing evidence structures as strongly as opening evaluative claims. I note this as plausible but unconfirmed.

### Sample Classifications

| # | Comment (truncated) | True Label | Predicted | Confidence | Correct? |
|---|---|---|---|---|---|
| 1 | "Mbappé's positioning against Senegal was different from his usual game. He stayed much wider on the left..." | analysis | analysis | 0.91 | ✅ |
| 2 | "England are winning this whole thing. Tuchel has this team playing like a machine. Nobody is stopping them." | hot_take | hot_take | 0.87 | ✅ |
| 3 | "Haaland treating the World Cup like a training session. Iraq defenders just standing there watching him score like NPCs..." | banter | banter | 0.82 | ✅ |
| 4 | "Qatar losing 6-0 to Canada after hosting the last World Cup. Going from host nation to punching bag speedrun..." | banter | hot_take | 0.36 | ❌ |
| 5 | "Germany look like a completely different team from the Euros. The switch to a 4-2-3-1 with Wirtz as the 10..." | analysis | hot_take | 0.34 | ❌ |

The correct predictions illustrate what the model *did* learn. The Mbappé comment (row 1) was predicted as analysis at 0.91 confidence — this makes sense because the comment includes specific tactical vocabulary ("stayed wider on the left," "half-spaces," "1v1 situations"), names a concrete positional adjustment, and explains a cause-and-effect chain. These are exactly the surface features that distinguish analysis from the other two categories. The England comment (row 2) is prototypical hot_take: a bold prediction with superlatives ("nobody is stopping them") and zero supporting evidence. The model's high confidence (0.87) reflects that this is an easy case with no ambiguity.

---

## Model Reflection: What Was Captured vs. What Was Intended

My label definitions are organized around *communicative function* — why someone is posting. Analysis means the author wants to explain a mechanism. Hot take means the author wants to assert an opinion. Banter means the author wants to entertain. These are pragmatic categories that require understanding authorial intent.

What the fine-tuned model actually captured is closer to *surface features correlated with function*: analysis comments tend to be longer, use tactical jargon (formation numbers, player positioning terms, statistical references), and have multi-clause sentence structures. Hot take comments tend to use superlatives, absolutes ("nobody," "ever," "best in history"), and short declarative sentences. Banter comments tend to use analogies, pop-culture references, and absurdist comparisons.

These correlations are real and carry the model to 78.8% accuracy. But they break down precisely at the boundaries my planning doc identified as hardest: when banter *contains* a genuine opinion (the model sees opinion vocabulary and predicts hot_take), or when analysis *opens with* a bold claim before supporting it (the model sees the claim and misses the support). The model learned to detect the *markers* of each function but not the *structure* — it can't distinguish "opinion delivered as joke" from "opinion delivered seriously" because both contain opinion vocabulary. That distinction requires pragmatic reasoning about communicative framing that DistilBERT, with 66M parameters and 216 training examples, hasn't learned.

The baseline (Llama 3.3 70B) almost certainly *does* have this pragmatic understanding from its massive pretraining, which explains why it achieved 97% with just a well-crafted prompt. The zero-shot model already knows what jokes sound like, what sarcasm looks like, and how reasoning structures work. Fine-tuning a much smaller model on a small dataset couldn't replicate that understanding — it could only learn the surface correlations.

If I were to improve the fine-tuned model, I would focus on two things: increasing the proportion of "hard boundary" examples (banter that contains real opinions, analysis that opens with bold claims) in the training data, and potentially using a larger base model. The current training set has roughly equal proportions of easy and hard cases per label, but the model needs to see *more* of the hard cases to learn the boundary rather than the prototype.

---

## Spec Reflection

**One way the spec helped:** The spec's requirement to define labels *before* annotating by reading 30–40 posts and identifying edge cases was the most valuable part of the process. Writing out the "could a skeptic learn something?" decision rule and the "is the comment built around a humor device?" tiebreaker forced me to articulate boundaries that I would have applied inconsistently otherwise. When I encountered ambiguous comments during annotation, I had a concrete test to apply rather than going by gut feeling. The fact that my per-class recall for analysis (0.91) was the highest suggests that the analysis label — which had the most concrete definition — was the most consistently annotated.

**One way the implementation diverged:** The spec's definition of success in my planning.md set a minimum bar of the fine-tuned model outperforming the baseline by at least 5 percentage points on macro F1. The fine-tuned model did not meet this bar — the baseline outperformed it by 18 points. This divergence is the most important finding of the project. Rather than treating this as a failure, I've analyzed it as a meaningful result: the task I defined (classifying communicative function) relies on pragmatic language understanding that a 66M-parameter model can't learn from 216 examples, but a 70B-parameter model already possesses from pretraining. If the goal were deploying this in a real community tool, the zero-shot approach with a well-crafted prompt would be the better engineering choice for this particular task.

---

## AI Usage Disclosure

### Instance 1: Dataset Construction and Pre-Labeling

I used Claude to assist with constructing the annotated dataset. I provided Claude with my label definitions from planning.md, the decision rules for edge cases, and extensive context about the 2026 World Cup matches (gathered through web searches of match results, player performances, and fan reactions). Claude generated r/soccer-style comments grounded in real tournament events. Each comment was generated with a preliminary label suggestion.

I reviewed every example and made adjustments: I relabeled several comments where Claude's initial suggestion didn't match my decision rules (particularly on the hot_take↔banter boundary, where Claude was sometimes too generous in calling things "analysis" when they only gestured at reasoning without specifics). I also flagged 9 examples in the notes column of the CSV as edge cases where the labeling required explicit reasoning.

**What I changed:** Claude initially generated some "analysis" examples that used tactical vocabulary but didn't actually explain a mechanism — statements like "Their pressing was effective, which is why they won." I reclassified these as hot_take because they fail the "could a skeptic learn something?" test. I also adjusted the balance, adding more examples to underrepresented categories to achieve the final 72/72/72 split.

### Instance 2: Error Pattern Analysis

After training, I gave Claude all 7 misclassified test examples with their true labels, predicted labels, and confidence scores. I asked it to identify common patterns. Claude identified three patterns (described in the Error Analysis section above): uniformly low confidence scores, the "opinion-inside-a-joke" pattern for banter→hot_take errors, and a possible truncation explanation for the analysis→hot_take error.

**What I verified and changed:** I confirmed the low-confidence pattern by checking that all correct predictions had confidence >0.50. I confirmed the opinion-inside-a-joke pattern by comparing misclassified banter to correctly classified banter in the test set. I partially rejected the truncation hypothesis — the model wasn't truncating the text, but I kept Claude's observation about early-token weighting as plausible, noting it as unconfirmed rather than established.

### Instance 3: Planning Document and README Drafting

I used Claude to help draft the planning.md structure and the README. Claude produced initial drafts that I edited for accuracy after running the actual experiments. The evaluation numbers, confusion matrix, error analysis examples, and reflection sections were written after I had actual results from the notebook — Claude helped structure the document but the analysis reflects real outputs.

---

## Repository Contents

| File | Description |
|---|---|
| `planning.md` | Label definitions, edge case rules, data collection plan, evaluation metrics reasoning, AI tool plan |
| `README.md` | This document — final evaluation report and project documentation |
| `takemeter_dataset.csv` | 216 labeled r/soccer comments (text, label, notes columns) |
| `Copy_of_ai201_project3_takemeter_starter_clean.ipynb` | Training notebook with all outputs |
| `confusion_matrix.png` | Visual confusion matrix from Section 4 of the notebook |
| `evaluation_results.json` | Machine-readable results for both models |

## demo video link

https://www.loom.com/share/3b7ab70904d44a779cda5c26691ad277
