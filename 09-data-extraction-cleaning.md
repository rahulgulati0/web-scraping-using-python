# Data Extraction & Cleaning

## Overview

Once you've scraped raw data from websites, the next crucial step is extracting meaningful information and cleaning it for analysis. This module covers techniques for processing, validating, and transforming scraped data into usable formats.

## Learning Objectives

By the end of this module, you'll be able to:
- Extract structured data from unstructured text
- Clean and normalize scraped data
- Handle missing and inconsistent data
- Validate data quality
- Transform data into analysis-ready formats

## 1. Text Processing and Extraction

### Basic Text Cleaning

```python
import re
import string

def clean_text(text):
    """Basic text cleaning function"""
    if not text:
        return ""
    
    # Convert to string and strip whitespace
    text = str(text).strip()
    
    # Remove extra whitespace
    text = re.sub(r'\s+', ' ', text)
    
    # Remove special characters (optional)
    text = re.sub(r'[^\w\s]', '', text)
    
    return text

def clean_html_text(text):
    """Clean text that may contain HTML entities"""
    import html
    
    # Decode HTML entities
    text = html.unescape(text)
    
    # Remove HTML tags
    text = re.sub(r'<[^>]+>', '', text)
    
    # Clean whitespace
    text = re.sub(r'\s+', ' ', text).strip()
    
    return text

# Examples
raw_text = "  Product Name&nbsp;&nbsp;with    extra   spaces  "
cleaned = clean_text(raw_text)
print(cleaned)  # "Product Name with extra spaces"

html_text = "Price: &pound;99.99 <span class='old-price'>&pound;129.99</span>"
clean_html = clean_html_text(html_text)
print(clean_html)  # "Price: £99.99 £129.99"
```

### Regular Expressions for Data Extraction

```python
import re

def extract_price(text):
    """Extract price from text"""
    # Pattern for price: currency symbol followed by digits
    price_pattern = r'[£$€¥]\s*(\d+(?:[,.]?\d{3})*(?:\.\d{2})?)'
    match = re.search(price_pattern, text)
    
    if match:
        price_str = match.group(1).replace(',', '')
        return float(price_str)
    
    # Alternative pattern: digits followed by currency
    alt_pattern = r'(\d+(?:[,.]?\d{3})*(?:\.\d{2})?)\s*[£$€¥]'
    match = re.search(alt_pattern, text)
    
    if match:
        price_str = match.group(1).replace(',', '')
        return float(price_str)
    
    return None

def extract_phone_numbers(text):
    """Extract phone numbers from text"""
    # Various phone number patterns
    patterns = [
        r'\+\d{1,3}[-.\s]?\d{3,4}[-.\s]?\d{3,4}[-.\s]?\d{3,4}',  # International
        r'\(\d{3}\)\s?\d{3}[-.\s]?\d{4}',  # (123) 456-7890
        r'\d{3}[-.\s]?\d{3}[-.\s]?\d{4}',  # 123-456-7890
        r'\d{10,}',  # 10+ digits
    ]
    
    phones = []
    for pattern in patterns:
        matches = re.findall(pattern, text)
        phones.extend(matches)
    
    return list(set(phones))  # Remove duplicates

def extract_emails(text):
    """Extract email addresses from text"""
    email_pattern = r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b'
    return re.findall(email_pattern, text)

def extract_urls(text):
    """Extract URLs from text"""
    url_pattern = r'https?://(?:[-\w.])+(?::\d+)?(?:/(?:[\w/_.])*(?:\?(?:[\w&=%.])*)?(?:#(?:[\w.])*)?)?'
    return re.findall(url_pattern, text)

# Examples
price_text = "Special offer: Was £129.99, now only £99.99!"
price = extract_price(price_text)
print(f"Extracted price: £{price}")

contact_text = "Call us at (555) 123-4567 or email support@example.com"
phones = extract_phone_numbers(contact_text)
emails = extract_emails(contact_text)
print(f"Phones: {phones}")
print(f"Emails: {emails}")
```

### Advanced Text Processing

