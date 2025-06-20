# Requests Library for HTTP

## Overview

The Requests library is the foundation of web scraping in Python. It provides a simple and elegant way to make HTTP requests to web servers. This module covers everything you need to know about using Requests for web scraping.

## Learning Objectives

By the end of this module, you'll be able to:
- Make different types of HTTP requests (GET, POST, etc.)
- Handle responses and status codes
- Work with headers, cookies, and sessions
- Implement authentication and authorization
- Handle errors and timeouts
- Use advanced features for web scraping

## 1. Getting Started with Requests

### Installation

```bash
pip install requests
```

### Basic Usage

```python
import requests

# Simple GET request
response = requests.get('https://api.github.com')
print(response.status_code)  # 200
print(response.text[:100])   # First 100 characters
```

### Response Object

```python
response = requests.get('https://example.com')

# Response properties
print(f"Status Code: {response.status_code}")
print(f"Headers: {response.headers}")
print(f"Content: {response.text}")
print(f"Encoding: {response.encoding}")
print(f"URL: {response.url}")
print(f"Elapsed Time: {response.elapsed}")
```

## 2. HTTP Methods

### GET Requests

```python
# Basic GET request
response = requests.get('https://api.github.com/users/octocat')

# GET with parameters
params = {
    'q': 'python web scraping',
    'page': 1,
    'per_page': 10
}
response = requests.get('https://api.github.com/search/repositories', params=params)
print(response.url)  # Shows the full URL with parameters
```

### POST Requests

```python
# POST with form data
data = {
    'username': 'john_doe',
    'password': 'secret123',
    'submit': 'Login'
}
response = requests.post('https://example.com/login', data=data)

# POST with JSON data
json_data = {
    'name': 'New Product',
    'price': 99.99,
    'category': 'Electronics'
}
response = requests.post('https://api.example.com/products', json=json_data)
```

### Other HTTP Methods

```python
# PUT request
response = requests.put('https://api.example.com/users/123', json={'name': 'Updated Name'})

# DELETE request
response = requests.delete('https://api.example.com/users/123')

# HEAD request (headers only)
response = requests.head('https://example.com')
print(response.headers)

# OPTIONS request
response = requests.options('https://api.example.com')
print(response.headers.get('Allow'))
```

## 3. Working with Headers

### Setting Headers

```python
# Custom headers
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
    'Accept-Language': 'en-US,en;q=0.5',
    'Accept-Encoding': 'gzip, deflate',
    'Connection': 'keep-alive',
    'Upgrade-Insecure-Requests': '1'
}

response = requests.get('https://example.com', headers=headers)
```

### Common Headers for Web Scraping

```python
def get_scraping_headers():
    """Get realistic headers for web scraping"""
    return {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36',
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8',
        'Accept-Language': 'en-US,en;q=0.5',
        'Accept-Encoding': 'gzip, deflate',
        'DNT': '1',
        'Connection': 'keep-alive',
        'Upgrade-Insecure-Requests': '1',
        'Sec-Fetch-Dest': 'document',
        'Sec-Fetch-Mode': 'navigate',
        'Sec-Fetch-Site': 'none',
        'Cache-Control': 'max-age=0'
    }

# Usage
headers = get_scraping_headers()
response = requests.get('https://example.com', headers=headers)
```

### Reading Response Headers

```python
response = requests.get('https://example.com')

# Get specific header
content_type = response.headers.get('Content-Type')
content_length = response.headers.get('Content-Length')

# Get all headers
for header, value in response.headers.items():
    print(f"{header}: {value}")

# Check if response is JSON
if 'application/json' in response.headers.get('Content-Type', ''):
    data = response.json()
```

## 4. URL Parameters and Query Strings

### Adding Parameters

```python
# Method 1: Using params argument
params = {
    'search': 'python',
    'category': 'programming',
    'page': 1
}
response = requests.get('https://example.com/search', params=params)
print(response.url)  # https://example.com/search?search=python&category=programming&page=1

# Method 2: Building URL manually
url = 'https://example.com/search?search=python&page=1'
response = requests.get(url)
```

### Complex Parameters

