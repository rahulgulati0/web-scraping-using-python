# Selenium for Dynamic Content

## Overview

Selenium is a powerful tool for automating web browsers, making it perfect for scraping websites with JavaScript-rendered content. This module covers how to use Selenium for web scraping when static HTML parsing isn't enough.

## Learning Objectives

By the end of this module, you'll be able to:
- Set up and configure Selenium WebDriver
- Navigate and interact with web pages
- Wait for dynamic content to load
- Handle JavaScript-heavy websites
- Extract data from dynamic elements
- Optimize Selenium for web scraping

## 1. Getting Started with Selenium

### Installation

```bash
pip install selenium
pip install webdriver-manager  # Automatic driver management
```

### Setting Up WebDriver

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from webdriver_manager.chrome import ChromeDriverManager

# Basic setup
driver = webdriver.Chrome()

# With automatic driver management
driver = webdriver.Chrome(ChromeDriverManager().install())

# With options
chrome_options = Options()
chrome_options.add_argument('--headless')  # Run in background
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')

driver = webdriver.Chrome(
    ChromeDriverManager().install(),
    options=chrome_options
)
```

### Basic Navigation

```python
from selenium import webdriver

driver = webdriver.Chrome()

# Navigate to a page
driver.get('https://example.com')

# Get page information
title = driver.title
current_url = driver.current_url
page_source = driver.page_source

# Navigation
driver.back()
driver.forward()
driver.refresh()

# Close browser
driver.quit()
```

## 2. Finding Elements

### Locator Strategies

```python
from selenium.webdriver.common.by import By

# Find by ID
element = driver.find_element(By.ID, 'my-id')

# Find by class name
element = driver.find_element(By.CLASS_NAME, 'my-class')

# Find by tag name
element = driver.find_element(By.TAG_NAME, 'h1')

# Find by CSS selector
element = driver.find_element(By.CSS_SELECTOR, '.content p')

# Find by XPath
element = driver.find_element(By.XPATH, '//div[@class="content"]//p')

# Find by link text
element = driver.find_element(By.LINK_TEXT, 'Click here')

# Find by partial link text
element = driver.find_element(By.PARTIAL_LINK_TEXT, 'Click')

# Find by name attribute
element = driver.find_element(By.NAME, 'username')
```

### Finding Multiple Elements

```python
# Find all matching elements
elements = driver.find_elements(By.CLASS_NAME, 'product-item')

for element in elements:
    print(element.text)

# Check if elements exist
products = driver.find_elements(By.CLASS_NAME, 'product')
if products:
    print(f"Found {len(products)} products")
else:
    print("No products found")
```

## 3. Waiting for Elements

### Explicit Waits

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By

# Wait for element to be present
wait = WebDriverWait(driver, 10)
element = wait.until(
    EC.presence_of_element_located((By.ID, 'dynamic-content'))
)

# Wait for element to be clickable
element = wait.until(
    EC.element_to_be_clickable((By.ID, 'submit-button'))
)

# Wait for element to be visible
element = wait.until(
    EC.visibility_of_element_located((By.CLASS_NAME, 'modal'))
)

# Wait for text to be present
wait.until(
    EC.text_to_be_present_in_element((By.ID, 'status'), 'Complete')
)
```

### Implicit Waits

```python
# Set implicit wait (applies to all find operations)
driver.implicitly_wait(10)  # Wait up to 10 seconds

# Now all find operations will wait
element = driver.find_element(By.ID, 'dynamic-element')
```

### Custom Wait Conditions

```python
def page_has_loaded(driver):
    """Custom condition to check if page has loaded"""
    return driver.execute_script("return document.readyState") == "complete"

def element_has_text(locator, text):
    """Custom condition to check if element contains text"""
    def _predicate(driver):
        element = driver.find_element(*locator)
        return text in element.text
    return _predicate

# Usage
wait = WebDriverWait(driver, 10)
wait.until(page_has_loaded)
wait.until(element_has_text((By.ID, 'status'), 'Ready'))
```

## 4. Interacting with Elements

