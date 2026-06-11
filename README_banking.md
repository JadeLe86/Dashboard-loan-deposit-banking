# 🏦 Banking Loan & Deposit Analytics Dashboard — Power BI BI Portfolio

> A consulting-grade Business Intelligence solution for Vietnamese commercial banks —  
> delivering unified loan portfolio analytics, deposit mobilization monitoring, branch performance benchmarking, customer segmentation, and real-time NPL risk surveillance on a single platform, aligned with SBV (NHNN) reporting standards.

---

## 🚀 Live Demo

| Language | Link |
|---|---|
| 🇬🇧 English | [banking_loan_deposit_analytics_portfolio_en.html](./banking_loan_deposit_analytics_portfolio_en.html) |
| 🇻🇳 Tiếng Việt | [banking_loan_deposit_analytics_portfolio_vi_v2.html](./banking_loan_deposit_analytics_portfolio_vi_v2.html) |

> Open directly in any browser — no installation, no server required. All charts and data are embedded.

---

## 📁 Repository Structure

```
📂 project-root/
├── 📄 README.md                                              ← You are here
├── 🌐 banking_loan_deposit_analytics_portfolio_en.html       ← English portfolio
├── 🌐 banking_loan_deposit_analytics_portfolio_vi_v2.html    ← Vietnamese portfolio (SBV-compliant)
```

---

## 🎯 Business Problem Solved

| Pain Point | Before | After |
|---|---|---|
| Loan reporting | Manual CoreBanking export, 3–4 days/month | Auto-refresh at 6:00 AM every day |
| Deposit tracking | Separate Excel, disconnected from loans | Unified in one data model |
| NPL monitoring | Discovered at month-end provision meeting | Real-time alert when threshold exceeded |
| Branch performance | Static PDF, no drill-down capability | Interactive dashboard, 3-tier RLS |
| LDR ratio | Known only after month-end settlement | Intra-month monitoring vs Circular 22 limit |

---

## 🗂️ Content — 10 Sections

| Section | Content |
|---|---|
| **1. Executive Summary** | Hero brief for CEO/CFO/CRO/Board, benefits table by executive role |
| **2. Business Case** | 5-group NPL classification (TT02), LDR Circular 22, Vietnamese banking market context 2024 |
| **3. Data Model** | Multi-fact star schema: 5 Fact + 7 Dimension tables, DimBranch flatten 2-tier design |
| **4. KPI Framework** | 20+ KPIs: Loans, Deposits, Combined, Risk — formulas + SBV thresholds |
| **5. Dashboard** | 6 interactive pages with real Chart.js data visualizations |
| **6. DAX Measures** | 14 production-ready formulas including 2-tier branch hierarchy and RLS |
| **7. Dataset Design** | Field-level design for Loan, Deposit, Customer, Branch tables + recommended volume |
| **8. Business Insights** | 20 automated banking insights with supporting charts |
| **9. Case Study** | Full consulting write-up: problem → solution → outcomes → lessons learned |
| **10. Website Content** | Portfolio website copy ready to publish |

---

## 🏗️ Data Architecture — Multi-Fact Star Schema

```
CoreBanking (T24 / FPT.BankingOS)
Credit System + Deposit System + NPL System
        │
        ▼
  SQL Staging Layer  ──── ETL / Power Query / Data Cleansing
        │
        ▼
┌────────────────────────────────────────────────────────────┐
│  FACT TABLES                                               │
│  FactLoanBalance    FactDepositBalance    FactNPL           │
│  FactLoanApplication    FactTransaction                    │
└───────────────────────┬────────────────────────────────────┘
                        │ Conformed Keys
         ┌──────────────┼──────────────┐
         ▼              ▼              ▼
    DimDate        DimCustomer    DimBranch (Flatten 2-tier)
    DimProduct     DimRegion      DimSegment
         │
         ▼
  Power BI Dashboard (6 pages)
  ├─ Page 1: Executive Overview
  ├─ Page 2: Loan Portfolio Analytics
  ├─ Page 3: Deposit Analytics
  ├─ Page 4: Branch Performance
  ├─ Page 5: Customer Segment Analytics
  └─ Page 6: Risk Monitoring (TT02)
```

---

## 🏢 DimBranch — Flatten 2-Tier Design

