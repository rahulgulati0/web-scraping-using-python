# HTML & CSS Fundamentals for Web Scraping

## Overview

Understanding HTML and CSS is crucial for web scraping because you need to know how web pages are structured to extract the right data. This module covers the essential concepts you'll encounter when scraping websites.

## Learning Objectives

By the end of this module, you'll be able to:
- Understand HTML document structure
- Identify HTML elements and their purposes
- Use CSS selectors for targeting elements
- Analyze web page structure for scraping
- Understand the relationship between HTML and CSS

## 1. HTML Basics

### What is HTML?

HTML (HyperText Markup Language) is the standard markup language for creating web pages. It describes the structure of a web page using a system of elements and tags.

### Basic HTML Document Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sample Web Page</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <header>
        <h1>Welcome to Our Website</h1>
        <nav>
            <ul>
                <li><a href="#home">Home</a></li>
                <li><a href="#about">About</a></li>
                <li><a href="#contact">Contact</a></li>
            </ul>
        </nav>
    </header>
    
    <main>
        <article>
            <h2>Main Content</h2>
            <p>This is the main content of the page.</p>
        </article>
    </main>
    
    <footer>
        <p>&copy; 2024 Sample Website</p>
    </footer>
</body>
</html>
```

### Key HTML Elements for Web Scraping

#### Headers (h1-h6)
```html
<h1>Main Title</h1>
<h2>Section Title</h2>
<h3>Subsection Title</h3>
<h4>Minor Section</h4>
<h5>Small Section</h5>
<h6>Smallest Section</h6>
```

#### Paragraphs and Text
```html
<p>This is a paragraph of text.</p>
<span>Inline text element</span>
<strong>Bold text</strong>
<em>Italic text</em>
<br> <!-- Line break -->
<hr> <!-- Horizontal rule -->
```

#### Lists
```html
<!-- Unordered list -->
<ul>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
</ul>

<!-- Ordered list -->
<ol>
    <li>First step</li>
    <li>Second step</li>
    <li>Third step</li>
</ol>

<!-- Definition list -->
<dl>
    <dt>Term</dt>
    <dd>Definition</dd>
</dl>
```

#### Links and Images
```html
<!-- Links -->
<a href="https://example.com">Visit Example</a>
<a href="#section">Jump to section</a>
<a href="mailto:email@example.com">Send email</a>

<!-- Images -->
<img src="image.jpg" alt="Description of image">
<img src="https://example.com/image.png" alt="Remote image">
```

#### Tables
```html
<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Age</th>
            <th>City</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>John Doe</td>
            <td>30</td>
            <td>New York</td>
        </tr>
        <tr>
            <td>Jane Smith</td>
            <td>25</td>
            <td>Los Angeles</td>
        </tr>
    </tbody>
</table>
```

#### Forms
```html
<form action="/submit" method="POST">
    <label for="name">Name:</label>
    <input type="text" id="name" name="name" required>
    
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    
    <label for="message">Message:</label>
    <textarea id="message" name="message" rows="4"></textarea>
    
    <button type="submit">Submit</button>
</form>
```

## 2. HTML Attributes

### Common Attributes

```html
<!-- ID attribute (unique identifier) -->
<div id="main-content">Content here</div>

<!-- Class attribute (for styling and grouping) -->
<p class="highlight important">Important text</p>

<!-- Data attributes (custom data) -->
<div data-product-id="12345" data-category="electronics">
    Product information
</div>

<!-- Style attribute (inline CSS) -->
<p style="color: red; font-size: 18px;">Styled text</p>

<!-- Title attribute (tooltip) -->
<a href="#" title="Click for more information">Link</a>
```

### Semantic HTML Elements

```html
<!-- Semantic structure -->
<header>Header content</header>
<nav>Navigation menu</nav>
<main>Main content</main>
<article>Article content</article>
<section>Section content</section>
<aside>Sidebar content</aside>
<footer>Footer content</footer>

<!-- Content elements -->
<figure>
    <img src="image.jpg" alt="Description">
    <figcaption>Image caption</figcaption>
</figure>

<blockquote>
    <p>Quoted text</p>
    <cite>— Author Name</cite>
</blockquote>
```

## 3. CSS Fundamentals

### What is CSS?

CSS (Cascading Style Sheets) is used to style and layout web pages. It controls the visual presentation of HTML elements.

### CSS Syntax

```css
selector {
    property: value;
    another-property: value;
}
```

### CSS Selectors (Crucial for Web Scraping)

#### Element Selectors
```css
/* Select all paragraphs */
p {
    color: blue;
}