### Basic Interactions

```python
# Get element properties
element = driver.find_element(By.ID, 'my-element')
text = element.text
html = element.get_attribute('outerHTML')
href = element.get_attribute('href')
is_displayed = element.is_displayed()
is_enabled = element.is_enabled()

# Click elements
button = driver.find_element(By.ID, 'submit-button')
button.click()

# Type text
input_field = driver.find_element(By.NAME, 'username')
input_field.clear()
input_field.send_keys('my_username')

# Submit forms
form = driver.find_element(By.TAG_NAME, 'form')
form.submit()
```

### Advanced Interactions

```python
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys

# Action chains for complex interactions
actions = ActionChains(driver)

# Hover over element
element = driver.find_element(By.ID, 'hover-menu')
actions.move_to_element(element).perform()

# Drag and drop
source = driver.find_element(By.ID, 'source')
target = driver.find_element(By.ID, 'target')
actions.drag_and_drop(source, target).perform()

# Key combinations
actions.key_down(Keys.CONTROL).send_keys('a').key_up(Keys.CONTROL).perform()

# Scroll to element
element = driver.find_element(By.ID, 'bottom-element')
driver.execute_script("arguments[0].scrollIntoView();", element)
```

## 5. Handling JavaScript and AJAX

### Executing JavaScript

```python
# Execute JavaScript
result = driver.execute_script("return document.title;")
print(result)

# Execute script with arguments
element = driver.find_element(By.ID, 'my-element')
driver.execute_script("arguments[0].style.border='3px solid red';", element)

# Get page data via JavaScript
data = driver.execute_script("""
    return {
        title: document.title,
        url: window.location.href,
        userAgent: navigator.userAgent
    };
""")
```

### Waiting for AJAX

```python
def wait_for_ajax(driver, timeout=30):
    """Wait for AJAX requests to complete"""
    wait = WebDriverWait(driver, timeout)
    try:
        wait.until(lambda driver: driver.execute_script("return jQuery.active == 0"))
    except:
        pass  # jQuery might not be available

def wait_for_page_load(driver, timeout=30):
    """Wait for page to fully load"""
    wait = WebDriverWait(driver, timeout)
    wait.until(
        lambda driver: driver.execute_script("return document.readyState") == "complete"
    )

# Usage
driver.get('https://example.com')
wait_for_page_load(driver)
wait_for_ajax(driver)
```

## 6. Handling Different Content Types

### Working with Dropdowns

```python
from selenium.webdriver.support.ui import Select

# Select dropdown
dropdown = Select(driver.find_element(By.ID, 'country-select'))

# Select by visible text
dropdown.select_by_visible_text('United States')

# Select by value
dropdown.select_by_value('us')

# Select by index
dropdown.select_by_index(1)

# Get all options
all_options = dropdown.options
for option in all_options:
    print(option.text)

# Get selected option
selected = dropdown.first_selected_option
print(selected.text)
```

### Working with Alerts and Popups

```python
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

# Wait for alert and handle it
wait = WebDriverWait(driver, 10)
alert = wait.until(EC.alert_is_present())

# Get alert text
alert_text = alert.text
print(alert_text)

# Accept alert
alert.accept()

# Dismiss alert
# alert.dismiss()

# Handle confirm dialog
# alert.accept()  # Click OK
# alert.dismiss()  # Click Cancel

# Handle prompt dialog
# alert.send_keys('my input')
# alert.accept()
```

### Working with Frames

```python
# Switch to frame by name or ID
driver.switch_to.frame('frame-name')

# Switch to frame by element
frame_element = driver.find_element(By.TAG_NAME, 'iframe')
driver.switch_to.frame(frame_element)

# Switch to frame by index
driver.switch_to.frame(0)

# Switch back to main content
driver.switch_to.default_content()

# Switch to parent frame
driver.switch_to.parent_frame()
```

### Working with Multiple Windows

