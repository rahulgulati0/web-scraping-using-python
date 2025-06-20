# Project: Building a Complete Web Scraper

## 🎯 Project Overview

In this capstone project, you'll build a complete web scraping solution that demonstrates all the skills learned throughout the course. You'll create a **Product Price Monitoring System** that scrapes product information from e-commerce sites, stores the data, and provides insights through visualizations.

## 🎪 Project Features

Your scraper will include:
- ✅ **Multi-site scraping** with configurable targets
- ✅ **Data persistence** with SQLite database
- ✅ **Price trend analysis** and alerts
- ✅ **Rate limiting** and respectful scraping
- ✅ **Error handling** and retry mechanisms
- ✅ **Data export** to CSV, JSON, and Excel
- ✅ **Simple web dashboard** for visualization
- ✅ **Logging and monitoring**
- ✅ **Configuration management**

## 📁 Project Structure

```
price_monitor/
├── src/
│   ├── __init__.py
│   ├── scraper.py           # Main scraping logic
│   ├── database.py          # Database operations  
│   ├── config.py            # Configuration management
│   ├── analyzers.py         # Data analysis functions
│   └── dashboard.py         # Web dashboard
├── data/
│   ├── products.db          # SQLite database
│   └── exports/             # Exported data files
├── logs/
│   └── scraper.log          # Application logs
├── config/
│   └── sites.yaml           # Site configurations
├── requirements.txt         # Python dependencies
├── main.py                  # Entry point
└── README.md               # Project documentation
```

## 🛠️ Phase 1: Project Setup

### 1. Create Project Structure

```python
import os

def create_project_structure():
    """Create the project directory structure"""
    dirs = [
        'price_monitor/src',
        'price_monitor/data/exports', 
        'price_monitor/logs',
        'price_monitor/config'
    ]
    
    for dir_path in dirs:
        os.makedirs(dir_path, exist_ok=True)
        print(f"Created directory: {dir_path}")

create_project_structure()
```

### 2. Requirements File

```txt
# requirements.txt
requests>=2.28.0
beautifulsoup4>=4.11.0
selenium>=4.8.0
pandas>=1.5.0
sqlite3
pyyaml>=6.0
flask>=2.2.0
plotly>=5.13.0
schedule>=1.2.0
```

### 3. Configuration System

```python
# src/config.py
import yaml
import os
from dataclasses import dataclass
from typing import List, Dict, Any

@dataclass
class ScrapingConfig:
    delay_between_requests: float = 2.0
    max_retries: int = 3
    timeout: int = 10
    user_agent: str = "PriceMonitor/1.0 (Educational Project)"
    max_concurrent: int = 3

@dataclass  
class SiteConfig:
    name: str
    base_url: str
    product_urls: List[str]
    selectors: Dict[str, str]
    custom_headers: Dict[str, str] = None

class ConfigManager:
    def __init__(self, config_path="config/sites.yaml"):
        self.config_path = config_path
        self.scraping_config = ScrapingConfig()
        self.sites = []
        self.load_config()
    
    def load_config(self):
        """Load configuration from YAML file"""
        if os.path.exists(self.config_path):
            with open(self.config_path, 'r') as f:
                config_data = yaml.safe_load(f)
                
            # Load scraping settings
            if 'scraping' in config_data:
                scraping_data = config_data['scraping']
                self.scraping_config = ScrapingConfig(**scraping_data)
            
            # Load site configurations
            if 'sites' in config_data:
                for site_data in config_data['sites']:
                    site = SiteConfig(**site_data)
                    self.sites.append(site)
        else:
            self.create_default_config()
    
    def create_default_config(self):
        """Create a default configuration file"""
        default_config = {
            'scraping': {
                'delay_between_requests': 2.0,
                'max_retries': 3,
                'timeout': 10,
                'user_agent': 'PriceMonitor/1.0 (Educational Project)',
                'max_concurrent': 3
            },
            'sites': [
                {
                    'name': 'example_store',
                    'base_url': 'https://example-store.com',
                    'product_urls': [
                        'https://example-store.com/product1',
                        'https://example-store.com/product2'
                    ],
                    'selectors': {
                        'title': 'h1.product-title',
                        'price': '.price-current',
                        'availability': '.stock-status',
                        'description': '.product-description'
                    }
                }
            ]
        }
        
        os.makedirs(os.path.dirname(self.config_path), exist_ok=True)
        with open(self.config_path, 'w') as f:
            yaml.dump(default_config, f, default_flow_style=False)
        
        print(f"Created default config at {self.config_path}")
```

