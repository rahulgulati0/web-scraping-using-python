# Advanced Techniques & Best Practices

## 🎯 Learning Objectives

By the end of this lesson, you will:
- Master advanced web scraping techniques
- Implement proper error handling and retry mechanisms
- Handle rate limiting and respectful scraping
- Use proxies and headers for stealth scraping
- Implement concurrent scraping for better performance
- Store and manage scraped data effectively

## 🔧 Advanced Scraping Techniques

### 1. Headers and User Agents

Always use proper headers to appear more like a real browser:

```python
import requests
from random import choice

# Common user agents
user_agents = [
    'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
    'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
    'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
]

headers = {
    'User-Agent': choice(user_agents),
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Accept-Encoding': 'gzip, deflate',
    'Connection': 'keep-alive',
    'Upgrade-Insecure-Requests': '1',
}

response = requests.get('https://example.com', headers=headers)
```

### 2. Session Management

Use sessions to maintain cookies and connection pooling:

```python
import requests
from time import sleep

class WebScraper:
    def __init__(self):
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        })
    
    def get_page(self, url, delay=1):
        """Get a page with delay and error handling"""
        sleep(delay)  # Be respectful
        try:
            response = self.session.get(url, timeout=10)
            response.raise_for_status()
            return response
        except requests.RequestException as e:
            print(f"Error fetching {url}: {e}")
            return None
    
    def close(self):
        self.session.close()

# Usage
scraper = WebScraper()
try:
    response = scraper.get_page('https://example.com')
    if response:
        print("Success!")
finally:
    scraper.close()
```

### 3. Robust Error Handling

Implement comprehensive error handling:

```python
import requests
from time import sleep
import logging

# Set up logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def robust_get(url, max_retries=3, delay=1):
    """
    Robust HTTP GET with retries and exponential backoff
    """
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=10)
            
            # Handle different status codes
            if response.status_code == 200:
                return response
            elif response.status_code == 429:  # Rate limited
                wait_time = delay * (2 ** attempt)
                logger.warning(f"Rate limited. Waiting {wait_time} seconds...")
                sleep(wait_time)
            elif response.status_code >= 500:  # Server error
                logger.warning(f"Server error {response.status_code}. Retrying...")
                sleep(delay * (attempt + 1))
            else:
                logger.error(f"HTTP {response.status_code}: {response.reason}")
                return None
                
        except requests.Timeout:
            logger.warning(f"Timeout on attempt {attempt + 1}")
            sleep(delay * (attempt + 1))
        except requests.ConnectionError:
            logger.warning(f"Connection error on attempt {attempt + 1}")
            sleep(delay * (attempt + 1))
        except Exception as e:
            logger.error(f"Unexpected error: {e}")
            return None
    
    logger.error(f"Failed to fetch {url} after {max_retries} attempts")
    return None
```

### 4. Rate Limiting and Delays

Implement intelligent rate limiting:

```python
import time
from collections import deque

class RateLimiter:
    def __init__(self, max_requests=10, time_window=60):
        self.max_requests = max_requests
        self.time_window = time_window
        self.requests = deque()
    
    def wait_if_needed(self):
        now = time.time()
        
        # Remove old requests outside the time window
        while self.requests and self.requests[0] <= now - self.time_window:
            self.requests.popleft()
        
        # If we're at the limit, wait
        if len(self.requests) >= self.max_requests:
            sleep_time = self.time_window - (now - self.requests[0])
            if sleep_time > 0:
                time.sleep(sleep_time)
                return self.wait_if_needed()  # Recursive call to recheck
        
        # Record this request
        self.requests.append(now)

# Usage
rate_limiter = RateLimiter(max_requests=5, time_window=60)

for url in urls:
    rate_limiter.wait_if_needed()
    response = requests.get(url)
    # Process response...
```

## 🚀 Performance Optimization

### 1. Concurrent Scraping

Use threading or asyncio for concurrent requests:

