# Data Structures in Python for Web Scraping

## Overview

Understanding data structures is crucial for web scraping because you need to efficiently organize, store, and process the data you extract from websites. This module covers the essential data structures and techniques you'll use when working with scraped data.

## Learning Objectives

By the end of this module, you'll be able to:
- Work effectively with lists, dictionaries, and sets
- Use advanced data structures like DataFrames and JSON
- Implement data filtering and transformation
- Handle large datasets efficiently
- Store and retrieve scraped data

## 1. Basic Data Structures Review

### Lists

Lists are ordered, mutable collections that can store different types of data.

```python
# Creating lists
urls = ["https://example.com", "https://site1.com", "https://site2.com"]
mixed_data = [1, "text", 3.14, True, [1, 2, 3]]

# List operations
urls.append("https://site3.com")
urls.extend(["https://site4.com", "https://site5.com"])
urls.insert(0, "https://homepage.com")

# List slicing
first_three = urls[:3]
last_two = urls[-2:]
every_other = urls[::2]

# List comprehension
squares = [x**2 for x in range(5)]
filtered_urls = [url for url in urls if "example" in url]
```

### Dictionaries

Dictionaries store key-value pairs and are perfect for structured data.

```python
# Creating dictionaries
product = {
    "name": "Laptop",
    "price": 999.99,
    "brand": "TechCorp",
    "in_stock": True
}

# Accessing values
name = product["name"]
price = product.get("price", 0)  # Safe access with default

# Adding/modifying items
product["warranty"] = "2 years"
product["price"] = 899.99

# Dictionary comprehension
prices = {item: price for item, price in [("laptop", 999), ("mouse", 25)]}
```

### Sets

Sets store unique, unordered elements and are great for removing duplicates.

```python
# Creating sets
unique_tags = {"python", "web-scraping", "data-science"}
visited_urls = set()

# Set operations
unique_tags.add("automation")
unique_tags.remove("data-science")
unique_tags.discard("nonexistent")  # Safe removal

# Set operations
tags1 = {"python", "web-scraping"}
tags2 = {"python", "machine-learning"}
union = tags1 | tags2
intersection = tags1 & tags2
difference = tags1 - tags2
```

## 2. Advanced Data Structures

### Named Tuples

Named tuples provide a way to create simple classes for storing data.

```python
from collections import namedtuple

# Define a named tuple
Product = namedtuple('Product', ['name', 'price', 'category'])

# Create instances
laptop = Product("Laptop", 999.99, "Electronics")
mouse = Product("Mouse", 25.50, "Accessories")

# Access fields
print(laptop.name)  # "Laptop"
print(laptop.price)  # 999.99

# Convert to dictionary
product_dict = laptop._asdict()
```

### Default Dictionaries

Default dictionaries automatically create default values for missing keys.

```python
from collections import defaultdict

# Count occurrences
word_count = defaultdict(int)
words = ["python", "web", "scraping", "python", "data"]

for word in words:
    word_count[word] += 1

print(word_count)  # defaultdict(<class 'int'>, {'python': 2, 'web': 1, 'scraping': 1, 'data': 1})

# Group by category
products_by_category = defaultdict(list)
products = [
    ("Laptop", "Electronics"),
    ("Mouse", "Accessories"),
    ("Keyboard", "Accessories"),
    ("Monitor", "Electronics")
]

for product, category in products:
    products_by_category[category].append(product)

print(products_by_category)
```

### Counters

Counters are specialized dictionaries for counting hashable objects.

```python
from collections import Counter

# Count elements
tags = ["python", "web-scraping", "python", "data-science", "python"]
tag_counts = Counter(tags)

print(tag_counts)  # Counter({'python': 3, 'web-scraping': 1, 'data-science': 1})

# Most common elements
most_common = tag_counts.most_common(2)
print(most_common)  # [('python', 3), ('web-scraping', 1)]

# Mathematical operations
tags1 = Counter(["python", "web-scraping", "python"])
tags2 = Counter(["python", "data-science"])
combined = tags1 + tags2
print(combined)  # Counter({'python': 3, 'web-scraping': 1, 'data-science': 1})
```

## 3. Working with JSON

### JSON Basics