## 🗄️ Phase 2: Database Design

```python
# src/database.py
import sqlite3
import json
from datetime import datetime
from typing import List, Dict, Optional
import logging

class ProductDatabase:
    def __init__(self, db_path="data/products.db"):
        self.db_path = db_path
        self.init_database()
        self.logger = logging.getLogger(__name__)
    
    def init_database(self):
        """Initialize database with required tables"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        # Products table
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS products (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                url TEXT UNIQUE NOT NULL,
                site_name TEXT NOT NULL,
                title TEXT,
                current_price REAL,
                currency TEXT DEFAULT 'USD',
                availability TEXT,
                description TEXT,
                first_seen TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        
        # Price history table
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS price_history (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                product_id INTEGER NOT NULL,
                price REAL NOT NULL,
                currency TEXT DEFAULT 'USD',
                recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                FOREIGN KEY (product_id) REFERENCES products (id)
            )
        ''')
        
        # Scraping logs table
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS scraping_logs (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                url TEXT NOT NULL,
                status TEXT NOT NULL,
                error_message TEXT,
                response_time REAL,
                scraped_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        
        # Create indexes
        cursor.execute('CREATE INDEX IF NOT EXISTS idx_product_url ON products(url)')
        cursor.execute('CREATE INDEX IF NOT EXISTS idx_price_history_product ON price_history(product_id)')
        cursor.execute('CREATE INDEX IF NOT EXISTS idx_price_history_date ON price_history(recorded_at)')
        
        conn.commit()
        conn.close()
        
        self.logger.info("Database initialized successfully")
    
    def add_or_update_product(self, product_data: Dict):
        """Add new product or update existing one"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        try:
            # Check if product exists
            cursor.execute('SELECT id, current_price FROM products WHERE url = ?', 
                         (product_data['url'],))
            existing = cursor.fetchone()
            
            if existing:
                product_id, old_price = existing
                
                # Update product
                cursor.execute('''
                    UPDATE products 
                    SET title = ?, current_price = ?, availability = ?, 
                        description = ?, last_updated = CURRENT_TIMESTAMP
                    WHERE url = ?
                ''', (
                    product_data.get('title'),
                    product_data.get('price'),
                    product_data.get('availability'),
                    product_data.get('description'),
                    product_data['url']
                ))
                
                # Add price history if price changed
                new_price = product_data.get('price')
                if new_price and new_price != old_price:
                    cursor.execute('''
                        INSERT INTO price_history (product_id, price, currency)
                        VALUES (?, ?, ?)
                    ''', (product_id, new_price, product_data.get('currency', 'USD')))
                    
                    self.logger.info(f"Price updated for {product_data['url']}: {old_price} -> {new_price}")
                
            else:
                # Insert new product
                cursor.execute('''
                    INSERT INTO products (url, site_name, title, current_price, 
                                        currency, availability, description)
                    VALUES (?, ?, ?, ?, ?, ?, ?)
                ''', (
                    product_data['url'],
                    product_data.get('site_name'),
                    product_data.get('title'),
                    product_data.get('price'),
                    product_data.get('currency', 'USD'),
                    product_data.get('availability'),
                    product_data.get('description')
                ))
                
                # Get the new product ID and add initial price history
                product_id = cursor.lastrowid
                if product_data.get('price'):
                    cursor.execute('''
                        INSERT INTO price_history (product_id, price, currency)
                        VALUES (?, ?, ?)
                    ''', (product_id, product_data['price'], product_data.get('currency', 'USD')))
                
                self.logger.info(f"New product added: {product_data['url']}")
            
            conn.commit()
            return product_id if 'product_id' in locals() else existing[0]
            
        except Exception as e:
            conn.rollback()
            self.logger.error(f"Database error: {e}")
            raise
        finally:
            conn.close()
    
    def get_products(self, site_name: Optional[str] = None) -> List[Dict]:
        """Get all products, optionally filtered by site"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        if site_name:
            cursor.execute('SELECT * FROM products WHERE site_name = ?', (site_name,))
        else:
            cursor.execute('SELECT * FROM products')
        
        columns = [desc[0] for desc in cursor.description]
        products = [dict(zip(columns, row)) for row in cursor.fetchall()]
        
        conn.close()
        return products
    
    def get_price_history(self, product_id: int, days: int = 30) -> List[Dict]:
        """Get price history for a product"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        cursor.execute('''
            SELECT price, currency, recorded_at 
            FROM price_history 
            WHERE product_id = ? AND recorded_at >= datetime('now', '-{} days')
            ORDER BY recorded_at ASC
        '''.format(days), (product_id,))
        
        columns = [desc[0] for desc in cursor.description]
        history = [dict(zip(columns, row)) for row in cursor.fetchall()]
        
        conn.close()
        return history
    
    def log_scraping_attempt(self, url: str, status: str, error_message: str = None, 
                           response_time: float = None):
        """Log scraping attempt"""
        conn = sqlite3.connect(self.db_path)
        cursor = conn.cursor()
        
        cursor.execute('''
            INSERT INTO scraping_logs (url, status, error_message, response_time)
            VALUES (?, ?, ?, ?)
        ''', (url, status, error_message, response_time))
        
        conn.commit()
        conn.close()
```

