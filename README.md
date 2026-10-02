# Decoding Liberal vs. Conservative Sentiment on Trump's Trade Tariffs: A Reddit Discussion Analysis

Samiksha Singh, Spring 2025

Sentiment and thematic analysis of Reddit comments about the Trump administration's 2025
trade tariffs on Canada, Mexico, and China, comparing a liberal-leaning community
(r/politics) with a conservative-leaning one (r/Conservative).

**Research question:** How do liberal and conservative Reddit communities express sentiment
differently when discussing Trump's trade tariffs?

## Data

- 500 first-level comments total: 250 from r/politics (liberal) and 250 from r/Conservative
  (conservative).
- Collected with PRAW from posts surfaced by Reddit's "hot" algorithm: 7 posts from
  r/Conservative and 2 from r/politics. Only first-level comments (made directly on the post)
  were kept.
- Filter keywords: `tariff` and `trade`, case-insensitive.
- Collection window: February 1, 2025 to March 31, 2025, starting from the date the tariff
  policy was enacted.
- Fields extracted per comment: author, text, date, score (upvotes/downvotes).
- Comments marked as removed/deleted, and obvious bot comments, were removed manually.

## Repository contents

```
SMM_Paper2.pdf / .docx                       The paper

Supplemental/
  Description.docx                           Short description of the four notebooks
  Code/
    1_SSingh_Paper_2_Data_Collection_PRAW.ipynb  Step 1 — comment collection via PRAW
    2_data_cleaning.ipynb                        Step 2 — preprocessing
    3_VADER_sentiment.ipynb                      Step 3 — VADER sentiment + Figs 1-4
    4_EMPATH_sentiment.ipynb                     Step 4 — EMPATH categories + Fig 5
  Data/
    LiberalsData/
      Comments1.csv                          Combined set of 250 comments used for analysis
      Comments2.csv                          Per-post raw export
      Preprocessed_Lib_Comments.csv.csv      Cleaned text
      Vader_score_Lib.csv                    Cleaned text + VADER scores
    ConservativesData/
      Comments1.csv                          Combined set of 250 comments used for analysis
      Comments2.csv ... Comments7.csv        Per-post raw exports
      Preprocessed_Con_Comments.csv          Cleaned text
      Vader_score_Con.csv                    Cleaned text + VADER scores
```

The notebooks in `Supplemental/Code/` store their outputs, so Figs 1-5 can be read without
re-running anything. Untracked working copies of the same notebooks and data sit in the
project root alongside their `.html` exports; see `.gitignore`.

## Setup

Python 3.13.2 was used. Install the dependencies:

```bash
pip install praw pandas numpy matplotlib seaborn nltk vaderSentiment empath
```

Download the VADER lexicon once:

```python
import nltk
nltk.download('vader_lexicon')
```

### Reddit API credentials

Step 1 needs a Reddit script application (https://www.reddit.com/prefs/apps). Supply your own
values in the `praw.Reddit(...)` call:

```python
reddit = praw.Reddit(client_id='...',
                     client_secret='...',
                     user_agent='script by u/<your_username>',
                     username='<your_username>',
                     password='<your_password>')
```

### File paths

The notebooks read and write absolute Windows paths of the form
`C:/Users/samik/Downloads/SMM/Paper2/...`. Change these to the location of this folder on
your machine before running.

## Running the analysis

Run the notebooks in `Supplemental/Code/` in order:

1. **`1_SSingh_Paper_2_Data_Collection_PRAW.ipynb`** — for each post ID, pulls first-level
   comments inside the Feb 1 – Mar 31, 2025 window and writes one
   `Comments<N>.csv` per post, with columns `Author, Text, Date, Score`. Move the outputs
   into `Data/LiberalsData/` or `Data/ConservativesData/` depending on the subreddit, remove
   deleted/removed and bot comments manually, then combine the per-post files into a single
   `Comments1.csv` of 250 comments per group — that combined file is what the later steps
   read.

2. **`2_data_cleaning.ipynb`** — lowercases text, strips hyperlinks, removes punctuation except
   `!` and `?` (kept because VADER uses them), collapses extra whitespace, and checks for
   missing values. Writes `Preprocessed_Lib_Comments.csv.csv` and
   `Preprocessed_Con_Comments.csv`.

3. **`3_VADER_sentiment.ipynb`** — scores each comment with VADER, adding `neg`, `neu`, `pos`,
   and `compound` columns, and writes `Vader_score_Lib.csv` and `Vader_score_Con.csv`.
   Compound scores are binned into five categories:

   | Compound score range | Sentiment category |
   | --- | --- |
   | score > 0.5 | Very Positive |
   | 0.1 < score ≤ 0.5 | Positive |
   | -0.1 < score ≤ 0.1 | Near Neutral |
   | -0.5 < score ≤ -0.1 | Negative |
   | score ≤ -0.5 | Very Negative |

   Produces, with matplotlib and seaborn:
   - Fig 1 — KDE plot of compound score distribution
   - Fig 2 — bar chart of sentiment category distribution (percentages)
   - Fig 3 — grouped bar plot of average polarity (`neg`, `neu`, `pos`, `compound`)
   - Fig 4 — line chart of average compound score per day

4. **`4_EMPATH_sentiment.ipynb`** — runs the EMPATH lexicon over the `Text` column of the
   `Vader_score_*.csv` files with `normalize=True`, summing scores for the categories
   `business`, `economics`, `criticism`, `inflation`, `anger`, `fear`, `suffering`,
   `swearing_terms`, aggregated separately per group.
   - Fig 5 — bar chart comparing EMPATH category scores between the two groups

## Column reference

| Column | Description |
| --- | --- |
| `Author` | Reddit username, or `[deleted]` |
| `Text` | Comment body (cleaned in the preprocessed and VADER files) |
| `Date` | Comment creation time, UTC |
| `Score` | Net upvotes |
| `neg`, `neu`, `pos` | VADER proportion scores |
| `compound` | VADER compound score, -1 (very negative) to 1 (very positive) |

## Findings

Liberals skewed Very Negative/Negative, emphasizing economic fallout and rising consumer
costs; conservatives were more neutral to positive, emphasizing national strength and
economic protectionism. Liberal sentiment declined steadily over the window while
conservative sentiment stayed relatively stable with intermittent spikes. In EMPATH,
conservatives concentrated in `business` and `economics`, liberals in `anger`, `fear`,
`suffering`, and `swearing_terms`.

## Limitations

The topic was ongoing and evolving during collection, with further tariffs announced for
April 2, 2025, so longer-term effects had not yet materialized. The dataset is limited to 500
comments and may not capture the full spectrum of public sentiment, and continuing policy
changes add complexity to the analysis.
