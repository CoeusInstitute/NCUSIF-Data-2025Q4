# OmniCUniverse-2025Q4 — Comprehensive Repository Reference Guide

**Prepared for:** Bolt.new Dev Agent (Supabase Schema & App Development)
**Repository:** `https://github.com/CoeusInstitute/OmniCUniverse-2025Q4`
**Data Period:** Primarily 2025 Q3–Q4 (with historical trend data back to ~2014)
**Total Size:** ~81 MB across 6 directories and 50+ files
**Prepared by:** Coeus Institute

---

## GLOSSARY OF KEY ACRONYMS

Before working with this data, understand these acronyms precisely:

| Acronym | Full Name | Meaning |
|---------|-----------|---------|
| **NCUA** | National Credit Union Administration | Independent federal agency that charters, regulates, and insures federal credit unions |
| **NCUSIF** | National Credit Union Share Insurance Fund | The federal insurance fund (backed by U.S. government) that insures member deposits at federally insured credit unions up to $250,000 |
| **FCU** | Federal Credit Union | Federally chartered, federally insured credit union |
| **FISCU** | Federally Insured State-Chartered Credit Union | State chartered credit union that opted into NCUA federal insurance |
| **NFICU** | Not Federally Insured Credit Union | State chartered credit union without NCUA insurance |
| **FICU** | Federally Insured Credit Union | Umbrella term for both FCU and FISCU |
| **5300 Call Report** | NCUA Form 5300 | Quarterly financial and statistical report all FICUs must file with NCUA |
| **FOICU** | Financial and Operational Information — Credit Unions | Master table in 5300 data containing institutional profile fields (name, location, charter type, peer group, etc.) |
| **FS220** | Financial Statement 220 | The core balance sheet / financial statement table in the 5300 Call Report data |
| **FS220A–S** | Financial Statement 220 sub-tables | Supplemental financial data tables (loans, delinquencies, investments, derivatives, etc.) |
| **FPR** | Financial Performance Report | NCUA's standardized financial summary report for a credit union (assets, liabilities, capital, income/expenses, ratios) |
| **CAMELS** | Capital, Asset quality, Management, Earnings, Liquidity, Sensitivity | Regulatory rating system (1=best, 5=worst) used by NCUA examiners to assess credit union health |
| **CUSO** | Credit Union Service Organization | A company that provides services to credit unions (technology, lending, payments, etc.) |
| **TIP** | Trade, Industry, and Profession | A type of credit union field of membership based on occupational groups |
| **FOM** | Field of Membership | The defined group of people eligible to join a particular credit union |
| **MDI** | Minority Depository Institution | A credit union where a majority of current members, board, and community served are minority |
| **RSSD** | Research Statistics Supervision Discount (ID) | Unique identifier assigned to financial institutions by the Federal Reserve |
| **ROA** | Return on Average Assets | Key profitability metric for credit unions |
| **YoY** | Year over Year | Comparison metric (current period vs. same period prior year) |
| **CECL** | Current Expected Credit Losses | Accounting standard (ASC 326) for recognizing credit losses |
| **SMSA** | Standard Metropolitan Statistical Area | Geographic classification used in FOICU data |
| **DXA/EMU** | Document XML Address / English Metric Units | Not data-related — ignore if seen in tooling |

---

## REPOSITORY STRUCTURE OVERVIEW

```
OmniCUniverse-2025Q4/
├── call-report-data-2025Q4/                    [~51 MB] NCUA 5300 Call Report raw data
├── financial-performance-report-september-2025/ [~1.8 MB] Aggregate FPR workbooks
├── insurance-report-activity-dec-2025/          [~154 KB] NCUA charter/merger/insurance activity
├── Various Credit Union Industry Excel Data Tables/ [~769 KB] Global & regional CU datasets
├── PDF Reports/                                [~5.9 MB] NCUA analytical PDF publications
└── NCUSIF 2025 Q4 - Credit Union Industry Summary Data/ [~3 MB] Visual summary dashboards (PNG)
```

---

## 1. DIRECTORY: `call-report-data-2025Q4/`

### Folder Summary
This is the largest and most granular dataset in the repository — the complete NCUA 5300 Call Report quarterly data dump for the cycle ending **September 30, 2025**. It contains individual-level financial data for all **4,419 federally insured credit unions** in the United States (2,715 FCU + 1,616 FISCU + 88 NFICU). The data is structured as comma-delimited text files where each row represents one credit union and columns represent specific financial accounts. The `AcctDesc.txt` file serves as the master data dictionary mapping account codes to their descriptions and parent tables.

This folder is the **primary source for per-credit-union financial, operational, and geographic data** and will power most of the Supabase schema.

### Files