The most critical architectural decision in this project: handling the branch hierarchy as it actually exists in Vietnamese commercial banks — **Parent Branch (Chi Nhánh)** and **Transaction Office (Phòng Giao Dịch, PGD)**.

```
BranchKey | Branch_code  | Branch_name        | IsParent | Parent_branch_code | Parent_branch_name
----------|--------------|--------------------|----------|--------------------|--------------------
1         | HN001        | CN Hà Nội          |    1     | HN001              | CN Hà Nội
2         | HN001-PGD01  | PGD Cầu Giấy       |    0     | HN001              | CN Hà Nội
3         | HN001-PGD02  | PGD Đống Đa        |    0     | HN001              | CN Hà Nội
4         | HN001-PGD03  | PGD Thanh Xuân     |    0     | HN001              | CN Hà Nội
5         | HCM001       | CN HCM Quận 1      |    1     | HCM001             | CN HCM Quận 1
6         | HCM001-PGD01 | PGD Bến Nghé       |    0     | HCM001             | CN HCM Quận 1
```

**The golden rule:** CoreBanking records transactions at PGD level. Branch-level figures = Parent Branch's own transactions + all PGD transactions under it. By storing `Parent_branch_code` on every row, Power BI automatically aggregates correctly when filtering by `Parent_branch_name` — no complex DAX required.

| Filter Applied | Rows Returned | Result |
|---|---|---|
| `Parent_branch_name = "CN Hà Nội"` | BranchKey {1, 2, 3, 4} | ✅ Correct branch total (CN + 3 PGDs) |
| `Branch_name = "PGD Cầu Giấy"` | BranchKey {2} | ✅ Correct PGD-only figure |

---

## 📐 KPI Framework — SBV (NHNN) Compliant

### Loan KPIs
| KPI | DAX Formula | SBV Threshold |
|---|---|---|
| Total Loan Outstanding | `SUM(FactLoanBalance[OutstandingBalance])` | Growth 14–15%/year |
| Loan Growth % | `DIVIDE(LoanCM - LoanPM, LoanPM)` | Within SBV-assigned quota |
| NPL Ratio | `DIVIDE([NPL Balance], [Total Loan])` | ⚠ Alert: > 3% |
| Portfolio at Risk (PAR) | `DIVIDE([Overdue Balance], [Total Loan])` | ⚠ Alert: > 5% |

### Deposit KPIs
| KPI | DAX Formula | Threshold |
|---|---|---|
| CASA Ratio | `DIVIDE([CASA Balance], [Total Deposits])` | Target: > 30% |
| LDR Ratio | `DIVIDE([Total Loans], [Total Deposits])` | ⚠ Circular 22: ≤ 80% (private banks) |
| Deposit Growth % | `DIVIDE(DepCM - DepPM, DepPM)` | Must track with loan growth |

### Loan Classification — Circular 02/2013/SBV (TT02)

| Group | Name | DPD | Provision Rate | Dashboard Color |
|---|---|---|---|---|
| Group 1 | Standard | 0 days | 0% | 🟢 Green |
| Group 2 | Special Mention | 1–30 days | 5% | 🟡 Yellow |
| Group 3 | Substandard | 31–90 days | 20% | 🟠 Orange |
| Group 4 | Doubtful | 91–180 days | 50% | 🔴 Red |
| Group 5 | Loss | > 180 days | 100% | ⬛ Dark Red |

> **NPL = Group 3 + Group 4 + Group 5**

---

## 📊 Dashboard — 6 Pages with Real Visualizations

### Page 1 — Executive Overview *(CEO · CFO · Board)*
KPI snapshot: Loans VND 46,250 Bn · Deposits VND 57,320 Bn · LDR 80.7% · NPL 2.80%
- Dual-line: Loan vs Deposit 12-month trend
- LDR trend with Circular 22 threshold line at 80%
- Product mix donuts (loans & deposits)
- Top 5 branches by loan outstanding

### Page 2 — Loan Portfolio Analytics *(Credit Department)*
- Combo bar+line: disbursement vs loan accumulation
- Loan by product and customer segment
- Maturity schedule by tenor (color-coded by urgency)

### Page 3 — Deposit Analytics *(Funding Department)*
- Combo: deposit balance + CASA ratio trend (dual axis)
- CASA vs term deposit stacked breakdown
- Term deposit maturity cliff — next 6 months

