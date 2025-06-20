# Python Basics for Web Scraping

## Overview

Before diving into web scraping, you need a solid foundation in Python programming. This module covers the essential Python concepts you'll use throughout your web scraping journey.

## Learning Objectives

By the end of this module, you'll be able to:
- Write and run Python scripts
- Work with variables, data types, and operators
- Use control structures (if/else, loops)
- Define and use functions
- Handle exceptions
- Work with files and modules

## 1. Getting Started with Python

### Installation

First, ensure Python is installed on your system:

```bash
# Check Python version
python --version
# or
python3 --version
```

### Your First Python Script

```python
# hello_world.py
print("Hello, Web Scraping World!")
```

### Running Python Scripts

```bash
python hello_world.py
# or
python3 hello_world.py
```

## 2. Variables and Data Types

### Basic Data Types

```python
# Strings - for text data
website_url = "https://example.com"
title = "Web Scraping Tutorial"

# Numbers
page_count = 10
price = 29.99

# Booleans
is_active = True
has_content = False

# Lists - ordered collections
urls = ["https://site1.com", "https://site2.com", "https://site3.com"]
prices = [10.99, 24.99, 15.50]

# Dictionaries - key-value pairs
page_data = {
    "title": "Sample Page",
    "url": "https://example.com",
    "content": "This is the page content"
}

# Sets - unique collections
unique_tags = {"h1", "h2", "p", "div", "span"}
```

### Type Checking and Conversion

```python
# Check data type
print(type(website_url))  # <class 'str'>
print(type(page_count))   # <class 'int'>

# Type conversion
number_string = "42"
number = int(number_string)
float_number = float(number_string)

# String formatting
name = "Python"
version = 3.9
print(f"Using {name} version {version}")
```

## 3. Control Structures

### Conditional Statements

```python
# Basic if/else
status_code = 200

if status_code == 200:
    print("Request successful!")
elif status_code == 404:
    print("Page not found")
else:
    print(f"Error: {status_code}")

# Multiple conditions
is_https = True
has_content = True

if is_https and has_content:
    print("Secure page with content")
elif is_https:
    print("Secure page, no content")
else:
    print("Not secure")
```

### Loops

```python
# For loop with range
for i in range(5):
    print(f"Processing page {i + 1}")

# For loop with list
urls = ["https://site1.com", "https://site2.com", "https://site3.com"]
for url in urls:
    print(f"Scraping: {url}")

# While loop
page = 1
while page <= 10:
    print(f"Processing page {page}")
    page += 1

# Loop with enumerate
for index, url in enumerate(urls, 1):
    print(f"{index}. {url}")

# List comprehension
squares = [x**2 for x in range(5)]  # [0, 1, 4, 9, 16]
```

## 4. Functions

### Basic Functions

```python
def greet(name):
    """Simple greeting function"""
    return f"Hello, {name}!"

# Function call
message = greet("Web Scraper")
print(message)

def scrape_page(url, timeout=30):
    """Scrape a single page with optional timeout"""
    print(f"Scraping {url} with {timeout}s timeout")
    # Placeholder for actual scraping logic
    return {"url": url, "status": "success"}

# Function with default parameters
result1 = scrape_page("https://example.com")
result2 = scrape_page("https://example.com", 60)
```

### Lambda Functions

```python
# Simple lambda function
square = lambda x: x**2
print(square(5))  # 25

# Lambda with filter
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
print(even_numbers)  # [2, 4, 6, 8, 10]
```

## 5. Working with Lists and Dictionaries

### List Operations

```python
# Creating and modifying lists
urls = []
urls.append("https://example.com")
urls.extend(["https://site1.com", "https://site2.com"])
urls.insert(0, "https://homepage.com")

# List slicing
first_three = urls[:3]
last_two = urls[-2:]

# List methods
urls.remove("https://example.com")
popped_url = urls.pop()
urls.sort()
urls.reverse()
```

### Dictionary Operations

```python
# Creating and accessing dictionaries
page_info = {
    "title": "Sample Page",
    "url": "https://example.com",
    "content_length": 1500
}

# Accessing values
title = page_info["title"]
content_length = page_info.get("content_length", 0)  # Safe access

# Adding and modifying
page_info["author"] = "John Doe"
page_info["title"] = "Updated Title"

# Dictionary methods
keys = list(page_info.keys())
values = list(page_info.values())
items = list(page_info.items())

# Dictionary comprehension
squares_dict = {x: x**2 for x in range(5)}
```

## 6. Exception Handling

### Try-Except Blocks

