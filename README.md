# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

For this project, I chose the public community r/soccer. After reading around
30–40 posts, I noticed three main distinctions in the content: match clips and
highlights, quotes or media/interview posts, and discussion or emotional posts
about players, teams, and events. Match-related clips appeared to make up the
largest portion of the posts I saw.



---

## Label Taxonomy

### `match_clip`

**Definition:** A post primarily sharing footage or an alternate angle of a specific match event, such as a goal, skill, foul, save, or celebration.

**Example 1:**
>Alternative angle of Jota's fake

**Example 2:**
>Fan angle of Yamal’s Golazo against Real Betis

### `news`

**Definition:** A post whose main purpose is to report a football-related event, development, announcement, discovery, or other piece of news.

**Example 1:** 
> A journalist uncovered three reports Barça scouts wrote about 13-year-old Messi. The first praised his technical ability, the second doubted his height would allow him to compete at the highest level, and the third compared him to Maradona and recommended signing him immediately.

**Example 2:** 
> Barcelona exceed €1 billion in revenue for 2025/26 – first time in history


### `stats_or_fact`

**Definition:** A post primarily presenting quantitative analysis, statistics, rankings, records, comparisons, or historical performance data about football.

**Example 1:** 
> Manchester United are winless in their last 6 games

**Example 2:** 
> [ThePopFoot] top 10 scorer in a season in the 21st century For Club & Country


### `quote_interview`

**Definition:** A post whose main purpose is to share what a person or organization said, including a direct quote, interview response, speech, or public statement, even if the statement itself contains an opinion.

**Example 1:** 
> [Oliver Holt] Strip Man City of ALL eight of their Premier League titles and relegate them

**Example 2:**
> “They’ve stolen memories from everyone else!” Steven Gerrard speaks about the memories he feels were taken away by Manchester City.


### The hardest boundary

**Which two labels:** `news` and `quote_interview`

**The decision rule I used every time:**

If the main purpose of the post is to report an event, development, announcement, or piece of journalism, I classify it as `news`.

If the main purpose of the post is to share what a person or organization said, such as a direct quote, interview response, speech, or statement, I classify it as `quote_interview`, even if the statement itself is opinionated.

If a post contains both news and a quote, I classify it based on what the title mainly emphasizes.



---

## The Dataset

**Where the posts came from:**

The posts came from the public r/soccer subreddit. I collected post titles and text from public posts covering match clips, football news, statistics and records, and quotes or interviews. I did not include Reddit flair labels in the text because those could reveal the category to the model and create an artificial shortcut.

**How I labelled them:**

I manually labelled the first 20 posts without AI assistance and marked them as `cold`. For the remaining posts, I used AI to pre-label batches using my four-label taxonomy. I then read every post myself, checked the suggested label against my definitions, and corrected or removed examples when I disagreed or thought the post was too ambiguous. These rows are marked `AI pre-labeled, verified`.

**Counts per label:**

| Label | Count | Share |
|---|---:|---:|
| `match_clip` | 64 | 32.0% |
| `news` | 48 | 24.0% |
| `quote_interview` | 48 | 24.0% |
| `stats_or_fact` | 40 | 20.0% |
| **Total** | **200** | **100%** |

**Three hard cases**

**1.**
> *The post:* Peter Schmeichel says Messi's goal shouldn't have stood because of a foul in the build up of the goal
>
> *Could have been:* `news` or `quote_interview`
>
> *I chose, because:* I chose `quote_interview` because the main purpose of the post is to communicate Schmeichel's opinion. Even though the title uses reported speech instead of a direct quotation, the important information is still what Schmeichel said.

**2.**
> *The post:* FIFA World Cup 2026 Goal of the tournament nominees
>
> *Could have been:* `stats_or_fact` or `news`
>
> *I chose, because:* I chose `news` because the post is announcing a new list of nominees rather than presenting statistics, rankings, records, or quantitative analysis.

**3.**
> *The post:* [Sami Mokbel] Enzo Maresca calls on Manchester City players to maintain focus and positivity in address to the squad upon their return to training following Premier League charges verdict.
>
> *Could have been:* `news` or `quote_interview`
>
> *I chose, because:* I chose `news` because the title reports what Maresca did and what happened, but does not directly present his words as a quote or interview response.

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate `2e-5`, batch size `16`,
maximum sequence length `128`, and seed `42`.

**Anything I changed from the defaults, and why:** I kept the default
training settings so that my first run used the provided baseline
configuration.

**Split sizes:** 139 training, 31 validation, and 30 test examples.

The test set contained 10 `match_clip`, 7 `news`, 6 `stats_or_fact`,
and 7 `quote_interview` examples. Because three labels had fewer than
8 examples in the test set, their individual F1 scores may vary
noticeably across different random seeds.