```python
import requests
from concurrent.futures import ThreadPoolExecutor, as_completed
import threading

class ConcurrentScraper:
    def __init__(self, max_workers=5):
        self.max_workers = max_workers
        self.session = requests.Session()
        self.lock = threading.Lock()
    
    def fetch_url(self, url):
        """Fetch a single URL"""
        try:
            response = self.session.get(url, timeout=10)
            response.raise_for_status()
            return {
                'url': url,
                'status': 'success',
                'content': response.text,
                'status_code': response.status_code
            }
        except Exception as e:
            return {
                'url': url,
                'status': 'error',
                'error': str(e)
            }
    
    def scrape_urls(self, urls):
        """Scrape multiple URLs concurrently"""
        results = []
        
        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            # Submit all tasks
            future_to_url = {executor.submit(self.fetch_url, url): url for url in urls}
            
            # Collect results
            for future in as_completed(future_to_url):
                result = future.result()
                results.append(result)
                
                # Thread-safe logging
                with self.lock:
                    print(f"Completed: {result['url']} - {result['status']}")
        
        return results

# Usage
scraper = ConcurrentScraper(max_workers=3)
urls = ['https://example.com', 'https://httpbin.org/json', 'https://httpbin.org/html']
results = scraper.scrape_urls(urls)
```

### 2. Async Scraping with aiohttp

For even better performance with async/await:

```python
import asyncio
import aiohttp
from aiohttp import ClientSession

class AsyncScraper:
    def __init__(self, concurrent_limit=10):
        self.concurrent_limit = concurrent_limit
        self.semaphore = asyncio.Semaphore(concurrent_limit)
    
    async def fetch_url(self, session: ClientSession, url: str):
        """Fetch a single URL asynchronously"""
        async with self.semaphore:
            try:
                async with session.get(url) as response:
                    content = await response.text()
                    return {
                        'url': url,
                        'status': 'success',
                        'content': content,
                        'status_code': response.status
                    }
            except Exception as e:
                return {
                    'url': url,
                    'status': 'error',
                    'error': str(e)
                }
    
    async def scrape_urls(self, urls):
        """Scrape multiple URLs asynchronously"""
        async with aiohttp.ClientSession() as session:
            tasks = [self.fetch_url(session, url) for url in urls]
            results = await asyncio.gather(*tasks, return_exceptions=True)
            return results

# Usage
async def main():
    scraper = AsyncScraper(concurrent_limit=5)
    urls = ['https://example.com'] * 10  # 10 URLs to test
    results = await scraper.scrape_urls(urls)
    print(f"Scraped {len(results)} URLs")

# Run the async function
# asyncio.run(main())
```

## 🛡️ Handling Anti-Scraping Measures

### 1. Rotating Proxies

```python
import requests
from itertools import cycle
import random

class ProxyRotator:
    def __init__(self, proxy_list):
        self.proxies = cycle(proxy_list)
        self.current_proxy = next(self.proxies)
    
    def get_proxy_dict(self):
        return {
            'http': self.current_proxy,
            'https': self.current_proxy
        }
    
    def rotate(self):
        self.current_proxy = next(self.proxies)
        return self.current_proxy

# Example proxy list (replace with real proxies)
proxy_list = [
    'http://proxy1.example.com:8080',
    'http://proxy2.example.com:8080',
    'http://proxy3.example.com:8080'
]

rotator = ProxyRotator(proxy_list)

def get_with_proxy(url):
    try:
        response = requests.get(
            url, 
            proxies=rotator.get_proxy_dict(),
            timeout=10
        )
        return response
    except:
        rotator.rotate()  # Switch proxy on failure
        return None
```

### 2. CAPTCHA Handling

```python
# For simple CAPTCHAs, you can use OCR libraries
# pip install pytesseract pillow

from PIL import Image
import pytesseract
import requests
from io import BytesIO

def solve_simple_captcha(captcha_url):
    """
    Attempt to solve simple text CAPTCHAs
    Note: This works only for very basic CAPTCHAs
    """
    try:
        response = requests.get(captcha_url)
        image = Image.open(BytesIO(response.content))
        
        # Preprocess image for better OCR
        image = image.convert('L')  # Convert to grayscale
        image = image.resize((image.width * 2, image.height * 2))  # Resize
        
        text = pytesseract.image_to_string(image)
        return text.strip()
    except Exception as e:
        print(f"CAPTCHA solving failed: {e}")
        return None
```