## 🕷️ Phase 3: Core Scraping Engine

```python
# src/scraper.py
import requests
from bs4 import BeautifulSoup
import time
import logging
from typing import Dict, Optional
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from datetime import datetime
import re

class ProductScraper:
    def __init__(self, config):
        self.config = config
        self.session = requests.Session()
        self.setup_session()
        self.driver = None
        self.logger = logging.getLogger(__name__)
    
    def setup_session(self):
        """Configure requests session"""
        self.session.headers.update({
            'User-Agent': self.config.scraping_config.user_agent,
            'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
            'Accept-Language': 'en-US,en;q=0.5',
            'Accept-Encoding': 'gzip, deflate',
            'Connection': 'keep-alive',
            'Upgrade-Insecure-Requests': '1'
        })
    
    def setup_selenium(self):
        """Set up Selenium WebDriver for JavaScript-heavy sites"""
        if self.driver is None:
            chrome_options = Options()
            chrome_options.add_argument('--headless')
            chrome_options.add_argument('--no-sandbox')
            chrome_options.add_argument('--disable-dev-shm-usage')
            chrome_options.add_argument(f'--user-agent={self.config.scraping_config.user_agent}')
            
            self.driver = webdriver.Chrome(options=chrome_options)
            self.logger.info("Selenium WebDriver initialized")
    
    def scrape_product(self, url: str, selectors: Dict[str, str], 
                      site_name: str, use_selenium: bool = False) -> Optional[Dict]:
        """Scrape a single product"""
        start_time = time.time()
        
        try:
            if use_selenium:
                return self._scrape_with_selenium(url, selectors, site_name)
            else:
                return self._scrape_with_requests(url, selectors, site_name)
                
        except Exception as e:
            response_time = time.time() - start_time
            self.logger.error(f"Failed to scrape {url}: {e}")
            return {
                'url': url,
                'site_name': site_name,
                'status': 'error',
                'error': str(e),
                'response_time': response_time
            }
    
    def _scrape_with_requests(self, url: str, selectors: Dict[str, str], 
                            site_name: str) -> Dict:
        """Scrape using requests and BeautifulSoup"""
        start_time = time.time()
        
        # Rate limiting
        time.sleep(self.config.scraping_config.delay_between_requests)
        
        response = self.session.get(
            url, 
            timeout=self.config.scraping_config.timeout
        )
        response.raise_for_status()
        
        soup = BeautifulSoup(response.content, 'html.parser')
        
        # Extract product data
        product_data = {
            'url': url,
            'site_name': site_name,
            'status': 'success',
            'response_time': time.time() - start_time
        }
        
        # Extract each field using provided selectors
        for field, selector in selectors.items():
            try:
                element = soup.select_one(selector)
                if element:
                    text = element.get_text(strip=True)
                    
                    # Special processing for price fields
                    if field == 'price':
                        text = self._extract_price(text)
                    
                    product_data[field] = text
                else:
                    product_data[field] = None
                    self.logger.warning(f"Could not find {field} with selector {selector} on {url}")
                    
            except Exception as e:
                self.logger.error(f"Error extracting {field} from {url}: {e}")
                product_data[field] = None
        
        return product_data
    
    def _scrape_with_selenium(self, url: str, selectors: Dict[str, str], 
                            site_name: str) -> Dict:
        """Scrape using Selenium for JavaScript-heavy sites"""
        if self.driver is None:
            self.setup_selenium()
        
        start_time = time.time()
        
        self.driver.get(url)
        
        # Wait for page to load
        WebDriverWait(self.driver, 10).until(
            EC.presence_of_element_located((By.TAG_NAME, "body"))
        )
        
        product_data = {
            'url': url,
            'site_name': site_name,
            'status': 'success',
            'response_time': time.time() - start_time
        }
        
        # Extract each field
        for field, selector in selectors.items():
            try:
                element = self.driver.find_element(By.CSS_SELECTOR, selector)
                text = element.text.strip()
                
                if field == 'price':
                    text = self._extract_price(text)
                
                product_data[field] = text
                
            except Exception as e:
                self.logger.warning(f"Could not find {field} with selector {selector} on {url}")
                product_data[field] = None
        
        return product_data
    
    def _extract_price(self, price_text: str) -> Optional[float]:
        """Extract numeric price from text"""
        if not price_text:
            return None
        
        # Remove currency symbols and extract number
        price_pattern = r'[\d,]+\.?\d*'
        match = re.search(price_pattern, price_text.replace(',', ''))
        
        if match:
            try:
                return float(match.group())
            except ValueError:
                pass
        
        return None
    
    def close(self):
        """Clean up resources"""
        if self.driver:
            self.driver.quit()
            self.driver = None
        
        self.session.close()
```