```python
def extract_product_specs(text):
    """Extract product specifications from text"""
    specs = {}
    
    # Pattern for "Key: Value" format
    spec_pattern = r'(\w+(?:\s+\w+)*)\s*[:\-]\s*([^\n\r;,]+)'
    matches = re.findall(spec_pattern, text, re.IGNORECASE)
    
    for key, value in matches:
        key = key.strip().lower().replace(' ', '_')
        value = value.strip()
        specs[key] = value
    
    return specs

def extract_dimensions(text):
    """Extract dimensions from text"""
    # Pattern for dimensions like "10 x 5 x 3 cm" or "10cm x 5cm x 3cm"
    dim_pattern = r'(\d+(?:\.\d+)?)\s*(?:cm|mm|in|inches|"|\')\s*[x×]\s*(\d+(?:\.\d+)?)\s*(?:cm|mm|in|inches|"|\')\s*[x×]\s*(\d+(?:\.\d+)?)\s*(?:cm|mm|in|inches|"|\')?'
    match = re.search(dim_pattern, text, re.IGNORECASE)
    
    if match:
        return {
            'length': float(match.group(1)),
            'width': float(match.group(2)),
            'height': float(match.group(3))
        }
    
    return None

def extract_ratings(text):
    """Extract ratings from text"""
    # Pattern for ratings like "4.5 out of 5" or "4.5/5" or "4.5 stars"
    rating_patterns = [
        r'(\d+(?:\.\d+)?)\s*(?:out\s*of\s*)?(\d+)(?:\s*stars?)?',
        r'(\d+(?:\.\d+)?)/(\d+)',
        r'(\d+(?:\.\d+)?)\s*stars?'
    ]
    
    for pattern in rating_patterns:
        match = re.search(pattern, text, re.IGNORECASE)
        if match:
            if len(match.groups()) == 2:
                rating = float(match.group(1))
                max_rating = float(match.group(2))
                return rating, max_rating
            else:
                rating = float(match.group(1))
                return rating, 5.0  # Assume 5-star scale
    
    return None, None

# Examples
spec_text = """
Color: Red
Weight: 2.5 kg
Material: Aluminum
Dimensions: 30cm x 20cm x 10cm
Rating: 4.5 out of 5 stars
"""

specs = extract_product_specs(spec_text)
dimensions = extract_dimensions(spec_text)
rating, max_rating = extract_ratings(spec_text)

print(f"Specs: {specs}")
print(f"Dimensions: {dimensions}")
print(f"Rating: {rating}/{max_rating}")
```

## 2. Data Validation and Quality Control

### Data Validation Functions

```python
def validate_email(email):
    """Validate email address"""
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return bool(re.match(pattern, email))

def validate_phone(phone):
    """Validate phone number"""
    # Remove all non-digit characters
    digits = re.sub(r'\D', '', phone)
    # Check if it's a valid length
    return len(digits) >= 10 and len(digits) <= 15

def validate_url(url):
    """Validate URL"""
    url_pattern = r'^https?://(?:[-\w.])+(?::\d+)?(?:/.*)?$'
    return bool(re.match(url_pattern, url))

def validate_price(price):
    """Validate price"""
    try:
        price_float = float(price)
        return price_float >= 0
    except (ValueError, TypeError):
        return False

def validate_date(date_string, formats=None):
    """Validate date string"""
    from datetime import datetime
    
    if formats is None:
        formats = [
            '%Y-%m-%d', '%d/%m/%Y', '%m/%d/%Y',
            '%Y-%m-%d %H:%M:%S', '%d/%m/%Y %H:%M:%S'
        ]
    
    for fmt in formats:
        try:
            datetime.strptime(date_string, fmt)
            return True
        except ValueError:
            continue
    
    return False

# Data validation class
class DataValidator:
    def __init__(self):
        self.validators = {
            'email': validate_email,
            'phone': validate_phone,
            'url': validate_url,
            'price': validate_price,
            'date': validate_date
        }
    
    def validate_record(self, record, schema):
        """Validate a data record against a schema"""
        errors = []
        
        for field, field_type in schema.items():
            if field in record:
                value = record[field]
                if field_type in self.validators:
                    if not self.validators[field_type](value):
                        errors.append(f"Invalid {field_type} in field '{field}': {value}")
            else:
                errors.append(f"Missing required field: {field}")
        
        return len(errors) == 0, errors

# Usage
validator = DataValidator()
record = {
    'email': 'user@example.com',
    'phone': '555-123-4567',
    'price': '99.99'
}
schema = {
    'email': 'email',
    'phone': 'phone',
    'price': 'price'
}

is_valid, errors = validator.validate_record(record, schema)
print(f"Valid: {is_valid}")
if not is_valid:
    print(f"Errors: {errors}")
```

### Data Quality Assessment