## 📊 Data Storage and Management

### 1. Database Storage

```python
import sqlite3
import json
from datetime import datetime

class ScrapingDatabase:
    def __init__(self, db_path='scraping_data.db'):
        self.db_path = db_path
        self.init_database()
    
    def init_database(self):
        """Initialize the database with required tables"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS scraped_data (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                url TEXT NOT NULL,
                title TEXT,
                content TEXT,
                metadata TEXT,
                scraped_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                status TEXT DEFAULT 'success'
            )
        ''')
        
        cursor.execute('''
            CREATE INDEX IF NOT EXISTS idx_url ON scraped_data(url);
        ''')
        
        conn.commit()
        conn.close()
    
    def save_scraped_data(self, url, title, content, metadata=None):
        """Save scraped data to database"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        cursor.execute('''
            INSERT INTO scraped_data (url, title, content, metadata)
            VALUES (?, ?, ?, ?)
        ''', (url, title, content, json.dumps(metadata) if metadata else None))
        
        conn.commit()
        conn.close()
    
    def get_scraped_data(self, url=None):
        """Retrieve scraped data"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        if url:
            cursor.execute('SELECT * FROM scraped_data WHERE url = ?', (url,))
        else:
            cursor.execute('SELECT * FROM scraped_data ORDER BY scraped_at DESC')
        
        results = cursor.fetchall()
        conn.close()
        return results

# Usage
db = ScrapingDatabase()
db.save_scraped_data(
    url='https://example.com',
    title='Example Page',
    content='Page content...',
    metadata={'word_count': 150, 'images': 3}
)
```

### 2. Structured Data Export

```python
import pandas as pd
import json
import csv

class DataExporter:
    def __init__(self, data):
        self.data = data
    
    def to_csv(self, filename):
        """Export data to CSV"""
        df = pd.DataFrame(self.data)
        df.to_csv(filename, index=False)
        print(f"Data exported to {filename}")
    
    def to_json(self, filename):
        """Export data to JSON"""
        with open(filename, 'w', encoding='utf-8') as f:
            json.dump(self.data, f, indent=2, ensure_ascii=False)
        print(f"Data exported to {filename}")
    
    def to_excel(self, filename):
        """Export data to Excel"""
        df = pd.DataFrame(self.data)
        df.to_excel(filename, index=False)
        print(f"Data exported to {filename}")

# Usage
scraped_data = [
    {'title': 'Article 1', 'url': 'https://example.com/1', 'content': 'Content 1'},
    {'title': 'Article 2', 'url': 'https://example.com/2', 'content': 'Content 2'}
]

exporter = DataExporter(scraped_data)
exporter.to_csv('scraped_data.csv')
exporter.to_json('scraped_data.json')
```

## 🔍 Monitoring and Logging

### 1. Comprehensive Logging

```python
import logging
from datetime import datetime
import sys

def setup_logging(log_file='scraping.log'):
    """Set up comprehensive logging"""
    
    # Create formatter
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    
    # File handler
    file_handler = logging.FileHandler(log_file)
    file_handler.setFormatter(formatter)
    file_handler.setLevel(logging.INFO)
    
    # Console handler
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setFormatter(formatter)
    console_handler.setLevel(logging.WARNING)
    
    # Configure root logger
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)
    logger.addHandler(file_handler)
    logger.addHandler(console_handler)
    
    return logger

# Usage
logger = setup_logging()

def scrape_with_logging(url):
    logger.info(f"Starting scrape of {url}")
    try:
        response = requests.get(url)
        logger.info(f"Successfully scraped {url} - Status: {response.status_code}")
        return response
    except Exception as e:
        logger.error(f"Failed to scrape {url}: {e}")
        return None
```

## 🎯 Best Practices Summary