```python
# Multiple values for same parameter
params = {
    'tags': ['python', 'web-scraping', 'tutorial'],
    'price_range': '0-100',
    'sort': 'price_asc'
}
response = requests.get('https://example.com/products', params=params)

# URL encoding is handled automatically
params = {
    'query': 'python web scraping tutorial',
    'category': 'programming & development'
}
response = requests.get('https://example.com/search', params=params)
```

## 5. Request and Response Content

### Different Content Types

```python
# Text content
response = requests.get('https://example.com')
html_content = response.text

# Binary content (for images, files)
response = requests.get('https://example.com/image.jpg')
image_data = response.content

# JSON content
response = requests.get('https://api.github.com/users/octocat')
user_data = response.json()

# Raw response
response = requests.get('https://example.com', stream=True)
raw_content = response.raw.read()
```

### Handling Different Encodings

```python
# Set encoding manually
response = requests.get('https://example.com')
response.encoding = 'utf-8'
content = response.text

# Detect encoding
import chardet

response = requests.get('https://example.com')
detected_encoding = chardet.detect(response.content)['encoding']
response.encoding = detected_encoding
content = response.text
```

### Streaming Large Responses

```python
# Stream large files
response = requests.get('https://example.com/large-file.txt', stream=True)

with open('large-file.txt', 'wb') as f:
    for chunk in response.iter_content(chunk_size=8192):
        if chunk:
            f.write(chunk)

# Stream with progress
import tqdm

response = requests.get('https://example.com/large-file.txt', stream=True)
total_size = int(response.headers.get('content-length', 0))

with open('large-file.txt', 'wb') as f:
    with tqdm.tqdm(total=total_size, unit='B', unit_scale=True) as pbar:
        for chunk in response.iter_content(chunk_size=8192):
            if chunk:
                f.write(chunk)
                pbar.update(len(chunk))
```

## 6. Sessions and Cookies

### Using Sessions

```python
# Create a session
session = requests.Session()

# Set default headers for all requests in this session
session.headers.update({
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8'
})

# Make requests with the session
response1 = session.get('https://example.com/login')
response2 = session.get('https://example.com/dashboard')  # Cookies are automatically sent
```

### Working with Cookies

```python
# Set cookies manually
cookies = {
    'session_id': 'abc123',
    'user_pref': 'dark_mode',
    'language': 'en'
}
response = requests.get('https://example.com', cookies=cookies)

# Get cookies from response
response = requests.get('https://example.com')
print(response.cookies)

# Session with cookies
session = requests.Session()
session.cookies.set('session_id', 'abc123')
session.cookies.set('user_pref', 'dark_mode')

response = session.get('https://example.com')
```

### Cookie Jar

```python
from requests.cookies import RequestsCookieJar

# Create a cookie jar
jar = RequestsCookieJar()
jar.set('session_id', 'abc123', domain='example.com', path='/')
jar.set('user_pref', 'dark_mode', domain='example.com', path='/')

response = requests.get('https://example.com', cookies=jar)
```

## 7. Authentication

### Basic Authentication

```python
# Method 1: Using auth parameter
from requests.auth import HTTPBasicAuth

response = requests.get('https://api.github.com/user', 
                       auth=HTTPBasicAuth('username', 'password'))

# Method 2: Using tuple
response = requests.get('https://api.github.com/user', 
                       auth=('username', 'password'))

# Method 3: Using headers
headers = {
    'Authorization': 'Basic dXNlcm5hbWU6cGFzc3dvcmQ='
}
response = requests.get('https://api.github.com/user', headers=headers)
```

### Bearer Token Authentication

```python
# Method 1: Using headers
headers = {
    'Authorization': 'Bearer your_token_here'
}
response = requests.get('https://api.github.com/user', headers=headers)

# Method 2: Custom auth class
class BearerAuth(requests.auth.AuthBase):
    def __init__(self, token):
        self.token = token
    
    def __call__(self, r):
        r.headers['Authorization'] = f'Bearer {self.token}'
        return r

response = requests.get('https://api.github.com/user', 
                       auth=BearerAuth('your_token_here'))
```

### OAuth Authentication

```python
# OAuth 2.0 with requests-oauthlib
from requests_oauthlib import OAuth2Session

# Create OAuth session
oauth = OAuth2Session(client_id='your_client_id',
                     redirect_uri='https://your-app.com/callback')

# Get authorization URL
authorization_url, state = oauth.authorization_url('https://provider.com/oauth/authorize')

# After user authorizes, get the authorization response URL
# Then get the token
token = oauth.fetch_token('https://provider.com/oauth/token',
                         authorization_response='https://your-app.com/callback?code=...')

# Make authenticated requests
response = oauth.get('https://api.provider.com/user')
```

