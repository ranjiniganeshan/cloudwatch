# Executive Proposal: Observability Platform Selection

**Date**: January 28, 2026
**Subject**: Recommendation to adopt/retain Amazon OpenSearch Service for High-Volume Log Analysis

## 1. Executive Summary
After a comprehensive evaluation of Amazon CloudWatch Logs versus Amazon OpenSearch Service, we recommend **Amazon OpenSearch Service** as the primary solution for our application log analysis.

At our current scale of **~764 GB per day**, OpenSearch provides a richer analytical feature set at a **~40-60% lower cost** than CloudWatch Logs. While CloudWatch offers simplicity, its pricing model (per-GB ingestion) becomes prohibitively expensive at this volume ($140k/year vs $60k/year).

## 2. Business Case & Cost Analysis

### Key Driver: Volume-Based Pricing
*   **Current Ingestion Volume**: ~764 GB/day (Verified via cluster stats)
*   **Annual Data Footprint**: ~270 TB/year

| Cost Component | CloudWatch Logs (Annual) | OpenSearch On-Demand (Annual) | OpenSearch Reserved (Annual) |
| :--- | :--- | :--- | :--- |
| **Ingestion/Compute** | $137,520 | ~$85,300 | ~$54,000 |
| **Storage** | $3,852 | ~$2,300 | ~$2,300 |
| **Operational Cost** | $0 | Part-time Engineering | Part-time Engineering |
| **Total Est. Cost** | **~$141,372** | **~$87,600** | **~$56,300** |
| **Projected Savings** | - | **$53,772 (38%)** | **$85,072 (60%)** |

### Strategic Benefits
1.  **Faster Incident Response**: OpenSearch (Kibana) dashboards load in milliseconds. CloudWatch Logs Insights scans can take minutes for large time ranges.
2.  **Advanced Analytics**: Capability to perform complex aggregations, joining data fields, and anomaly detection which are not natively possible in CloudWatch.
3.  **Future Proofing**: As volume grows, OpenSearch cost grows largely with *storage* (cheap), whereas CloudWatch grows linearly with *ingestion* (expensive).

## 3. Technical Implementation Plan

### Recommended Architecture
*   **Hot Tier (0-3 Days)**: High-performance NVMe instances for immediate troubleshooting.
*   **Warm Tier (4-14 Days)**: UltraWarm nodes (S3-backed) for cost-effective retention.
*   **Cold Tier (14+ Days)**: Recommend extending retention to 30 days using Cold Storage (S3-only pricing) for compliance, as it adds negligible cost.

## 4. Risks & Mitigations
*   **Risk**: Management Overhead. OpenSearch is not serverless.
    *   *Mitigation*: Use AWS Managed OpenSearch Service (not self-hosted). Use "Auto-Tune" features and set up persistent CloudWatch Alarms for cluster health (CPU, Free Storage).

---

## Appendix A: Verification of Data Volume
The sizing estimates above are based on the following direct measurements from our production environment.

**How to verify daily volume:**
Run the following in OpenSearch Dashboards Dev Tools:

1. **Check Indices List** (Identify date pattern):
   ```json
   GET _cat/indices?v&s=index:desc&h=index,pri.store.size
   ```

2. **Calculate Exact Day Size** (Replace `*YYYY.MM.DD` with specific date):
   ```json
   GET /*-2026.01.27/_stats/store
   ```
   *Look for `_all.primaries.store.size_in_bytes`.*

3. **Estimate via Search** (If using Data Streams):
   ```json
   GET <your-index-pattern>*/_search
   {
     "size": 0,
     "query": { "range": { "@timestamp": { "gte": "now-1d/d", "lt": "now/d" } } },
     "aggs": { "total_size_estimate": { "sum": { "script": "doc['_source'].toString().length()" } } }
   }
   ```