JSON (JavaScript Object Notation) is a lightweight data format commonly used for web APIs and data storage.

```python
import json

# Python to JSON
data = {
    "name": "John Doe",
    "age": 30,
    "skills": ["Python", "Web Scraping", "Data Analysis"],
    "active": True
}

json_string = json.dumps(data, indent=2)
print(json_string)

# JSON to Python
parsed_data = json.loads(json_string)
print(parsed_data["name"])

# Working with files
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

with open("data.json", "r") as f:
    loaded_data = json.load(f)
```

### JSON in Web Scraping

```python
import json
import requests

# Scraping JSON API
response = requests.get("https://api.example.com/products")
products = response.json()

# Processing JSON data
for product in products:
    print(f"Name: {product['name']}, Price: ${product['price']}")

# Saving scraped data as JSON
scraped_data = []
for product in products:
    scraped_data.append({
        "name": product["name"],
        "price": product["price"],
        "category": product.get("category", "Unknown")
    })

with open("scraped_products.json", "w") as f:
    json.dump(scraped_data, f, indent=2)
```

## 4. Pandas DataFrames

### Introduction to Pandas

Pandas is a powerful library for data manipulation and analysis, perfect for working with scraped data.

```python
import pandas as pd

# Creating DataFrames
data = {
    "name": ["Laptop", "Mouse", "Keyboard", "Monitor"],
    "price": [999.99, 25.50, 75.00, 299.99],
    "category": ["Electronics", "Accessories", "Accessories", "Electronics"],
    "in_stock": [True, True, False, True]
}

df = pd.DataFrame(data)
print(df)

# Reading from different sources
df_csv = pd.read_csv("products.csv")
df_json = pd.read_json("products.json")
df_excel = pd.read_excel("products.xlsx")
```

### DataFrame Operations

```python
# Basic operations
print(df.head())  # First 5 rows
print(df.tail())  # Last 5 rows
print(df.info())  # DataFrame info
print(df.describe())  # Statistical summary

# Selecting data
print(df["name"])  # Single column
print(df[["name", "price"]])  # Multiple columns
print(df.iloc[0:3])  # Rows by index
print(df.loc[df["price"] > 50])  # Filtered rows

# Filtering
expensive_products = df[df["price"] > 100]
electronics = df[df["category"] == "Electronics"]
in_stock = df[df["in_stock"] == True]

# Sorting
sorted_by_price = df.sort_values("price", ascending=False)
sorted_by_category = df.sort_values(["category", "price"])
```

### Data Cleaning and Transformation

```python
# Handling missing values
df["price"].fillna(0, inplace=True)
df.dropna(inplace=True)

# Data type conversion
df["price"] = df["price"].astype(float)
df["in_stock"] = df["in_stock"].astype(bool)

# String operations
df["name_upper"] = df["name"].str.upper()
df["name_length"] = df["name"].str.len()

# Grouping and aggregation
category_stats = df.groupby("category")["price"].agg(["mean", "count", "sum"])
print(category_stats)

# Pivot tables
pivot_table = df.pivot_table(
    values="price",
    index="category",
    columns="in_stock",
    aggfunc="mean"
)
print(pivot_table)
```

## 5. Data Processing Patterns

### Data Extraction Patterns

```python
# Extracting data from HTML
def extract_product_data(html_elements):
    """Extract product data from HTML elements"""
    products = []
    
    for element in html_elements:
        product = {
            "name": element.find("h3").text.strip(),
            "price": float(element.find("span", class_="price").text.replace("$", "")),
            "rating": element.find("div", class_="rating").text.strip(),
            "url": element.find("a")["href"]
        }
        products.append(product)
    
    return products

# Processing scraped data
def process_scraped_data(raw_data):
    """Process and clean scraped data"""
    processed_data = []
    
    for item in raw_data:
        # Clean and validate data
        if item.get("name") and item.get("price"):
            processed_item = {
                "name": item["name"].strip(),
                "price": float(item["price"]),
                "category": item.get("category", "Unknown"),
                "timestamp": pd.Timestamp.now()
            }
            processed_data.append(processed_item)
    
    return processed_data
```

### Data Filtering and Validation