```python
import pandas as pd
import numpy as np

def assess_data_quality(df):
    """Assess the quality of a DataFrame"""
    quality_report = {}
    
    for column in df.columns:
        col_data = df[column]
        
        quality_report[column] = {
            'total_count': len(col_data),
            'missing_count': col_data.isnull().sum(),
            'missing_percentage': (col_data.isnull().sum() / len(col_data)) * 100,
            'unique_count': col_data.nunique(),
            'duplicate_count': col_data.duplicated().sum(),
            'data_type': str(col_data.dtype)
        }
        
        # For numeric columns
        if col_data.dtype in ['int64', 'float64']:
            quality_report[column].update({
                'mean': col_data.mean(),
                'std': col_data.std(),
                'min': col_data.min(),
                'max': col_data.max(),
                'outliers_count': len(col_data[(np.abs(col_data - col_data.mean()) > 3 * col_data.std())])
            })
        
        # For string columns
        elif col_data.dtype == 'object':
            quality_report[column].update({
                'avg_length': col_data.astype(str).str.len().mean(),
                'empty_strings': (col_data == '').sum()
            })
    
    return quality_report

def print_quality_report(quality_report):
    """Print a formatted data quality report"""
    for column, stats in quality_report.items():
        print(f"\n=== {column} ===")
        for key, value in stats.items():
            if isinstance(value, float):
                print(f"{key}: {value:.2f}")
            else:
                print(f"{key}: {value}")
```

## 3. Data Normalization and Standardization

### Text Normalization

```python
def normalize_text(text):
    """Normalize text for consistency"""
    if not text:
        return ""
    
    # Convert to lowercase
    text = text.lower()
    
    # Remove extra whitespace
    text = re.sub(r'\s+', ' ', text).strip()
    
    # Normalize quotes
    text = text.replace('"', '"').replace('"', '"')
    text = text.replace(''', "'").replace(''', "'")
    
    return text

def normalize_names(name):
    """Normalize person/company names"""
    if not name:
        return ""
    
    # Title case
    name = name.title()
    
    # Handle common abbreviations
    abbreviations = {
        'Inc.': 'Inc',
        'Corp.': 'Corp',
        'Ltd.': 'Ltd',
        'Co.': 'Co'
    }
    
    for old, new in abbreviations.items():
        name = name.replace(old, new)
    
    return name.strip()

def normalize_phone(phone):
    """Normalize phone numbers"""
    if not phone:
        return ""
    
    # Remove all non-digit characters
    digits = re.sub(r'\D', '', phone)
    
    # Format as XXX-XXX-XXXX for US numbers
    if len(digits) == 10:
        return f"{digits[:3]}-{digits[3:6]}-{digits[6:]}"
    elif len(digits) == 11 and digits[0] == '1':
        return f"{digits[1:4]}-{digits[4:7]}-{digits[7:]}"
    
    return phone  # Return original if can't normalize
```

### Price and Currency Normalization

```python
def normalize_price(price_text, target_currency='USD'):
    """Normalize price to a standard format"""
    if not price_text:
        return None
    
    # Currency conversion rates (in practice, use an API)
    exchange_rates = {
        'USD': 1.0,
        'EUR': 0.85,
        'GBP': 0.73,
        'JPY': 110.0
    }
    
    # Extract currency and amount
    currency_symbols = {'$': 'USD', '€': 'EUR', '£': 'GBP', '¥': 'JPY'}
    
    # Find currency symbol
    currency = None
    for symbol, curr_code in currency_symbols.items():
        if symbol in price_text:
            currency = curr_code
            break
    
    # Extract numeric value
    price_match = re.search(r'[\d,]+\.?\d*', price_text.replace(',', ''))
    if not price_match:
        return None
    
    amount = float(price_match.group())
    
    # Convert to target currency
    if currency and currency != target_currency:
        if currency in exchange_rates and target_currency in exchange_rates:
            # Convert via USD
            usd_amount = amount / exchange_rates[currency]
            amount = usd_amount * exchange_rates[target_currency]
    
    return round(amount, 2)

def normalize_date(date_string):
    """Normalize date to ISO format"""
    from datetime import datetime
    
    date_formats = [
        '%Y-%m-%d', '%d/%m/%Y', '%m/%d/%Y',
        '%B %d, %Y', '%d %B %Y',
        '%Y-%m-%d %H:%M:%S'
    ]
    
    for fmt in date_formats:
        try:
            dt = datetime.strptime(date_string, fmt)
            return dt.strftime('%Y-%m-%d')
        except ValueError:
            continue
    
    return None

# Examples
price_examples = ["$99.99", "€85.00", "£73.50"]
for price in price_examples:
    normalized = normalize_price(price, 'USD')
    print(f"{price} -> ${normalized}")

date_examples = ["2024-01-15", "15/01/2024", "January 15, 2024"]
for date in date_examples:
    normalized = normalize_date(date)
    print(f"{date} -> {normalized}")
```

