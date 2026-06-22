# TakeMeter — r/berkeley Discourse Classifier

## Community

I chose **r/berkeley**, the subreddit for UC Berkeley students. I'm a current Berkeley student, so I have the context to judge what counts as a thoughtful take versus noise in this community — threads about housing, classes (e.g. CS61B vs CS70 difficulty), campus safety incidents, professors, and admissions generate constant opinions with wildly different rigor levels. A post about whether a dorm is safe at night can range from a detailed account with specifics to a one-line vent. That range — from grounded, specific commentary to pure reaction — is what makes the discourse worth classifying.

## Labels

- **analysis**: A post that makes a specific, supportable claim about a Berkeley-related topic (housing, classes, safety, admissions, etc.) using concrete details, personal experience, or evidence, and reasons through it rather than just asserting it.
- **hot_take**: A post that states a strong, deliberately provocative or contrarian opinion about a Berkeley-related topic with little to no supporting reasoning or evidence.
- **reaction**: A post that primarily expresses an emotional response (excitement, frustration, relief, humor) to something Berkeley-related, without making a claim or argument at all.

**Examples:**

*analysis*
1. "I lived in Clark Kerr freshman year and Unit 1 sophomore year — Clark Kerr's dining hall access and quieter halls made it way better for actually sleeping, even though Unit 1 is closer to campus."
2. "CS61B is harder than CS70 for most people not because of the math but because of the time cost of debugging Java assignments — 70's hardest material is compressed into fewer, shorter problem sets."

*hot_take*
1. "Southside is objectively better than Northside and anyone who disagrees just hasn't lived there."
2. "Office hours are a waste of time, just read the textbook."

*reaction*
1. "Just got off the Unit 3 elevator that's been broken for two weeks lol I'm so done"
2. "First sunny day of the semester and Memorial Glade is PACKED, love this campus"

## Hard edge cases

The genuinely ambiguous case is a post that opens with an emotional reaction and then pivots into a claim — e.g. "I'm so tired of people saying Northside is safer, I lived there for two years and saw more incidents than my friends in Southside ever reported." That's reaction-flavored but contains a comparable, evidence-backed claim underneath.

**Rule I'll apply during annotation:** I'll label based on what the post is doing by its final sentence, not its opening tone. If the post ends on a supported claim, it's `analysis`. If it ends on an assertion with no support, it's `hot_take`. If it ends on pure feeling with no claim at all, it's `reaction`. I'll log every case where I had to apply this tiebreak rule in a `hard_cases.md` file so I can audit consistency afterward.

## Data collection plan

I'll pull posts and top-level comments from r/berkeley using the Reddit API (PRAW), filtered to threads tagged or about housing, classes, safety, and admissions — sampling across the "hot," "top," and "new" feeds to avoid over-indexing on whatever's currently trending. Target: roughly 70 examples per label (210 total, buffer above the 200 minimum) split 70/15/15 train/val/test.

If a label is underrepresented after the initial pull (most likely `analysis`, since long-form posts are rarer than quick reactions), I'll search specific high-effort threads (e.g. course review megathreads, housing comparison threads) rather than padding with borderline examples, since forcing weak posts into a label just to hit a quota would teach the model the wrong signal.

## Evaluation metrics

Accuracy alone won't tell me much here because the classes aren't symmetric in cost or difficulty — confusing `analysis` for `hot_take` is a much more meaningful failure than confusing `hot_take` for `reaction`. So I'll report:
- **Per-class precision and recall** — to see whether the model is systematically over-predicting one label (e.g. defaulting to `reaction` whenever it's unsure, which would inflate `reaction` recall while tanking `analysis` recall).
- **Macro F1** — since label counts may not be perfectly even, macro F1 keeps a minority label's errors from being washed out by overall accuracy.
- **Confusion matrix** — to see *which* labels get confused with which, not just how often the model is wrong overall.

## Definition of success

For this to be genuinely useful — say, as a tool that surfaces high-effort `analysis` posts to the top of a thread — I'd want **macro F1 above 0.70** and, critically, **`analysis` recall above 0.75**, since missing genuine analysis defeats the tool's purpose even if overall accuracy looks fine. I'd accept "good enough for deployment" at those numbers as long as the confusion matrix shows errors concentrated between `hot_take` and `reaction` (a low-stakes mix-up) rather than between `analysis` and either of the other two (a high-stakes mix-up that misses the actual goal).