## 📊 Phase 4: Data Analysis & Monitoring

```python
# src/analyzers.py
import pandas as pd
import plotly.graph_objects as go
import plotly.express as px
from datetime import datetime, timedelta
from typing import List, Dict
import logging

class PriceAnalyzer:
    def __init__(self, database):
        self.db = database
        self.logger = logging.getLogger(__name__)
    
    def get_price_trends(self, product_id: int, days: int = 30) -> Dict:
        """Analyze price trends for a product"""
        history = self.db.get_price_history(product_id, days)
        
        if len(history) < 2:
            return {'status': 'insufficient_data', 'message': 'Not enough price history'}
        
        df = pd.DataFrame(history)
        df['recorded_at'] = pd.to_datetime(df['recorded_at'])
        
        # Calculate trend metrics
        current_price = df['price'].iloc[-1]
        previous_price = df['price'].iloc[-2]
        min_price = df['price'].min()
        max_price = df['price'].max()
        avg_price = df['price'].mean()
        
        # Price change
        price_change = current_price - previous_price
        price_change_percent = (price_change / previous_price) * 100 if previous_price > 0 else 0
        
        # Trend direction
        if len(df) >= 3:
            recent_trend = df['price'].tail(3).pct_change().mean()
            if recent_trend > 0.02:
                trend = 'increasing'
            elif recent_trend < -0.02:
                trend = 'decreasing'
            else:
                trend = 'stable'
        else:
            trend = 'unknown'
        
        return {
            'status': 'success',
            'current_price': current_price,
            'price_change': price_change,
            'price_change_percent': price_change_percent,
            'min_price': min_price,
            'max_price': max_price,
            'avg_price': avg_price,
            'trend': trend,
            'data_points': len(df)
        }
    
    def generate_price_chart(self, product_id: int, days: int = 30) -> str:
        """Generate a price chart for a product"""
        history = self.db.get_price_history(product_id, days)
        
        if not history:
            return None
        
        df = pd.DataFrame(history)
        df['recorded_at'] = pd.to_datetime(df['recorded_at'])
        
        fig = go.Figure()
        
        fig.add_trace(go.Scatter(
            x=df['recorded_at'],
            y=df['price'],
            mode='lines+markers',
            name='Price',
            line=dict(color='blue', width=2),
            marker=dict(size=6)
        ))
        
        fig.update_layout(
            title=f'Price History (Last {days} days)',
            xaxis_title='Date',
            yaxis_title='Price ($)',
            hovermode='x unified',
            showlegend=False
        )
        
        # Save chart as HTML
        chart_path = f'data/exports/price_chart_{product_id}_{datetime.now().strftime("%Y%m%d")}.html'
        fig.write_html(chart_path)
        
        return chart_path
    
    def find_price_alerts(self, threshold_percent: float = 10.0) -> List[Dict]:
        """Find products with significant price changes"""
        products = self.db.get_products()
        alerts = []
        
        for product in products:
            analysis = self.get_price_trends(product['id'], days=7)
            
            if analysis['status'] == 'success':
                change_percent = abs(analysis['price_change_percent'])
                
                if change_percent >= threshold_percent:
                    alerts.append({
                        'product_id': product['id'],
                        'title': product['title'],
                        'url': product['url'],
                        'price_change': analysis['price_change'],
                        'price_change_percent': analysis['price_change_percent'],
                        'current_price': analysis['current_price'],
                        'alert_type': 'price_drop' if analysis['price_change'] < 0 else 'price_increase'
                    })
        
        return alerts
    
    def export_data(self, format_type: str = 'csv', filename: str = None) -> str:
        """Export scraped data to various formats"""
        products = self.db.get_products()
        df = pd.DataFrame(products)
        
        if filename is None:
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            filename = f"data/exports/products_{timestamp}"
        
        if format_type.lower() == 'csv':
            filepath = f"{filename}.csv"
            df.to_csv(filepath, index=False)
        elif format_type.lower() == 'excel':
            filepath = f"{filename}.xlsx"
            df.to_excel(filepath, index=False)
        elif format_type.lower() == 'json':
            filepath = f"{filename}.json"
            df.to_json(filepath, orient='records', indent=2)
        else:
            raise ValueError(f"Unsupported format: {format_type}")
        
        self.logger.info(f"Data exported to {filepath}")
        return filepath
```