/* Select all headings */
h1, h2, h3 {
    font-weight: bold;
}
```

#### Class Selectors
```css
/* Select elements with class "highlight" */
.highlight {
    background-color: yellow;
}

/* Select elements with multiple classes */
.important.urgent {
    color: red;
}
```

#### ID Selectors
```css
/* Select element with id "main-content" */
#main-content {
    width: 800px;
    margin: 0 auto;
}
```

#### Attribute Selectors
```css
/* Select elements with specific attributes */
[data-product-id] {
    border: 1px solid #ccc;
}

[href*="example.com"] {
    color: green;
}

[class^="btn-"] {
    padding: 10px;
}
```

#### Descendant and Child Selectors
```css
/* Descendant selector (any level) */
div p {
    margin-bottom: 10px;
}

/* Child selector (direct child only) */
div > p {
    font-weight: bold;
}
```

#### Pseudo-classes
```css
/* Link states */
a:link { color: blue; }
a:visited { color: purple; }
a:hover { color: red; }
a:active { color: orange; }

/* Form elements */
input:focus {
    border-color: blue;
}

/* Position-based */
p:first-child { font-size: 18px; }
p:last-child { margin-bottom: 0; }
li:nth-child(odd) { background-color: #f0f0f0; }
```

## 4. CSS Box Model

### Understanding the Box Model

Every HTML element is treated as a box with the following properties:

```css
.box {
    /* Content */
    width: 200px;
    height: 100px;
    
    /* Padding (inner space) */
    padding: 10px;
    padding-top: 10px;
    padding-right: 15px;
    padding-bottom: 10px;
    padding-left: 15px;
    
    /* Border */
    border: 2px solid black;
    border-width: 2px;
    border-style: solid;
    border-color: black;
    
    /* Margin (outer space) */
    margin: 20px;
    margin-top: 20px;
    margin-right: 25px;
    margin-bottom: 20px;
    margin-left: 25px;
}
```

### Box Sizing

```css
/* Default behavior */
.element {
    box-sizing: content-box;
    width: 200px; /* Only content width */
}

/* Include padding and border in width */
.element {
    box-sizing: border-box;
    width: 200px; /* Total width including padding and border */
}
```

## 5. CSS Layout

### Display Properties

```css
/* Block elements (full width, new line) */
.block-element {
    display: block;
}

/* Inline elements (content width, same line) */
.inline-element {
    display: inline;
}

/* Inline-block (content width, but respects width/height) */
.inline-block-element {
    display: inline-block;
}

/* Hidden elements */
.hidden-element {
    display: none;
}
```

### Position Properties

```css
/* Static (default) */
.static {
    position: static;
}

/* Relative (relative to normal position) */
.relative {
    position: relative;
    top: 10px;
    left: 20px;
}

/* Absolute (relative to nearest positioned parent) */
.absolute {
    position: absolute;
    top: 0;
    right: 0;
}

/* Fixed (relative to viewport) */
.fixed {
    position: fixed;
    top: 0;
    left: 0;
}
```

## 6. Responsive Design

### Media Queries

```css
/* Mobile first approach */
.container {
    width: 100%;
    padding: 10px;
}

/* Tablet */
@media (min-width: 768px) {
    .container {
        width: 750px;
        margin: 0 auto;
    }
}

/* Desktop */
@media (min-width: 1024px) {
    .container {
        width: 1000px;
    }
}
```

### Flexbox

```css
/* Flex container */
.flex-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-direction: row;
}

/* Flex items */
.flex-item {
    flex: 1;
    flex-grow: 1;
    flex-shrink: 1;
    flex-basis: auto;
}
```

## 7. Common Web Page Structures

### E-commerce Product Page

```html
<div class="product-page">
    <div class="product-images">
        <img src="product-main.jpg" alt="Product">
        <div class="thumbnail-images">
            <img src="thumb1.jpg" alt="Thumbnail 1">
            <img src="thumb2.jpg" alt="Thumbnail 2">
        </div>
    </div>
    
    <div class="product-info">
        <h1 class="product-title">Product Name</h1>
        <div class="product-price">$99.99</div>
        <div class="product-rating">
            <span class="stars">★★★★☆</span>
            <span class="review-count">(125 reviews)</span>
        </div>
        <div class="product-description">
            <p>Product description here...</p>
        </div>
        <button class="add-to-cart">Add to Cart</button>
    </div>
