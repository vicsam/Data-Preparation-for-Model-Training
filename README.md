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
  - Corporate Affairs Commission (CAC): Strike-off lists provided in PDF format. 

- **Reasoning:**  
  These websites provide publicly available directories of businesses operating in Lagos, including water companies. The CAC strike-off list ensures inactive or defunct companies are excluded from the dataset.
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

# CAC Strike-Off List

## 4.1 Purpose
To ensure the dataset only includes active water companies, we obtained the latest strike-off list from the Corporate Affairs Commission (CAC). This list identifies companies marked for deregistration due to non-compliance or inactivity.

## 4.2 Process

### Request Data from CAC:
- Received the strike-off list in PDF format titled **STRIKE-OFF-LIST-UPDATED-NOV-23-2023.pdf**.

### Convert PDF to Excel:
- Used tools like **Tabula (tabula-py)** or **Adobe Acrobat** to extract tabular data from the PDF.
- Saved the extracted data as an Excel file: **STRIKE-OFF-LIST-UPDATED-NOV-23-2023.xlsx**.

### Export to CSV:
- Converted the Excel file to a CSV file for easier processing: **STRIKE-OFF-LIST-UPDATED-NOV-23-2023.csv**.

## 4.3 Cleaning Script

```python
import pandas as pd
import sys

# File paths
input_file_path = r"C:\Users\YourUsername\Documents\water-data\STRIKE-OFF-LIST-UPDATED-NOV-23-2023.csv"
output_file_path = r"C:\Users\YourUsername\Documents\water-data\cleaned_companies.csv"

# Function to print progress with timestamp
def print_progress(message):
    print(f"[Progress - {pd.Timestamp.now()}] {message}", file=sys.stdout, flush=True)

try:
    # Step 1: Read the CSV
    print_progress("Starting to read the CSV file...")
    df = pd.read_csv(input_file_path, dtype=str, engine='python')
    print_progress(f"Successfully read the CSV file with {len(df)} initial rows.")

    # Step 2: Rename columns for clarity
    print_progress("Renaming columns...")
    df = df.rename(columns={
        'SN': 'SN',
        'Company_Name': 'Company_Name',
        'Registration_RNeugmisbterartion_Date': 'Registration_RNeugmisbterartion_Date'
    })

    # Step 3: Drop unnecessary columns
    print_progress("Dropping 'Unnamed: 1' column if it exists...")
    df = df.drop(columns=['Unnamed: 1'], errors='ignore')

    # Step 4: Remove duplicate header rows
    print_progress("Removing duplicate header rows where Company_Name is 'Company_Name'...")
    initial_rows = len(df)
    df = df[df['Company_Name'] != 'Company_Name']
    print_progress(f"Removed {initial_rows - len(df)} duplicate header rows. Remaining rows: {len(df)}")

    # Step 5: Split Registration_Number and Date
    print_progress("Splitting Registration_RNeugmisbterartion_Date into Registration_Number and Date...")
    df[['Registration_Number', 'Date']] = df['Registration_RNeugmisbterartion_Date'].str.split(' ', n=1, expand=True)
    df = df.drop(columns=['Registration_RNeugmisbterartion_Date'])
    print_progress("Split completed.")

    # Step 6: Standardize Date to YYYY-MM-DD
    print_progress("Standardizing Date format to YYYY-MM-DD...")
    def standardize_date(date_str):
        if pd.isna(date_str):
            return None
        if isinstance(date_str, str):
            if len(date_str) == 8 and date_str.endswith('19'):
                date_str += "91"  # Assume 1991 for partial dates
            parts = date_str.split('/')
            if len(parts) == 3:
                day, month, year = parts
                day = day.zfill(2)
                month = month.zfill(2)
                year = year.zfill(4)
                return f"{year}-{month}-{day}"
        return date_str

    df['Date'] = df['Date'].apply(standardize_date)
    print_progress("Date standardization completed.")

    # Step 7: Renumber SN sequentially
    print_progress("Renumbering SN sequentially starting from 1...")
    df = df.reset_index(drop=True)
    df['SN'] = range(1, len(df) + 1)
    print_progress(f"Renumbered SN for {len(df)} rows.")

    # Step 8: Reorder columns
    print_progress("Reordering columns to match desired structure...")
    df = df[['SN', 'Company_Name', 'Registration_Number', 'Date']]
    print_progress("Column reordering completed.")

    # Step 9: Save cleaned dataset
    print_progress("Saving the cleaned dataset to a new CSV file...")
    df.to_csv(output_file_path, index=False)
    print_progress(f"Cleaned dataset saved to {output_file_path}")

except Exception as e:
    print(f"[Error - {pd.Timestamp.now()}] An error occurred: {e}")
finally:
    print(f"[Finally - {pd.Timestamp.now()}] Script execution completed. Check {output_file_path} for the cleaned dataset.")

```

## 4.4 Insights

### Exclusion of Inactive Companies:
- Analyzed companies marked for strike-off by the CAC to exclude inactive or defunct entities.