#### `Readme.txt` (5.5 KB)
Official NCUA documentation explaining how to use the 5300 Call Report data package. Describes file formats (comma-delimited text), lists all included files, and provides brief descriptions. Useful as metadata but contains no data itself.

#### `Report1.txt` (3 KB)
Summary statistics for the data cycle: lists the count of FCUs (2,715), FISCUs (1,616), NFICUs (88), totaling 4,419 credit unions. Also lists the row count per data file (4,419 data records + 1 header each for financial tables; 22,068 for branches; 17,506 for ATMs; 982 for trade names). Use this as a validation reference when importing data.

#### `FOICUDES.txt` (3 KB, 30 rows)
Data dictionary for the `FOICU.txt` table. Contains field names, descriptions, and table name for all 28 columns in FOICU. Essential reference for understanding FOICU column semantics (e.g., `CU_TYPE` = charter type, `TOM_CODE` = field of membership type, `Peer_Group` = asset-size tier 1–6, `IsMDI` = minority depository institution flag).

#### `FOICU.txt` (~944 KB, 4,420 rows, 28 columns)
**Master credit union profile table.** One row per credit union. Contains: charter number (`CU_NUMBER`), name, city/state/zip, county code, congressional district, SMSA code, NCUA region (1–5), year opened, charter issue date, peer group (1=<$2M through 6=>$500M), MDI flag, insurance date, and annual meeting date. This is the **core entity table** — every other file joins to it via `CU_NUMBER`. Critical for geographic mapping (CesiumJS globe) and institutional identity.

#### `AcctDesc.txt` (~852 KB, 3,377 rows, 12 columns)
**Master account code dictionary.** Maps every account code (e.g., `Acct_007`) to its human-readable name, full description, parent table (FS220, FS220A, etc.), and flags indicating whether the account is used in CBS, FPR, or statistical reports. This file is essential for the dev agent to understand what each column in the FS220 tables means. The `Status` column indicates Active vs. Inactive accounts.

#### `Acct-DescTradeNames.txt` (~512 bytes, 5 rows)
Tiny data dictionary for the `TradeNames.txt` file. Describes the 4 fields: `CU_NUMBER`, `CycleDate`, `JoinNumber`, `CU_NAME`, `TradeNamesId`, and `TradeName`.

#### `FS220.txt` (~3.1 MB, 4,420 rows, 246 columns)
**Core balance sheet / financial statement.** Contains 242 financial accounts per credit union covering: land & buildings, fixed assets, other assets, total assets, shares/deposits, borrowings, net worth, and equity. This is the primary financial data table and the foundation for balance sheet analysis, asset distribution, and net worth calculations.

#### `FS220A.txt` (~3.1 MB, 4,420 rows, 249 columns)
**Loan portfolio and supplemental financial data.** Contains 246 accounts covering: total loans by type (other loans, leases receivable), average daily assets, interest income/expense breakdowns, and fee income. Critical for loan composition analysis, yield calculations, and income statement metrics.

#### `FS220B.txt` (~2.7 MB, 4,420 rows, 247 columns)
**Delinquency detail — mortgage and consumer loans.** Contains 247 accounts tracking delinquent loan amounts by type and aging bucket (1–2 months, 2–6 months, 6–12 months, 12+ months). Focuses on interest-only mortgages, payment-option loans, and first/second lien real estate. Essential for credit risk analysis and portfolio health assessment.

#### `FS220C.txt` (~2.4 MB, 4,420 rows, 247 columns)
**Loans held for sale, member business loan delinquency, and agricultural loan data.** Contains 247 accounts covering loans held for sale balances, detailed delinquency breakdowns for commercial/business loans and agricultural loans by aging bucket. Important for assessing commercial lending risk exposure.

#### `FS220D.txt` (~4.1 MB, 4,420 rows, 223 columns)
**Operational and compliance data.** Contains 218 accounts plus leadership fields (CEO, Fax, Phone, Preparer, President, Principal). Covers Bank Secrecy Act (BSA) compliance accounts (dealers in foreign exchange, check cashers, money services), operational metrics, and regulatory reporting flags. Unique in that it includes officer name fields.

#### `FS220G.txt` (~2.4 MB, 4,420 rows, 249 columns)
**Intangible assets, goodwill, and supplemental balance sheet items.** Contains 249 accounts covering intangible assets, goodwill, other real estate owned (OREO), and additional asset/liability detail not captured in the core FS220 table.

#### `FS220H.txt` (~2.3 MB, 4,420 rows, 246 columns)
**Modified and troubled loan delinquency data.** Contains 246 accounts tracking delinquent modified consumer loans (not secured by real estate), modified business loans, non-federally guaranteed student loans, and other restructured debt by aging buckets. Key for analyzing troubled debt restructuring (TDR) exposure.