## 8. Error Handling and Timeouts

### Basic Error Handling

```python
import requests
from requests.exceptions import RequestException

try:
    response = requests.get('https://example.com')
    response.raise_for_status()  # Raises an HTTPError for bad responses
except requests.exceptions.HTTPError as e:
    print(f"HTTP Error: {e}")
except requests.exceptions.ConnectionError as e:
    print(f"Connection Error: {e}")
except requests.exceptions.Timeout as e:
    print(f"Timeout Error: {e}")
except requests.exceptions.RequestException as e:
    print(f"Request Error: {e}")
```

### Timeouts

```python
# Set timeout for all requests
response = requests.get('https://example.com', timeout=10)

# Set different timeouts for connect and read
response = requests.get('https://example.com', timeout=(3.05, 27))

# No timeout (not recommended)
response = requests.get('https://example.com', timeout=None)
```

### Retry Logic

```python
import time
from requests.exceptions import RequestException

def make_request_with_retry(url, max_retries=3, delay=1):
    """Make request with retry logic"""
    for attempt in range(max_retries):
        try:
            response = requests.get(url, timeout=30)
            response.raise_for_status()
            return response
        except RequestException as e:
            print(f"Attempt {attempt + 1} failed: {e}")
            if attempt < max_retries - 1:
                time.sleep(delay * (2 ** attempt))  # Exponential backoff
            else:
                raise
    return None
```

### Status Code Handling

```python
def handle_response(response):
    """Handle different HTTP status codes"""
    if response.status_code == 200:
        return response.json()
    elif response.status_code == 404:
        print("Resource not found")
        return None
    elif response.status_code == 429:
        print("Rate limited. Waiting...")
        time.sleep(60)
        return None
    elif response.status_code >= 500:
        print(f"Server error: {response.status_code}")
        return None
    else:
        print(f"Unexpected status code: {response.status_code}")
        return None
```

## 9. Advanced Features

### Proxies

```python
# Single proxy
proxies = {
    'http': 'http://proxy.example.com:8080',
    'https': 'https://proxy.example.com:8080'
}
response = requests.get('https://example.com', proxies=proxies)

# Proxy with authentication
proxies = {
    'http': 'http://user:pass@proxy.example.com:8080',
    'https': 'https://user:pass@proxy.example.com:8080'
}
response = requests.get('https://example.com', proxies=proxies)

# Session with proxies
session = requests.Session()
session.proxies.update(proxies)
response = session.get('https://example.com')
```

### SSL Certificate Verification

```python
# Disable SSL verification (not recommended for production)
response = requests.get('https://example.com', verify=False)

# Custom certificate
response = requests.get('https://example.com', 
                       cert=('/path/to/cert.pem', '/path/to/key.pem'))

# Custom CA bundle
response = requests.get('https://example.com', 
                       verify='/path/to/ca-bundle.crt')
```

### File Uploads

```python
# Upload a file
with open('file.txt', 'rb') as f:
    files = {'file': f}
    response = requests.post('https://example.com/upload', files=files)

# Upload multiple files
files = {
    'file1': open('file1.txt', 'rb'),
    'file2': open('file2.txt', 'rb')
}
response = requests.post('https://example.com/upload', files=files)

# Upload with additional data
files = {'file': open('file.txt', 'rb')}
data = {'description': 'My uploaded file'}
response = requests.post('https://example.com/upload', files=files, data=data)
```

## 10. Web Scraping Examples

### Basic Web Scraper

