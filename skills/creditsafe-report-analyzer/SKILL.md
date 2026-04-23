---
name: creditsafe-report-analyzer
description: >
  Parse Creditsafe PDF credit reports and generate structured risk narrative
  summaries for merchant onboarding decisions. Use when the user asks to analyze
  a Creditsafe report, assess credit risk for UK entities, or produce due
  diligence narratives.
metadata:
  author: mcsanchez
  version: "0.1.0"
  status: experimental
---

# Creditsafe Report Analyzer

## When to Use
- User asks to analyze a Creditsafe PDF credit report
- User needs a structured risk narrative for merchant onboarding
- User mentions credit risk assessment or due diligence for UK entities

## Workflow
1. User provides a Creditsafe PDF report (local file path)
2. Extract text using `pdftotext` (from poppler)
3. Parse key data points (see below)
4. Generate a structured risk narrative in paragraph-table format

## Data Points to Extract
- Company name, registration number, incorporation date
- Credit score and credit limit
- CCJ (County Court Judgments) history
- Payment performance / DBT (Days Beyond Terms)
- Director information and cross-directorships
- Financial summary (revenue, net worth, profit/loss)
- Industry SIC codes
- Mortgage and charge information

## Output Format

### Single Entity Analysis
Generate a paragraph-table with these sections:
| Section | Content |
|---------|---------|
| Company Overview | Name, reg number, incorporation date, SIC codes |
| Credit Assessment | Score, limit, rating trend |
| Financial Health | Revenue, net worth, profit/loss trends |
| Legal / CCJ | Outstanding and satisfied judgments |
| Payment Behaviour | DBT score, payment trends |
| Directors | Key individuals, cross-directorships, risk flags |
| Risk Summary | Overall risk narrative with recommendation |

### Group Analysis (Multiple Entities)
When analyzing multiple related entities, also address:
- Cross-directorship patterns across the group
- Consolidated financial health
- Shared risk factors
- Group-level recommendation

## Key Analytical Questions (Group Analysis)
- Are there shared directors across entities?
- Do any entities have significantly worse credit profiles?
- Are there patterns in CCJ history across the group?
- Is there concentration risk in a single industry?
