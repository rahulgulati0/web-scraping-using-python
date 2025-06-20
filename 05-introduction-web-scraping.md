# Introduction to Web Scraping

## Overview

Web scraping is the process of extracting data from websites automatically. It's a powerful technique for gathering information from the web for analysis, research, automation, and building data-driven applications.

## Learning Objectives

By the end of this module, you'll be able to:
- Understand what web scraping is and when to use it
- Identify different types of web scraping approaches
- Choose appropriate tools for different scraping tasks
- Understand the legal and ethical considerations
- Plan and structure web scraping projects

## 1. What is Web Scraping?

### Definition

Web scraping (also called web harvesting, web data extraction, or web crawling) is the process of automatically extracting data from websites using software tools.

### How Web Scraping Works

```
1. Send HTTP request to website
2. Receive HTML response
3. Parse HTML to extract data
4. Store/process extracted data
```

### Common Use Cases

- **Price Monitoring**: Track product prices across e-commerce sites
- **Market Research**: Gather competitor information
- **Data Collection**: Collect data for analysis and research
- **Content Aggregation**: Gather news, articles, or other content
- **Lead Generation**: Collect contact information from business directories
- **Social Media Analysis**: Monitor social media trends and sentiment

## 2. Types of Web Scraping

### Static Content Scraping

Scraping content that's directly embedded in the HTML source code.

```python
# Example: Scraping a simple HTML page
import requests
from bs4 import BeautifulSoup

url = "https://example.com/products"
response = requests.get(url)
soup = BeautifulSoup(response.content, 'html.parser')

# Extract product titles
titles = soup.find_all('h2', class_='product-title')
for title in titles:
    print(title.text.strip())
```

### Dynamic Content Scraping

Scraping content that's loaded via JavaScript after the initial page load.

```python
# Example: Using Selenium for dynamic content
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

driver = webdriver.Chrome()
driver.get("https://example.com/dynamic-content")

# Wait for content to load
wait = WebDriverWait(driver, 10)
elements = wait.until(EC.presence_of_all_elements_located((By.CLASS_NAME, "dynamic-content")))

for element in elements:
    print(element.text)

driver.quit()
```

### API Scraping

Extracting data from web APIs (Application Programming Interfaces).

```python
# Example: Scraping JSON API
import requests
import json

url = "https://api.example.com/products"
response = requests.get(url)
data = response.json()

for product in data['products']:
    print(f"Name: {product['name']}, Price: ${product['price']}")
```

## 3. Web Scraping Tools and Libraries

### Python Libraries

#### Requests
- **Purpose**: Making HTTP requests
- **Best for**: Simple GET/POST requests
- **Installation**: `pip install requests`

```python
import requests

response = requests.get('https://example.com')
print(response.status_code)
print(response.text)
```

#### BeautifulSoup
- **Purpose**: HTML/XML parsing
- **Best for**: Extracting data from HTML
- **Installation**: `pip install beautifulsoup4`

```python
from bs4 import BeautifulSoup

html = '<html><body><h1>Title</h1><p>Content</p></body></html>'
soup = BeautifulSoup(html, 'html.parser')
title = soup.find('h1').text
print(title)  # "Title"
```

#### Selenium
- **Purpose**: Browser automation
- **Best for**: Dynamic content, JavaScript-heavy sites
- **Installation**: `pip install selenium`

```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get('https://example.com')
content = driver.find_element_by_tag_name('body').text
driver.quit()
```

#### Scrapy
- **Purpose**: Full-featured web scraping framework
- **Best for**: Large-scale scraping projects
- **Installation**: `pip install scrapy`

```python
import scrapy

class ProductSpider(scrapy.Spider):
    name = 'products'
    start_urls = ['https://example.com/products']
    
    def parse(self, response):
        for product in response.css('.product'):
            yield {
                'name': product.css('.title::text').get(),
                'price': product.css('.price::text').get()
            }
```

### Other Tools

#### LXML
- **Purpose**: Fast XML/HTML processing
- **Best for**: Performance-critical applications

#### Pandas
- **Purpose**: Data manipulation and analysis
- **Best for**: Processing scraped data

#### SQLite/PostgreSQL
- **Purpose**: Data storage
- **Best for**: Storing scraped data

## 4. Web Scraping Process

### Step 1: Planning and Analysis