## 4. Handling Missing Data

### Missing Data Detection

```python
def detect_missing_data(df):
    """Detect various types of missing data"""
    missing_indicators = ['', 'N/A', 'n/a', 'NULL', 'null', 'None', 'none', '-', 'TBD', 'TBA']
    
    # Replace common missing indicators with NaN
    for col in df.columns:
        df[col] = df[col].replace(missing_indicators, np.nan)
    
    # Detect missing data patterns
    missing_summary = {
        'total_missing': df.isnull().sum().sum(),
        'missing_by_column': df.isnull().sum().to_dict(),
        'missing_percentage': (df.isnull().sum() / len(df) * 100).to_dict(),
        'rows_with_missing': df.isnull().any(axis=1).sum(),
        'complete_rows': df.dropna().shape[0]
    }
    
    return missing_summary

def fill_missing_data(df, strategy='smart'):
    """Fill missing data using various strategies"""
    df_filled = df.copy()
    
    for column in df_filled.columns:
        if df_filled[column].isnull().any():
            if strategy == 'smart':
                # Smart strategy based on data type
                if df_filled[column].dtype in ['int64', 'float64']:
                    # For numeric: use median
                    df_filled[column].fillna(df_filled[column].median(), inplace=True)
                else:
                    # For categorical: use mode
                    mode_value = df_filled[column].mode()
                    if not mode_value.empty:
                        df_filled[column].fillna(mode_value[0], inplace=True)
                    else:
                        df_filled[column].fillna('Unknown', inplace=True)
            
            elif strategy == 'forward_fill':
                df_filled[column].fillna(method='ffill', inplace=True)
            
            elif strategy == 'backward_fill':
                df_filled[column].fillna(method='bfill', inplace=True)
            
            elif strategy == 'interpolate':
                if df_filled[column].dtype in ['int64', 'float64']:
                    df_filled[column].interpolate(inplace=True)
    
    return df_filled
```

### Intelligent Data Imputation

```python
def intelligent_imputation(df, column, method='knn'):
    """Intelligent imputation using various methods"""
    from sklearn.impute import KNNImputer
    from sklearn.preprocessing import LabelEncoder
    
    if method == 'knn':
        # Prepare data for KNN imputation
        df_encoded = df.copy()
        label_encoders = {}
        
        # Encode categorical variables
        for col in df_encoded.columns:
            if df_encoded[col].dtype == 'object':
                le = LabelEncoder()
                non_null_values = df_encoded[col].dropna()
                if len(non_null_values) > 0:
                    le.fit(non_null_values)
                    label_encoders[col] = le
                    df_encoded[col] = df_encoded[col].map(lambda x: le.transform([x])[0] if pd.notna(x) else np.nan)
        
        # Apply KNN imputation
        imputer = KNNImputer(n_neighbors=5)
        df_imputed = pd.DataFrame(imputer.fit_transform(df_encoded), columns=df_encoded.columns)
        
        # Decode categorical variables
        for col, le in label_encoders.items():
            df_imputed[col] = df_imputed[col].round().astype(int)
            df_imputed[col] = df_imputed[col].map(lambda x: le.inverse_transform([x])[0])
        
        return df_imputed[column]
    
    return None
```

## 5. Data Transformation and Enrichment

### Feature Engineering

