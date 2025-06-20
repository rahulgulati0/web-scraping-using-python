# BeautifulSoup for HTML Parsing

## Overview

BeautifulSoup is a Python library for parsing HTML and XML documents. It creates a parse tree from page source code that can be used to extract data in a more readable and intuitive way.

## Learning Objectives

By the end of this module, you'll be able to:
- Parse HTML documents with BeautifulSoup
- Navigate and search through HTML trees
- Extract text, attributes, and structured data
- Handle different HTML structures and edge cases
- Combine BeautifulSoup with Requests for web scraping

## 1. Getting Started with BeautifulSoup

### Installation

```bash
pip install beautifulsoup4
pip install lxml
pip install html5lib
```

### Basic Usage

```python
from bs4 import BeautifulSoup
import requests

# Parse HTML string
html = '<html><body><h1>Hello World</h1></body></html>'
soup = BeautifulSoup(html, 'html.parser')
print(soup.h1.text)  # "Hello World"

# Parse from web request
response = requests.get('https://example.com')
soup = BeautifulSoup(response.content, 'html.parser')
```

### Choosing a Parser

```python
# html.parser (built-in)
soup = BeautifulSoup(html, 'html.parser')

# lxml (fast)
soup = BeautifulSoup(html, 'lxml')

# html5lib (very lenient)
soup = BeautifulSoup(html, 'html5lib')
```

## 2. Finding Elements

### Basic Methods

```python
# Find first occurrence
first_p = soup.find('p')

# Find all occurrences
all_p = soup.find_all('p')

# Find by class
content_div = soup.find('div', class_='content')

# Find by multiple attributes
element = soup.find('div', {'class': 'content', 'id': 'main'})
```

### CSS Selectors

```python
# Select by tag
paragraphs = soup.select('p')

# Select by class
content = soup.select('.content')

# Select by ID
main = soup.select('#main-content')

# Descendant selectors
content_p = soup.select('.content p')

# Attribute selectors
links = soup.select('a[href^="http"]')
```

## 3. Extracting Data

### Getting Text and Attributes

```python
# Get text content
element = soup.find('h1')
text = element.text  # or element.get_text()

# Get attribute value
link = soup.find('a')
href = link.get('href')

# Check if attribute exists
if link.has_attr('href'):
    print("Link has href")
```

### Practical Example

```python
def extract_products(soup):
    """Extract product information"""
    products = []
    
    for product in soup.find_all('div', class_='product'):
        data = {
            'name': product.find('h3').text.strip(),
            'price': product.find('span', class_='price').text.strip(),
            'url': product.find('a')['href']
        }
        products.append(data)
    
    return products
```

## 4. Error Handling

### Safe Extraction

```python
def safe_extract(element, default=''):
    """Safely extract text from element"""
    try:
        return element.text.strip() if element else default
    except AttributeError:
        return default

def extract_product_safely(soup):
    """Extract with error handling"""
    products = []
    
    for element in soup.find_all('div', class_='product'):
        try:
            name_elem = element.find('h3', class_='product-name')
            price_elem = element.find('span', class_='price')
            
            product = {
                'name': safe_extract(name_elem, 'Unknown'),
                'price': safe_extract(price_elem, '0')
            }
            
            if product['name'] != 'Unknown':
                products.append(product)
                
        except Exception as e:
            print(f"Error processing product: {e}")
            continue
    
    return products
```

## 5. Complete Example

```python
import requests
from bs4 import BeautifulSoup
import json

class WebScraper:
    def __init__(self):
        self.session = requests.Session()
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        })
    
    def scrape_page(self, url):
        """Scrape a single page"""
        try:
            response = self.session.get(url, timeout=30)
            response.raise_for_status()
            
            soup = BeautifulSoup(response.content, 'html.parser')
            return self.extract_data(soup)
        
        except Exception as e:
            print(f"Error scraping {url}: {e}")
            return None
    
    def extract_data(self, soup):
        """Extract data from soup"""
        # Extract title
        title = soup.find('title')
        page_title = title.text.strip() if title else ''
        
        # Extract all links
        links = []
        for link in soup.find_all('a', href=True):
            links.append({
                'text': link.text.strip(),
                'url': link['href']
            })
        
        # Extract images
        images = []
        for img in soup.find_all('img', src=True):
            images.append({
                'src': img['src'],
                'alt': img.get('alt', '')
            })
        
        return {
            'title': page_title,
            'links': links,
            'images': images
        }

# Usage
scraper = WebScraper()
data = scraper.scrape_page('https://example.com')
print(json.dumps(data, indent=2))
```

## Key Takeaways

- BeautifulSoup makes HTML parsing intuitive
- Use CSS selectors for flexible element selection
- Always handle errors gracefully
- Combine with Requests for complete solutions
- Choose the right parser for your needs

Remember: Practice with real websites to master BeautifulSoup! 