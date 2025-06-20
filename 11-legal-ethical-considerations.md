# Legal & Ethical Considerations

## 🎯 Learning Objectives

By the end of this lesson, you will:
- Understand the legal landscape of web scraping
- Know how to identify and respect website terms of service
- Implement ethical scraping practices
- Understand robots.txt and its importance
- Learn about copyright and data protection laws
- Develop responsible scraping guidelines

## ⚖️ Legal Framework

### 1. Terms of Service (ToS)

**Always check and respect website Terms of Service:**

```python
def check_terms_of_service():
    """
    Manual checklist for Terms of Service review
    """
    checklist = [
        "Have I read the website's Terms of Service?",
        "Does the ToS explicitly prohibit automated access?",
        "Are there specific restrictions on data use?",
        "Do I need permission for commercial use?",
        "Are there rate limiting requirements?",
        "What data can I collect vs. what I should avoid?"
    ]
    
    for item in checklist:
        print(f"☐ {item}")
```

**Common ToS restrictions to watch for:**
- Prohibition of automated access
- Rate limiting requirements
- Restrictions on data redistribution
- Commercial use limitations
- Personal data collection rules

### 2. Copyright Law

**Understand what you can and cannot scrape:**

- **Facts and data**: Generally not copyrightable
- **Original content**: Protected by copyright
- **Database rights**: May apply to collections of data
- **Fair use**: Limited exceptions for research/criticism

```python
def content_classification():
    """
    Guide for classifying scrapeable content
    """
    guidelines = {
        "Generally OK to scrape": [
            "Public facts and data",
            "Prices and product specifications", 
            "Business listings and contact info",
            "News headlines (not full articles)",
            "Public social media posts (with care)"
        ],
        "Requires caution": [
            "Full news articles",
            "Product descriptions",
            "User-generated content",
            "Images and media files",
            "Structured databases"
        ],
        "Generally avoid": [
            "Copyrighted images/videos",
            "Premium/paid content",
            "Personal/private information",
            "Content behind paywalls",
            "Copyrighted creative works"
        ]
    }
    
    for category, items in guidelines.items():
        print(f"\n{category}:")
        for item in items:
            print(f"  • {item}")
```

### 3. Data Protection Laws

**Major regulations to be aware of:**

- **GDPR (EU)**: Strict rules for personal data
- **CCPA (California)**: Consumer privacy rights
- **PIPEDA (Canada)**: Personal information protection
- **National data protection laws**: Vary by country

```python
def gdpr_compliance_check():
    """
    GDPR compliance considerations for web scraping
    """
    considerations = {
        "Legal basis": "Do you have a legitimate interest?",
        "Data minimization": "Are you collecting only necessary data?",
        "Purpose limitation": "Is data used only for stated purpose?", 
        "Storage limitation": "Do you delete data when no longer needed?",
        "Rights of individuals": "Can people request data deletion?",
        "Data security": "Is scraped data properly secured?"
    }
    
    print("GDPR Compliance Checklist:")
    for principle, question in considerations.items():
        print(f"• {principle}: {question}")
```

## 🤖 Technical Ethics

### 1. robots.txt Protocol

**Always check and respect robots.txt:**

```python
import urllib.robotparser
import requests
from urllib.parse import urljoin

class RobotsChecker:
    def __init__(self, base_url):
        self.base_url = base_url.rstrip('/')
        self.robots_url = urljoin(self.base_url, '/robots.txt')
        self.rp = urllib.robotparser.RobotFileParser()
        self.load_robots()
    
    def load_robots(self):
        """Load and parse robots.txt"""
        try:
            self.rp.set_url(self.robots_url)
            self.rp.read()
            print(f"✓ Successfully loaded robots.txt from {self.robots_url}")
            return True
        except Exception as e:
            print(f"⚠ Could not load robots.txt: {e}")
            return False
    
    def can_fetch(self, url, user_agent='*'):
        """Check if URL can be fetched"""
        return self.rp.can_fetch(user_agent, url)
    
    def get_crawl_delay(self, user_agent='*'):
        """Get recommended crawl delay"""
        return self.rp.crawl_delay(user_agent)
    
    def get_request_rate(self, user_agent='*'):
        """Get request rate if specified"""
        return self.rp.request_rate(user_agent)
    
    def show_robots_summary(self):
        """Display robots.txt summary"""
        try:
            response = requests.get(self.robots_url)
            if response.status_code == 200:
                print(f"\nRobots.txt content for {self.base_url}:")
                print("-" * 50)
                print(response.text)
                print("-" * 50)
            else:
                print(f"No robots.txt found (HTTP {response.status_code})")
        except Exception as e:
            print(f"Error fetching robots.txt: {e}")

# Usage example
checker = RobotsChecker('https://example.com')
checker.show_robots_summary()

if checker.can_fetch('https://example.com/products'):
    delay = checker.get_crawl_delay()
    print(f"✓ Can scrape with delay: {delay} seconds")
else:
    print("✗ Scraping not allowed by robots.txt")
```