```python
def validate_product_data(product):
    """Validate product data"""
    errors = []
    
    if not product.get("name"):
        errors.append("Missing product name")
    
    if not product.get("price"):
        errors.append("Missing price")
    elif not isinstance(product["price"], (int, float)):
        errors.append("Invalid price format")
    
    if product.get("price", 0) < 0:
        errors.append("Negative price")
    
    return len(errors) == 0, errors

def filter_products(products, min_price=0, max_price=float('inf'), category=None):
    """Filter products based on criteria"""
    filtered = []
    
    for product in products:
        price = product.get("price", 0)
        product_category = product.get("category", "")
        
        if (min_price <= price <= max_price and 
            (category is None or category.lower() in product_category.lower())):
            filtered.append(product)
    
    return filtered
```

## 6. Data Storage and Retrieval

### CSV Files

```python
import csv

# Writing CSV
def save_to_csv(data, filename):
    """Save data to CSV file"""
    if not data:
        return
    
    fieldnames = data[0].keys()
    
    with open(filename, 'w', newline='', encoding='utf-8') as csvfile:
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(data)

# Reading CSV
def read_from_csv(filename):
    """Read data from CSV file"""
    data = []
    
    with open(filename, 'r', encoding='utf-8') as csvfile:
        reader = csv.DictReader(csvfile)
        for row in reader:
            data.append(row)
    
    return data

# Usage
products = [
    {"name": "Laptop", "price": "999.99", "category": "Electronics"},
    {"name": "Mouse", "price": "25.50", "category": "Accessories"}
]

save_to_csv(products, "products.csv")
loaded_products = read_from_csv("products.csv")
```

### Database Operations (SQLite)

```python
import sqlite3

# Create database and table
def setup_database():
    """Setup SQLite database for storing scraped data"""
    conn = sqlite3.connect("scraped_data.db")
    cursor = conn.cursor()
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS products (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            price REAL,
            category TEXT,
            url TEXT,
            scraped_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    ''')
    
    conn.commit()
    conn.close()

# Insert data
def insert_products(products):
    """Insert products into database"""
    conn = sqlite3.connect("scraped_data.db")
    cursor = conn.cursor()
    
    for product in products:
        cursor.execute('''
            INSERT INTO products (name, price, category, url)
            VALUES (?, ?, ?, ?)
        ''', (product["name"], product["price"], 
              product.get("category"), product.get("url")))
    
    conn.commit()
    conn.close()

# Query data
def get_products_by_category(category):
    """Get products by category"""
    conn = sqlite3.connect("scraped_data.db")
    cursor = conn.cursor()
    
    cursor.execute('''
        SELECT * FROM products WHERE category = ?
    ''', (category,))
    
    products = cursor.fetchall()
    conn.close()
    
    return products
```

## 7. Data Analysis and Visualization

### Basic Analysis

```python
import pandas as pd
import matplotlib.pyplot as plt

# Load data
df = pd.read_csv("scraped_products.csv")

# Basic statistics
print("Price Statistics:")
print(df["price"].describe())

print("\nProducts by Category:")
print(df["category"].value_counts())

# Price analysis by category
category_prices = df.groupby("category")["price"].agg(["mean", "min", "max"])
print("\nPrice Analysis by Category:")
print(category_prices)

# Visualization
plt.figure(figsize=(10, 6))

# Price distribution
plt.subplot(1, 2, 1)
df["price"].hist(bins=20)
plt.title("Price Distribution")
plt.xlabel("Price")
plt.ylabel("Frequency")

# Category counts
plt.subplot(1, 2, 2)
df["category"].value_counts().plot(kind="bar")
plt.title("Products by Category")
plt.xlabel("Category")
plt.ylabel("Count")

plt.tight_layout()
plt.show()
```

### Advanced Analysis

```python
# Time series analysis (if you have timestamps)
df["scraped_at"] = pd.to_datetime(df["scraped_at"])
df.set_index("scraped_at", inplace=True)

# Price trends over time
daily_prices = df.resample("D")["price"].mean()
daily_prices.plot(title="Average Daily Prices")

# Correlation analysis
numeric_columns = df.select_dtypes(include=[np.number])
correlation_matrix = numeric_columns.corr()
print("Correlation Matrix:")
print(correlation_matrix)
```

