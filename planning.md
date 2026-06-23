# TakeMeter — Planning Document

## Community Choice

**Community:** r/soccer (reddit.com/r/soccer) — 8.7 million members, the largest general football/soccer forum on the internet.

**Why this community:** r/soccer is one of the most text-heavy sports subreddits on Reddit, and the 2026 FIFA World Cup (currently underway across the US, Canada, and Mexico) has pushed daily activity to its annual peak. What makes r/soccer especially well-suited for a classification task is that the community itself constantly argues about what counts as a good contribution. Match threads and post-match threads generate thousands of comments per game that range from single-word reactions to multi-paragraph tactical breakdowns with statistics. The regulars have a sharp intuition for the difference between someone who actually watched the match and analyzed what happened versus someone firing off a hot take for attention, and they are vocal about it — the phrase "least reactionary r/soccer user" is a running joke. Banter and humor are a massive part of the culture (inherited from football fan culture more broadly), but the community also produces genuinely insightful analysis involving expected-goals data, pressing maps, and formation discussions. These three modes — analysis, hot takes, and banter — coexist in every thread, are recognizable to anyone who spends time there, and are distinct enough to classify.

---

## Label Definitions

### Label 1: `analysis`

**Definition:** A comment that makes a claim about a team, player, match, or tactic and supports it with specific reasoning, evidence, or contextual knowledge — the reader learns *why* the author holds their position, not just *what* it is.

**Example A:**
> "The US looked so much better once they switched to a 4-2-3-1 in the second half against Australia. Musah dropping deeper freed up Pulisic to drift inside, and you could see the Australians struggling to track him between the lines. That's been the pattern all tournament — they're more dangerous when Pulisic doesn't have defensive responsibilities."

**Example B:**
> "Brazil's biggest problem against Morocco was that Vinícius kept drifting central and clogging the same space as Rodrygo. Neither of them was making runs in behind, so Morocco's back line could sit deep and stay compact. They need someone stretching the defense vertically or it's just going to be sideways passing until they concede on the counter."

**Uncertain case:**
> "Messi breaking the all-time World Cup scoring record is insane when you think about everything he's been through. Five World Cups, all those heartbreaks, and he's still delivering at 39."

This is tricky — it references specific factual context (five World Cups, his age, the record) and contains a framing argument about longevity. But it doesn't analyze *how* or *why* he's performing, and the emotional framing ("insane," "heartbreaks") pushes it toward a reaction. **Decision rule:** if the comment's core contribution is explaining a mechanism (tactical, statistical, historical causation), it's analysis. If the core contribution is expressing magnitude of feeling with some facts sprinkled in, it's a hot take. This one goes `hot_take`.

---

### Label 2: `hot_take`

**Definition:** A comment that states an opinion, prediction, or evaluative claim about the sport without providing reasoning or evidence to back it up — the reader knows what the author thinks but not why they think it.

**Example A:**
> "England are winning this whole thing. Tuchel has this team playing like a machine. Nobody is stopping them."

**Example B:**
> "Ronaldo should have retired after 2022. He's actively hurting Portugal's chances every time he starts."

**Uncertain case:**
> "Yamal is already better than Neymar ever was at a World Cup. Fight me."

This sits between `hot_take` and `banter` — "Fight me" is a humor/provocation signal typical of banter, but the core content is a bold evaluative claim (comparative player ranking). **Decision rule:** if the comment's primary purpose is advancing an opinion the author appears to actually hold, it's `hot_take`, even if it's delivered with attitude or humor. If the primary purpose is getting a laugh or provoking a reaction and the claim is clearly absurd or ironic, it's `banter`. This one goes `hot_take` — the author probably means it.

---

### Label 3: `banter`

**Definition:** A comment whose primary purpose is humor, mockery, trash talk, wordplay, or communal in-jokes — it may reference football but its goal is entertainment or social bonding rather than advancing an argument.

**Example A:**
> "Germany fan cycling 26,000km to get to Houston only to watch them concede from a set piece in the 89th minute. Peak German efficiency in finding new ways to suffer."

**Example B:**
> "VAR checking if that was offside like me checking if I can afford groceries this week."

**Uncertain case:**
> "Imagine telling someone in 2018 that Mbappé would have 100 caps, a Ballon d'Or, and still somehow be the most frustrating player to watch on any given night."

This mixes factual information (100 caps, career achievements) with an ironic observation. It could be classified as a `hot_take` (arguing Mbappé is frustrating to watch) or `banter` (the "imagine telling someone" setup is a classic joke structure). **Decision rule:** look at the structure. If the comment is built around a humor device (irony, absurdity, callback, setup/punchline), it's `banter` even if the underlying opinion is real. If it's a straightforward claim delivered with some attitude, it's `hot_take`. This one's structure is comedic — it goes `banter`.