### Page 4 — Branch Performance *(Retail Banking)*
- Grouped bar: Loan vs Deposit for 8 branches
- NPL by branch with SBV 3% threshold line
- Loan growth ranking per branch

### Page 5 — Customer Segment Analytics *(CRM)*
- Loan & deposit by 5 segments: Retail / SME / Corporate / VIP / Mass Market
- Customer count + avg loan per customer (dual-axis combo)
- NPL by segment — SME and Mass Market highest

### Page 6 — Risk Monitoring *(CRO)*
- NPL trend 12 months + SBV 3% threshold line
- 5-group TT02 loan classification stacked (monthly)
- Aging bucket: Current → 1-30 → 31-60 → 61-90 → 91-180 → >180 DPD
- NPL by product — SME and business loans highest

---

## ⚙️ Key DAX Measures

```dax
-- Branch-level total (auto-aggregates PGDs via flatten)
Total Loan Outstanding = SUM( FactLoanBalance[OutstandingBalance] )

-- NPL Ratio (Group 3+4+5 per TT02)
NPL Ratio =
DIVIDE(
    CALCULATE( SUM(FactLoanBalance[OutstandingBalance]),
               FactLoanBalance[LoanGroup] >= 3 ),
    [Total Loan Outstanding], 0
)

-- LDR (Circular 22/2019/SBV)
LDR Ratio = DIVIDE( [Total Loan Outstanding], [Total Deposit Balance], 0 )

-- CASA Ratio
CASA Ratio = DIVIDE( [CASA Balance], [Total Deposit Balance], 0 )

-- Auto drill-down across branch hierarchy
Loan By Level =
IF(
    ISINSCOPE( DimBranch[Branch_name] ),
    SUM( FactLoanBalance[OutstandingBalance] ),
    CALCULATE(
        SUM( FactLoanBalance[OutstandingBalance] ),
        FILTER( ALL(DimBranch),
            DimBranch[Parent_branch_code] = MAX(DimBranch[Parent_branch_code]) )
    )
)

-- RLS: Branch Director sees own branch + all PGDs
[Parent_branch_code] = LOOKUPVALUE(
    DimStaff[Parent_branch_code],
    DimStaff[Email], USERPRINCIPALNAME()
)
```

---

## 📈 Business Outcomes

| Metric | Result |
|---|---|
| Reduction in manual reporting effort | **~75%** (3–4 days → < 4 hours) |
| Data freshness | **Daily** (automated 6:00 AM refresh) |
| NPL alerts triggered | **3 branches** caught in Week 1 of deployment |
| Systems unified | **3 → 1** (CoreBanking + Deposits + NPL) |
| Access control tiers | **3 levels** RLS: PGD Head / Branch Director / Regional Head |

---

## 🛠️ Technology Stack

| Layer | Technology | Purpose |
|---|---|---|
| Visualization | Power BI Desktop & Service | 6-page dashboard, RLS, scheduled refresh |
| Data Modeling | DAX / Power BI Model | Star schema, KPI measures, time-intelligence |
| Transformation | Power Query (M Language) | ETL, cleansing, CoreBanking normalization |
| Database | SQL Server / T-SQL | Staging layer, data history |
| Source Data | CoreBanking Export (CSV/Excel) | T24, FPT.BankingOS or equivalent |
| Security | Row-Level Security (RLS) | 3-tier access: PGD / Branch / Region |
| Automation | Power Automate | Daily KPI email digest for Branch Directors |
| Compliance | TT02/2013, TT22/2019 (SBV) | 5-group NPL classification, LDR threshold |

---

## 🗝️ Topics

```
power-bi  dax  sql  banking  business-intelligence  dashboard
fintech  nhnn  sbv  loan-analytics  deposit-analytics  npl
ldr  casa  star-schema  row-level-security  corebanking  vietnam
```

---

## 📬 Contact

- 🔗 LinkedIn: https://www.linkedin.com/in/thuy-linh-le-2aa7b810a/
- 📧 Email: linh.ltt06@gmail.com
- 🌐 Portfolio: [yoursite.com](https://yoursite.com)

---

*Built as a consulting portfolio project to demonstrate end-to-end BI delivery for the Vietnamese banking and financial services sector.*
