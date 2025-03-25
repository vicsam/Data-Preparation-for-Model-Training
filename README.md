# Water Companies Data Collection for Nigerian Farm Industry  

## 1. Project Objective  
The objective of this project was to collect and preprocess data about water companies in Lagos, Nigeria, as a foundational step toward developing a specialized model dedicated to the Nigerian farm industry. By focusing on the water sector, the dataset includes company names, locations, profile links, and other relevant information to train the **DeepSeek-R1-Distill-Qwen-1.5B-ONNX** model for applications in:  
- Agricultural water management  
- Supply chain optimization  
- Sustainability analysis  

## 2. Data Collection  

### 2.1 Source Identification  
- **Primary Sources:**  
  - [Lagos InfoisInfo](https://www.infoisinfo.ng/)  
  - [NigeriaGalleria](https://www.nigeriagalleria.com/)  

- **Reasoning:**  
  These websites provide publicly available directories of businesses operating in Lagos, including water companies.  

### 2.2 Web Scraping  

#### **Tools Used**  
- **Python Libraries:**  
  - `requests`  
  - `BeautifulSoup`  
  - `Selenium`  
  - `Playwright`  

- **Web Drivers:**  
  - ChromeDriver (for Selenium)  

#### **Scraping Steps**  
1. **Inspected the website structure** using browser developer tools.  
2. **Identified pagination patterns**:  
   - **Single-letter pages** (e.g., `Water_a2.html`, `Water_b.html`).  
   - **Paired-letter pages** (e.g., `Water_u-v.html`, `Water_y-z.html`).  
3. **Constructed a Python script** to scrape all relevant pages:  
   - Handled both **single-letter** and **paired-letter** URLs programmatically.  
   - Extracted **company names, locations, and profile links**.  
4. **Saved the scraped data** into a CSV file:  
   - Output file: `water_companies.csv`.  

### 2.3 Sample Code  
Below is the Python script used for data cleaning and preprocessing:  

```python
from playwright.sync_api import sync_playwright
from bs4 import BeautifulSoup
import csv

base_url = "https://www.nigeriagalleria.com/Food_Beverages/{}"
all_companies = []
single_letter_pages = [
    "Water_a2.html", "Water_b.html", "Water_c.html", "Water_d.html",
    "Water_e.html", "Water_f.html", "Water_g.html", "Water_h.html",
    "Water_i.html", "Water_j.html", "Water_k.html", "Water_l.html",
    "Water_m.html", "Water_n.html", "Water_o.html", "Water_p.html",
    "Water_q.html", "Water_s.html", "Water_t.html"
]
paired_letter_pages = ["Water_u-v.html", "Water_w-x.html", "Water_y-z.html"]

def clean_text(text):
    return text.strip() if text else "N/A"

def scrape_page(page, url):
    print(f"Scraping page: {url}")
    response = page.goto(url)
    if response.status == 200:
        html_content = page.content()
        soup = BeautifulSoup(html_content, 'html.parser')
        container = soup.find('div', class_='col-contentgen col-t-contentgen')
        if container:
            lines = container.get_text(separator="<br>").split("<br>")
            company = {}
            for line in lines:
                line = line.strip()
                if line.startswith("Address:"):
                    company["Location"] = clean_text(line.replace("Address:", ""))
                elif line and not line.startswith(("Phone:", "Fax:", "Email:", "Website:")):
                    if "Name" not in company:
                        company["Name"] = clean_text(line)
                    else:
                        all_companies.append(company)
                        company = {"Name": clean_text(line)}
    else:
        print(f"Failed to fetch page {url}. Status code: {response.status}")

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    page = browser.new_page()
    for page_name in single_letter_pages:
        url = base_url.format(page_name)
        scrape_page(page, url)
    for page_name in paired_letter_pages:
        url = base_url.format(page_name)
        scrape_page(page, url)
    browser.close()

keys = ["Name", "Location"]
with open("water_companies_full.csv", "w", newline="", encoding="utf-8") as csvfile:
    writer = csv.DictWriter(csvfile, fieldnames=keys)
    writer.writeheader()
    writer.writerows(all_companies)

print(f"Scraped {len(all_companies)} companies and saved to water_companies_full.csv")

```

# 3. Data Analysis

## 3.1 Initial Inspection
- Loaded the CSV file (`water_companies.csv`) into a Pandas DataFrame.
- Checked for missing values, duplicates, and inconsistencies in names and locations.

## 3.2 Cleaning
### Removed Irrelevant Entries
- Filtered out non-water-related companies (e.g., telecoms, construction).

### Standardized Formatting
- Ensured consistent formatting for company names and locations.
- Cleaned up extra spaces and special characters.

## 3.3 Insights
### Geographical Distribution
- Most companies are concentrated in industrial areas like Ogba, Surulere, Ilupeju, and Amuwo-Odofin.

### Market Leaders
- Identified key players such as Nestlé Nig PLC, Astral Water, and SO-Safe Water Technologies Ltd.

### Product Focus
- Differentiated between sachet water producers and bottled water manufacturers.

## 3.4 Sample Code
Below is the Python script used for analysis:

```python
import pandas as pd

df = pd.read_csv("water_companies.csv")
df = df[df["Name"].str.contains("Water|Pure|Drinks", case=False, na=False)]
df["Location"] = df["Location"].str.strip()
df["Name"] = df["Name"].str.strip()
df.to_csv("cleaned_water_companies.csv", index=False)
```
## 3.5 Three Additional Data Analyses

### 1. Distribution of Companies by Location

```python
import matplotlib.pyplot as plt

location_counts = df["Location"].value_counts().head(10)
plt.figure(figsize=(10,5))
location_counts.plot(kind="bar", color="skyblue")
plt.title("Top 10 Locations for Water Companies in Lagos")
plt.xlabel("Location")
plt.ylabel("Number of Companies")
plt.xticks(rotation=45)
plt.show()
```
## Insights

**1. Geographical Concentration of Water Companies**  
This visualization helps determine where water companies are most concentrated in Lagos.

---

### 2. Identifying Market Leaders

```python
df["Company_Length"] = df["Name"].apply(len)
top_companies = df.nlargest(10, "Company_Length")[["Name", "Location"]]
print(top_companies)
```

## Insights on Company Names

### 1. Longest Company Names  
The longest company names often belong to established brands with subsidiaries, helping identify market leaders.

### 2. Word Frequency in Company Names  

```python
from collections import Counter
import re

words = " ".join(df["Name"]).lower()
word_list = re.findall(r'\b[a-z]+\b', words)
common_words = Counter(word_list).most_common(10)

print("Top 10 Most Common Words in Company Names:", common_words)

````

**Insight:** Common words like *"water," "pure,"* and *"drinks"* show naming trends among water companies.