## 🌐 Phase 5: Web Dashboard

```python
# src/dashboard.py
from flask import Flask, render_template, request, jsonify
import plotly.graph_objects as go
import plotly.utils
import json
from datetime import datetime

app = Flask(__name__)

class Dashboard:
    def __init__(self, database, analyzer):
        self.db = database
        self.analyzer = analyzer
        self.app = app
        self.setup_routes()
    
    def setup_routes(self):
        @self.app.route('/')
        def index():
            products = self.db.get_products()
            return render_template('index.html', products=products)
        
        @self.app.route('/product/<int:product_id>')
        def product_detail(product_id):
            # Get product info
            products = self.db.get_products()
            product = next((p for p in products if p['id'] == product_id), None)
            
            if not product:
                return "Product not found", 404
            
            # Get price analysis
            analysis = self.analyzer.get_price_trends(product_id)
            
            # Get price history chart
            history = self.db.get_price_history(product_id)
            chart_json = self.create_price_chart(history)
            
            return render_template('product.html', 
                                 product=product, 
                                 analysis=analysis,
                                 chart_json=chart_json)
        
        @self.app.route('/api/alerts')
        def api_alerts():
            threshold = request.args.get('threshold', 10.0, type=float)
            alerts = self.analyzer.find_price_alerts(threshold)
            return jsonify(alerts)
        
        @self.app.route('/export/<format_type>')
        def export_data(format_type):
            try:
                filepath = self.analyzer.export_data(format_type)
                return jsonify({'status': 'success', 'file': filepath})
            except Exception as e:
                return jsonify({'status': 'error', 'message': str(e)})
    
    def create_price_chart(self, history):
        """Create price chart for dashboard"""
        if not history:
            return None
        
        dates = [item['recorded_at'] for item in history]
        prices = [item['price'] for item in history]
        
        fig = go.Figure()
        fig.add_trace(go.Scatter(x=dates, y=prices, mode='lines+markers'))
        fig.update_layout(title='Price History', xaxis_title='Date', yaxis_title='Price')
        
        return json.dumps(fig, cls=plotly.utils.PlotlyJSONEncoder)
    
    def run(self, debug=True, port=5000):
        self.app.run(debug=debug, port=port)

# HTML Templates (create templates/ directory)
# templates/index.html
index_template = '''
<!DOCTYPE html>
<html>
<head>
    <title>Price Monitor Dashboard</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>
    <div class="container mt-4">
        <h1>Price Monitor Dashboard</h1>
        
        <div class="row mt-4">
            <div class="col-md-12">
                <h2>Monitored Products</h2>
                <table class="table table-striped">
                    <thead>
                        <tr>
                            <th>Product</th>
                            <th>Site</th>
                            <th>Current Price</th>
                            <th>Last Updated</th>
                            <th>Actions</th>
                        </tr>
                    </thead>
                    <tbody>
                        {% for product in products %}
                        <tr>
                            <td>{{ product.title or 'N/A' }}</td>
                            <td>{{ product.site_name }}</td>
                            <td>${{ "%.2f"|format(product.current_price) if product.current_price else 'N/A' }}</td>
                            <td>{{ product.last_updated }}</td>
                            <td>
                                <a href="/product/{{ product.id }}" class="btn btn-sm btn-primary">View Details</a>
                            </td>
                        </tr>
                        {% endfor %}
                    </tbody>
                </table>
            </div>
        </div>
        
        <div class="row mt-4">
            <div class="col-md-6">
                <button class="btn btn-success" onclick="exportData('csv')">Export CSV</button>
                <button class="btn btn-info" onclick="exportData('excel')">Export Excel</button>
                <button class="btn btn-warning" onclick="exportData('json')">Export JSON</button>
            </div>
        </div>
    </div>
    
    <script>
        function exportData(format) {
            fetch('/export/' + format)
                .then(response => response.json())
                .then(data => {
                    if (data.status === 'success') {
                        alert('Data exported to: ' + data.file);
                    } else {
                        alert('Export failed: ' + data.message);
                    }
                });
        }
    </script>
</body>
</html>
'''
```