</div>
```

### News Article Page

```html
<article class="news-article">
    <header class="article-header">
        <h1 class="article-title">Article Title</h1>
        <div class="article-meta">
            <span class="author">By John Doe</span>
            <span class="date">January 15, 2024</span>
            <span class="category">Technology</span>
        </div>
    </header>
    
    <div class="article-content">
        <p class="article-summary">Article summary...</p>
        <div class="article-body">
            <p>Article content...</p>
        </div>
    </div>
    
    <footer class="article-footer">
        <div class="tags">
            <span class="tag">Python</span>
            <span class="tag">Web Scraping</span>
        </div>
    </footer>
</article>
```

### Search Results Page

```html
<div class="search-results">
    <div class="search-header">
        <h2>Search Results for "web scraping"</h2>
        <span class="result-count">1,234 results found</span>
    </div>
    
    <div class="results-list">
        <div class="result-item">
            <h3 class="result-title">
                <a href="/article1">Web Scraping with Python</a>
            </h3>
            <div class="result-url">https://example.com/article1</div>
            <div class="result-snippet">
                Learn how to scrape websites using Python...
            </div>
        </div>
        
        <div class="result-item">
            <h3 class="result-title">
                <a href="/article2">Advanced Web Scraping Techniques</a>
            </h3>
            <div class="result-url">https://example.com/article2</div>
            <div class="result-snippet">
                Advanced techniques for web scraping...
            </div>
        </div>
    </div>
    
    <div class="pagination">
        <a href="?page=1" class="page-link">1</a>
        <a href="?page=2" class="page-link">2</a>
        <a href="?page=3" class="page-link">3</a>
        <a href="?page=next" class="page-link">Next</a>
    </div>
</div>
```

## 8. Analyzing Web Pages for Scraping

### Inspecting Elements

1. **Right-click and "Inspect Element"** - Opens browser developer tools
2. **Look for patterns** - Similar elements often have similar class names or structures
3. **Check for IDs** - Unique identifiers are great for targeting specific elements
4. **Examine data attributes** - Custom data attributes often contain useful information

### Common Patterns to Look For

```html
<!-- Product listings -->
<div class="product-item" data-product-id="123">
    <img class="product-image" src="product.jpg">
    <h3 class="product-title">Product Name</h3>
    <span class="product-price">$99.99</span>
</div>

<!-- Article listings -->
<article class="article-card">
    <h2 class="article-title">Article Title</h2>
    <p class="article-excerpt">Article excerpt...</p>
    <time class="article-date">2024-01-15</time>
</article>

<!-- Navigation menus -->
<nav class="main-nav">
    <ul class="nav-list">
        <li class="nav-item"><a href="/home">Home</a></li>
        <li class="nav-item"><a href="/about">About</a></li>
    </ul>
</nav>
```

## 9. CSS Selectors for Web Scraping

### Most Useful Selectors

```css
/* Target specific elements by class */
.product-title { }
.price { }
.rating { }

/* Target elements with specific attributes */
[data-product-id] { }
[href*="product"] { }

/* Target nested elements */
.product-item .title { }
.article .meta .date { }

/* Target elements by position */
.product-item:nth-child(1) { }
.article:first-child { }

/* Target elements by state */
.product-item:hover { }
.link:visited { }
```

## 10. Exercises

### Exercise 1: HTML Structure Analysis
Create a simple HTML page with a product listing and identify the key elements you would scrape.

### Exercise 2: CSS Selector Practice
Write CSS selectors to target specific elements in a complex HTML structure.

### Exercise 3: Page Structure Mapping
Analyze a real website and map out its HTML structure for scraping purposes.

### Exercise 4: Responsive Design Analysis
Examine how a website's structure changes on different screen sizes.

## 11. Next Steps

You now understand the fundamentals of HTML and CSS! Before moving to the next module, practice by:

1. Inspecting real websites using browser developer tools
2. Identifying patterns in HTML structure
3. Understanding how CSS affects page layout
4. Mapping out data you want to scrape

When you're comfortable with these concepts, proceed to [HTTP & Web Protocols](03-http-web-protocols.md).

## Key Takeaways

- HTML provides the structure and content of web pages
- CSS controls the visual presentation and layout
- Understanding HTML structure is essential for effective web scraping
- CSS selectors help you target specific elements
- Semantic HTML makes pages more accessible and easier to scrape
- Responsive design affects how content is structured

Remember: The better you understand HTML and CSS, the more effectively you can scrape web pages! 