#### `FS220I.txt` (~2.4 MB, 4,420 rows, 248 columns)
**Granular loan delinquency by product type.** Contains 248 accounts with delinquency data broken down by specific loan categories: new vehicles, used vehicles, member business loans secured/not secured by real estate, credit cards, and other consumer loans, all by 30–59, 60–89, 90–179, and 180+ day aging buckets.

#### `FS220J.txt` (~2.3 MB, 4,420 rows, 252 columns)
**Derivatives and hedging instruments.** Contains 252 accounts covering derivative assets/liabilities, interest rate swaps (pay-fixed, receive-fixed), caps, floors, options, futures, forwards, credit default swaps, and other derivative positions with notional amounts and fair values. Relevant primarily for larger credit unions with active derivatives portfolios.

#### `FS220K.txt` (~1.4 MB, 4,420 rows, 145 columns)
**Derivatives fair value by maturity.** Contains 145 accounts showing net fair value gains/losses on derivative contracts bucketed by time remaining in contract (≤1 year, 1–3 years, 3–5 years, 5–10 years, >10 years) for each derivative type (swaps, caps, floors, options, etc.). Supplements FS220J with maturity risk detail.

#### `FS220L.txt` (~2.5 MB, 4,420 rows, 241 columns)
**Commercial loan delinquency detail.** Contains 241 accounts with granular delinquency data for member commercial loans (secured/not secured by real estate), agricultural loans, and participation loans — all by aging bucket. Important for analyzing commercial credit risk concentration.

#### `FS220M.txt` (~835 KB, 4,420 rows, 67 columns)
**Reverse mortgages and unfunded commitments.** Contains 67 accounts covering Home Equity Conversion Mortgages (HECM/reverse mortgages) committed directly vs. through third parties, unfunded commitments, and related metrics. Smaller and more specialized dataset.

#### `FS220N.txt` (~1.2 MB, 4,420 rows, 78 columns)
**CECL-adjusted assets and cash positions.** Contains 78 accounts covering held-to-maturity securities (net of CECL allowance for credit losses), cash on deposit at Federal Reserve, coin and currency, and other cash-equivalent line items. Reflects the newer CECL accounting standard adoption.

#### `FS220P.txt` (~2.3 MB, 4,420 rows, 189 columns)
**Cash deposits, investment detail, and liquidity.** Contains 189 accounts covering time deposits at banks/credit unions, other deposits, total cash positions, and detailed investment classifications. Key for liquidity analysis and investment portfolio breakdown.

#### `FS220Q.txt` (~2.5 MB, 4,420 rows, 199 columns)
**Off-balance sheet exposures and contingent liabilities.** Contains 199 accounts covering loans transferred with limited recourse (commercial and consumer), unfunded loan commitments, letters of credit, and other off-balance sheet items. Important for understanding total risk exposure beyond the balance sheet.

#### `FS220R.txt` (~3.0 MB, 4,420 rows, 227 columns)
**Borrowing arrangements and liquidity facilities.** Contains 227 accounts detailing borrowing capacity and outstanding balances at: Central Liquidity Facility (CLF), Federal Reserve Bank, FRB Paycheck Protection Program, Federal Home Loan Banks, correspondent credit unions, and other borrowing sources. Critical for understanding liquidity risk and funding diversification.

#### `FS220S.txt` (~685 KB, 4,420 rows, 35 columns)
**Technology and fintech indicators.** Contains 35 accounts including Legal Entity Identifier (LEI), virtual credit union flag, fintech partnership usage flag, and other modern operational indicators. Smallest of the FS220 series but valuable for digital transformation analysis.

#### `ATM Locations.txt` (~3.1 MB, 17,507 rows, 16 columns)
**Complete ATM location directory.** One row per ATM. Fields include: credit union number/name, site ID, site name, site type (branch office, etc.), site function (ATM), and full physical address (line1, line2, city, state, zip, county, country). Directly usable for geographic plotting on the CesiumJS globe.

#### `Credit Union Branch Information.txt` (~7.0 MB, 22,069 rows, 27 columns)
**Complete branch location directory.** One row per branch. Fields include: credit union number/name, site ID, site name, site type, main office flag, full physical address, full mailing address, phone number, hours of operation, and service flags (member services, ATM, drive-through, shared service center network). The richest geolocation dataset in the repo. Essential for geographic mapping.

#### `TradeNames.txt` (~70 KB, 983 rows, 6 columns)
**Alternate/trade names for credit unions.** Maps credit union charter numbers to their trade names (the consumer-facing brand name that may differ from the legal charter name). For example, charter #6 "THE NEW ORLEANS FIREMEN'S" has trade name "NOFFCU". Useful for name resolution and search functionality.

---

## 2. DIRECTORY: `financial-performance-report-september-2025/`