---

## Mutual Exclusivity Check

The three labels are organized around *function*, not topic. Any football subject (a player, a match, a tactic) can appear in all three categories. What distinguishes them:

| | Has a specific claim? | Supports it with reasoning? | Primary goal is humor? |
|---|---|---|---|
| `analysis` | Yes | Yes | No |
| `hot_take` | Yes | No | No |
| `banter` | Maybe | No | Yes |

The main overlap risk is between `hot_take` and `banter`, since both lack supporting reasoning. The tiebreaker is intent: if the comment is trying to be funny first and opinionated second, it's `banter`. If it's trying to stake out a position (even an inflammatory one), it's `hot_take`. In practice this is clear ~85% of the time. The few ambiguous cases will be decided by asking: "If you removed the joke, would there still be a claim the author wants you to take seriously?" If yes, `hot_take`. If the joke *is* the whole point, `banter`.

---

## Hard Edge Cases

The genuinely difficult boundary is **hot_take vs. analysis**. Some comments make a strong claim and gesture at a reason without fully developing it:

> "Argentina look beatable. Their midfield can't control games the way it did in 2022."

This has a claim ("beatable") and a reason ("midfield can't control games"), but the reason is vague — it doesn't specify *what* about the midfield is different or *why* it matters. A strict reading puts this in `hot_take` (no specific evidence). A generous reading puts it in `analysis` (there's at least an attempt at reasoning).

**Handling rule for annotation:** I will apply a "could a skeptic learn something?" test. If the comment provides enough specificity that someone who disagrees could engage with the *reasoning* (not just the conclusion), it's `analysis`. If all a skeptic could say is "no you're wrong," it's `hot_take`. The Argentina example above goes `hot_take` — a skeptic can't meaningfully engage with "can't control games" without more detail.

Another hard edge: **banter that contains real tactical observation.** Football humor often works *because* the observation is accurate. I'll label based on the primary communicative function, not whether the content happens to be true.

---

## Data Collection Plan

**Source:** Comments from r/soccer post-match threads, pre-match threads, and transfer/news discussion threads during the 2026 World Cup group stage (June 11–24, 2026). Post-match threads are ideal because they generate the full spectrum of comment types in a single location.

**Method:** I will use Reddit's public interface (old.reddit.com for easier bulk reading) to manually collect comments. I will sample from at least 8–10 different match threads across different groups and teams to avoid over-representing any single fanbase or match narrative. I will collect top-level comments and high-engagement replies, excluding comments that are purely procedural (match stats bots, lineup posts) or under 10 words (too short to classify meaningfully).

**Target distribution:**
- `analysis`: ~70 examples (35%)
- `hot_take`: ~70 examples (35%)
- `banter`: ~60 examples (30%)

**Total target:** 210–220 examples

**If a label is underrepresented:** `analysis` is the most likely to be underrepresented because substantive comments are less common than reactions or jokes. If I'm short on analysis examples after scraping post-match threads, I will specifically seek out the "Serious Discussion" and "Post-Match Analysis" threads that r/soccer moderators sometimes pin, as well as the "Daily Discussion" threads where longer-form takes tend to appear. I will also sample from pre-tournament tactical preview threads (e.g., "How will X team line up?") which skew heavily toward analysis.

**If a label is overrepresented:** `banter` will likely be overrepresented since jokes dominate upvoted content. I will cap banter collection once I reach ~70 examples and redirect collection effort toward the other two labels.

---

## Evaluation Metrics

**Primary metric: Macro-averaged F1 score.** This is the right primary metric because my three labels are roughly balanced but not perfectly so, and I care equally about the model's ability to identify each category. Accuracy alone would be misleading — a model that's excellent at `banter` (the most structurally distinct category) but terrible at the harder `analysis`/`hot_take` boundary could still report 65%+ accuracy while being useless at the distinction that actually matters.

**Secondary metrics:**
- **Per-class precision and recall for each label.** The analysis/hot_take boundary is where I expect the most errors, and I need to know the direction of failure. If the model has high recall for `analysis` but low precision, it's promoting hot takes to analysis status — that's worse for a community tool than the reverse, because users would lose trust in "analysis" as a filter. Per-class metrics let me diagnose this.
- **Confusion matrix.** I need to see the full error pattern: are misclassifications concentrated on one boundary (analysis↔hot_take) or spread across all pairs? This directly informs whether my label definitions need tightening or whether the model just needs more data.
- **Overall accuracy** will be reported as a simple summary statistic for comparison against the zero-shot Groq baseline, since both models see the same test set.