### 2. Rate Limiting & Server Respect

**Implement respectful request patterns:**

```python
import time
import requests
from datetime import datetime, timedelta

class RespectfulScraper:
    def __init__(self, delay=1.0, max_requests_per_minute=30):
        self.delay = delay
        self.max_requests_per_minute = max_requests_per_minute
        self.request_times = []
        self.session = requests.Session()
        
        # Set a reasonable User-Agent
        self.session.headers.update({
            'User-Agent': 'Research Bot 1.0 (Educational Purpose)'
        })
    
    def _wait_if_needed(self):
        """Implement rate limiting"""
        now = datetime.now()
        
        # Remove requests older than 1 minute
        self.request_times = [
            req_time for req_time in self.request_times 
            if now - req_time < timedelta(minutes=1)
        ]
        
        # Check if we need to wait
        if len(self.request_times) >= self.max_requests_per_minute:
            wait_time = 60 - (now - self.request_times[0]).total_seconds()
            if wait_time > 0:
                print(f"Rate limit reached. Waiting {wait_time:.1f} seconds...")
                time.sleep(wait_time)
        
        # Add mandatory delay between requests
        if self.request_times:
            last_request = self.request_times[-1]
            time_since_last = (now - last_request).total_seconds()
            if time_since_last < self.delay:
                time.sleep(self.delay - time_since_last)
    
    def get(self, url):
        """Make a respectful GET request"""
        self._wait_if_needed()
        
        try:
            response = self.session.get(url, timeout=10)
            self.request_times.append(datetime.now())
            
            # Be extra careful with server errors
            if response.status_code >= 500:
                print(f"Server error {response.status_code}. Backing off...")
                time.sleep(self.delay * 5)  # Extra delay for server issues
            
            return response
        except Exception as e:
            print(f"Request failed: {e}")
            return None

# Usage
scraper = RespectfulScraper(delay=2.0, max_requests_per_minute=20)
response = scraper.get('https://example.com')
```

### 3. Data Usage Guidelines

**Ethical data handling practices:**

```python
class EthicalDataHandler:
    def __init__(self):
        self.personal_data_patterns = [
            r'\b\d{3}-\d{2}-\d{4}\b',  # SSN pattern
            r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',  # Email
            r'\b\d{3}[\s.-]?\d{3}[\s.-]?\d{4}\b',  # Phone numbers
        ]
    
    def sanitize_content(self, content):
        """Remove potential personal information"""
        import re
        
        sanitized = content
        for pattern in self.personal_data_patterns:
            sanitized = re.sub(pattern, '[REDACTED]', sanitized, flags=re.IGNORECASE)
        
        return sanitized
    
    def data_usage_guidelines(self):
        """Display ethical data usage guidelines"""
        guidelines = {
            "Collection": [
                "Only collect data you actually need",
                "Avoid personal/sensitive information when possible",
                "Document what data you're collecting and why",
                "Check if consent is required"
            ],
            "Storage": [
                "Secure stored data appropriately",
                "Set data retention policies",
                "Implement access controls",
                "Consider encryption for sensitive data"
            ],
            "Usage": [
                "Use data only for stated purposes",
                "Don't sell or redistribute without permission",
                "Respect attribution requirements",
                "Consider impact on data subjects"
            ],
            "Sharing": [
                "Anonymize data before sharing",
                "Check legal requirements for data transfer",
                "Obtain necessary permissions",
                "Document data sharing agreements"
            ]
        }
        
        for category, rules in guidelines.items():
            print(f"\n{category}:")
            for rule in rules:
                print(f"  ✓ {rule}")

# Usage
handler = EthicalDataHandler()
handler.data_usage_guidelines()
```

## 🚫 What NOT to Do

### 1. Prohibited Practices

```python
def scraping_red_flags():
    """
    Practices that are likely problematic or illegal
    """
    prohibited = {
        "Technical violations": [
            "Ignoring robots.txt completely",
            "Making thousands of rapid requests",
            "Bypassing rate limiting without permission",
            "Using fake/misleading User-Agent strings",
            "Circumventing technical protection measures"
        ],
        "Legal violations": [
            "Scraping copyrighted content for republication",
            "Collecting personal data without legal basis",
            "Violating clearly stated Terms of Service",
            "Scraping content behind authentication",
            "Using scraped data for harassment/spam"
        ],
        "Ethical violations": [
            "Overloading servers (causing denial of service)",
            "Scraping competitors for harmful purposes",
            "Collecting private/sensitive information",
            "Republishing content without attribution",
            "Ignoring takedown requests"
        ]
    }
    
    print("🚫 AVOID THESE PRACTICES:")
    for category, practices in prohibited.items():
        print(f"\n{category}:")
        for practice in practices:
            print(f"  ✗ {practice}")

scraping_red_flags()
```