```python
import requests
from bs4 import BeautifulSoup
import time
import random

class WebScraper:
    def __init__(self, base_url, delay_range=(1, 3)):
        self.session = requests.Session()
        self.base_url = base_url
        self.delay_range = delay_range
        
        # Set default headers
        self.session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
            'Accept-Language': 'en-US,en;q=0.5'
        })
    
    def get_page(self, path):
        """Get a page with polite delay"""
        url = f"{self.base_url}{path}"
        
        # Random delay
        delay = random.uniform(*self.delay_range)
        time.sleep(delay)
        
        try:
            response = self.session.get(url, timeout=30)
            response.raise_for_status()
            return response
        except requests.exceptions.RequestException as e:
            print(f"Error fetching {url}: {e}")
            return None
    
    def scrape_products(self, page_count=5):
        """Scrape products from multiple pages"""
        all_products = []
        
        for page in range(1, page_count + 1):
            response = self.get_page(f"/products?page={page}")
            if response:
                soup = BeautifulSoup(response.content, 'html.parser')
                products = self.extract_products(soup)
                all_products.extend(products)
                print(f"Scraped {len(products)} products from page {page}")
        
        return all_products
    
    def extract_products(self, soup):
        """Extract product data from HTML"""
        products = []
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

# Usage
scraper = WebScraper('https://example.com')
products = scraper.scrape_products(3)
print(f"Total products scraped: {len(products)}")
```

### API Scraper

```python
class APIScraper:
    def __init__(self, base_url, api_key=None):
        self.session = requests.Session()
        self.base_url = base_url
        
        if api_key:
            self.session.headers.update({
                'Authorization': f'Bearer {api_key}',
                'Content-Type': 'application/json'
            })
    
    def get_data(self, endpoint, params=None):
        """Get data from API endpoint"""
        url = f"{self.base_url}{endpoint}"
        
        try:
            response = self.session.get(url, params=params, timeout=30)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            print(f"API request failed: {e}")
            return None
    
    def post_data(self, endpoint, data):
        """Post data to API endpoint"""
        url = f"{self.base_url}{endpoint}"
        
        try:
            response = self.session.post(url, json=data, timeout=30)
            response.raise_for_status()
            return response.json()
        except requests.exceptions.RequestException as e:
            print(f"API request failed: {e}")
            return None

# Usage
api_scraper = APIScraper('https://api.github.com')
user_data = api_scraper.get_data('/users/octocat')
print(user_data)
```

## 11. Best Practices

### Performance Optimization

```python
# Use sessions for multiple requests
session = requests.Session()
session.headers.update({'User-Agent': 'MyBot/1.0'})

for url in urls:
    response = session.get(url)  # Reuses connection

# Use connection pooling
session = requests.Session()
adapter = requests.adapters.HTTPAdapter(
    pool_connections=10,
    pool_maxsize=10
)
session.mount('http://', adapter)
session.mount('https://', adapter)
```

### Memory Management

```python
# Stream large responses
response = requests.get('https://example.com/large-file', stream=True)
with open('large-file.txt', 'wb') as f:
    for chunk in response.iter_content(chunk_size=8192):
        if chunk:
            f.write(chunk)

# Close sessions properly
session = requests.Session()
try:
    response = session.get('https://example.com')
finally:
    session.close()
```

### Error Recovery

```python
def robust_request(url, max_retries=3, backoff_factor=0.3):
    """Make robust request with exponential backoff"""
    session = requests.Session()
    
    for attempt in range(max_retries):
        try:
            response = session.get(url, timeout=30)
            response.raise_for_status()
            return response
        except requests.exceptions.RequestException as e:
            if attempt == max_retries - 1:
                raise
            
            wait_time = backoff_factor * (2 ** attempt)
            time.sleep(wait_time)
    
    return None
```

## 12. Exercises

### Exercise 1: Basic HTTP Client
Create a simple HTTP client that can make GET and POST requests.

### Exercise 2: Session Management
Build a scraper that maintains a session across multiple requests.

### Exercise 3: Error Handling
Implement comprehensive error handling for a web scraper.

### Exercise 4: API Integration
Create a client for a public API with authentication and rate limiting.

## 13. Next Steps

You now understand how to use the Requests library effectively! Before moving to the next module, practice by:

1. Building simple HTTP clients
2. Working with different content types
3. Implementing authentication
4. Handling errors and timeouts

When you're comfortable with these concepts, proceed to [BeautifulSoup for HTML Parsing](07-beautifulsoup-html-parsing.md).

## Key Takeaways

- Requests is the foundation for HTTP communication in Python
- Sessions help maintain state across multiple requests
- Proper error handling makes scrapers robust
- Headers and authentication are crucial for successful scraping
- Timeouts and retries improve reliability
- Always be respectful with request rates

Remember: The Requests library is your gateway to the web! 