## 🚀 Phase 6: Main Application

```python
# main.py
import logging
import schedule
import time
import threading
from src.config import ConfigManager
from src.database import ProductDatabase
from src.scraper import ProductScraper
from src.analyzers import PriceAnalyzer
from src.dashboard import Dashboard
import argparse

# Set up logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('logs/scraper.log'),
        logging.StreamHandler()
    ]
)

class PriceMonitor:
    def __init__(self):
        self.config = ConfigManager()
        self.database = ProductDatabase()
        self.scraper = ProductScraper(self.config)
        self.analyzer = PriceAnalyzer(self.database)
        self.dashboard = Dashboard(self.database, self.analyzer)
        self.logger = logging.getLogger(__name__)
    
    def scrape_all_products(self):
        """Scrape all configured products"""
        self.logger.info("Starting scraping cycle")
        
        for site in self.config.sites:
            self.logger.info(f"Scraping site: {site.name}")
            
            for url in site.product_urls:
                try:
                    # Scrape product
                    product_data = self.scraper.scrape_product(
                        url=url,
                        selectors=site.selectors,
                        site_name=site.name
                    )
                    
                    if product_data and product_data.get('status') == 'success':
                        # Save to database
                        self.database.add_or_update_product(product_data)
                        self.database.log_scraping_attempt(url, 'success', response_time=product_data.get('response_time'))
                    else:
                        error_msg = product_data.get('error', 'Unknown error') if product_data else 'No data returned'
                        self.database.log_scraping_attempt(url, 'failed', error_message=error_msg)
                        
                except Exception as e:
                    self.logger.error(f"Error scraping {url}: {e}")
                    self.database.log_scraping_attempt(url, 'error', error_message=str(e))
        
        self.logger.info("Scraping cycle completed")
    
    def check_alerts(self):
        """Check for price alerts"""
        alerts = self.analyzer.find_price_alerts(threshold_percent=10.0)
        
        if alerts:
            self.logger.info(f"Found {len(alerts)} price alerts")
            for alert in alerts:
                self.logger.warning(
                    f"Price alert: {alert['title']} - "
                    f"{alert['price_change_percent']:.1f}% change to ${alert['current_price']:.2f}"
                )
    
    def run_scheduler(self):
        """Run the scheduled scraping"""
        # Schedule scraping every 6 hours
        schedule.every(6).hours.do(self.scrape_all_products)
        schedule.every(6).hours.do(self.check_alerts)
        
        # Run immediately once
        self.scrape_all_products()
        self.check_alerts()
        
        self.logger.info("Scheduler started. Scraping every 6 hours.")
        
        while True:
            schedule.run_pending()
            time.sleep(60)  # Check every minute
    
    def run_dashboard(self):
        """Run the web dashboard"""
        self.logger.info("Starting web dashboard on http://localhost:5000")
        self.dashboard.run(debug=False, port=5000)
    
    def cleanup(self):
        """Clean up resources"""
        self.scraper.close()

def main():
    parser = argparse.ArgumentParser(description='Price Monitor Application')
    parser.add_argument('--mode', choices=['scrape', 'dashboard', 'both'], 
                       default='both', help='Run mode')
    parser.add_argument('--once', action='store_true', 
                       help='Run scraping once instead of continuously')
    
    args = parser.parse_args()
    
    monitor = PriceMonitor()
    
    try:
        if args.mode == 'scrape':
            if args.once:
                monitor.scrape_all_products()
                monitor.check_alerts()
            else:
                monitor.run_scheduler()
        
        elif args.mode == 'dashboard':
            monitor.run_dashboard()
        
        elif args.mode == 'both':
            # Run scheduler in background thread
            scheduler_thread = threading.Thread(target=monitor.run_scheduler, daemon=True)
            scheduler_thread.start()
            
            # Run dashboard in main thread
            monitor.run_dashboard()
    
    except KeyboardInterrupt:
        print("\nShutting down...")
    finally:
        monitor.cleanup()

if __name__ == '__main__':
    main()
```