### 2. Common Legal Pitfalls

```python
def legal_pitfalls():
    """
    Common mistakes that can lead to legal issues
    """
    pitfalls = [
        {
            "Issue": "Assuming public = freely usable",
            "Reality": "Public data may still be copyrighted or protected",
            "Solution": "Check copyright and terms of use"
        },
        {
            "Issue": "Not reading Terms of Service",
            "Reality": "ToS violations can lead to lawsuits",
            "Solution": "Always review and follow website ToS"
        },
        {
            "Issue": "Ignoring data protection laws",
            "Reality": "GDPR and similar laws have strict penalties",
            "Solution": "Understand applicable privacy regulations"
        },
        {
            "Issue": "Republishing without permission",
            "Reality": "Can be copyright infringement",
            "Solution": "Get permission or ensure fair use applies"
        },
        {
            "Issue": "Not identifying your bot",
            "Reality": "Deceptive practices may violate laws",
            "Solution": "Use clear, honest User-Agent strings"
        }
    ]
    
    print("⚠️ COMMON LEGAL PITFALLS:")
    for pitfall in pitfalls:
        print(f"\n• {pitfall['Issue']}")
        print(f"  Reality: {pitfall['Reality']}")
        print(f"  Solution: {pitfall['Solution']}")

legal_pitfalls()
```

## ✅ Best Practices

### 1. Pre-Scraping Checklist

```python
def pre_scraping_checklist():
    """
    Comprehensive checklist before starting any scraping project
    """
    checklist = {
        "Legal Review": [
            "Read and understand the website's Terms of Service",
            "Check for any API alternatives",
            "Review applicable copyright laws", 
            "Consider data protection regulations (GDPR, CCPA, etc.)",
            "Evaluate fair use vs. commercial use"
        ],
        "Technical Review": [
            "Check robots.txt file",
            "Identify appropriate crawl delays",
            "Plan respectful request patterns",
            "Choose appropriate User-Agent",
            "Design error handling and backoff strategies"
        ],
        "Ethical Review": [
            "Define legitimate business/research purpose",
            "Minimize data collection to what's necessary",
            "Plan for data security and retention",
            "Consider impact on website and users",
            "Prepare for takedown requests"
        ],
        "Documentation": [
            "Document your scraping purpose and methods",
            "Keep records of permissions obtained",
            "Maintain compliance documentation",
            "Plan for audit trails",
            "Create incident response procedures"
        ]
    }
    
    print("📋 PRE-SCRAPING CHECKLIST:")
    for category, items in checklist.items():
        print(f"\n{category}:")
        for item in items:
            print(f"  ☐ {item}")

pre_scraping_checklist()
```

### 2. Responsible Scraping Framework

```python
class ResponsibleScraper:
    """
    Framework for ethical web scraping
    """
    def __init__(self, target_site, purpose):
        self.target_site = target_site
        self.purpose = purpose
        self.compliance_log = []
        
    def legal_compliance_check(self):
        """Check legal compliance requirements"""
        checks = [
            "Terms of Service reviewed",
            "Robots.txt checked", 
            "Copyright implications considered",
            "Data protection laws reviewed",
            "API alternatives investigated"
        ]
        
        print(f"Legal compliance for {self.target_site}:")
        for check in checks:
            # In practice, these would be actual checks
            status = "✓"  # or "✗" based on actual compliance
            print(f"  {status} {check}")
            self.compliance_log.append(f"{check}: {status}")
    
    def technical_compliance_setup(self):
        """Set up technical compliance measures"""
        measures = {
            "User-Agent": f"ResponsibleBot/1.0 (Purpose: {self.purpose})",
            "Crawl-Delay": "2 seconds minimum",
            "Request-Rate": "30 requests per minute maximum",
            "Error-Handling": "Exponential backoff on errors",
            "Respectful-Hours": "Avoid peak traffic times"
        }
        
        print("\nTechnical compliance measures:")
        for measure, setting in measures.items():
            print(f"  • {measure}: {setting}")
    
    def data_handling_policy(self):
        """Define data handling policy"""
        policy = {
            "Collection": "Minimal necessary data only",
            "Storage": "Secure, encrypted storage",
            "Retention": "Delete after purpose fulfilled",
            "Access": "Restricted to authorized personnel",
            "Sharing": "Only with explicit permission"
        }
        
        print("\nData handling policy:")
        for aspect, rule in policy.items():
            print(f"  • {aspect}: {rule}")

# Usage
scraper = ResponsibleScraper("example.com", "Academic research on pricing trends")
scraper.legal_compliance_check()
scraper.technical_compliance_setup()
scraper.data_handling_policy()
```