### Folder Summary
Contains three aggregate Financial Performance Report (FPR) Excel workbooks from NCUA for September 2025. Each workbook contains **36 sheets** covering comprehensive financial analysis: summary financials, key ratios, supplemental ratios, historical ratios, assets, liabilities/equity, income/expense, loans (8 sheets), investments (5 sheets), shares/deposits, derivatives, liquidity, supplemental info, and graphs. The three files likely represent different aggregation levels (all FICUs, FCU-only, FISCU-only) based on their different file IDs (426946, 426947, 426948).

### Files

#### `FPR (Sep 2025) Aggregate (426946) 508.xlsx` (~589 KB, 36 sheets)
Aggregate Financial Performance Report — likely **all federally insured credit unions combined**. Contains 36 tabs including: Summary Financial, Key Ratios (~100 rows of ratio calculations), Historical Ratios (multi-year trend), Assets breakdown, Liabilities & Shareholder Equity, Income & Expense, 8 loan-related sheets (delinquency, indirect, participation, purchased/sold, real estate, commercial, charge-offs), 5 investment sheets (classification, maturity, memoranda), derivatives, liquidity/borrowings, share distribution/maturity, and 2 graph data sheets. The `Master` sheet likely contains underlying raw data. Comparable to a comprehensive call report aggregate.

#### `FPR (Sep 2025) Aggregate (426947) 508.xlsx` (~588 KB, 36 sheets)
Same structure as above — likely **Federal Credit Unions (FCU) only** aggregate. Allows comparison of FCU-specific performance against the full system.

#### `FPR (Sep 2025) Aggregate (426948) 508.xlsx` (~593 KB, 36 sheets)
Same structure as above — likely **Federally Insured State-Chartered Credit Unions (FISCU) only** aggregate. Allows FCU vs. FISCU comparison analysis.

---

## 3. DIRECTORY: `insurance-report-activity-dec-2025/`

### Folder Summary
Contains NCUA regulatory activity reports for December 2025, documenting the administrative actions that shape the credit union landscape: new charters, field-of-membership expansions, mergers, conversions, and underserved area designations. This data reveals industry consolidation trends, growth strategies, and NCUA regulatory priorities. The data covers all 5 NCUA regions.

### Files

#### `insurance-report-activity-detail-2025-12.xlsx` (~58 KB, 9 sheets)
**Detailed regulatory action register.** Contains individual records for: applications approved, single common bond expansions, multiple common bond expansions (48 records — the most active category), community expansions, community conversions, underserved area approvals, mergers (38 records), charter conversions, and insurance applications. Each record includes charter number, region, credit union name, location, approval date, potential membership impact, and field of membership text. Key for understanding industry consolidation (mergers) and growth (expansions).

#### `insurance-report-activity-summary-2025-12.xlsx` (~47 KB, 14 sheets)
**Regional summary statistics for all regulatory actions.** Aggregates the detail data by NCUA region (1–5) showing counts of approved/denied/deferred applications for: single and multiple common bond expansions, community expansions/conversions, low-income community expansions, underserved areas, charter conversions (FISCU↔FCU), insurance applications/conversions, charter applications, housekeeping mergers, TIP (trade/industry/profession) actions, and merger counts/assets by type. Provides the macro view of regulatory activity.

#### `community-charter-report-2025-12.xlsx` (~20 KB, 1 sheet, ~41 rows)
**Community charter credit union directory.** Lists credit unions operating under community charters with: region, charter number, credit union name, action taken, state, effective date, potential members, duplicate flag, and field of membership description. Shows which credit unions serve geographic communities vs. occupational or associational groups.

#### `underserved-area-report-2025-12.xlsx` (~13 KB, 1 sheet, ~40 rows)
**Underserved area expansion approvals.** Lists credit unions approved to expand into areas deemed underserved by NCUA, with: region, charter number, CU name, state of underserved area, asset size, potential members, and approval date. Important for understanding financial inclusion efforts and community development.

#### `trade-industry-profession-report-2025-year-end.xlsx` (~12 KB, 1 sheet, ~5 rows)
**TIP (Trade, Industry, and Profession) expansion activity.** Lists credit unions with TIP-based field of membership expansions including charter number, CU name, TIP type, potential membership, effective date, and field of membership description. Small dataset — TIP activity is relatively rare.

---

## 4. DIRECTORY: `Various Credit Union Industry Excel Data Tables/`

### Folder Summary
A curated collection of structured datasets providing **global, regional, and U.S.-specific** credit union industry data for 2025. This folder bridges the gap between the U.S.-only NCUA data and the worldwide credit union movement, with data covering North America, Latin America, the Caribbean, and Africa. Also includes economic indicator data (unemployment rates, house price growth) that provides macro-economic context for credit union performance analysis.

### Files — Global & Regional Data

