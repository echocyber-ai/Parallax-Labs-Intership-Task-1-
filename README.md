# Reddit NLP Pipeline — Week 1: Data Collection + Cleaning

This repo contains my Week 1 deliverable for the internship: collecting one month of Reddit posts and comments from a chosen subreddit, cleaning the data, and running basic tokenization/corpus stats on it.

## Subreddit & Time Range

- **Subreddit:** r/artificial
- **Date range:** June 1, 2026 – June 30, 2026
- **Data source:** [Arctic Shift API](https://github.com/ArthurHeitmann/arctic_shift) (used instead of PRAW since it allows querying historical posts/comments by exact date range without needing a Reddit developer app)

## Environment Setup

This project was built and run in **Google Colab**, so there's no local install required. If you want to run it locally instead, you'll need Python 3.9+ and the following packages:

```bash
pip install pandas spacy sentence-transformers faiss-cpu requests
python -m spacy download en_core_web_sm
```

If running in Colab, just run the first cell of the notebook — it installs everything automatically:

```python
!pip install pandas spacy sentence-transformers faiss-cpu requests
!python -m spacy download en_core_web_sm
```

## How to Run

The notebook is organized into sequential cells. Run them in order, top to bottom:

1. **Setup cell** — installs dependencies (see above)
2. **Fetch posts** — pulls all posts from r/artificial for June 2026 via the Arctic Shift API, with retry logic for timeouts/connection drops. Saves progress to `posts_progress.json` after every batch so nothing is lost if the session disconnects.
3. **Fetch comments** — same idea, but for comments. This step takes longer (10-15 min) since there are ~9x more comments than posts. Also auto-saves to `comments_progress.json`.
4. **Load into pandas** — converts the raw JSON into `posts_df` and `comments_df` DataFrames.
5. **Clean the data** — removes `[deleted]`/`[removed]` content, filters out bot accounts (e.g. AutoModerator), strips URLs and markdown formatting using regex, and drops duplicate rows.
6. **Tokenize with spaCy** — lowercases text, removes stopwords and punctuation, and generates a word-frequency table (top 20 most common terms) as a basic corpus stats summary.

Note: the tokenization step runs on a random sample of 2,000 cleaned comments rather than the full dataset, since spaCy's per-sentence processing is slow at 20k+ rows. This keeps the notebook fast to run while still giving representative stats. The full dataset is still saved separately and can be tokenized in full later if needed.

## Output Files

- `posts_progress.json` — raw collected posts
- `comments_progress.json` — raw collected comments
- Cleaned/tokenized data and stats are generated inline in the notebook (see final cells)

## Data Summary

- **Raw posts collected:** ~2,317
- **Raw comments collected:** ~20,447
- After removing deleted/removed content, bot comments, and duplicates, the cleaned dataset retains the majority of this volume as usable natural-language text.

## Known Limitations

- The Arctic Shift API occasionally times out or drops the connection on long-running requests; the fetch functions include retry logic (up to 5 retries with a delay) to handle this.
- Bot filtering currently only excludes `AutoModerator` by username — other bots may still be present in smaller numbers and could be added to the filter list in future iterations.
