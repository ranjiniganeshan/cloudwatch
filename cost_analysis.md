# Cost Analysis: CloudWatch Logs vs OpenSearch
## Sources https://aws.amazon.com/cloudwatch/pricing/
# Cost Analysis: CloudWatch Logs vs OpenSearch

**Basis of Estimate**
*   **Daily Ingestion**: 764 GB / day
*   **Retention**: 14 Days (derived from ISM policy)
    *   **Hot**: 3 Days
    *   **Warm**: 11 Days
*   **Region**: Assuming `us-east-1` (Standard pricing)

---

## 1. Amazon CloudWatch Logs
CloudWatch pricing is simple but linear. It scales directly with volume.

### Monthly Costs
1.  **Ingestion**:
    *   764 GB/day × 30 days = **22,920 GB/month**
    *   Price: $0.50 per GB
    *   Cost: **$11,460 / month**
2.  **Storage**:
    *   Accumulated Storage: ~10.7 TB (Average steady state for 14 days)
    *   Price: $0.03 per GB
    *   Cost: **$321 / month**
3.  **Logs Insights (Querying)**:
    *   *Variable*: Assumed 1 TB scanned per day (light usage).
    *   Price: $0.005 per GB.
    *   Cost: **$150 / month**

### **Total CloudWatch: ~$11,931 / month**
> [!NOTE]
> Zero operational overhead. Price is pure "Pay for what you use".

---

## 2. Amazon OpenSearch Service (Managed)
OpenSearch requires provisioned infrastructure. At 764 GB/day, you need a robust cluster.

### Architecture Assumptions
*   **Hot Tier**: 3 Days × 764 GB × 2 (1 Replica) = **4.6 TB** required.
    *   *Recommendation*: 3x `r6g.2xlarge.search` (Data Nodes) + EBS.
*   **Warm Tier**: 11 Days × 764 GB = **8.4 TB** (UltraWarm uses S3, no replicas needed).
    *   *Recommendation*: 2x `ultrawarm1.large.search` (Holds up to 20TB).

### Monthly Costs (Estimated)
1.  **Hot Nodes (Compute)**:
    *   3 instances (`r6g.2xlarge.search`) × ~$1.06/hr × 730 hrs
    *   Cost: **~$2,321 / month**
2.  **Hot Storage (EBS gp3)**:
    *   5 TB Provisioned × $0.08/GB
    *   Cost: **~$400 / month**
3.  **UltraWarm Nodes**:
    *   2 instances (`ultrawarm1.large.search`) × ~$2.68/hr × 730 hrs
    *   Cost: **~$3,912 / month**
4.  **Master Nodes**:
    *   3 instances (`c6g.large.search`) × ~$0.13/hr × 730 hrs
    *   Cost: **~$285 / month**
5.  **UltraWarm Storage (S3)**:
    *   8.4 TB × $0.023/GB
    *   Cost: **~$193 / month**

### **Total OpenSearch: ~$7,111 / month**
*Includes Reserved Instance savings? No (On-Demand pricing used).*
*> [!TIP]
> **Savings Opportunity**: Purchasing 1-year All Upfront RIs for OpenSearch can reduce compute costs by ~30-40%, bringing this closer to **$4,500/month**.*

---

## Summary Verdict
| Metric | CloudWatch Logs | OpenSearch (On-Demand) | OpenSearch (Reserved) |
| :--- | :--- | :--- | :--- |
| **Monthly Cost** | **$11,931** | **~$7,111** | **~$4,500** |
| **Difference** | Baseline | **~40% Cheaper** | **~60% Cheaper** |

**Conclusion**: At **764 GB/day**, OpenSearch is significantly more cost-effective, saving between $50k - $80k annually. CloudWatch is only recommended if you absolutely cannot spare engineering time to manage the OpenSearch cluster configuration.