## 8. Performance Optimization

### Memory Efficiency

```python
# Using generators for large datasets
def process_large_dataset(file_path):
    """Process large dataset using generator"""
    with open(file_path, 'r') as file:
        for line in file:
            yield json.loads(line)

# Usage
for item in process_large_dataset("large_data.json"):
    # Process each item individually
    process_item(item)

# Chunked processing with pandas
chunk_size = 1000
for chunk in pd.read_csv("large_file.csv", chunksize=chunk_size):
    # Process each chunk
    process_chunk(chunk)
```

### Data Structure Optimization

```python
# Using sets for fast lookups
visited_urls = set()
urls_to_visit = ["url1", "url2", "url3"]

for url in urls_to_visit:
    if url not in visited_urls:  # O(1) lookup
        visited_urls.add(url)
        # Process URL

# Using defaultdict for counting
from collections import defaultdict
category_counts = defaultdict(int)

for product in products:
    category_counts[product["category"]] += 1
```

## 9. Practical Examples

### Complete Data Processing Pipeline

```python
import pandas as pd
import json
from datetime import datetime

class DataProcessor:
    def __init__(self):
        self.raw_data = []
        self.processed_data = []
    
    def add_raw_data(self, data):
        """Add raw scraped data"""
        self.raw_data.extend(data)
    
    def process_data(self):
        """Process and clean raw data"""
        for item in self.raw_data:
            processed_item = self._clean_item(item)
            if processed_item:
                self.processed_data.append(processed_item)
    
    def _clean_item(self, item):
        """Clean individual data item"""
        try:
            return {
                "name": str(item.get("name", "")).strip(),
                "price": float(item.get("price", 0)),
                "category": str(item.get("category", "Unknown")).strip(),
                "url": str(item.get("url", "")).strip(),
                "scraped_at": datetime.now().isoformat()
            }
        except (ValueError, TypeError):
            return None
    
    def save_to_csv(self, filename):
        """Save processed data to CSV"""
        df = pd.DataFrame(self.processed_data)
        df.to_csv(filename, index=False)
    
    def save_to_json(self, filename):
        """Save processed data to JSON"""
        with open(filename, 'w') as f:
            json.dump(self.processed_data, f, indent=2)
    
    def get_statistics(self):
        """Get data statistics"""
        df = pd.DataFrame(self.processed_data)
        return {
            "total_items": len(df),
            "price_range": (df["price"].min(), df["price"].max()),
            "categories": df["category"].value_counts().to_dict(),
            "average_price": df["price"].mean()
        }

# Usage
processor = DataProcessor()

# Add scraped data
scraped_data = [
    {"name": "Laptop", "price": "999.99", "category": "Electronics"},
    {"name": "Mouse", "price": "25.50", "category": "Accessories"}
]

processor.add_raw_data(scraped_data)
processor.process_data()
processor.save_to_csv("processed_products.csv")
processor.save_to_json("processed_products.json")

stats = processor.get_statistics()
print("Statistics:", stats)
```

## 10. Exercises

### Exercise 1: Data Cleaning
Create a function that cleans and validates scraped product data.

### Exercise 2: Data Analysis
Build a data analysis pipeline that processes scraped data and generates insights.

### Exercise 3: Data Storage
Implement a system to store and retrieve scraped data using different formats.

### Exercise 4: Performance Optimization
Optimize a data processing script for handling large datasets efficiently.

## 11. Next Steps

You now understand data structures and processing! Before moving to the next module, practice by:

1. Working with different data formats (CSV, JSON, databases)
2. Building data processing pipelines
3. Analyzing and visualizing data
4. Optimizing data operations

When you're comfortable with these concepts, proceed to [Introduction to Web Scraping](05-introduction-web-scraping.md).

## Key Takeaways

- Choose the right data structure for your specific use case
- Pandas is powerful for data manipulation and analysis
- JSON is the standard format for web APIs and data exchange
- Proper data cleaning and validation are essential
- Consider performance when working with large datasets
- Data storage and retrieval are crucial for persistent scraping

Remember: Good data organization leads to better analysis and insights! 