# HTTP & Web Protocols for Web Scraping

## Overview

Understanding HTTP (HyperText Transfer Protocol) and web protocols is fundamental to web scraping. This knowledge helps you make proper requests, handle responses, and understand the communication between your scraper and web servers.

## Learning Objectives

By the end of this module, you'll be able to:
- Understand HTTP request/response cycle
- Identify different HTTP methods and their purposes
- Handle HTTP status codes and headers
- Work with cookies and sessions
- Understand authentication mechanisms
- Deal with rate limiting and robots.txt

## 1. What is HTTP?

### Definition

HTTP (HyperText Transfer Protocol) is the foundation of data communication on the World Wide Web. It's a request-response protocol that allows clients (like your web scraper) to communicate with servers.

### How HTTP Works

```
Client (Your Scraper) ←→ HTTP ←→ Server (Website)
```

1. **Client sends a request** to the server
2. **Server processes the request** and generates a response
3. **Server sends the response** back to the client

## 2. HTTP Request Structure

### Basic HTTP Request

```
GET /page.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive

[Request Body - usually empty for GET requests]
```

### Request Components

#### Request Line
```
METHOD /path HTTP/version
```

#### Headers
```
Header-Name: Header-Value
```

#### Body (Optional)
```
[Data for POST/PUT requests]
```

## 3. HTTP Methods

### GET
- **Purpose**: Retrieve data from server
- **Body**: Usually empty
- **Idempotent**: Yes (safe to repeat)
- **Use case**: Fetching web pages, getting data

```http
GET /products HTTP/1.1
Host: example.com
```

### POST
- **Purpose**: Submit data to server
- **Body**: Contains data
- **Idempotent**: No
- **Use case**: Submitting forms, creating resources

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=john&password=secret123
```

### PUT
- **Purpose**: Update existing resource
- **Body**: Contains updated data
- **Idempotent**: Yes
- **Use case**: Updating user profiles, modifying data

### DELETE
- **Purpose**: Remove resource
- **Body**: Usually empty
- **Idempotent**: Yes
- **Use case**: Deleting accounts, removing data

### HEAD
- **Purpose**: Get headers only (no body)
- **Use case**: Checking if resource exists, getting metadata

### OPTIONS
- **Purpose**: Get available methods for resource
- **Use case**: Checking server capabilities

## 4. HTTP Response Structure

### Basic HTTP Response

```
HTTP/1.1 200 OK
Date: Mon, 15 Jan 2024 10:30:00 GMT
Server: Apache/2.4.41
Content-Type: text/html; charset=UTF-8
Content-Length: 1234
Cache-Control: max-age=3600

<!DOCTYPE html>
<html>
<head>
    <title>Example Page</title>
</head>
<body>
    <h1>Hello World</h1>