```python
# Analyze the target website
def analyze_website(url):
    """Analyze website structure for scraping"""
    import requests
    from bs4 import BeautifulSoup
    
    response = requests.get(url)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    # Check for robots.txt
    robots_url = url.replace('/page', '/robots.txt')
    robots_response = requests.get(robots_url)
    
    # Analyze page structure
    structure = {
        'title': soup.find('title').text if soup.find('title') else None,
        'headings': [h.text for h in soup.find_all(['h1', 'h2', 'h3'])],
        'links': len(soup.find_all('a')),
        'forms': len(soup.find_all('form')),
        'robots_txt': robots_response.text if robots_response.status_code == 200 else None
    }
    
    return structure
```

### Step 2: Data Extraction Strategy

```python
# Define data extraction strategy
def extract_product_data(soup):
    """Extract product data from HTML"""
    products = []
    
    # Find product containers
    product_elements = soup.find_all('div', class_='product-item')
    
    for element in product_elements:
        product = {
            'name': element.find('h3', class_='product-title').text.strip(),
            'price': element.find('span', class_='price').text.strip(),
            'url': element.find('a')['href'],
            'image': element.find('img')['src'] if element.find('img') else None
        }
        products.append(product)
    
    return products
```

### Step 3: Data Processing and Storage

```python
# Process and store extracted data
def process_and_store_data(products):
    """Process and store scraped data"""
    import json
    import csv
    
    # Save as JSON
    with open('products.json', 'w') as f:
        json.dump(products, f, indent=2)
    
    # Save as CSV
    with open('products.csv', 'w', newline='') as f:
        if products:
            writer = csv.DictWriter(f, fieldnames=products[0].keys())
            writer.writeheader()
            writer.writerows(products)
    
    print(f"Saved {len(products)} products")
```

## 5. Common Challenges and Solutions

### Challenge 1: Rate Limiting

**Problem**: Websites may block requests that come too quickly.

**Solution**: Implement delays between requests.

```python
import time
import random

def polite_request(url, delay_range=(1, 3)):
    """Make a polite request with random delay"""
    delay = random.uniform(*delay_range)
    time.sleep(delay)
    
    response = requests.get(url)
    return response
```

### Challenge 2: CAPTCHA and Bot Detection

**Problem**: Websites may use CAPTCHAs or other methods to detect bots.

**Solution**: Use realistic headers and behavior patterns.

```python
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Accept-Encoding': 'gzip, deflate',
    'Connection': 'keep-alive',
    'Upgrade-Insecure-Requests': '1'
}

response = requests.get(url, headers=headers)
```

### Challenge 3: Dynamic Content

**Problem**: Content loaded via JavaScript may not be available in the initial HTML.

**Solution**: Use Selenium or wait for content to load.

```python
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

driver = webdriver.Chrome()
driver.get(url)

# Wait for specific element to load
wait = WebDriverWait(driver, 10)
element = wait.until(EC.presence_of_element_located((By.CLASS_NAME, "dynamic-content")))

content = element.text
driver.quit()
```

### Challenge 4: Pagination

**Problem**: Data spread across multiple pages.

**Solution**: Implement pagination handling.

```python
def scrape_paginated_content(base_url, max_pages=10):
    """Scrape content from multiple pages"""
    all_data = []
    
    for page in range(1, max_pages + 1):
        url = f"{base_url}?page={page}"
        response = requests.get(url)
        
        if response.status_code != 200:
            break
            
        soup = BeautifulSoup(response.content, 'html.parser')
        page_data = extract_data(soup)
        
        if not page_data:
            break
            
        all_data.extend(page_data)
        
        # Be polite
        time.sleep(2)
    
    return all_data
```

## 6. Legal and Ethical Considerations

### Legal Considerations

- **Terms of Service**: Check website's terms of service
- **robots.txt**: Respect robots.txt file
- **Copyright**: Be aware of copyright laws
- **Rate Limiting**: Don't overwhelm servers

### Ethical Guidelines

```python
# Ethical scraping checklist
def ethical_scraping_checklist(url):
    """Check if scraping is ethical"""
    import requests
    
    checks = {
        'robots_txt': False,
        'terms_of_service': False,
        'rate_limiting': True,
        'data_usage': 'research_only'
    }
    
    # Check robots.txt
    robots_url = url.replace('/page', '/robots.txt')
    try:
        robots_response = requests.get(robots_url, timeout=5)
        if robots_response.status_code == 200:
            checks['robots_txt'] = True
    except:
        pass
    
    return checks
```

### Best Practices

1. **Respect robots.txt**
2. **Use reasonable delays between requests**
3. **Identify your scraper in User-Agent**
4. **Don't scrape personal or sensitive data**
5. **Check terms of service**
6. **Use data responsibly**

## 7. Project Structure

### Recommended Project Structure