## 📋 Phase 7: Testing & Deployment

### 1. Testing the Scraper

```python
# test_scraper.py
import unittest
from unittest.mock import Mock, patch
from src.scraper import ProductScraper
from src.config import ConfigManager

class TestProductScraper(unittest.TestCase):
    def setUp(self):
        self.config = ConfigManager()
        self.scraper = ProductScraper(self.config)
    
    def test_price_extraction(self):
        """Test price extraction from text"""
        test_cases = [
            ("$19.99", 19.99),
            ("Price: $1,234.56", 1234.56),
            ("€25.50", 25.50),
            ("No price", None)
        ]
        
        for text, expected in test_cases:
            result = self.scraper._extract_price(text)
            self.assertEqual(result, expected)
    
    @patch('requests.Session.get')
    def test_scrape_product(self, mock_get):
        # Mock HTML response
        mock_response = Mock()
        mock_response.status_code = 200
        mock_response.content = '''
        <html>
            <h1 class="title">Test Product</h1>
            <span class="price">$29.99</span>
            <div class="stock">In Stock</div>
        </html>
        '''
        mock_get.return_value = mock_response
        
        selectors = {
            'title': '.title',
            'price': '.price', 
            'availability': '.stock'
        }
        
        result = self.scraper.scrape_product(
            'https://example.com/product', 
            selectors, 
            'test_site'
        )
        
        self.assertEqual(result['status'], 'success')
        self.assertEqual(result['title'], 'Test Product')
        self.assertEqual(result['price'], 29.99)

if __name__ == '__main__':
    unittest.main()
```