</body>
</html>
```

### Response Components

#### Status Line
```
HTTP/version STATUS_CODE STATUS_MESSAGE
```

#### Headers
```
Header-Name: Header-Value
```

#### Body
```
[Response content - HTML, JSON, etc.]
```

## 5. HTTP Status Codes

### 1xx - Informational
- **100 Continue**: Request received, continue
- **101 Switching Protocols**: Server switching protocols

### 2xx - Success
- **200 OK**: Request successful
- **201 Created**: Resource created successfully
- **204 No Content**: Request successful, no content returned

### 3xx - Redirection
- **301 Moved Permanently**: Resource moved permanently
- **302 Found**: Resource temporarily moved
- **304 Not Modified**: Resource not modified (use cached version)

### 4xx - Client Errors
- **400 Bad Request**: Invalid request syntax
- **401 Unauthorized**: Authentication required
- **403 Forbidden**: Access denied
- **404 Not Found**: Resource not found
- **429 Too Many Requests**: Rate limit exceeded

### 5xx - Server Errors
- **500 Internal Server Error**: Server error
- **502 Bad Gateway**: Gateway error
- **503 Service Unavailable**: Server temporarily unavailable

## 6. Important HTTP Headers

### Request Headers

#### User-Agent
```
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
```
- Identifies your client to the server
- Important for avoiding detection as a bot

#### Accept
```
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
```
- Tells server what content types you can handle

#### Authorization
```
Authorization: Bearer token123
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```
- Used for authentication

#### Cookie
```
Cookie: session=abc123; user=john
```
- Sends cookies to server

#### Referer
```
Referer: https://example.com/previous-page
```
- Indicates where request came from

### Response Headers

#### Content-Type
```
Content-Type: text/html; charset=UTF-8
Content-Type: application/json
```
- Specifies response content type

#### Set-Cookie
```
Set-Cookie: session=abc123; Path=/; HttpOnly
```
- Server sets cookies for client

#### Location
```
Location: https://new-example.com/redirect
```
- Used for redirects

#### Cache-Control
```
Cache-Control: max-age=3600
Cache-Control: no-cache
```
- Controls caching behavior

## 7. Cookies and Sessions

### What are Cookies?

Cookies are small pieces of data stored by the browser and sent with each request to the same domain.

### Types of Cookies

#### Session Cookies
- Temporary, deleted when browser closes
- Used for session management

#### Persistent Cookies
- Stored on disk, survive browser restarts
- Used for preferences, authentication

### Working with Cookies in Python

```python
import requests

# Create a session to maintain cookies
session = requests.Session()

# First request - server sets cookies
response = session.get('https://example.com/login')
print(response.cookies)

# Second request - cookies automatically sent
response = session.get('https://example.com/dashboard')
print(response.cookies)

# Manual cookie handling
cookies = {
    'session_id': 'abc123',
    'user_pref': 'dark_mode'
}
response = requests.get('https://example.com', cookies=cookies)
```

## 8. Authentication Methods

### Basic Authentication

```python
import requests
from requests.auth import HTTPBasicAuth

# Method 1: Using auth parameter
response = requests.get('https://api.example.com/data', 
                       auth=HTTPBasicAuth('username', 'password'))

# Method 2: Using headers
headers = {
    'Authorization': 'Basic dXNlcm5hbWU6cGFzc3dvcmQ='
}
response = requests.get('https://api.example.com/data', headers=headers)
```

### Bearer Token Authentication

```python
headers = {
    'Authorization': 'Bearer your_token_here'
}
response = requests.get('https://api.example.com/data', headers=headers)
```

### Form-based Authentication

```python
# Login form submission
login_data = {
    'username': 'your_username',
    'password': 'your_password',
    'submit': 'Login'
}

session = requests.Session()
response = session.post('https://example.com/login', data=login_data)

# Now session contains authentication cookies
response = session.get('https://example.com/protected-page')
```

## 9. Rate Limiting and Politeness

### What is Rate Limiting?

Rate limiting is a technique used by servers to control the number of requests a client can make in a given time period.

### Common Rate Limiting Headers

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1642233600
Retry-After: 60
```

### Implementing Polite Scraping

```python
import time
import random

def polite_request(url, delay_range=(1, 3)):
    """Make a polite request with random delay"""
    # Random delay between requests
    delay = random.uniform(*delay_range)
    time.sleep(delay)
    
    response = requests.get(url)
    
    # Check for rate limiting
    if response.status_code == 429:
        retry_after = int(response.headers.get('Retry-After', 60))
        print(f"Rate limited. Waiting {retry_after} seconds...")
        time.sleep(retry_after)
        return polite_request(url, delay_range)
    
    return response

# Usage
urls = ['https://example.com/page1', 'https://example.com/page2']
for url in urls:
    response = polite_request(url)
    print(f"Scraped: {url}")
```

## 10. Robots.txt

### What is robots.txt?

Robots.txt is a text file that tells web crawlers which parts of a website they can and cannot access.

### Example robots.txt