```python
# Get current window handle
main_window = driver.current_window_handle

# Get all window handles
all_windows = driver.window_handles

# Open new window/tab
driver.execute_script("window.open('https://example.com');")

# Switch to new window
for window in driver.window_handles:
    if window != main_window:
        driver.switch_to.window(window)
        break

# Close current window
driver.close()

# Switch back to main window
driver.switch_to.window(main_window)
```

## 7. Practical Scraping Examples

### Dynamic Product Listing

```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options

class DynamicProductScraper:
    def __init__(self, headless=True):
        chrome_options = Options()
        if headless:
            chrome_options.add_argument('--headless')
        chrome_options.add_argument('--no-sandbox')
        chrome_options.add_argument('--disable-dev-shm-usage')
        
        self.driver = webdriver.Chrome(options=chrome_options)
        self.wait = WebDriverWait(self.driver, 10)
    
    def scrape_products(self, url, max_pages=3):
        """Scrape products from paginated results"""
        all_products = []
        
        self.driver.get(url)
        
        for page in range(max_pages):
            print(f"Scraping page {page + 1}")
            
            # Wait for products to load
            self.wait.until(
                EC.presence_of_all_elements_located((By.CLASS_NAME, 'product-item'))
            )
            
            # Extract products from current page
            products = self.extract_products()
            all_products.extend(products)
            
            # Try to go to next page
            try:
                next_button = self.driver.find_element(By.CLASS_NAME, 'next-page')
                if next_button.is_enabled():
                    next_button.click()
                    time.sleep(2)  # Wait for page to load
                else:
                    print("No more pages")
                    break
            except:
                print("Next button not found")
                break
        
        return all_products
    
    def extract_products(self):
        """Extract product data from current page"""
        products = []
        product_elements = self.driver.find_elements(By.CLASS_NAME, 'product-item')
        
        for element in product_elements:
            try:
                # Use JavaScript to get computed styles if needed
                product = {
                    'name': element.find_element(By.CLASS_NAME, 'product-name').text,
                    'price': element.find_element(By.CLASS_NAME, 'price').text,
                    'rating': len(element.find_elements(By.CLASS_NAME, 'star-filled')),
                    'url': element.find_element(By.TAG_NAME, 'a').get_attribute('href')
                }
                products.append(product)
            except Exception as e:
                print(f"Error extracting product: {e}")
                continue
        
        return products
    
    def close(self):
        """Close the browser"""
        self.driver.quit()

# Usage
scraper = DynamicProductScraper(headless=True)
try:
    products = scraper.scrape_products('https://example-store.com/products', max_pages=5)
    print(f"Scraped {len(products)} products")
    for product in products[:5]:  # Print first 5
        print(f"Name: {product['name']}, Price: {product['price']}")
finally:
    scraper.close()
```

### Infinite Scroll Scraping

```python
class InfiniteScrollScraper:
    def __init__(self):
        chrome_options = Options()
        chrome_options.add_argument('--headless')
        self.driver = webdriver.Chrome(options=chrome_options)
        self.wait = WebDriverWait(self.driver, 10)
    
    def scrape_infinite_scroll(self, url, max_scrolls=10):
        """Scrape content from infinite scroll page"""
        self.driver.get(url)
        
        # Wait for initial content
        self.wait.until(
            EC.presence_of_element_located((By.CLASS_NAME, 'content-item'))
        )
        
        last_height = self.driver.execute_script("return document.body.scrollHeight")
        scrolls = 0
        
        while scrolls < max_scrolls:
            # Scroll to bottom
            self.driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
            
            # Wait for new content to load
            time.sleep(2)
            
            # Calculate new scroll height
            new_height = self.driver.execute_script("return document.body.scrollHeight")
            
            if new_height == last_height:
                print("No more content to load")
                break
            
            last_height = new_height
            scrolls += 1
            print(f"Scroll {scrolls}: New content loaded")
        
        # Extract all content
        return self.extract_all_items()
    
    def extract_all_items(self):
        """Extract all loaded items"""
        items = []
        elements = self.driver.find_elements(By.CLASS_NAME, 'content-item')
        
        for element in elements:
            try:
                item = {
                    'title': element.find_element(By.CLASS_NAME, 'title').text,
                    'description': element.find_element(By.CLASS_NAME, 'description').text,
                    'url': element.find_element(By.TAG_NAME, 'a').get_attribute('href')
                }
                items.append(item)
            except Exception as e:
                continue
        
        return items
    
    def close(self):
        self.driver.quit()

# Usage
scraper = InfiniteScrollScraper()
try:
    items = scraper.scrape_infinite_scroll('https://example.com/feed', max_scrolls=20)
    print(f"Scraped {len(items)} items")
finally:
    scraper.close()
```