---

## How I Used AI

**Moment 1**

- *What I asked for:* I asked AI to help me stress-test my initial label definitions and identify places where the categories could overlap.
- *What came back:* It pointed out possible overlap between categories such as `news`, `quote_interview`, and `stats_or_fact`, and gave examples of posts that could reasonably fit more than one label.
- *What I changed:* I refined the taxonomy so that labels are based mainly on the purpose of the post. I narrowed `stats_or_fact` to statistics, records, rankings, comparisons, and quantitative information, and I created a specific decision rule for distinguishing `news` from `quote_interview`.

**Moment 2**

- *What I asked for:* After manually labelling the first 20 posts, I asked AI to collect public r/soccer posts and suggest labels for them using my taxonomy.
- *What came back:* AI returned batches of posts with suggested labels such as `match_clip`, `news`, `stats_or_fact`, and `quote_interview`.
- *What I changed:* I personally reviewed every suggested label. I changed classifications when I disagreed, removed examples that were too ambiguous, and used several difficult examples to test and clarify my decision rules. For example, I changed the World Cup Goal of the Tournament nominees post from `stats_or_fact` to `news`.

**Pre-labelling disclosure:** After manually labelling the first 20 posts, I used AI to pre-label the remaining examples. I personally read and verified every AI-labelled post and corrected or removed labels when necessary.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.667 | 0.700 | +0.033 |
| Macro F1 | 0.567 | 0.620 | +0.053 |
| F1 — `match_clip` | 0.720 | 0.769 | +0.049 |
| F1 — `news` | 0.625 | 0.000 | -0.625 |
| F1 — `quote_interview` | 0.923 | 0.800 | -0.123 |
| F1 — `stats_or_fact` | 0.000 | 0.909 | +0.909 |

**What I predicted before I looked:**
I expect my trained model to outperform the baseline overall, particularly on `match_clip` and `stats_or_fact`, because those categories have strong recurring patterns in my labeled dataset. I am less confident about `news`, since my trained model performed poorly on that label in the initial test run.

**What the gap actually means:**
The trained model slightly outperformed the zero-shot baseline overall, with 0.700 accuracy compared with 0.667 and a macro F1 of 0.620 compared with 0.567. However, the per-label results show that the improvement was not consistent across categories. Fine-tuning greatly improved `stats_or_fact`, where the trained model reached an F1 of 0.909 while the baseline scored 0.000. In contrast, the baseline performed much better on `news`, with an F1 of 0.625 compared with 0.000 for the trained model. This suggests that fine-tuning helped the model learn some task-specific patterns very well, but it may also have learned an unstable or misleading boundary for `news`.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.55 | 0.700 | 0.867 | 0.800 | MET |
| 2. Every label F1 | ≥ 0.50 | MISSED (`news` = 0.000) | MET | MISSED (`news` = 0.444) | MISSED |
| 3. Dataset balance | No label > 45% | 32.0% max | 32.0% max | 32.0% max | MET |
| 4. Label consistency | ≥ 80% agreement | Not tested yet | Not tested yet | Not tested yet | Pending |
| 5. High-confidence predictions | >0.70 confidence → ≥75% correct | Not fully tested yet | Not fully tested yet | Not fully tested yet | Pending |

### Confusion matrix

| true \ predicted | match_clip | news | stats_or_fact | quote_interview |
|---|---:|---:|---:|---:|
| **match_clip** | 10 | 0 | 0 | 0 |
| **news** | 5 | 0 | 0 | 2 |
| **stats_or_fact** | 0 | 1 | 5 | 0 |
| **quote_interview** | 1 | 0 | 0 | 6 |

**My biggest off-diagonal number, and what it means:**

The largest confusion was 5 real `news` posts being predicted as `match_clip`, while no real `match_clip` posts were predicted as `news`. This shows a strongly directional confusion: on this split, the model tended to absorb some `news` examples into `match_clip` rather than confusing the two labels equally.


---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

**Agreement rate:** 30 / 30 = 100%

I matched the staff labels on all 30 posts, so there were no disagreements to adjudicate.

I think the set was relatively clear once I applied the staff decision rules carefully. The most important distinction was whether a specific fact actually carried the argument for `analysis`, rather than simply appearing inside a strong claim. The `reaction` examples were also fairly distinct because they focused on something that had just happened to the writer and their immediate response.

**What the pattern in my disagreements tells me:**

There were no disagreements in the staff set. This suggests that I was able to apply a written taxonomy consistently when the decision rules were explicit. Because the staff taxonomy is different from my own TakeMeter taxonomy, this does not prove that every label in my 200-post dataset was perfectly consistent, but it provides evidence that inconsistency in applying written rules was probably not the main cause of my model's unstable `news` performance.

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