### 1. Respect robots.txt

```python
import urllib.robotparser

def can_fetch(url, user_agent='*'):
    """Check if we can fetch a URL according to robots.txt"""
    rp = urllib.robotparser.RobotFileParser()
    rp.set_url(url + '/robots.txt')
    rp.read()
    return rp.can_fetch(user_agent, url)

# Usage
if can_fetch('https://example.com/page'):
    # Proceed with scraping
    pass
else:
    print("Scraping not allowed by robots.txt")
```

### 2. Implement Caching

```python
import requests
import hashlib
import pickle
import os
from datetime import datetime, timedelta

class CachingScraper:
    def __init__(self, cache_dir='cache', cache_duration=3600):
        self.cache_dir = cache_dir
        self.cache_duration = cache_duration
        os.makedirs(cache_dir, exist_ok=True)
    
    def _get_cache_path(self, url):
        """Generate cache file path for URL"""
        url_hash = hashlib.md5(url.encode()).hexdigest()
        return os.path.join(self.cache_dir, f"{url_hash}.pkl")
    
    def _is_cache_valid(self, cache_path):
        """Check if cache file is still valid"""
        if not os.path.exists(cache_path):
            return False
        
        file_time = datetime.fromtimestamp(os.path.getmtime(cache_path))
        return datetime.now() - file_time < timedelta(seconds=self.cache_duration)
    
    def get_page(self, url):
        """Get page with caching"""
        cache_path = self._get_cache_path(url)
        
        # Try to load from cache
        if self._is_cache_valid(cache_path):
            with open(cache_path, 'rb') as f:
                return pickle.load(f)
        
        # Fetch fresh data
        response = requests.get(url)
        if response.status_code == 200:
            # Save to cache
            with open(cache_path, 'wb') as f:
                pickle.dump(response, f)
            return response
        
        return None
```

## 🧪 Testing Your Scrapers

```python
import unittest
from unittest.mock import Mock, patch

class TestWebScraper(unittest.TestCase):
    def setUp(self):
        self.scraper = WebScraper()
    
    @patch('requests.get')
    def test_successful_request(self, mock_get):
        # Mock successful response
        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.text = '<html><title>Test</title></html>'
        mock_get.return_value = mock_response
        
        result = self.scraper.get_page('https://example.com')
        self.assertIsNotNone(result)
        self.assertEqual(result.status_code, 200)
    
    @patch('requests.get')
    def test_failed_request(self, mock_get):
        # Mock failed response
        mock_get.side_effect = requests.RequestException("Connection error")
        
        result = self.scraper.get_page('https://example.com')
        self.assertIsNone(result)

if __name__ == '__main__':
    unittest.main()
```

## 🔧 Practical Exercises

### Exercise 1: Build a Robust News Scraper
Create a scraper that:
- Implements rate limiting
- Uses proper error handling
- Stores data in a database
- Exports results to multiple formats

### Exercise 2: Concurrent Product Scraper
Build a scraper that:
- Scrapes product information from multiple pages concurrently
- Implements proxy rotation
- Includes comprehensive logging
- Handles anti-scraping measures

### Exercise 3: Monitoring Dashboard
Create a simple dashboard to:
- Monitor scraping performance
- Display success/failure rates
- Show data collection statistics
- Alert on errors

## 📝 Key Takeaways

1. **Always be respectful**: Implement delays and follow robots.txt
2. **Handle errors gracefully**: Implement retry logic and proper exception handling
3. **Use proper headers**: Make requests look more natural
4. **Monitor your scrapers**: Implement logging and metrics
5. **Store data efficiently**: Use databases and structured formats
6. **Test thoroughly**: Write tests for your scraping code
7. **Stay within legal bounds**: Always check terms of service

## 🚀 Next Steps

- Apply these techniques in your final project
- Learn about specific anti-scraping bypass techniques
- Explore machine learning for content extraction
- Study website fingerprinting and detection avoidance

Remember: The goal is to build robust, maintainable, and respectful web scrapers that can handle real-world challenges! 