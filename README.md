# SEC Financial Statement Dashboard

A dashboard that pulls annual financial data for public companies straight from the SEC's public records, cleans it up, and turns it into an interactive tool for comparing companies over time.

Data source comes from a free, public government source, and refreshes whenever a company files a new annual report.

**Companies:** Apple, Microsoft, Amazon

**Tracked:** Revenue, Total Assets, Total Liabilities, Accounts Payable

**Coverage:** Fiscal years 2007 to 2025, 212 data points total

---

## What it looks like

**Company overview.** Pick one company and see its key numbers, a trend line, and year-over-year growth.

<img width="1000" height="600" alt="Overview_page" src="https://github.com/user-attachments/assets/4c6700b1-63ae-4a46-9b25-3ffeabe00850" />


For example, here is how Apple (AAPL) should look like:

<img width="1000" height="700" alt="AAPL_Overview" src="https://github.com/user-attachments/assets/d86c4625-c976-4b69-9883-820282f2a41d" />


**Comparison.** All three companies side by side, growth rebased to a common starting point so differently sized companies are easy to compare.

<img width="1000" height="700" alt="Comparision_page" src="https://github.com/user-attachments/assets/46b0da1b-fc3d-47f0-b9d8-f4eb6e0b5d65" />


**Filing details.** Every number traced back to the actual SEC filing it came from.

<img width="1000" height="700" alt="Filings_page" src="https://github.com/user-attachments/assets/9c73ed29-5e2f-4005-8231-b048187c05ff" />

**Data Modeling View**

<img width="1000" height="700" alt="Data_Modeling" src="https://github.com/user-attachments/assets/287d06a1-f272-4a5c-8ceb-fd1a6da6035b" />


---

## How it's built

Two steps:

1. **Excel (Power Query)** pulls the raw data from the SEC's free API, cleans it up (removing duplicate or restated figures and quarterly numbers mistakenly filed as annual), and combines everything into one simple table.

2. **Power BI** turns that table into the dashboard: a data model plus the ratios (leverage, asset turnover and so on) used on the Comparison page.

Power BI reads the Excel file directly

Adding a fourth company later: pull its data the same way and the dashboard picks it up automatically with no changes needed to the model or the visuals.

---

## Adding a new company, step by step

The example below walks through adding Alphabet (ticker GOOGL) to show how the whole pipeline actually works, start to finish, in Excel.

**Before you start:** look up the company's CIK number (its SEC filer ID) on the SEC's [company search](https://www.sec.gov/cgi-bin/browse-edgar).

1. **Open the workbook and the Power Query editor.**
   Open `SEC_Data.xlsx`. The `financial_combined` sheet holds the finished result, one row per company, metric, and year. To see how it's built: Data tab -> Get Data -> Launch Power Query Editor.

<img width="928" height="736" alt="lauch-editor" src="https://github.com/user-attachments/assets/4430a7d8-54a1-4df8-b559-8acf20176e1b" />


2. **Find which SEC tags the company uses.**
   Companies don't all label the same thing the same way, and labels change over time. Run the `fn_FindTag` helper with the company's CIK and a keyword like "revenue." Repeat with "sales," "liabilit," and "payable" so nothing gets missed.

<img width="1297" height="800" alt="findtag" src="https://github.com/user-attachments/assets/32a4a2fd-5145-4085-afef-620b278066c9" />


3. **Pull each number.**
   Run the `fn_GetConcept` helper once for Assets, once for Liabilities, and once for Accounts Payable, using the CIK and the exact tag name found in step 2. Each run comes back as a clean table, one row per fiscal year.

<img width="1296" height="800" alt="getconcepts" src="https://github.com/user-attachments/assets/a77895dc-052d-49ba-86bc-6252e48398a6" />


4. **Build Revenue.**
   Revenue almost always needs more than one tag stitched together, since the label companies use for it tends to change over the years. Pull each tag, combine the results into one table, and remove any years that show up twice.


5. **Handle the occasional missing Liabilities tag**
   A few companies never file a single "Total Liabilities" tag. When that happens, it can be calculated instead: Total Assets minus Shareholders' Equity.

6. **Name the new tables so the model finds them automatically**
   Rename each finished table to the pattern `fact_TICKER_Concept`, for example `fact_GOOGL_Revenue`. That naming pattern is the only thing that tells the workbook which tables belong in the combined dataset, nothing else needs to change.


7. **Refresh the combined table.**
   Right-click `Financial_Combined` and refresh. It automatically finds every table named `fact_...` and stacks them together, so the new company's rows appear right alongside the existing ones.


8. **Save it back to the sheet.**
   Home tab, Close & Load. This writes the updated table into the `financial_combined` sheet that Power BI reads from.

<img width="1099" height="609" alt="data-table" src="https://github.com/user-attachments/assets/e0b6d032-460f-4921-92b9-4cfca22d3573" />


9. **Double check the work.**
   Confirm the row count grew by roughly what you'd expect and spot check one number against the company's actual 10-K filing before trusting it.

10. **Refresh Power BI.**
    Open the Power BI file and refresh. Company and metric lists are read straight from the data rather than hardcoded, so the new company shows up on every page without anyone touching the model or the visuals.

---

## Dashboard features

**Page navigation.** Three buttons on the left switch between Company overview, Comparison and Filing details instantly, no tabs or scrolling required. The page you're on is highlighted so you always know where you are. Hold Ctrl + Click to the buttons to switch pages and "Reset" pages.

**Reset button.** Clears every filter and slicer back to its default state in one click, useful after digging into one company or year range and wanting a clean slate.

**Dynamic title.** The heading at the top of the page rewrites itself based on what's selected, for example "Apple Inc., revenue, FY2007 to FY2025," so it's always clear exactly what's being shown.

**Company, Metric, View mode and Fiscal year range filters.** Pick a company, pick a metric (Revenue, Assets, Liabilities, Accounts Payable), pick how to view it (raw dollars, year-over-year percent, or indexed to 100), and narrow the year range. These carry over between pages.

**KPI cards with year-over-year change.** Each headline number shows its change from the prior year underneath it, colored green for growth and red for decline, so the trend is visible without reading the chart.

---

## Some key notes

1. **Some years are missing** 

Companies weren't required to tag their filings electronically until 2009, so a number of cells (mostly 2007 to 2009) are blank for some companies and metrics. The dashboard shows these as gaps rather than as zeros, so it never quietly implies a number that was never filed.

2. **Fiscal years don't line up across companies** 

Apple's fiscal year ends in late September, Microsoft's in June, Amazon's in December. "FY2025" covers a different twelve month window for each company, worth keeping in mind when comparing them directly.

---

## If you want to run this yourself

1. Open the Excel workbook and enter your name and email where it asks for a "User-Agent" in both `fn_GetConcept` and `fn_FindTag` by open the advancce query editor. The SEC requires this on every request.
2. Refresh the workbook to pull the latest data.
3. Open the Power BI file and point it at your copy of the workbook.