#### `Credit_Union_Industry_Data_2025.xlsx` (~40 KB, 10 sheets)
**Comprehensive global credit union industry workbook.** The most complete cross-regional dataset. Sheets cover: Overview (global totals), North America, Latin America, Caribbean, Africa, Demographics (gender/age breakdowns), Loan Composition, Performance metrics, Top Institutions rankings, and Data Sources. Note: headers may be null in row 1 (formatted as a report with merged cells) — parsing requires skipping decorative rows.

#### `Global_Regional_Comparison_Credit_Union_Data_2025.xlsx` (~8.5 KB, 4 sheets)
**Side-by-side regional comparison.** Clean, structured sheets: Regional Comparison (members, savings, loans, assets by region with % of global totals), Demographics by Region (youth membership %, women members %, women on boards %), Global Trends (10 key metrics), and Strategic Priorities & Risks (ranked lists of industry challenges). Well-structured for direct database import.

#### `North_America_Credit_Union_Data_2025.xlsx` (~14 KB, 9 sheets)
**U.S. and Canada detailed financials.** Sheets: Regional Summary (US vs. Canada), US Detailed Financials (27 metrics Q3 2025 vs Q3 2024 with YoY change), US Loan Portfolio (by category), US Delinquency Rates (by loan type with YoY basis point changes), US Deposits Breakdown, US CU by Asset Size (8 tiers with growth metrics), US Consolidation/Mergers, US Demographics, and Canada Details. Excellent for dashboard KPIs and trend analysis.

#### `Latin_America_Credit_Union_Data_2025.xlsx` (~8.5 KB, 3 sheets)
**Latin American credit union data by country.** Sheets: Country Summary (16 countries with CU count, members, savings, loans, capital, assets), Updated Country Data (newer metrics for select countries), Demographics (regional averages for membership composition). Sources include WOCCU (World Council of Credit Unions) and national federations.

#### `Africa_Credit_Union_Data_2025.xlsx` (~9.5 KB, 3 sheets)
**African credit union data by country.** Sheets: Country Summary (28 countries), Updated Country Data (14 rows of recent metrics), Demographics. Coverage spans from Kenya (largest African CU market) to smaller markets. Sources include ACCOSCA (African Confederation of Co-operative Savings & Credit Associations).

### Files — U.S. Granular CSVs

#### `CU_Country_Level_Data.csv` (~7.5 KB)
**Country-level credit union data worldwide.** Columns: region, sub_region, country, ISO code, CU count, members, savings, loans, reserves, assets, penetration rate, data source, data period, notes. Clean, well-structured CSV ready for import. Covers all major CU markets globally.

#### `CU_Regional_Aggregates.csv` (~512 bytes)
**Regional rollup totals.** Aggregates country-level data into regions (North America, Latin America, Caribbean, Africa, Asia, Europe, Oceania) with member counts, financial totals, and country counts.

#### `CU_Demographics_By_Region.csv` (~512 bytes)
**Demographic breakdown by world region.** Women membership %, youth under-35 %, average member age, women on boards %, women CEO %. Compact but valuable for diversity/inclusion dashboards.

#### `CU_Top_Institutions_Africa.csv` (~512 bytes)
**Ranked list of largest African credit unions.** By assets (USD millions and Kenyan Shillings billions). Important for comparative analysis.

#### `CU_Top_Institutions_LatAm.csv` (~1.5 KB)
**Ranked list of largest Latin American credit unions.** Includes % of national financial system, % of national CU system, and % of LatAm CU system. Richer than the Africa equivalent.

#### `CU_Cooperative_Banks_LatAm.csv` (~1 KB)
**Cooperative banks/credit unions in Latin America.** Lists institution names with assets, legal form (cooperative bank vs. credit union), and business type. Captures the broader cooperative financial sector.

#### `CU_LatAm_Financial_Depth.csv` (~1 KB)
**Financial depth metrics for Latin American CU markets.** Shows total national financial system size vs. CU assets and CU % of the national system. Measures how significant credit unions are in each country's financial infrastructure.

#### `CU_US_Loan_Composition.csv` (~1 KB)
**U.S. credit union loan portfolio breakdown.** Columns: loan category, Q1 2025, Q4 2024, Q2 2024, Q1 2024 (all in USD billions). Categories include real estate, auto, credit card, commercial, etc. Great for trend charting.

#### `CU_US_Quarterly_Performance.csv` (~512 bytes)
**U.S. credit union system quarterly KPIs.** Columns: period, CU count, members, total assets, total loans, shares/deposits, net worth ratio, 60+ day delinquency rate, net charge-off rate. Compact time-series data for dashboard metrics.

### Files — Economic Context