**Why not just accuracy:** With three roughly balanced classes, random-chance accuracy is ~33%. A model hitting 55% accuracy could be doing well on all three labels or could be near-perfect on banter and near-random on the other two. Macro F1 forces the model to perform on every class to get a good score.

---

## Definition of Success

**Genuinely useful (stretch goal):** Macro F1 ≥ 0.72, with per-class F1 ≥ 0.65 for every label including the hardest one (likely `hot_take`). At this level, the classifier is correct often enough that a community tool could reliably surface analysis-tagged comments or filter hot takes, and users would trust the labels most of the time.

**Good enough (realistic target):** Macro F1 ≥ 0.62, with no single class F1 below 0.50. At this level the model is clearly better than random, meaningfully outperforms the zero-shot baseline, and the error patterns are interpretable (concentrated on the known-hard boundary, not random). A community tool would need a confidence threshold but could still be useful.

**Minimum acceptable:** The fine-tuned model must outperform the zero-shot Groq baseline on macro F1 by at least 5 percentage points. If fine-tuning on 200 domain-specific examples can't beat a general-purpose LLM prompted with the same label definitions, something is wrong with either the labels or the data.

**Objective check:** At the end of the project, I will compare the fine-tuned model's macro F1, per-class F1, and the fine-tuned-vs-baseline delta against these three thresholds. The thresholds are numeric, so the assessment is unambiguous.

---

## AI Tool Plan

### 1. Label Stress-Testing

**Before annotation begins**, I will give Claude (or another LLM) my three label definitions, the decision rules for edge cases, and the "could a skeptic learn something?" test. I will prompt it to generate 8–10 synthetic r/soccer comments that deliberately sit on each of the two major boundaries:

- **analysis ↔ hot_take boundary:** Comments that make a claim with vague or minimal reasoning (e.g., "Their pressing was terrible" — is one word of tactical vocabulary enough to count as analysis?).
- **hot_take ↔ banter boundary:** Comments that express a real opinion through a joke structure (e.g., an ironic setup that contains a genuine claim about a player).

For each generated comment, I will attempt to classify it using my own definitions. If I can't do it cleanly, the definition needs tightening *before* I start annotating 200 examples — fixing a definitional gap after 150 annotations is much more expensive than fixing it after 10 synthetic tests. I expect to iterate on the "could a skeptic learn something?" criterion at this stage, possibly adding a specificity requirement (e.g., "names at least one concrete observation about positioning, a statistic, or a causal mechanism").

### 2. Annotation Assistance

**I will use LLM pre-labeling selectively.** After I have manually annotated the first 50 examples myself (to lock in my own calibration), I will use Claude to pre-label the remaining ~160 comments in batches of 20. For each batch, I will provide the label definitions, 3 examples per label from my existing annotations, and the batch of unlabeled comments. Claude will output a predicted label for each.

**I will review every pre-labeled example myself** and override any label I disagree with. I will track pre-labeling in my CSV with a boolean column `ai_prelabeled` (TRUE if the initial label suggestion came from an LLM, FALSE if I labeled it from scratch). This lets me report in the final writeup what percentage of my labels were AI-suggested and what percentage I overrode, as evidence that the labels reflect my judgment, not the model's.

**Why this approach:** Pre-labeling is useful because the bottleneck in annotation is the *initial decision*, not the review. Seeing a suggested label and asking "do I agree?" is faster than classifying from a blank slate, especially for the clear cases (obvious banter, obvious analysis). For ambiguous cases, the AI suggestion is less useful — I'll need to apply my decision rules either way.

### 3. Failure Analysis

After training and evaluation, I will take the full list of wrong predictions from the test set (printed in Section 4 of the notebook) and give them to Claude with the following prompt structure:

- Here are my three label definitions and decision rules.
- Here are N misclassified examples, showing: the text, the true label, the predicted label, and the model's confidence score.
- Identify patterns: Are there structural or linguistic features that the model consistently gets wrong? Are the errors concentrated on one label boundary? Do the errors correlate with comment length, the presence of football jargon, or the use of humor/irony?

**What I'll look for specifically:**
- Whether analysis→hot_take errors involve short comments (suggesting the model equates length with substance).
- Whether hot_take→banter errors involve sarcasm or rhetorical questions (suggesting the model reads any informal tone as humor).
- Whether banter→hot_take errors involve banter that contains a real opinion (the hardest edge case by design).

**Verification:** I will read every misclassified example myself before accepting any pattern the AI identifies. If Claude claims "the model struggles with sarcasm," I will check whether the examples it cites are actually sarcastic or whether the label was just wrong. I will also check whether the pattern holds for correctly-classified examples (e.g., are there sarcastic comments the model *did* get right? If so, what's different about them?). The AI identifies candidate patterns; I verify them against the full data.
