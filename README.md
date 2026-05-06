# Claims-Summary-Report

## Table of Contents
- [Project Overview](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning & Preparation](#data-cleaning--preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Dashboard](#dashboard)
- [Results & Findings](#results--findings)
- [Recommendations](#recommendations)
- [Limitations](#limitations)
- [References](#references)

---

  
### Project Overview
Healthcare managed care contracting analytics dashboard built in Power BI. Analyzes payer reimbursement performance across 8 payers using DAX-driven metrics to support contract negotiation. This dashboard provides managed care contracting teams with data-driven insights to support payer contract negotiations, benchmark reimbursement rates, and identify revenue optimization opportunities.

Key features include YoY trending, Reimbursement Index, Varaiance from Benchmark, and Rate Increase Needed metrics - all built on a star schema data model in Power BI

---

### Data Sources
- **FactClaims.csv** - 5,000 medical claims records (2022-2023)
  containing paid amounts, claims types, diagnosis and procedure codes
- **DimMember.csv** - 500 member records with demographics, plan type, and enrollment data
- **DimProvider.csv** - 50 provider records with specialty, NPI, and provider type
- **DimPayer.csv** - 8 payer records across Commercial, Medicare, Medicaid, and Medicare Advantage
- **DimDate.csv** - 2022-2023 date dimension table


> Note: All datasets are synthetic and were generated for
> analytical purposes. No real PHI or PII is contained
> in this repository.

---

### Tools
- **Power BI Desktop** - Data Modelling, DAX, and dashboard visualization
- **DAX** - KPI measures, time intelligence.
- **Power Query** - Data loading and type validation

---

### Data Cleaning & Preparation
The following steps were performed during data prepartion:

1. Loaded all 5 csv files into Power BI
2. Validated column data types (dates, decimals, whole numbers)
3. Established star schema relationships.
4. Marked DimDate as official date table
5. Configured MonthName sort by MonthNum for correct chronological ordering.

---
   
### Exploratory Data Analysis
- What is the the total paid amount, claim amount, and unique member count across all payers?
- How does average cost per claim and cost per member vary by payer type?
- Which payers are reimbursing above or below the portfolio benchmark?
- How has total paid amount trended month over month compared to the prior year?
- Which provider specialties drive the highest total paid amounts?

---

### Data Analysis
The following DAX measures were developed to support analysis

**Base Measures**
```dax
Total Amount Paid = SUM(FactClaims[PaidAmount])
Total Claims = COUNTROWS(FactClaims)
Unique Members = DISTINCTCOUNT(FactClaims[MemberID])
Avg Cost per Member = DIVIDE([Total Amount Paid],[Unique Members])
Avg Cost Per Claim = DIVIDE([Total Amount Paid],[Total Claims],0)
```
**Time Intelligence**
```dax
PY Total Amount Paid = 
  CALCULATE(
      SUM(
          FactClaims[PaidAmount]
      ),
      SAMEPERIODLASTYEAR(DimDate[Date])
  )
PY Total Claims = 
  CALCULATE(
      COUNTROWS(
          FactClaims
      ),
      SAMEPERIODLASTYEAR(DimDate[Date]))
PY Unique Members = 
  CALCULATE(
      DISTINCTCOUNT(
          FactClaims[MemberID]
      ),
      SAMEPERIODLASTYEAR(DimDate[Date])
  )
PY Avg Cost Per Claim = 
  DIVIDE(
      [PY Total Amount Paid],
      [PY Total Claims],
      0
  )
PY Avg  Cost per Member = 
  DIVIDE(
      [PY Total Amount Paid],
      [PY Unique Members],
      0
  )

```
**Reimbursement Analytics**
```dax

Reimbursement Index = 
  VAR OverallAvg = 
      CALCULATE(
          [Avg Cost Per Claim],
          ALL(DimPayer[PayerName])
      )
  VAR PayerAvg = [Avg Cost Per Claim]
  RETURN
      DIVIDE(PayerAvg, OverallAvg,0)

Reimbursement Variance = 
  VAR OverallAvg = 
      CALCULATE(
          [Avg Cost Per Claim],
          ALL(DimPayer[PayerName])
      )
  RETURN
      [Avg Cost Per Claim] - OverallAvg

Rate Increase Needed =
  VAR OverallAvg = 
      CALCULATE(
          [Avg Cost Per Claim],
          ALL(DimPayer[PayerName])
      )
  VAR PayerAvg = [Avg Cost Per Claim]
  RETURN
      DIVIDE(
          OverallAvg - PayerAvg,
          PayerAvg,
          0
      )

```
---

### Dashboard
[**Explore Dashboard Here**](https://app.powerbi.com/view?r=eyJrIjoiNzhjZWJkY2EtNmEzNC00Nzg4LWIxMDAtNzU5MGY0ZDJkMWFjIiwidCI6IjJmMDFkOTZmLTgyYjMtNDY5My1hNTM3LTAwMTkxMGVkNmUyYSIsImMiOjZ9)

---

### Results & Findings
1. **Total Paid amount grew 2.8% YoY** ($33.66M ➡️ $34.59) driven primarily by increased claim volume (+1.7%) and higher average cost per claim (+1.1%)
   
2. **Aetna and WellCare Medicaid are the lowest reimbursing payers** with Reimbursement Index Scores of 0.94 and 0.96 respectively -  both below the portfolio benchmark
   
3. **Humana Medicare Advantage and Cigna are the strongest performers** with index scores of 1.03, reimbursing 3% above portfolio avarage
 
4. **Oncology drives the highest specialty spend** followed by Cardiology and Internal Medicine
  
5. **Unique member count remained flat YoY (0.0% change)** suggetsing cost growth ius driven by utilization intensity and rate changes rather membership growth

### Recommendations
1. **Initiate contract renegotiation with Aetna** - a 6% rate increase would bring reimbursement to portfolio benchmark. Open at 8-9% to allow negotiation room.

2. **Review Wellcare Medicaid contract terms** - consider requesting care management supplement payments given the SDOH burden associated with Medicaid populations.

3. **Monitor BlueCross BlueShield** - currently in the yellow zone (0.99 Index). Flag for 2-3% trend increase at next renewal cycle.

4. **Investigate flat member count alongside rising costs** - unchanged membership with growing spend suggests increasing case mix complexity requiring utilization management review.

5. **Expand specialty cost analysis** - conduct deeper dive into oncology spend given its disproportionate share of total paid amount.

### Limitations

### References

🚀🚀🚀