#### `State and City Indicators for December 2025.xlsx` (~447 KB, 4 sheets)
**Macroeconomic indicators by U.S. metro area and state.** Sheets: Metro Area Unemployment Rates (~398 metros, annual from 2000 + rolling 12-month), State Unemployment Rates (50 states + DC, annual from 2000 + quarterly from 2009 + monthly from 2011, ~289 columns), Metro Area House Price Growth (~391 metros, annual FHFA index from 2000), State House Price Growth (50 states + DC, annual from 2000). Sourced from BLS and FHFA. Critical for correlating credit union performance with local economic conditions on the geographic map.

#### `accountDescription_December2025.xlsx` (~65 KB, 1 sheet, 1,113 rows)
**Extended account code reference.** Maps account codes to their Type, Category, Description, and charter-type applicability flags (FCU, FISCU, NFICU), plus page reference. More comprehensive than AcctDesc.txt — includes 1,113 codes vs. 3,377 (different structure, likely cross-referenced by code vs. the txt which lists duplicates across tables).

#### `community-charter-report-2025-12.xlsx` (~20 KB)
**Duplicate** of the same file in `insurance-report-activity-dec-2025/`. Same community charter directory.

#### `underserved-area-report-2025-12.xlsx` (~13 KB)
**Duplicate** of the same file in `insurance-report-activity-dec-2025/`. Same underserved area report.

#### `FINANCIAL TRENDS IN FEDERALLY INSURED CREDIT UNIONS 2025Q3.xlsx` (~127 KB, 42 sheets)
**Source data for the NCUA Financial Trends report.** Each of the 42 sheets corresponds to one chart from the published report (Chart 1 through Chart 42). Covers: CU count trends, asset distribution, growth rates, net worth ratios, ROA breakdown, yield vs cost of funds, loan/share distributions, delinquency rates, investment classifications/maturities, borrowings, CAMELS rating distributions, failures, mergers/liquidations, and trend summaries by asset group and CU type. Contains ~10 years of quarterly time-series data per chart. Extremely valuable for trend visualization.

---

## 5. DIRECTORY: `PDF Reports/`

### Folder Summary
Published NCUA analytical reports in PDF format, providing narrative context, visualization, and analysis that complements the raw data files. These are the official quarterly publications that credit union leaders, regulators, and analysts read to understand industry trends. They contain charts, maps, and commentary that can inform the design of your interactive dashboard.

### Files

#### `FINANCIAL TRENDS IN FEDERALLY INSURED CREDIT UNIONS 2025Q3.pdf` (2 MB, 16 pages)
**Flagship NCUA trends report.** Covers overall industry trends for Q3 2025: number of reporting CUs (FCU vs FISCU), asset distribution, asset growth vs membership growth, loan growth vs share growth, net worth ratio, return on assets, yield vs cost of funds, loan distribution by type, delinquency and charge-off trends, investment classifications/maturities, share distribution, liquidity ratios, CAMELS rating distributions, failures, and merger/liquidation counts. The corresponding data is in the 42-sheet Excel file above.

#### `quarterly-data-summary-2025-Q3.pdf` (550 KB, 20 pages)
**Quarterly Credit Union Data Summary.** Provides a broader performance overview of federally insured credit unions for Q3 2025 with system-level metrics, charts, and tables covering assets, loans, deposits, net worth, income, and trends. More narrative-driven than the Financial Trends report.

#### `quarterly-map-review-third-quarter-2025.pdf` (3.3 MB, 14 pages)
**NCUA Quarterly U.S. Map Review.** State-by-state geographic analysis of credit union health indicators using color-coded maps. Covers key metrics (asset growth, loan growth, delinquency, net worth) visualized geographically. Directly relevant to CesiumJS globe design — the map approach in this PDF can inspire the choropleth/heatmap layer design for the interactive globe.

#### `cusos-at-a-glance-2023.pdf` (133 KB, 1 page)
**CUSO (Credit Union Service Organization) snapshot.** Single-page infographic showing CUSO statistics as of December 2023: count of registered CUSOs, types of services provided, and industry participation. Older data (2023) but provides context on the CU service ecosystem. Note: this is the only file not from 2025.

---

## 6. DIRECTORY: `NCUSIF 2025 Q4 - Credit Union Industry Summary Data/`

### Folder Summary
A set of **9 dashboard-style PNG images** capturing NCUSIF (National Credit Union Share Insurance Fund) summary data for Q4 2025. These are visual snapshots — likely screen captures from NCUA's Power BI dashboards or similar reporting tools — showing the aggregate health of the credit union insurance fund and the industry it insures. These images provide design inspiration for the interactive dashboard and represent the latest data point in the repo (Q4 2025).

### Files

#### `Summary.png` (507 KB)
**High-level industry overview dashboard.** Likely shows total insured credit unions, total assets, total shares, membership, and key aggregate metrics for the NCUSIF-insured system.