```
User-agent: *
Disallow: /admin/
Disallow: /private/
Allow: /public/

User-agent: Googlebot
Allow: /

Crawl-delay: 10
```

### Parsing robots.txt

```python
import urllib.robotparser

def check_robots_txt(domain, path):
    """Check if a path is allowed by robots.txt"""
    rp = urllib.robotparser.RobotFileParser()
    rp.set_url(f"https://{domain}/robots.txt")
    rp.read()
    
    return rp.can_fetch("*", f"https://{domain}{path}")

# Usage
domain = "example.com"
path = "/products"
if check_robots_txt(domain, path):
    print(f"Allowed to scrape: {path}")
else:
    print(f"Not allowed to scrape: {path}")
```

## 11. HTTPS and Security

### What is HTTPS?

HTTPS (HTTP Secure) is HTTP over SSL/TLS encryption, providing secure communication.

### SSL/TLS Certificates

```python
import requests

# Disable SSL verification (not recommended for production)
response = requests.get('https://example.com', verify=False)

# Custom SSL certificate
response = requests.get('https://example.com', 
                       cert=('/path/to/cert.pem', '/path/to/key.pem'))
```

### Handling SSL Errors

```python
import requests
from requests.exceptions import SSLError

try:
    response = requests.get('https://example.com')
except SSLError as e:
    print(f"SSL Error: {e}")
    # Handle SSL error appropriately
```

## 12. Common HTTP Issues in Web Scraping

### Connection Errors

```python
import requests
from requests.exceptions import ConnectionError, Timeout

try:
    response = requests.get('https://example.com', timeout=10)
except ConnectionError:
    print("Connection failed")
except Timeout:
    print("Request timed out")
```

### Redirect Handling

```python
# Allow redirects (default behavior)
response = requests.get('https://example.com', allow_redirects=True)

# Disable redirects
response = requests.get('https://example.com', allow_redirects=False)

# Check if response was redirected
if response.history:
    print(f"Request was redirected from {response.history[0].url}")
    print(f"Final URL: {response.url}")
```

### Custom Headers

```python
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

## 13. Practical Examples

### Complete Web Scraping Session

```python
import requests
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
            response = self.session.get(url, timeout=10)
            response.raise_for_status()
            return response
        except requests.exceptions.RequestException as e:
            print(f"Error fetching {url}: {e}")
            return None
    
    def login(self, username, password):
        """Login to the website"""
        login_data = {
            'username': username,
            'password': password
        }
        
        response = self.session.post(f"{self.base_url}/login", data=login_data)
        return response.status_code == 200

# Usage
scraper = WebScraper('https://example.com')

# Login
if scraper.login('username', 'password'):
    print("Login successful")
    
    # Scrape pages
    pages = ['/page1', '/page2', '/page3']
    for page in pages:
        response = scraper.get_page(page)
        if response:
            print(f"Scraped: {page}")
```

## 14. Exercises

### Exercise 1: HTTP Request Analysis
Use browser developer tools to analyze HTTP requests and responses for a website.

### Exercise 2: Status Code Handling
Create a function that handles different HTTP status codes appropriately.

### Exercise 3: Session Management
Build a scraper that maintains a session across multiple requests.

### Exercise 4: Rate Limiting Implementation
Implement a rate limiting mechanism that respects server limits.

## 15. Next Steps

You now understand HTTP and web protocols! Before moving to the next module, practice by:

1. Analyzing HTTP requests in browser developer tools
2. Building simple HTTP clients
3. Handling different response types
4. Implementing proper error handling

When you're comfortable with these concepts, proceed to [Data Structures in Python](04-data-structures-python.md).

## Key Takeaways

- HTTP is the foundation of web communication
- Understanding request/response structure is crucial for scraping
- Proper headers and authentication are essential
- Rate limiting and politeness are important for sustainable scraping
- Sessions help maintain state across requests
- Error handling makes scrapers robust

Remember: Respectful scraping practices ensure long-term success! 