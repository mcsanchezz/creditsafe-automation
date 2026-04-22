# Creditsafe Report Analyzer

A Claude Code skill that parses Creditsafe PDF credit reports and generates structured risk narrative summaries for merchant onboarding decisions.

## Overview

This tool is designed for credit risk analysts evaluating whether to onboard merchants or approve new payment processing relationships. It extracts key risk indicators from Creditsafe PDF reports and produces a 4-paragraph risk assessment narrative.

## Skill: `/analyze-creditsafe`

### Invocation

```
/analyze-creditsafe <filename1.pdf> [filename2.pdf] [filename3.pdf]
```

**Examples:**

```bash
# Single entity analysis
/analyze-creditsafe ASPINAL_OF_LONDON_LIMITED_GB-0-05081390_2026-04-21_08-19-09.pdf

# Group analysis (standalone + parent holding company)
/analyze-creditsafe ASPINAL_OF_LONDON_LIMITED_GB-0-05081390_2026-04-21_08-19-09.pdf ASPINAL_OF_LONDON_GROUP_LIMITED_GB-0-06412916_2026-04-21_08-19-25.pdf
```

### Input

| Parameter | Description |
|-----------|-------------|
| `files` | One or more Creditsafe PDF filenames. The user specifies which reports belong to the same group. |
| `folder` | **Hardcoded** to the team's shared Google Drive folder (currently requires local download — see [Future: Drive Integration](#future-drive-integration)). |

### How It Works

1. **PDF Text Extraction** — Uses `pdftotext` (from poppler) to extract structured text from each Creditsafe PDF
2. **Field Parsing** — Identifies and extracts key data points from standard Creditsafe report sections
3. **Single vs Group Detection** — Based on the number of files provided:
   - **1 file** → Single entity analysis
   - **2+ files** → Group analysis (standalone + holding company / related entities)
4. **Narrative Generation** — Produces a structured risk assessment in paragraph form

---

## Output Format

### Single Entity Analysis (1 report)

| Paragraph | Content |
|-----------|---------|
| **1. Entity Overview & CCJs** | Company identifiers, risk score, international score, credit limit, CCJ status, payment behaviour (DBT, invoices), enquiry activity |
| **2. Financials** | Turnover, pre-tax profit, shareholder's funds (with YoY trends), balance sheet health, liquidity indicators, cash flows, share capital, charges/security, workforce, Creditsafe commentary |
| **3. Historical & Negative Information** | CVAs, court notices, score history trajectory, limit history, previous company names |
| **4. Overall Assessment** | Risk summary, suitability for onboarding, flags for the credit file, monitoring recommendations |

### Group Analysis (2+ reports)

When multiple reports are provided (e.g. standalone subsidiary + ultimate holding company), the analysis covers both entities in conjunction:

| Paragraph | Content |
|-----------|---------|
| **1. Entity Overview & CCJs** | Identifies both entities and their relationship (e.g. subsidiary of consolidating parent). Reports scores, limits, status, CCJs, and payment behaviour for **both** entities side by side |
| **2. Financials (Consolidated & Standalone)** | Leads with consolidated group financials as the "true" picture, then compares standalone. Flags divergences (e.g. subsidiary loss-making while group profitable). Charges at both levels — common lenders identified |
| **3. Historical & Negative Information** | Negative events at **both** levels — identifies where they originated (group vs subsidiary). Score trajectories compared — recovering in sync or diverging? |
| **4. Overall Assessment** | Combined risk view: standalone viability + group backing. Dependency risk assessment. Contagion risk. Recommendation with group context |

### Key Analytical Questions (Group Analysis)

The group analysis specifically addresses:

- Is the merchant financially viable on a standalone basis, or dependent on group support?
- Are negative events (CVAs, CCJs) isolated to one entity or group-wide?
- Do consolidated accounts mask weakness at the subsidiary level (or vice versa)?
- Are charges/security mirrored across entities (same lenders = same exposure)?

---

## Data Points Extracted

### From Each Report