### Standardization of Dates:
- Ensured all registration dates follow the **YYYY-MM-DD** format for consistency.

# 5. Merging Datasets

## Purpose
To consolidate the web-scraped data (**water_companies.csv**) with the CAC strike-off list (**cleaned_companies.csv**), ensuring a comprehensive and accurate dataset.

## Steps
1. Loaded both datasets into Pandas DataFrames.
2. Filtered out inactive or defunct companies from the CAC strike-off list.
3. Merged the two datasets based on common fields (e.g., **Company_Name**).
4. Ensured no duplicate entries in the final dataset.

## Sample Code

```python
import pandas as pd

# Load datasets
web_scraped_data = pd.read_csv(r"C:\Users\YourUsername\Documents\water-data\water_companies.csv")
cac_strike_off_data = pd.read_csv(r"C:\Users\YourUsername\Documents\water-data\cleaned_companies.csv")

# Filter out inactive companies
active_companies = cac_strike_off_data[cac_strike_off_data['Date'].isnull()]  # Null dates indicate active status

# Merge datasets
merged_data = pd.merge(
    web_scraped_data,
    active_companies[['Company_Name', 'Registration_Number']],
    on='Company_Name',
    how='left'
)

# Save merged dataset
merged_data.to_csv(r"C:\Users\YourUsername\Documents\water-data\merged_water_companies.csv", index=False)
print("Merged dataset saved successfully!")

```
# 6. Conversion to JSON

## Purpose
JSON files are lightweight and easily consumable by machine learning models. Converting the merged CSV data into JSON ensures compatibility with the **DeepSeek-R1-Distill-Qwen-1.5B-ONNX** model.

## Steps
1. Loaded the merged CSV file (**merged_water_companies.csv**) into a Pandas DataFrame.
2. Converted the DataFrame into a JSON format suitable for model consumption.
3. Saved the JSON file for training.

## Sample Code

```python
import pandas as pd
import json

# Load merged dataset
df = pd.read_csv(r"C:\Users\YourUsername\Documents\water-data\merged_water_companies.csv")

# Convert DataFrame to JSON
json_data = df.to_dict(orient="records")

# Save JSON file
with open(r"C:\Users\YourUsername\Documents\water-data\water_companies.json", "w", encoding="utf-8") as jsonfile:
    json.dump(json_data, jsonfile, ensure_ascii=False, indent=4)

print("JSON file generated successfully!")

```

# 7. Summary

## Key Steps

### Data Collection:
- Scraped water company data from public directories.
- Obtained the CAC strike-off list in PDF format, converted it to CSV, and cleaned it.

### Data Cleaning:
- Standardized formatting for both datasets.
- Filtered out inactive or defunct companies.
- Merged the datasets into a single comprehensive file.

### Data Conversion:
- Converted the cleaned CSV data into JSON format for model consumption.

### Model Feeding:
- Prepared the JSON data for training the **DeepSeek-R1-Distill-Qwen-1.5B-ONNX** model.

## Tools Used
- **Web Scraping:** `requests`, `BeautifulSoup`, `Selenium`, `Playwright`.
- **Data Analysis:** `pandas`.
- **PDF Conversion:** `tabula-py`, **Adobe Acrobat**.
- **JSON Conversion:** Python’s built-in `json` library.

## Outputs
- **Web Scraped CSV:** `water_companies.csv`.
- **CAC Strike-Off CSV:** `cleaned_companies.csv`.
- **Merged CSV:** `merged_water_companies.csv`.
- **JSON File:** `water_companies.json`.
- **JSON Lines File:** `water_companies.jsonl`.

  

# 8. Conclusion

This documentation outlines the complete process of collecting, analyzing, and preparing data for training the **DeepSeek-R1-Distill-Qwen-1.5B-ONNX** model. By integrating data from multiple sources (**public directories** and the **CAC strike-off list**), we ensured that the dataset was accurate, clean, and compatible with the model's requirements.

The collected data on **water companies in Lagos** serves as a foundational step toward developing specialized models for the **Nigerian farm industry**. This dataset will be instrumental in applications such as:
- **Agricultural water management**
- **Supply chain optimization**
- **Sustainability analysis**

Additionally, the challenges encountered while running the **DeepSeek-R1-Distill-Qwen-1.5B-ONNX** model locally have been documented in the following GitHub repository:

🔗 **[GitHub Repository](https://github.com/vicsam/DeepSeek-R1-Distill-Qwen-1.5B-ONNX-Locally-Doc)**

This repository contains:
- Detailed issues faced during local deployment.
- Solutions implemented to resolve them.
- Valuable insights for future users and contributors.

It is important to note that this work represents only a **fragment** of the broader initiative to develop **comprehensive models tailored to Nigeria's agricultural and industrial needs**. Future phases will:
- Expand the **scope of data collection**.
- Refine **model performance**.
- Incorporate additional **sectors critical to the Nigerian economy**.

