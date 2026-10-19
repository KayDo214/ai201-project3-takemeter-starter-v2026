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

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**
<!-- Milestone 1 asks you to write this BEFORE seeing the trained numbers. A
     prediction made afterwards isn't one. -->

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



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

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

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