### Form Automation

```python
class FormAutomation:
    def __init__(self):
        self.driver = webdriver.Chrome()
        self.wait = WebDriverWait(self.driver, 10)
    
    def fill_and_submit_form(self, url, form_data):
        """Fill and submit a form"""
        self.driver.get(url)
        
        # Wait for form to load
        self.wait.until(EC.presence_of_element_located((By.TAG_NAME, 'form')))
        
        # Fill form fields
        for field_name, value in form_data.items():
            try:
                field = self.driver.find_element(By.NAME, field_name)
                
                if field.tag_name == 'select':
                    # Handle dropdown
                    select = Select(field)
                    select.select_by_visible_text(value)
                elif field.get_attribute('type') == 'checkbox':
                    # Handle checkbox
                    if value and not field.is_selected():
                        field.click()
                    elif not value and field.is_selected():
                        field.click()
                else:
                    # Handle text input
                    field.clear()
                    field.send_keys(value)
                    
            except Exception as e:
                print(f"Error filling field {field_name}: {e}")
        
        # Submit form
        submit_button = self.driver.find_element(By.CSS_SELECTOR, 'input[type="submit"], button[type="submit"]')
        submit_button.click()
        
        # Wait for response
        time.sleep(3)
        
        return self.driver.current_url
    
    def close(self):
        self.driver.quit()

# Usage
form_scraper = FormAutomation()
try:
    form_data = {
        'username': 'testuser',
        'email': 'test@example.com',
        'country': 'United States',
        'newsletter': True
    }
    result_url = form_scraper.fill_and_submit_form('https://example.com/signup', form_data)
    print(f"Form submitted, redirected to: {result_url}")
finally:
    form_scraper.close()
```

## 8. Performance Optimization

### Browser Configuration

```python
def get_optimized_driver():
    """Get an optimized Chrome driver for scraping"""
    chrome_options = Options()
    
    # Run in headless mode
    chrome_options.add_argument('--headless')
    
    # Disable images, CSS, and JavaScript (if not needed)
    prefs = {
        "profile.managed_default_content_settings.images": 2,
        "profile.default_content_setting_values.notifications": 2,
        "profile.managed_default_content_settings.stylesheets": 2,
        "profile.managed_default_content_settings.javascript": 1,  # Keep JS enabled
    }
    chrome_options.add_experimental_option("prefs", prefs)
    
    # Additional performance options
    chrome_options.add_argument('--no-sandbox')
    chrome_options.add_argument('--disable-dev-shm-usage')
    chrome_options.add_argument('--disable-gpu')
    chrome_options.add_argument('--disable-features=VizDisplayCompositor')
    chrome_options.add_argument('--window-size=1920,1080')
    
    return webdriver.Chrome(options=chrome_options)
```

### Resource Management

```python
from contextlib import contextmanager

@contextmanager
def selenium_driver():
    """Context manager for Selenium driver"""
    driver = get_optimized_driver()
    try:
        yield driver
    finally:
        driver.quit()

# Usage
with selenium_driver() as driver:
    driver.get('https://example.com')
    # Scraping code here
    pass
# Driver automatically closed
```

### Parallel Processing

