# NewsDigest-py
NewsDigest-Py is a compact Python project that fetches headlines or keyword-based articles from the News API, saves them as CSV/Excel, and then crawls each article URL to extract main text content. It’s perfect for quickly assembling your own dataset of news articles for research, NLP tasks, or data journalism.


# Step 1: Get a News API Key

1. Go to https://newsapi.org/

2. Create an account and get your API Key (it’s free for basic usage).


# Step 2: Install Required Packages

Open your terminal and install:

```
pip install newsapi-python
```

(For later crawling, you’ll also need: **pip install requests beautifulsoup4 pandas openpyxl.**)
