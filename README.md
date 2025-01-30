# NewsDigest-py


![Untitled design](https://github.com/user-attachments/assets/25ae1fe0-feaa-4693-a3a3-c66820f6ee2b)



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


# Step 4: Crawl Each Article URL to Extract Full Text

Next, save this code as crawl_urls.py. It reads the Excel file produced above, crawls each link, and appends the article text as a new column.

```
import requests
from bs4 import BeautifulSoup
import pandas as pd

def extract_article_text_from_url(url):
    """
    Given a URL, fetch the page content and return a simple extracted text.
    Returns an empty string if there's any error.
    """
    try:
        # 1) GET request to fetch the page
        headers = {
            # Some sites block default requests; adding a user-agent helps.
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) "
                          "AppleWebKit/537.36 (KHTML, like Gecko) "
                          "Chrome/58.0.3029.110 Safari/537.3"
        }
        response = requests.get(url, headers=headers, timeout=10)
        response.raise_for_status()  # Raise HTTPError if status != 200

        # 2) Parse HTML with BeautifulSoup
        soup = BeautifulSoup(response.text, 'html.parser')

        # 3) Extract text from <p> tags (basic approach)
        #    You can refine for specific site structures.
        paragraphs = soup.find_all('p')
        # Join them into a single text
        article_text = ' '.join(p.get_text(strip=True) for p in paragraphs)

        return article_text

    except Exception as e:
        print(f"Error extracting from {url}: {e}")
        return ""

def crawl_articles_from_excel(input_excel, output_excel):
    """
    Reads URLs from the input Excel file (expects a column named 'url'),
    extracts article text, and saves the results to output Excel.
    """
    # 1) Read the file into a DataFrame
    df = pd.read_excel(input_excel)
    
    # If you're using CSV instead:
    # df = pd.read_csv("news_results.csv")
    
    # Ensure 'url' column exists
    if 'url' not in df.columns:
        print(f"Error: No 'url' column found in {input_excel}.")
        return

    # 2) Create a new column 'article_text'
    article_texts = []
    
    for idx, row in df.iterrows():
        url = row['url']
        if not isinstance(url, str) or not url.startswith("http"):
            # Skip if invalid URL
            article_texts.append("")
            continue
        
        print(f"Crawling URL {idx+1}/{len(df)}: {url}")
        article_text = extract_article_text_from_url(url)
        article_texts.append(article_text)
    
    # 3) Add the new column to the DataFrame
    df['article_text'] = article_texts
    
    # 4) Save to a new Excel file
    df.to_excel(output_excel, index=False)
    print(f"Extraction complete. Results saved to {output_excel}.")

if __name__ == "__main__":
    # Example usage
    input_file = "news_results2.xlsx"   # or "news_results.csv"
    output_file = "extracted_articles.xlsx"
    
    crawl_articles_from_excel(input_file, output_file)
```


**That’s It!**

You’ve:

.  Fetched news from the **News API** (headlines or by keyword).

.  Saved data to **CSV/Excel**. (Check it out at example_data, news_results.xlsx & extracted_articles.xlsx)

.  Crawled each **article URL**, extracting and storing full text in a new file.

You can now use extracted_articles.xlsx for text analysis, NLP projects, or whatever you need. Enjoy!