```python
def extract_features_from_text(df, text_column):
    """Extract features from text data"""
    df_features = df.copy()
    
    # Text length
    df_features[f'{text_column}_length'] = df[text_column].astype(str).str.len()
    
    # Word count
    df_features[f'{text_column}_word_count'] = df[text_column].astype(str).str.split().str.len()
    
    # Contains email
    df_features[f'{text_column}_has_email'] = df[text_column].astype(str).str.contains('@', na=False)
    
    # Contains phone
    df_features[f'{text_column}_has_phone'] = df[text_column].astype(str).str.contains(r'\d{3}[-.\s]?\d{3}[-.\s]?\d{4}', na=False)
    
    # Contains URL
    df_features[f'{text_column}_has_url'] = df[text_column].astype(str).str.contains('http', na=False)
    
    # Sentiment analysis (basic)
    positive_words = ['good', 'great', 'excellent', 'amazing', 'love', 'best']
    negative_words = ['bad', 'terrible', 'awful', 'hate', 'worst', 'horrible']
    
    text_lower = df[text_column].astype(str).str.lower()
    df_features[f'{text_column}_positive_words'] = text_lower.apply(
        lambda x: sum(word in x for word in positive_words)
    )
    df_features[f'{text_column}_negative_words'] = text_lower.apply(
        lambda x: sum(word in x for word in negative_words)
    )
    
    return df_features

def create_price_features(df, price_column):
    """Create features from price data"""
    df_features = df.copy()
    
    # Price bands
    df_features[f'{price_column}_band'] = pd.cut(
        df[price_column], 
        bins=[0, 50, 100, 200, 500, float('inf')],
        labels=['Low', 'Medium', 'High', 'Premium', 'Luxury']
    )
    
    # Price relative to mean
    mean_price = df[price_column].mean()
    df_features[f'{price_column}_vs_mean'] = df[price_column] / mean_price
    
    # Price percentile
    df_features[f'{price_column}_percentile'] = df[price_column].rank(pct=True)
    
    return df_features
```

### Data Enrichment

```python
def enrich_product_data(df):
    """Enrich product data with additional information"""
    df_enriched = df.copy()
    
    # Brand extraction from product name
    known_brands = ['Apple', 'Samsung', 'Sony', 'Nike', 'Adidas', 'Canon', 'Nikon']
    def extract_brand(name):
        if pd.isna(name):
            return 'Unknown'
        for brand in known_brands:
            if brand.lower() in name.lower():
                return brand
        return 'Other'
    
    if 'name' in df.columns:
        df_enriched['brand'] = df['name'].apply(extract_brand)
    
    # Category prediction based on keywords
    category_keywords = {
        'Electronics': ['phone', 'laptop', 'computer', 'camera', 'tablet'],
        'Clothing': ['shirt', 'pants', 'dress', 'shoes', 'jacket'],
        'Books': ['book', 'novel', 'guide', 'manual', 'textbook'],
        'Sports': ['ball', 'equipment', 'fitness', 'exercise', 'sport']
    }
    
    def predict_category(name):
        if pd.isna(name):
            return 'Unknown'
        name_lower = name.lower()
        for category, keywords in category_keywords.items():
            if any(keyword in name_lower for keyword in keywords):
                return category
        return 'Other'
    
    if 'name' in df.columns:
        df_enriched['predicted_category'] = df['name'].apply(predict_category)
    
    return df_enriched

def add_external_data(df, id_column, external_source):
    """Add external data based on ID column"""
    # This is a placeholder for external data enrichment
    # In practice, you might call APIs or query databases
    
    enrichment_data = {
        'comp001': {'industry': 'Technology', 'size': 'Large'},
        'comp002': {'industry': 'Retail', 'size': 'Medium'},
        # ... more data
    }
    
    df_enriched = df.copy()
    
    # Add external data
    for idx, row in df_enriched.iterrows():
        entity_id = row[id_column]
        if entity_id in enrichment_data:
            for key, value in enrichment_data[entity_id].items():
                df_enriched.at[idx, key] = value
    
    return df_enriched
```

## 6. Complete Data Processing Pipeline

### Production-Ready Pipeline