## 📋 Takedown and Incident Response

### 1. Handling Takedown Requests

```python
def takedown_response_plan():
    """
    Plan for responding to takedown requests
    """
    response_steps = [
        "Immediately stop scraping the specified content",
        "Acknowledge receipt of the request promptly", 
        "Review the validity of the request",
        "Remove/delete requested data if appropriate",
        "Document the incident for compliance records",
        "Implement measures to prevent recurrence",
        "Follow up with the requesting party"
    ]
    
    print("🚨 TAKEDOWN REQUEST RESPONSE PLAN:")
    for i, step in enumerate(response_steps, 1):
        print(f"{i}. {step}")
    
    print("\nSample response template:")
    template = """
Subject: Re: Takedown Request for [Website/Content]

Dear [Requester],

We have received your request dated [DATE] regarding content scraped from [WEBSITE]. 

We take these requests seriously and have immediately:
- Stopped all automated access to the specified content
- Initiated a review of our data collection practices
- [Additional specific actions taken]

We will complete our review within [TIMEFRAME] and provide an update on our response.

Thank you for bringing this to our attention.

Best regards,
[Your contact information]
"""
    print(template)
```

### 2. Incident Documentation

```python
import datetime
import json

class ComplianceLogger:
    def __init__(self, project_name):
        self.project_name = project_name
        self.log_file = f"{project_name}_compliance_log.json"
        self.incidents = []
    
    def log_incident(self, incident_type, description, action_taken):
        """Log compliance incidents"""
        incident = {
            "timestamp": datetime.datetime.now().isoformat(),
            "type": incident_type,
            "description": description,
            "action_taken": action_taken,
            "status": "recorded"
        }
        
        self.incidents.append(incident)
        self._save_log()
        print(f"Incident logged: {incident_type}")
    
    def log_compliance_check(self, check_type, result, notes=""):
        """Log compliance verification"""
        check = {
            "timestamp": datetime.datetime.now().isoformat(),
            "type": "compliance_check",
            "check_type": check_type,
            "result": result,
            "notes": notes
        }
        
        self.incidents.append(check)
        self._save_log()
    
    def _save_log(self):
        """Save log to file"""
        try:
            with open(self.log_file, 'w') as f:
                json.dump({
                    "project": self.project_name,
                    "created": datetime.datetime.now().isoformat(),
                    "entries": self.incidents
                }, f, indent=2)
        except Exception as e:
            print(f"Error saving log: {e}")

# Usage
logger = ComplianceLogger("ecommerce_price_monitoring")
logger.log_compliance_check("robots.txt", "compliant", "Respecting crawl delay of 2 seconds")
logger.log_incident("rate_limit", "Received 429 status", "Increased delay to 5 seconds")
```

## 🎓 Key Takeaways

### Legal Principles
1. **Always read Terms of Service** - They are legally binding contracts
2. **Respect robots.txt** - It's the website's explicit guidance  
3. **Understand copyright** - Facts vs. creative expression
4. **Know data protection laws** - GDPR, CCPA, and local regulations
5. **Document everything** - Keep records of compliance efforts

### Ethical Guidelines
1. **Be transparent** - Identify your bot honestly
2. **Be respectful** - Don't overload servers
3. **Be minimal** - Collect only what you need
4. **Be secure** - Protect any data you collect
5. **Be responsive** - Address complaints promptly

### Technical Standards
1. **Follow robots.txt** - Check and obey restrictions
2. **Implement rate limiting** - Be considerate of server resources
3. **Handle errors gracefully** - Back off when problems occur
4. **Use appropriate headers** - Don't masquerade as regular browsers
5. **Monitor your impact** - Watch for signs of problems

## 🚀 Moving Forward

Before starting any scraping project:

1. **Conduct a legal review** using the checklists provided
2. **Check for API alternatives** - They're usually better than scraping
3. **Design with compliance in mind** - Build respect into your architecture
4. **Document your compliance efforts** - Keep detailed records
5. **Prepare for incidents** - Have response plans ready

Remember: **The goal is to extract value while respecting rights, laws, and the broader web ecosystem.**

When in doubt, consult with legal experts and err on the side of caution. The web scraping landscape continues to evolve, and staying informed about legal developments is crucial for responsible practitioners. 