```python
from concurrent.futures import ThreadPoolExecutor
import threading

class ParallelScraper:
    def __init__(self, max_workers=3):
        self.max_workers = max_workers
        self.local = threading.local()
    
    def get_driver(self):
        """Get thread-local driver"""
        if not hasattr(self.local, 'driver'):
            self.local.driver = get_optimized_driver()
        return self.local.driver
    
    def scrape_url(self, url):
        """Scrape a single URL"""
        driver = self.get_driver()
        try:
            driver.get(url)
            # Wait and extract data
            wait = WebDriverWait(driver, 10)
            wait.until(EC.presence_of_element_located((By.TAG_NAME, 'body')))
            return {'url': url, 'title': driver.title}
        except Exception as e:
            return {'url': url, 'error': str(e)}
    
    def scrape_urls(self, urls):
        """Scrape multiple URLs in parallel"""
        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            results = list(executor.map(self.scrape_url, urls))
        
        # Clean up drivers
        self.cleanup()
        return results
    
    def cleanup(self):
        """Clean up all drivers"""
        if hasattr(self.local, 'driver'):
            self.local.driver.quit()
```

## 9. Error Handling and Debugging

### Robust Error Handling

```python
from selenium.common.exceptions import (
    TimeoutException, 
    NoSuchElementException,
    StaleElementReferenceException,
    WebDriverException
)

def robust_element_interaction(driver, locator, action='click', timeout=10):
    """Robust element interaction with retries"""
    wait = WebDriverWait(driver, timeout)
    max_retries = 3
    
    for attempt in range(max_retries):
        try:
            element = wait.until(EC.element_to_be_clickable(locator))
            
            if action == 'click':
                element.click()
            elif action == 'text':
                return element.text
            elif action == 'attribute':
                return element.get_attribute('href')
            
            return True
            
        except StaleElementReferenceException:
            print(f"Stale element, retrying... (attempt {attempt + 1})")
            time.sleep(1)
        except TimeoutException:
            print(f"Timeout waiting for element: {locator}")
            return None
        except Exception as e:
            print(f"Error interacting with element: {e}")
            if attempt == max_retries - 1:
                raise
    
    return None
```

### Debugging Tools

```python
def take_screenshot(driver, filename):
    """Take screenshot for debugging"""
    driver.save_screenshot(filename)
    print(f"Screenshot saved: {filename}")

def log_page_info(driver):
    """Log current page information"""
    print(f"Current URL: {driver.current_url}")
    print(f"Page Title: {driver.title}")
    print(f"Page Source Length: {len(driver.page_source)}")

def highlight_element(driver, element):
    """Highlight element for debugging"""
    driver.execute_script(
        "arguments[0].style.border='3px solid red';", 
        element
    )
```

## 10. Best Practices

### General Guidelines

```python
class BestPracticesScraper:
    def __init__(self):
        self.driver = self.setup_driver()
        self.wait = WebDriverWait(self.driver, 10)
    
    def setup_driver(self):
        """Setup driver with best practices"""
        options = Options()
        options.add_argument('--headless')
        options.add_argument('--no-sandbox')
        options.add_argument('--disable-dev-shm-usage')
        
        # Set user agent
        options.add_argument('--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36')
        
        driver = webdriver.Chrome(options=options)
        
        # Set timeouts
        driver.set_page_load_timeout(30)
        driver.implicitly_wait(10)
        
        return driver
    
    def safe_get(self, url, retries=3):
        """Navigate to URL with retries"""
        for attempt in range(retries):
            try:
                self.driver.get(url)
                return True
            except Exception as e:
                print(f"Attempt {attempt + 1} failed: {e}")
                if attempt == retries - 1:
                    raise
                time.sleep(2)
        return False
    
    def safe_find_element(self, locator, timeout=10):
        """Find element safely"""
        try:
            return self.wait.until(
                EC.presence_of_element_located(locator)
            )
        except TimeoutException:
            return None
    
    def cleanup(self):
        """Clean up resources"""
        if hasattr(self, 'driver'):
            self.driver.quit()
```

## Key Takeaways

- Selenium is essential for JavaScript-heavy websites
- Always use explicit waits for dynamic content
- Optimize browser settings for better performance
- Handle errors gracefully with proper exception handling
- Use headless mode for production scraping
- Clean up resources properly

Remember: Selenium is powerful but resource-intensive - use it wisely! 