# RCM-Solution-Pipeline and Executive Dashboard
The pipeline cut end-to-end processing time by 20% overall and empowered executives to track KPIs daily rather than weekly. Reducing report time 95%, data reconciliation errors by 90%.
The problem: The Revenue cycle management (RCM) team at our healthcare institution struggled with manual claims data extraction from Epic and Databricks. Reports took 3- 4 days to produce, were error-prone due to manual reconciliation, and executives lacked real-time visibility into denial rates, average reimbursement time, and provider performance. The goal was to build an automated pipeline that:
•	Ingested claims data from multiple sources
•	Cleaned and standardized the data
•	Loaded it into a structured warehouse
•	Powered automated dashboards for executive decision-making
Tools Used
Layers	Tools
Orchestration	SQL Server Integration Services (SSIS)
Database/Warehouse	SQL Server/ SSMS
Data Source	Databricks (raw claims) + Epic (EHR Reference data)
Visualization	Power BI + Excel (automated refresh)
Validation	SQL scripts + custom SSIS data flow tasks

Steps
Phase 1: Requirements & Data Mapping 
· Met with RCM SMEs to define KPIs: denial rate by payer, average days to payment, top denied procedure codes, provider-level approval rates
· Mapped source fields from Databricks (claims, payments, patients) and Epic (providers, facilities) to a target star schema
· Defined data quality rules: no null ClaimIDs, valid date ranges, ClaimStatus  {Approved, Denied, Pending}

Phase 2: ETL Development in SSIS 
· Built SSIS packages with:
  · Data Flow Tasks to extract from Databricks (via ODBC) and Epic (via API/flat files)
  · Lookup Transformations to map Provider IDs and Procedure Codes
  · Slowly Changing Dimensions (SCD Type 1 & 2) for provider and facility attributes
· Used For-Each Loop containers to process claims in batches (~50K rows per batch), handling ~3M rows total
· Implemented error logging to a SQL table for failed rows with detailed reason codes

Phase 3: Validation & Reconciliation 
· Created SQL validation scripts to:
  · Compare row counts between source and staging
  · Check referential integrity (every claim has a valid ProviderID)
  · Flag claims with amounts outside expected ranges
· Built a reconciliation report in Excel that automated analysts could run each morning to verify data completeness before dashboard refresh

Phase 4: Dashboard Development 
· Designed Power BI dashboards with:
  · Executive summary: 6 KPI cards (total claims, denial %, avg payment time, outstanding $, top 5 payers by denial rate, provider ranking)
  · Trend lines: claims volume and denial rate by week
  · Drill-through click any provider to see their detailed claims history
· Set up automated data refresh via SSIS scheduled jobs running at 6:00 AM daily

Phase 5: Testing & Deployment (Week 9)
· Ran UAT with RCM team: compared dashboard metrics against manual reports for 2 weeks
· Trained 5 team members on dashboard usage and basic troubleshooting
· Deployed to production with monitoring alerts for pipeline failures

Measurable Impact
Metric Before After Improvement
Report generation time 3–4 days ~15 minutes (automated) ~95% reduction
Data reconciliation errors ~12 per month 0–1 per month ~90% reduction
Executive decision speed Weekly review Daily review Real-time visibility
Claim denial identification 7–10 days post-submission 24–48 hours Early intervention enabled
This project successfully cut end-to-end processing time by 20% overall and empowered executives to track KPIs daily rather than weekly.

Sample Code Snippet (SSIS Data Flow Validation)
```sql
-- Post-load validation check
SELECT 
    'Orphaned Claims' as CheckType,
    COUNT(*) as IssueCount
FROM stg_claims c
LEFT JOIN dim_provider p ON c.ProviderID = p.ProviderID
WHERE p.ProviderID IS NULL
UNION ALL
SELECT 
    'Invalid ClaimStatus',
    COUNT(*) 
FROM stg_claims 
WHERE ClaimStatus NOT IN ('Approved', 'Denied', 'Pending');
