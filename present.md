 Verify exact AWS CloudWatch Logs pricing tiers via search
 Calculate monthly ingestion costs based on 1.64 TB/day



 # CloudWatch Logs Cost Estimation
**Daily Ingestion:** 1.64 TB  
**Monthly Ingestion:** ~49.2 TB (based on 30 days)  
**Total Monthly GB:** 50,381 GB
## 1. Summary of Estimated Costs
| Log Class | Estimated Monthly Cost | Price Per GB |
| :--- | :--- | :--- |
| **Standard Logs** (Default) | **$25,190.40** | $0.50 (Flat) |
| **Infrequent Access Logs** | **$12,595.20** | $0.25 (Flat) |
| **Vended Logs** (e.g., VPC Flow) | **~$12,206.08** | Tiered ($0.50 - $0.10) |
> [!IMPORTANT]
> **Standard application logs** typically incur the **Standard** flat rate. The tiered pricing shown for Vended Logs generally applies only to specific AWS service logs (VPC Flow, Route 53) or if you are using the new Lambda logging tiers (checking availability).
## 2. Detailed Breakdown
### Scenario A: Standard Logs (Most Common)
For custom application logs sent via CloudWatch Agent, Fluentd, etc.
*   **Rate:** $0.50 per GB (us-east-1)
*   **Calculation:** 50,381 GB * $0.50
*   **Total:** **$25,190.40**
### Scenario B: Infrequent Access Class
If you do not need features like Live Tail, metric extraction, or wildcards in subscriptions.
*   **Rate:** $0.25 per GB
*   **Calculation:** 50,381 GB * $0.25
*   **Total:** **$12,595.20**
### Scenario C: Vended Logs (Volume Tiers)
Applies *only* to natively vended logs like VPC Flow Logs or Route 53.
*   **First 10 TB:** 10,240 GB * $0.50 = $5,120.00
*   **Next 20 TB:** 20,480 GB * $0.25 = $5,120.00
*   **Next 19.2 TB:** 19,661 GB * $0.10 = $1,966.10
*   **Total:** **$12,206.10**
## 3. Storage Costs (Additional)
In addition to ingestion, you pay for storage.
*   **Rate:** $0.03 per GB/month.
*   Archiving 49.2 TB would cost an additional **~$1,511** per month (cumulative as data grows).