#### `Balance Sheet.png` (337 KB)
**Aggregate balance sheet visualization.** Shows industry-wide asset composition, liabilities, and net worth at the system level.

#### `Revenue and Expenses.png` (331 KB)
**Income statement summary.** Visualizes total interest income, non-interest income, operating expenses, provisions for loan losses, and net income for the industry.

#### `Portfolio Performance.png` (378 KB)
**Loan portfolio health dashboard.** Likely shows aggregate loan growth, delinquency rates, charge-offs, and portfolio composition trends.

#### `CAMELS Overview.png` (328 KB)
**CAMELS rating distribution.** Shows how many credit unions fall into each CAMELS composite rating (1–5), visualizing the overall supervisory health of the system.

#### `CAMELS Coded 3-4-5.png` (315 KB)
**Problem credit unions detail.** Focuses on credit unions rated CAMELS 3 (cause for concern), 4 (significant risk), and 5 (critically deficient). Shows counts, assets, and trends for these troubled institutions.

#### `Equity Ratio and Normal Operating Level.png` (425 KB)
**NCUSIF fund health metrics.** Shows the insurance fund's equity ratio relative to the Normal Operating Level (NOL) set by the NCUA Board. The equity ratio measures fund reserves relative to insured shares and indicates the fund's ability to absorb losses.

#### `Reserves and Losses Overview.png` (266 KB)
**Insurance fund reserves and loss experience.** Shows NCUSIF reserve balances, historical losses, and loss provisioning trends. Indicates the financial stability of the deposit insurance system.

#### `Credit Union VS Bank rates - 2025 Q4.png` (140 KB)
**Rate comparison between credit unions and banks.** Compares average rates on common products (savings, CDs, auto loans, mortgages, credit cards) between credit unions and commercial banks. Demonstrates the credit union value proposition.

---

## SUPABASE SCHEMA RECOMMENDATIONS

### Core Entity Tables
1. **`credit_unions`** — From `FOICU.txt`: charter number (PK), name, city, state, zip, county, congressional district, region, peer group, year opened, charter type, MDI flag, trade names (joined from `TradeNames.txt`)
2. **`branches`** — From `Credit Union Branch Information.txt`: site ID (PK), CU number (FK), physical address, phone, hours, service flags (member services, ATM, drive-through, shared service center)
3. **`atm_locations`** — From `ATM Locations.txt`: site ID (PK), CU number (FK), physical address, site type/function

### Financial Data Tables
4. **`financial_statements`** — Consolidated from FS220 + FS220A through FS220S. Consider: (a) a wide single table with all ~3,000+ accounts as columns, or (b) a normalized EAV (Entity-Attribute-Value) table: `(cu_number, cycle_date, account_code, value)` joined to an `account_descriptions` lookup table from `AcctDesc.txt`. **Recommendation: Use the EAV approach** — it's more flexible for querying, doesn't require schema changes when NCUA adds/removes accounts, and works better with Supabase's PostgreSQL. Create materialized views for common financial snapshots.
5. **`account_descriptions`** — From `AcctDesc.txt` + `accountDescription_December2025.xlsx`: account code (PK), name, description, parent table, category, type, status, charter-type applicability

### Performance & Trends
6. **`fpr_aggregate`** — From FPR Excel files: pre-computed aggregate ratios and financial summaries for the system (all FICUs, FCU-only, FISCU-only)
7. **`financial_trends`** — From the 42-chart Financial Trends Excel: historical time-series data for each trend metric (quarterly from ~2014–2025)
8. **`us_quarterly_performance`** — From `CU_US_Quarterly_Performance.csv`: compact KPI time series

### Regulatory Activity
9. **`regulatory_actions`** — From insurance-report-activity files: charter number, action type (expansion, merger, conversion, etc.), region, date, potential membership impact, field of membership
10. **`community_charters`** — From community charter report
11. **`underserved_areas`** — From underserved area report

### Global/Regional Data
12. **`global_cu_data`** — From `CU_Country_Level_Data.csv` + regional Excel files: country-level credit union metrics
13. **`regional_aggregates`** — From `CU_Regional_Aggregates.csv`: regional rollup totals
14. **`global_demographics`** — From `CU_Demographics_By_Region.csv`: diversity metrics by region
15. **`top_institutions`** — From Africa and LatAm top institution CSVs: ranked institution lists

### Economic Context
16. **`economic_indicators`** — From `State and City Indicators for December 2025.xlsx`: metro and state unemployment rates, house price growth. Normalize into: `(geography_type, geography_name, metric, period, value)`

---

## INTERACTIVE APP RECOMMENDATIONS

Based on the combined data, here are recommended features for the CesiumJS + dashboard application:

### 1. Geographic Globe (CesiumJS)
- **U.S. Credit Union Heatmap**: Plot all 4,419 credit unions by headquarters (from FOICU). Color-code by: asset size tier, CAMELS-equivalent health (can approximate from financial ratios), net worth ratio, loan growth, or delinquency rate.
- **Branch & ATM Network**: 22,068 branches + 17,506 ATMs plotted as markers. Toggle by credit union or by region. Cluster at zoom levels to prevent overload.
- **State/Metro Choropleth Overlays**: Overlay unemployment rates, house price growth, or aggregated CU performance metrics by state from the economic indicators data.
- **Global View**: Zoom out to show international CU presence using `CU_Country_Level_Data.csv`. Size bubbles by membership or assets. Color by penetration rate.
- **Regulatory Activity Layer**: Plot mergers, expansions, and underserved area approvals as animated event markers showing where the industry is growing/consolidating.

### 2. Interactive Dashboard
- **System Health KPIs**: Total CUs, members, assets, loans, shares, net worth ratio, delinquency rate, ROA — all from North America and quarterly performance data. Show YoY deltas.
- **Financial Trends Explorer**: Interactive charts for all 42 financial trend metrics with time-series slider (2014–2025). Allow comparison of FCU vs. FISCU vs. all FICU.
- **Loan Composition Drill-Down**: Pie/donut charts of loan portfolio breakdown, with click-through to delinquency detail by loan type and aging bucket.
- **Peer Group Comparison**: Filter/compare CUs by peer group (asset tier 1–6), region, charter type. Show distribution histograms for key ratios.
- **Rate Comparison Widget**: CU vs. bank rate comparison visualization (from the Q4 image data — could recreate dynamically).
- **CAMELS Distribution**: Interactive bar/pie showing CAMELS rating distribution, with drill-down into 3-4-5 coded institutions.
- **Merger & Consolidation Tracker**: Timeline of mergers, with assets involved, by region. Show consolidation acceleration/deceleration.

### 3. AI Chatbot (trained on this data)
- Train on: all structured data + PDF report narrative content + account descriptions
- Enable questions like: "What is the average delinquency rate for credit unions in Texas?", "How has loan growth compared to share growth over the past 3 years?", "Which peer group has the strongest net worth ratio?", "Compare FCU and FISCU performance on ROA"
- Use RAG (Retrieval-Augmented Generation) with Supabase pgvector to embed account descriptions, trend narratives, and financial definitions for accurate domain-specific responses.

### 4. Advanced Features
- **Credit Union Lookup**: Search by name, charter number, city, or trade name. Show full financial profile card with key ratios, branch map, peer comparison.
- **Risk Heatmap**: Composite risk score per CU derived from delinquency + net worth + loan concentration + borrowing reliance. Visualize on globe and in sortable tables.
- **Regional Economic Correlation**: Scatter plots showing CU delinquency vs. local unemployment, or CU growth vs. house price appreciation by metro/state.
- **Global Inclusion Dashboard**: Visualize credit union penetration rates, women's membership, youth participation across world regions.
- **What-If Scenario Tool**: Allow users to adjust rate assumptions and see projected impact on CU income/expense using the FPR ratio framework.

---

## DATA QUALITY NOTES FOR DEV AGENT

1. **CSV/TXT files**: Comma-delimited with double-quote text qualifiers. First row is always headers. Use standard CSV parsing.
2. **Excel files with null headers**: `Credit_Union_Industry_Data_2025.xlsx` has null row-1 headers due to merged cells — skip decorative rows and detect actual header row programmatically.
3. **Duplicate files**: `community-charter-report-2025-12.xlsx` and `underserved-area-report-2025-12.xlsx` exist in BOTH `insurance-report-activity-dec-2025/` and `Various Credit Union Industry Excel Data Tables/`. Import once only.
4. **Date format**: FOICU uses `9/30/2025 0:00:00` format. Normalize to ISO 8601.
5. **Numeric fields**: Financial amounts in call report data are stored as raw numbers (not pre-formatted as currency). Some may be stored as strings with double quotes.
6. **CRLF line endings**: Most .txt files use Windows-style `\r\n` line endings.
7. **Account status**: `AcctDesc.txt` includes both Active and Inactive accounts — filter by `STATUS` column when building the live schema.
8. **Cycle date mismatch**: Call report data is for Q3 2025 (9/30/2025), while the NCUSIF summary images are labeled Q4 2025. FPR data is September 2025. Insurance activity is December 2025. Track these different reporting periods in your schema.
9. **Image files**: The 9 PNG files in the NCUSIF folder are raster images, not structured data. Either OCR them for data extraction or treat them as reference/documentation assets only.
10. **FPR file IDs**: The three FPR files (426946, 426947, 426948) need to be labeled in the schema as their respective aggregation levels — confirm by checking their cover page data.
