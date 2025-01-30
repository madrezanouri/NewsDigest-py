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


# Step 3: Fetch News Using the API

Save the code below as fetch_and_save_news.py. It prompts you to choose:

.  Top Headlines by source (e.g., bbc-news)
.  Everything by keyword (e.g., world war 3)

It then retrieves the articles and saves them to **CSV** and **Excel**.

```
from newsapi import NewsApiClient
import pandas as pd

def fetch_and_save_news(api_key):
    """
    Fetches news headlines or news articles based on user's choice,
    saves them to CSV and Excel, and handles empty inputs and errors.
    """

    # Initialize the News API client
    newsapi = NewsApiClient(api_key=api_key)
    
    # 1) Ask the user which mode they'd like: top headlines or everything
    print("Choose an option:")
    print("1) Get top headlines (by news source)")
    print("2) Get all articles (by keyword)")
    choice = input("Enter 1 or 2: ").strip()
    
    try:
        # 2) Get user input for the chosen mode
        if choice == "1":
            # Prompt until the user provides a non-empty source
            source = ""
            while not source:
                source = input("Enter the news source (e.g., 'bbc-news'): ").strip()
                if not source:
                    print("Source cannot be empty. Please try again.")
            
            # Fetch top headlines for the provided source
            response = newsapi.get_top_headlines(sources=source)
        
        elif choice == "2":
            # Prompt until the user provides a non-empty keyword
            keyword = ""
            while not keyword:
                keyword = input("Enter a keyword (e.g., 'bitcoin'): ").strip()
                if not keyword:
                    print("Keyword cannot be empty. Please try again.")
            
            # Fetch all articles for the provided keyword
            # Adding extra params to reduce "too broad" errors
            response = newsapi.get_everything(
                q=keyword,
                language='en',      # filter by English articles
                #sort_by, 'relevancy' # sort results by relevancy
                # You can add from_param, to, etc. if needed, e.g.:
                #from_param='2024-01-01',
                #to='2025-01-30'
            )
        
        else:
            print("Invalid choice. Please run again.")
            return
        
        # 3) Extract articles from the response
        articles = response.get("articles", [])
        
        # If no articles found, just exit
        if not articles:
            print("No articles found for your request.")
            return
        
        # 4) Flatten the articles into a list of dictionaries
        flattened_data = []
        for article in articles:
            src = article.get("source", {})
            flattened_data.append({
                "source_id":   src.get("id"),
                "source_name": src.get("name"),
                "author":      article.get("author"),
                "title":       article.get("title"),
                "description": article.get("description"),
                "url":         article.get("url"),
                "urlToImage":  article.get("urlToImage"),
                "publishedAt": article.get("publishedAt"),
                "content":     article.get("content")
            })
        
        # 5) Convert to DataFrame
        df = pd.DataFrame(flattened_data)
        
        # 6) Save to CSV and Excel
        df.to_csv("news_results2.csv", index=False, encoding='utf-8')
        df.to_excel("news_results2.xlsx", index=False, engine="openpyxl")
        
        print("News data has been saved to 'news_results2.csv' and 'news_results2.xlsx'.")
    
    except NewsAPIException as e:
        # Handle NewsAPI-specific errors
        print(f"An error occurred with NewsAPI: {e}")
    except Exception as e:
        # Handle any other unexpected errors
        print(f"An unexpected error occurred: {e}")

if __name__ == "__main__":
    # Replace with your own News API key
    API_KEY = "XXXXXXXXXXXXXXXXXXXXXXXXXX"
    fetch_and_save_news(API_KEY)
```