```python
import pandas as pd
import numpy as np
import re
from datetime import datetime
import logging

class DataProcessor:
    def __init__(self, config=None):
        self.config = config or {}
        self.setup_logging()
        
    def setup_logging(self):
        """Setup logging for the pipeline"""
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s'
        )
        self.logger = logging.getLogger(__name__)
    
    def process_scraped_data(self, raw_data, schema=None):
        """Main processing pipeline"""
        self.logger.info(f"Starting data processing for {len(raw_data)} records")
        
        # Step 1: Convert to DataFrame
        df = pd.DataFrame(raw_data)
        self.logger.info(f"Created DataFrame with shape {df.shape}")
        
        # Step 2: Basic cleaning
        df = self.basic_cleaning(df)
        
        # Step 3: Data validation
        if schema:
            df = self.validate_data(df, schema)
        
        # Step 4: Handle missing data
        df = self.handle_missing_data(df)
        
        # Step 5: Normalize data
        df = self.normalize_data(df)
        
        # Step 6: Feature engineering
        df = self.feature_engineering(df)
        
        # Step 7: Data enrichment
        df = self.enrich_data(df)
        
        # Step 8: Final validation
        df = self.final_validation(df)
        
        self.logger.info(f"Data processing completed. Final shape: {df.shape}")
        return df
    
    def basic_cleaning(self, df):
        """Basic data cleaning"""
        self.logger.info("Performing basic cleaning")
        
        # Remove completely empty rows
        df = df.dropna(how='all')
        
        # Strip whitespace from string columns
        string_columns = df.select_dtypes(include=['object']).columns
        for col in string_columns:
            df[col] = df[col].astype(str).str.strip()
        
        # Replace empty strings with NaN
        df = df.replace('', np.nan)
        
        return df
    
    def validate_data(self, df, schema):
        """Validate data against schema"""
        self.logger.info("Validating data")
        
        validator = DataValidator()
        valid_rows = []
        
        for idx, row in df.iterrows():
            is_valid, errors = validator.validate_record(row.to_dict(), schema)
            if is_valid:
                valid_rows.append(idx)
            else:
                self.logger.warning(f"Row {idx} failed validation: {errors}")
        
        df_valid = df.loc[valid_rows]
        self.logger.info(f"Validation complete. {len(valid_rows)} valid rows out of {len(df)}")
        
        return df_valid
    
    def handle_missing_data(self, df):
        """Handle missing data"""
        self.logger.info("Handling missing data")
        
        missing_summary = detect_missing_data(df)
        self.logger.info(f"Missing data summary: {missing_summary['total_missing']} missing values")
        
        # Apply filling strategy
        df_filled = fill_missing_data(df, strategy='smart')
        
        return df_filled
    
    def normalize_data(self, df):
        """Normalize data"""
        self.logger.info("Normalizing data")
        
        # Normalize specific columns if they exist
        if 'price' in df.columns:
            df['price'] = df['price'].apply(lambda x: normalize_price(str(x)) if pd.notna(x) else x)
        
        if 'phone' in df.columns:
            df['phone'] = df['phone'].apply(lambda x: normalize_phone(str(x)) if pd.notna(x) else x)
        
        if 'date' in df.columns:
            df['date'] = df['date'].apply(lambda x: normalize_date(str(x)) if pd.notna(x) else x)
        
        return df
    
    def feature_engineering(self, df):
        """Create new features"""
        self.logger.info("Engineering features")
        
        # Add timestamp
        df['processed_at'] = datetime.now()
        
        # Extract features from text columns
        text_columns = ['description', 'title', 'content']
        for col in text_columns:
            if col in df.columns:
                df = extract_features_from_text(df, col)
        
        # Create price features if price column exists
        if 'price' in df.columns:
            df = create_price_features(df, 'price')
        
        return df
    
    def enrich_data(self, df):
        """Enrich data with additional information"""
        self.logger.info("Enriching data")
        
        # Add product-specific enrichment
        if 'name' in df.columns or 'title' in df.columns:
            df = enrich_product_data(df)
        
        return df
    
    def final_validation(self, df):
        """Final validation and cleanup"""
        self.logger.info("Performing final validation")
        
        # Remove duplicates
        initial_count = len(df)
        df = df.drop_duplicates()
        final_count = len(df)
        
        if initial_count != final_count:
            self.logger.info(f"Removed {initial_count - final_count} duplicate rows")
        
        # Data quality assessment
        quality_report = assess_data_quality(df)
        self.logger.info("Data quality assessment completed")
        
        return df
    
    def save_processed_data(self, df, output_path, format='csv'):
        """Save processed data"""
        if format == 'csv':
            df.to_csv(output_path, index=False)
        elif format == 'json':
            df.to_json(output_path, orient='records', indent=2)
        elif format == 'parquet':
            df.to_parquet(output_path)
        
        self.logger.info(f"Data saved to {output_path}")

# Usage example
processor = DataProcessor()

# Sample raw data
raw_data = [
    {'name': 'iPhone 13', 'price': '$699.99', 'rating': '4.5 stars'},
    {'name': 'Samsung Galaxy', 'price': '€599.00', 'rating': '4.2/5'},
    {'name': '', 'price': '$invalid', 'rating': 'N/A'},
]

# Define schema
schema = {
    'name': 'text',
    'price': 'price',
    'rating': 'text'
}

# Process data
processed_df = processor.process_scraped_data(raw_data, schema)
print(processed_df.head())

# Save processed data
processor.save_processed_data(processed_df, 'processed_data.csv')
```

## Key Takeaways

- Data cleaning is crucial for reliable analysis
- Use regular expressions for pattern extraction
- Validate data quality at multiple stages
- Handle missing data intelligently
- Normalize data for consistency
- Create meaningful features from raw data
- Build reusable processing pipelines

Remember: Clean data is the foundation of good analysis! 