```python
def safe_divide(a, b):
    try:
        result = a / b
        return result
    except ZeroDivisionError:
        print("Error: Division by zero!")
        return None
    except TypeError:
        print("Error: Invalid data types!")
        return None

# Multiple exceptions
def process_url(url):
    try:
        # Simulate URL processing
        if not url.startswith("http"):
            raise ValueError("Invalid URL format")
        return f"Processed: {url}"
    except ValueError as e:
        print(f"Value error: {e}")
    except Exception as e:
        print(f"Unexpected error: {e}")
    finally:
        print("Processing completed")

# Using with statement (for file handling)
try:
    with open("data.txt", "r") as file:
        content = file.read()
except FileNotFoundError:
    print("File not found!")
```

## 7. File Operations

### Reading and Writing Files

```python
# Writing to a file
with open("scraped_data.txt", "w") as file:
    file.write("Title: Sample Page\n")
    file.write("URL: https://example.com\n")
    file.write("Content: This is the page content\n")

# Reading from a file
with open("scraped_data.txt", "r") as file:
    content = file.read()
    print(content)

# Reading line by line
with open("urls.txt", "r") as file:
    for line in file:
        url = line.strip()
        print(f"Processing: {url}")

# Working with CSV
import csv

# Writing CSV
with open("data.csv", "w", newline="") as file:
    writer = csv.writer(file)
    writer.writerow(["Title", "URL", "Content"])
    writer.writerow(["Page 1", "https://site1.com", "Content 1"])
    writer.writerow(["Page 2", "https://site2.com", "Content 2"])

# Reading CSV
with open("data.csv", "r") as file:
    reader = csv.DictReader(file)
    for row in reader:
        print(f"Title: {row['Title']}, URL: {row['URL']}")
```

## 8. Modules and Imports

### Using Built-in Modules

```python
import os
import sys
import time
import random
import json

# OS operations
current_dir = os.getcwd()
files = os.listdir(current_dir)

# Time operations
start_time = time.time()
time.sleep(1)  # Wait for 1 second
end_time = time.time()
print(f"Elapsed time: {end_time - start_time:.2f} seconds")

# Random operations
random_number = random.randint(1, 100)
random_choice = random.choice(["option1", "option2", "option3"])

# JSON operations
data = {
    "title": "Sample Page",
    "url": "https://example.com",
    "tags": ["web", "scraping", "python"]
}

# Convert to JSON string
json_string = json.dumps(data, indent=2)
print(json_string)

# Parse JSON string
parsed_data = json.loads(json_string)
print(parsed_data["title"])
```

## 9. Practical Examples for Web Scraping

### URL Processing

```python
def process_urls(url_list):
    """Process a list of URLs and extract domain names"""
    domains = []
    for url in url_list:
        if url.startswith("http"):
            # Simple domain extraction
            domain = url.split("//")[1].split("/")[0]
            domains.append(domain)
    return domains

urls = [
    "https://example.com/page1",
    "https://google.com/search",
    "https://github.com/user/repo"
]

domains = process_urls(urls)
print(domains)  # ['example.com', 'google.com', 'github.com']
```

### Data Validation

```python
def validate_page_data(data):
    """Validate scraped page data"""
    required_fields = ["title", "url", "content"]
    
    for field in required_fields:
        if field not in data:
            return False, f"Missing required field: {field}"
    
    if not data["url"].startswith("http"):
        return False, "Invalid URL format"
    
    if len(data["content"]) < 10:
        return False, "Content too short"
    
    return True, "Data is valid"

# Test validation
test_data = {
    "title": "Sample Page",
    "url": "https://example.com",
    "content": "This is a sample page content"
}

is_valid, message = validate_page_data(test_data)
print(f"Validation result: {is_valid}, Message: {message}")
```

## 10. Exercises

### Exercise 1: URL List Processor
Create a function that takes a list of URLs and returns only those that are valid (start with http/https).

### Exercise 2: Data Aggregator
Write a function that takes a list of dictionaries (each containing page data) and returns summary statistics.

### Exercise 3: File Logger
Create a logging function that writes scraping results to a file with timestamps.

### Exercise 4: Error Handler
Build a robust error handling system for processing multiple URLs.

## 11. Next Steps

You now have a solid foundation in Python! Before moving to the next module, practice these concepts by:

1. Writing small scripts to process data
2. Working with different data types
3. Handling files and exceptions
4. Building simple functions

When you're comfortable with these concepts, proceed to [HTML & CSS Fundamentals](02-html-css-fundamentals.md).

## Key Takeaways

- Python is the foundation for web scraping
- Understanding data types and structures is crucial
- Exception handling makes your code robust
- File operations are essential for saving scraped data
- Functions help organize and reuse code

Remember: Practice makes perfect! Experiment with the code examples and build your own small projects to reinforce these concepts. 