| Section | Fields |
|---------|--------|
| **Header/Scores** | Risk Score, International Score, Credit Limit, Contract Limit, Status, DBT, Industry DBT |
| **Company Info** | Company Number, Name, Status, Incorporation Date, Type, SIC Code, Principal Activity |
| **Key Financials** | Turnover (3yr), Pre-Tax Profit (3yr), Shareholder's Funds (3yr), Employees |
| **Payment Info** | Avg Invoice Value, Invoices Paid/Outstanding, Age of Debt |
| **CCJ Summary** | CCJ count, values, trends |
| **Negative Info** | CVAs, Court Notices (HM Court), Petitions |
| **Score History** | Historical scores with descriptions and dates |
| **Limit History** | Historical credit limits with dates |
| **Commentary** | Creditsafe narrative commentary |
| **Group Structure** | Parent/subsidiaries, consolidation flag, individual scores |
| **Charges** | Outstanding/satisfied, lenders, charge types |
| **Directors** | Current board, recent appointments/resignations |
| **Previous Names** | Historical company names (phoenix company flags) |

---

## Example Output

### Single Entity: Aspinal of London Group Limited

> **Aspinal of London Group Limited** (06412916) currently holds a Creditsafe risk score of 87 ("Very Low Risk") with an international score of A and a recommended credit limit of £1.05M. The company is active with accounts filed, incorporated on 30/10/2007, and operates as a private limited company in the retail sector (SIC: 47789). **No CCJs are recorded against this company.** Payment behaviour is clean — 129 invoices paid with zero outstanding and no recorded Days Beyond Terms, comfortably below the industry average DBT of 8 days. The company has 43 credit enquiries in the past 12 months, indicating active commercial engagement.
>
> Financially, the company has shown significant improvement. Turnover grew to £45.2M in FY2025 (+17.3% YoY, up from £38.6M), with pre-tax profit surging to £7.1M — a 410% increase on the prior year's £1.4M. Shareholder's funds have returned to positive territory at £7.4M, a material recovery from -£3.5M in 2024 and -£5.2M in 2023, indicating the balance sheet is now solvent and strengthening. Share capital stands at £144,550. The company carries 6 outstanding charges — primarily fixed and floating charges held by Santander UK PLC (as security trustee), alongside charges held by private individuals (Susan Garwood, Meiju Xu, Aleksei Bazhenov). The workforce has reduced slightly from 186 to 168 employees. Creditsafe commentary notes high profitability, high capital available for reinvestment, and high equity within the business.
>
> The company has material historical negative information that warrants attention. A Company Voluntary Arrangement (CVA) application was filed in November 2020 (case CR-2020-MAN-000954, represented by Squire Patton Boggs), with completion recorded in August 2021. The score history reflects this: the company was "Not Scored" or "High Risk" (score 29) between late 2020 and early 2022, before gradually recovering — moving through "Moderate Risk" (50) in February 2022, "Low Risk" (61-69) through 2024, and reaching "Very Low Risk" (87) in January 2026. The credit limit followed a similar trajectory, sitting at £0 through the CVA period before recovering to £100K in 2022 and jumping to £1.05M in January 2026. The company also previously traded under two different names (Yrret Limited, HS 441 Limited).
>
> Overall, the current profile suggests a strong recovery trajectory suitable for onboarding. The CVA completed over four years ago, and every financial and behavioural metric has improved materially since. No CCJs, clean payment history, and positive equity all support a favourable credit view. However, the CVA history and the concentration of outstanding charges should be noted in the credit file, and periodic monitoring is recommended given the relatively recent return to financial health.

---

## Technical Requirements

- **`pdftotext`** (from poppler) must be installed: `brew install poppler`
- **Claude Code** with access to the local filesystem

---

## Future: Drive Integration

The team's Creditsafe reports are stored in a shared Google Drive folder:

```
https://drive.google.com/drive/u/0/folders/12vrftmzwvJVF9f038EdIJqkiWmBr3AUU
```

**Current workflow:** Download the relevant PDFs locally, then run the skill.

**Planned enhancement:** Integrate with Google Drive (via Drive for Desktop sync or API) so the skill can read reports directly from the shared folder without manual download. Options under consideration:

1. **Google Drive for Desktop** — Syncs the shared folder to a local path, which the skill reads from directly
2. **Google Drive API** — Programmatic access to download PDFs on demand
3. **MCP Server** — A Claude Code MCP server that connects to Google Drive

---

## Setup

```bash
# Install poppler (required for PDF text extraction)
brew install poppler

# Clone the repo
git clone https://github.com/mcsanchezz/creditsafe-automation.git
```