```
web_scraping_project/
├── scrapers/
│   ├── __init__.py
│   ├── product_scraper.py
│   └── news_scraper.py
├── data/
│   ├── raw/
│   └── processed/
├── config/
│   └── settings.py
├── utils/
│   ├── __init__.py
│   ├── helpers.py
│   └── database.py
├── requirements.txt
├── README.md
└── main.py
```

### Example Project Setup

```python
# config/settings.py
import os

# Base settings
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
DATA_DIR = os.path.join(BASE_DIR, 'data')

# Scraping settings
DELAY_RANGE = (1, 3)
MAX_RETRIES = 3
TIMEOUT = 30

# Database settings
DATABASE_URL = 'sqlite:///scraped_data.db'

# Headers
DEFAULT_HEADERS = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5'
}
```

```python
# utils/helpers.py
import time
import random
import requests
from config.settings import DELAY_RANGE, DEFAULT_HEADERS

def make_request(url, headers=None, delay=True):
    """Make a polite HTTP request"""
    if delay:
        time.sleep(random.uniform(*DELAY_RANGE))
    
    if headers is None:
        headers = DEFAULT_HEADERS
    
    response = requests.get(url, headers=headers, timeout=30)
    return response

def save_data(data, filename, format='json'):
    """Save data in specified format"""
    import json
    import csv
    
    if format == 'json':
        with open(filename, 'w') as f:
            json.dump(data, f, indent=2)
    elif format == 'csv':
        with open(filename, 'w', newline='') as f:
            if data:
                writer = csv.DictWriter(f, fieldnames=data[0].keys())
                writer.writeheader()
                writer.writerows(data)
```

## 8. Testing and Validation

### Data Validation

```python
def validate_scraped_data(data, required_fields=None):
    """Validate scraped data"""
    if required_fields is None:
        required_fields = ['name', 'price']
    
    valid_data = []
    invalid_data = []
    
    for item in data:
        is_valid = True
        missing_fields = []
        
        for field in required_fields:
            if field not in item or not item[field]:
                is_valid = False
                missing_fields.append(field)
        
        if is_valid:
            valid_data.append(item)
        else:
            invalid_data.append({
                'item': item,
                'missing_fields': missing_fields
            })
    
    return valid_data, invalid_data
```

### Error Handling

```python
def robust_scraping(url, max_retries=3):
    """Robust scraping with error handling"""
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=30)
            response.raise_for_status()
            return response
        except requests.exceptions.RequestException as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)  # Exponential backoff
            else:
                raise
```

## 9. Performance Optimization

### Concurrent Scraping

```python
import concurrent.futures
import threading

def scrape_urls_concurrent(urls, max_workers=5):
    """Scrape multiple URLs concurrently"""
    results = []
    
    with concurrent.futures.ThreadPoolExecutor(max_workers=max_workers) as executor:
        future_to_url = {executor.submit(make_request, url): url for url in urls}
        
        for future in concurrent.futures.as_completed(future_to_url):
            url = future_to_url[future]
            try:
                response = future.result()
                results.append((url, response))
            except Exception as e:
                print(f"Error scraping {url}: {e}")
    
    return results
```

### Memory Management

```python
def process_large_dataset(urls, batch_size=100):
    """Process large datasets in batches"""
    for i in range(0, len(urls), batch_size):
        batch = urls[i:i + batch_size]
        
        # Process batch
        batch_results = []
        for url in batch:
            response = make_request(url)
            # Process response...
            batch_results.append(processed_data)
        
        # Save batch results
        save_batch_results(batch_results, f"batch_{i//batch_size}.json")
        
        # Clear memory
        del batch_results
```

## 10. Exercises

### Exercise 1: Website Analysis
Analyze a website's structure and identify what data can be scraped.

### Exercise 2: Simple Scraper
Build a basic scraper that extracts data from a simple HTML page.

### Exercise 3: Error Handling
Implement robust error handling for a web scraper.

### Exercise 4: Data Validation
Create a data validation system for scraped content.

## 11. Next Steps

You now understand the fundamentals of web scraping! Before moving to the next module, practice by:

1. Analyzing different websites for scraping potential
2. Building simple scrapers for static content
3. Understanding the legal and ethical considerations
4. Planning scraping projects

When you're comfortable with these concepts, proceed to [Requests Library for HTTP](06-requests-library.md).

## Key Takeaways

- Web scraping is extracting data from websites automatically
- Choose the right tool for your specific scraping needs
- Always respect websites' terms of service and robots.txt
- Implement proper error handling and rate limiting
- Plan your scraping projects carefully
- Consider legal and ethical implications

Remember: Responsible scraping ensures sustainable access to web data! 