### 2. Running the Project

```bash
# Install dependencies
pip install -r requirements.txt

# Run scraper once
python main.py --mode scrape --once

# Run continuous monitoring
python main.py --mode scrape

# Run dashboard only
python main.py --mode dashboard

# Run both (recommended)
python main.py --mode both
```

## 🎯 Project Extensions

### 1. Add Email Alerts

```python
import smtplib
from email.mime.text import MIMEText

def send_price_alert(alert_data, email_config):
    msg = MIMEText(f"Price alert: {alert_data['title']} is now ${alert_data['current_price']}")
    msg['Subject'] = 'Price Monitor Alert'
    msg['From'] = email_config['from']
    msg['To'] = email_config['to']
    
    server = smtplib.SMTP(email_config['smtp_server'], email_config['port'])
    server.send_message(msg)
    server.quit()
```

### 2. Add More Analysis Features

- Price prediction using machine learning
- Competitor price comparison
- Historical trend analysis
- Seasonal price patterns

### 3. Enhance the Dashboard

- Real-time updates with WebSockets
- Interactive charts with Plotly Dash
- User authentication and multiple watchlists
- Mobile-responsive design

## 📝 Project Deliverables

Your completed project should include:

1. ✅ **Working scraper** that can monitor multiple products
2. ✅ **Database** with proper schema and relationships
3. ✅ **Web dashboard** for visualization and management
4. ✅ **Configuration system** for easy site addition
5. ✅ **Comprehensive logging** and error handling
6. ✅ **Data export** functionality
7. ✅ **Price alerts** and monitoring
8. ✅ **Unit tests** for core functionality
9. ✅ **Documentation** and setup instructions

## 🏆 Success Criteria

Your project demonstrates mastery when it:

- **Respects robots.txt** and implements proper delays
- **Handles errors gracefully** with retry mechanisms
- **Stores data efficiently** with proper normalization
- **Provides useful insights** through analysis features
- **Follows best practices** for code organization
- **Includes proper documentation** and testing
- **Demonstrates ethical scraping** practices

## 🚀 Next Steps

After completing this project:

1. **Deploy to cloud** (AWS, Heroku, DigitalOcean)
2. **Add more sites** and products to monitor
3. **Implement machine learning** for price predictions
4. **Create mobile app** interface
5. **Scale with multiple workers** and job queues
6. **Add API endpoints** for external integration

Congratulations! You've built a professional-grade web scraping system that demonstrates all the skills learned in this course. This project serves as an excellent portfolio piece and foundation for more advanced scraping applications.

Remember to always scrape responsibly and respect website terms of service! 