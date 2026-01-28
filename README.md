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

   {
  "_shards": {
    "total": 920,
    "successful": 920,
    "failed": 0
  },
  "_all": {
    "primaries": {
      "store": {
        "size_in_bytes": 820047661984,
        "reserved_in_bytes": 0
      }
    },
    "total": {
      "store": {
        "size_in_bytes": 1641943655527,
        "reserved_in_bytes": 0
      }
    }
  },
  "indices": {
    "mvision-eventhub-engine-2026.01.27": {
      "uuid": "jTPWoqAZTBOl0s0jKhEBbA",
      "primaries": {
        "store": {
          "size_in_bytes": 140323667,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 281552957,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw003-dxl_broker_hub-2026.01.27": {
      "uuid": "HG-8JJScRsqbo2C3sbjbCA",
      "primaries": {
        "store": {
          "size_in_bytes": 65095569,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 131309595,
          "reserved_in_bytes": 0
        }
      }
    },
    "dlp2-messages-all-2026.01.27": {
      "uuid": "-_4--ZcxTfOClANqR6kNsw",
      "primaries": {
        "store": {
          "size_in_bytes": 6883880214,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 13806579278,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw002-scheduler-2026.01.27": {
      "uuid": "7LfaCVHFRimouuNecbxAqA",
      "primaries": {
        "store": {
          "size_in_bytes": 8314727,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 29193901,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw004-app_server-2026.01.27": {
      "uuid": "ri3AVjVuSLW6QQuxt35TWw",
      "primaries": {
        "store": {
          "size_in_bytes": 777911515,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 2144233139,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw001-cds_server-2026.01.27": {
      "uuid": "cBzf76s7TGW6fDoaMYfiBg",
      "primaries": {
        "store": {
          "size_in_bytes": 4042214092,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 8162645776,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw004-agent_handler-2026.01.27": {
      "uuid": "KT1b8Je9TvGA9DHFxOow3g",
      "primaries": {
        "store": {
          "size_in_bytes": 2591620863,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 5362893592,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw003-tie_server-2026.01.27": {
      "uuid": "AMOyMX8IRd6hvKNgs5oy1g",
      "primaries": {
        "store": {
          "size_in_bytes": 56681023780,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 113366523859,
          "reserved_in_bytes": 0
        }
      }
    },
    "routingservice-2026.01.27": {
      "uuid": "UhvoLpmjTiex9yqT0shNvQ",
      "primaries": {
        "store": {
          "size_in_bytes": 29821399469,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 59659917612,
          "reserved_in_bytes": 0
        }
      }
    },
    "mvision-access-log-2026.01.27": {
      "uuid": "ynaUxYO2Q16woOpSCqVFZQ",
      "primaries": {
        "store": {
          "size_in_bytes": 121308934,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 241687617,
          "reserved_in_bytes": 0
        }
      }
    },
    "trigger-2026.01.27": {
      "uuid": "nPRAg3LZQdStohO-CfHMZA",
      "primaries": {
        "store": {
          "size_in_bytes": 386134316,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 769669674,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw005-app_server-2026.01.27": {
      "uuid": "A5i5bnZfQri8qtkNsYaT6g",
      "primaries": {
        "store": {
          "size_in_bytes": 1458951228,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 3220329146,
          "reserved_in_bytes": 0
        }
      }
    },
    "dlp2-stats-unprocessed-2026.01.27": {
      "uuid": "X6kj3mJLSB2sMJK2DLayUw",
      "primaries": {
        "store": {
          "size_in_bytes": 128037,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 256074,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw002-tie_server-2026.01.27": {
      "uuid": "XiM0GVLNQsan6CSjYJM44w",
      "primaries": {
        "store": {
          "size_in_bytes": 27972888581,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 55971459487,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-mcafee-tie_server-traffic-2026.01.27": {
      "uuid": "reaKt62eQAeUShZhVGgNzQ",
      "primaries": {
        "store": {
          "size_in_bytes": 315630326,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 626873847,
          "reserved_in_bytes": 0
        }
      }
    },
    "%{service}-2026.01.27": {
      "uuid": "2O2KaNmjTxiGstwNUfyHGg",
      "primaries": {
        "store": {
          "size_in_bytes": 637791924,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 1271903557,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw005-tie_server-traffic-2026.01.27": {
      "uuid": "eDbI9lVdQwafSsT_fgeOHw",
      "primaries": {
        "store": {
          "size_in_bytes": 1762140602,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 3548127745,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw005-gold_master-2026.01.27": {
      "uuid": "XsK1xbKAR-uhlzFBeFLfXw",
      "primaries": {
        "store": {
          "size_in_bytes": 14704556,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 28422998,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw002-tie_server-traffic-2026.01.27": {
      "uuid": "RjAlGnSASEqSLgYB4133_A",
      "primaries": {
        "store": {
          "size_in_bytes": 7074333941,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 14140571752,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-mcafee-tie_server-2026.01.27": {
      "uuid": "H3ll1ML0SviHFo55xbfC8Q",
      "primaries": {
        "store": {
          "size_in_bytes": 29098932672,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 58211693270,
          "reserved_in_bytes": 0
        }
      }
    },
    "tde-disk-and-volume-recovery-service-2026.01.27": {
      "uuid": "mqQ35C8aQnOfRRlwNC58sA",
      "primaries": {
        "store": {
          "size_in_bytes": 36653290,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 72824082,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw004-dxl_broker_hub-2026.01.27": {
      "uuid": "SQd6ZzLjQLCxO4vC8XfeZw",
      "primaries": {
        "store": {
          "size_in_bytes": 115141123,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 230192395,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw001-gold_master-2026.01.27": {
      "uuid": "pH8nTxNGS8e9mMyPyKyj7A",
      "primaries": {
        "store": {
          "size_in_bytes": 20492204,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 53751892,
          "reserved_in_bytes": 0
        }
      }
    },
    "event-filter-2026.01.27": {
      "uuid": "3eYLOleuQVuNOBIsy274hg",
      "primaries": {
        "store": {
          "size_in_bytes": 18507077872,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 37026212768,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw003-scheduler-2026.01.27": {
      "uuid": "XAxVLar-Ra6pgeesq2Lv4A",
      "primaries": {
        "store": {
          "size_in_bytes": 6694360,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 20405496,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw002-app_server-2026.01.27": {
      "uuid": "Dnx3MBDPR7Ow1F9Ro130sA",
      "primaries": {
        "store": {
          "size_in_bytes": 1349885622,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 2705726435,
          "reserved_in_bytes": 0
        }
      }
    },
    "dp-dlpgsmw-mw-2026.01.27": {
      "uuid": "zIiKrodyS-SAVRxN_bJlTg",
      "primaries": {
        "store": {
          "size_in_bytes": 98211900,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 195572900,
          "reserved_in_bytes": 0
        }
      }
    },
    "mvision-api-developerhub-2026.01.27": {
      "uuid": "ss6Z1Z1WRRi4AC95TTWWWA",
      "primaries": {
        "store": {
          "size_in_bytes": 108173,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 216346,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw003-cds_server-2026.01.27": {
      "uuid": "ICFyoWj5RCeUk_F_IHhw_g",
      "primaries": {
        "store": {
          "size_in_bytes": 3818126687,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 7751812903,
          "reserved_in_bytes": 0
        }
      }
    },
    "system-tree-sync-service-2026.01.27": {
      "uuid": "J66qdgAwQLWvlHdIDvE8mQ",
      "primaries": {
        "store": {
          "size_in_bytes": 1579850400,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 3130128058,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw001-tie_server-2026.01.27": {
      "uuid": "vnyJSruZQsaBYXGi95RcWQ",
      "primaries": {
        "store": {
          "size_in_bytes": 108291219435,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 216549786054,
          "reserved_in_bytes": 0
        }
      }
    },
    "tde-audit-service-2026.01.27": {
      "uuid": "-RdHBZPdTaKAikWf3Zc3wQ",
      "primaries": {
        "store": {
          "size_in_bytes": 10588290,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 20775756,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw005-dxl_broker_hub-2026.01.27": {
      "uuid": "5_zzbRDLSWStkNPMaMzXtw",
      "primaries": {
        "store": {
          "size_in_bytes": 21450035,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 43165730,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-mcafee-scheduler-2026.01.27": {
      "uuid": "PAr3nckwQ6yCuhg1rUEdWA",
      "primaries": {
        "store": {
          "size_in_bytes": 125350784,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 252495167,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw001-agent_handler-2026.01.27": {
      "uuid": "ibz2sTs_SpiAZJ2YzshwdQ",
      "primaries": {
        "store": {
          "size_in_bytes": 3654112660,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 7300440533,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-mcafee-gold_master-2026.01.27": {
      "uuid": "7pKkel5ER-am-zfk5f8plg",
      "primaries": {
        "store": {
          "size_in_bytes": 24774305,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 59324641,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw003-tie_server-traffic-2026.01.27": {
      "uuid": "6eupEPTKQjeFeJgYur0Kgw",
      "primaries": {
        "store": {
          "size_in_bytes": 10847543261,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 21670099812,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw003-app_server-2026.01.27": {
      "uuid": "EQapSmCjTeuZ-zwX2NcU2Q",
      "primaries": {
        "store": {
          "size_in_bytes": 2101401451,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 4452271579,
          "reserved_in_bytes": 0
        }
      }
    },
    "evaluator-cluster-2026.01.27": {
      "uuid": "O713mk6oTiuOMG9-llmfLg",
      "primaries": {
        "store": {
          "size_in_bytes": 69724235199,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 139410818705,
          "reserved_in_bytes": 0
        }
      }
    },
    "threat-event-parser-2026.01.27": {
      "uuid": "0ZZXmgCTS_GETcIrgaQkTg",
      "primaries": {
        "store": {
          "size_in_bytes": 53868978046,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 107670610870,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw002-agent_handler-2026.01.27": {
      "uuid": "IsI_s1xlQGmj7dqB0jYMZw",
      "primaries": {
        "store": {
          "size_in_bytes": 3676731798,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 7331632174,
          "reserved_in_bytes": 0
        }
      }
    },
    "mvision-cdc-service-2026.01.27": {
      "uuid": "zPyOICiLTJa4azt7TL-JgQ",
      "primaries": {
        "store": {
          "size_in_bytes": 221118394,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 441344492,
          "reserved_in_bytes": 0
        }
      }
    },
    "dlp2-messages-unprocessed-2026.01.27": {
      "uuid": "b2NZjSaTTReUGn_uymMFKg",
      "primaries": {
        "store": {
          "size_in_bytes": 66646,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 133292,
          "reserved_in_bytes": 0
        }
      }
    },
    "mvision-d2c-2026.01.27": {
      "uuid": "66CMYsxWQp2xcqOt4j7BPQ",
      "primaries": {
        "store": {
          "size_in_bytes": 1536936928,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 3020688236,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw004-gold_master-2026.01.27": {
      "uuid": "mI9udbMYQ5i1Yp9wtdKVDQ",
      "primaries": {
        "store": {
          "size_in_bytes": 20738750,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 41480259,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw002-cds_server-2026.01.27": {
      "uuid": "M2VczdOdT4aaLdB1_HdEdQ",
      "primaries": {
        "store": {
          "size_in_bytes": 10660187147,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 21494228438,
          "reserved_in_bytes": 0
        }
      }
    },
    "tde-user-data-service-2026.01.27": {
      "uuid": "ocU4hMFBTiiLNdqItmg3-g",
      "primaries": {
        "store": {
          "size_in_bytes": 184010060,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 365710777,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw004-tie_server-traffic-2026.01.27": {
      "uuid": "ofgHtkqsT3StRqOCvFqaWw",
      "primaries": {
        "store": {
          "size_in_bytes": 8611758601,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 17246113302,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw003-agent_handler-2026.01.27": {
      "uuid": "0cF_xXTRRK-VOtug964ibg",
      "primaries": {
        "store": {
          "size_in_bytes": 2582693943,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 5175310136,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw001-dxl_broker-2026.01.27": {
      "uuid": "yA9VQ76rRdm-an6xO9MN3g",
      "primaries": {
        "store": {
          "size_in_bytes": 4564045140,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 9149235966,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw001-tie_server-traffic-2026.01.27": {
      "uuid": "X5lKvfkaQEqnPSVUe9C_nw",
      "primaries": {
        "store": {
          "size_in_bytes": 18754545339,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 37479448699,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw005-cds_server-2026.01.27": {
      "uuid": "DomNjs7hTTKpxAFl-EFa1Q",
      "primaries": {
        "store": {
          "size_in_bytes": 426326791,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 878235184,
          "reserved_in_bytes": 0
        }
      }
    },
    "cds-notification-service-2026.01.27": {
      "uuid": "dGy-pfD-SKmSNJY7MqXnXA",
      "primaries": {
        "store": {
          "size_in_bytes": 1304853645,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 2603451403,
          "reserved_in_bytes": 0
        }
      }
    },
    "epo-api-2026.01.27": {
      "uuid": "vN1fgDdKSDaKtfvh2Eac1A",
      "primaries": {
        "store": {
          "size_in_bytes": 35627184,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 71160578,
          "reserved_in_bytes": 0
        }
      }
    },
    "tacc-reputation-2026.01.27": {
      "uuid": "Tiu5TaQpSXmS34fYYqN2pA",
      "primaries": {
        "store": {
          "size_in_bytes": 28568958,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 56771507,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw002-dxl_broker-2026.01.27": {
      "uuid": "RT4__oosRlW3KcKeePKN3g",
      "primaries": {
        "store": {
          "size_in_bytes": 4044865119,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 8089091998,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-mcafee-dxl_broker-2026.01.27": {
      "uuid": "x8vNAAOnRdyqQG9jwOiS4w",
      "primaries": {
        "store": {
          "size_in_bytes": 38260656,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 131158588,
          "reserved_in_bytes": 0
        }
      }
    },
    "mvision-api-servicehub-2026.01.27": {
      "uuid": "ol1eBjsOQyqkEyTbLk-nqw",
      "primaries": {
        "store": {
          "size_in_bytes": 63597867,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 127188994,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw004-cds_server-2026.01.27": {
      "uuid": "6djsNzFsTA6MhwKVJzONKQ",
      "primaries": {
        "store": {
          "size_in_bytes": 2698901464,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 5460902742,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw005-agent_handler-2026.01.27": {
      "uuid": "aUdyc02yQIOjeGG5hY3qcA",
      "primaries": {
        "store": {
          "size_in_bytes": 416911851,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 929854170,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-mcafee-agent_handler-2026.01.27": {
      "uuid": "yHb0t0SLRUOSo7UhtVLNvQ",
      "primaries": {
        "store": {
          "size_in_bytes": 208379460,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 400585567,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw004-scheduler-2026.01.27": {
      "uuid": "1ZjJDKd_Tm-WIf0cDfUiow",
      "primaries": {
        "store": {
          "size_in_bytes": 179474778,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 368854641,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw001-app_server-2026.01.27": {
      "uuid": "AHbEdnozSNuswcRFYi7hcw",
      "primaries": {
        "store": {
          "size_in_bytes": 1259276126,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 2391141385,
          "reserved_in_bytes": 0
        }
      }
    },
    "jwt-authorizer-2026.01.27": {
      "uuid": "seXhZbpVSh2NvT9trJy-6Q",
      "primaries": {
        "store": {
          "size_in_bytes": 505282366,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 1006791351,
          "reserved_in_bytes": 0
        }
      }
    },
    "dp-im-mw-2026.01.27": {
      "uuid": "oDtk8K-PR_WTaprZ2y2nlw",
      "primaries": {
        "store": {
          "size_in_bytes": 48820994,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 98262573,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw003-dxl_broker-2026.01.27": {
      "uuid": "G76FNiJET_yF_DKKF74t7A",
      "primaries": {
        "store": {
          "size_in_bytes": 3318709020,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 6641758931,
          "reserved_in_bytes": 0
        }
      }
    },
    "mvision-api-webhookcron-2026.01.27": {
      "uuid": "aREJ5RlpS-CtWf6SBV2IgQ",
      "primaries": {
        "store": {
          "size_in_bytes": 1081322,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 2145116,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw003-gold_master-2026.01.27": {
      "uuid": "8LKIDsKRQJyOgRWRmx5InQ",
      "primaries": {
        "store": {
          "size_in_bytes": 21099566,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 42056995,
          "reserved_in_bytes": 0
        }
      }
    },
    "mvision-d2c-access-2026.01.27": {
      "uuid": "DSJvqidRSq2nMoOv1uXwlg",
      "primaries": {
        "store": {
          "size_in_bytes": 2200282981,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 4396600196,
          "reserved_in_bytes": 0
        }
      }
    },
    "events-api-2026.01.27": {
      "uuid": "0c_gLjqNReW3NpgCPvb3zg",
      "primaries": {
        "store": {
          "size_in_bytes": 270273471,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 539282531,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw005-tie_server-2026.01.27": {
      "uuid": "KiZ_ovb3QKODsk08ahFkAQ",
      "primaries": {
        "store": {
          "size_in_bytes": 12071967092,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 24161809902,
          "reserved_in_bytes": 0
        }
      }
    },
    "enricher-2026.01.27": {
      "uuid": "ERrIAXw-QmWdiT5HNieUpA",
      "primaries": {
        "store": {
          "size_in_bytes": 23222132459,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 46418509023,
          "reserved_in_bytes": 0
        }
      }
    },
    "dp-sm-mw-2026.01.27": {
      "uuid": "GJIcaZJVQ1GKcnTXjRRD1g",
      "primaries": {
        "store": {
          "size_in_bytes": 2211574,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 4229301,
          "reserved_in_bytes": 0
        }
      }
    },
    "tacc-service-2026.01.27": {
      "uuid": "gKRWSHPzR_anibqmUsN7pA",
      "primaries": {
        "store": {
          "size_in_bytes": 371149,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 742298,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw001-dxl_broker_hub-2026.01.27": {
      "uuid": "uG-iKxLCRV2MFS3D31G6rw",
      "primaries": {
        "store": {
          "size_in_bytes": 35628444,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 71326648,
          "reserved_in_bytes": 0
        }
      }
    },
    "tde-user-assignment-service-2026.01.27": {
      "uuid": "1fCDDD-7QGeZhYQtsaPlDQ",
      "primaries": {
        "store": {
          "size_in_bytes": 2878803560,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 5749614723,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-mcafee-app_server-2026.01.27": {
      "uuid": "bgVcvOFCRp2gfzISIQQizA",
      "primaries": {
        "store": {
          "size_in_bytes": 261309731,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 693679410,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw004-dxl_broker-2026.01.27": {
      "uuid": "ZbMy0sJLQZiNJvSAXES81g",
      "primaries": {
        "store": {
          "size_in_bytes": 5107471907,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 10237929854,
          "reserved_in_bytes": 0
        }
      }
    },
    "dlp2-stats-systemstats-2026.01.27": {
      "uuid": "zB8cSKJPQqeMJK7_qmJGTA",
      "primaries": {
        "store": {
          "size_in_bytes": 35539197,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 71705422,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw002-dxl_broker_hub-2026.01.27": {
      "uuid": "F6rbzs5wQJ6kALmRsMbt9Q",
      "primaries": {
        "store": {
          "size_in_bytes": 119545385,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 240452578,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw001-scheduler-2026.01.27": {
      "uuid": "6GeGkCcRTNKqWLd3bY7DCw",
      "primaries": {
        "store": {
          "size_in_bytes": 5225279,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 19141426,
          "reserved_in_bytes": 0
        }
      }
    },
    "adapter-2026.01.27": {
      "uuid": "iUfP3THVRMWwRt6zJyKUlA",
      "primaries": {
        "store": {
          "size_in_bytes": 28232612290,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 56406451155,
          "reserved_in_bytes": 0
        }
      }
    },
    "evaluator-2026.01.27": {
      "uuid": "-TlMsMV0Q9iFqcdIKbS14w",
      "primaries": {
        "store": {
          "size_in_bytes": 134024416449,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 267875962312,
          "reserved_in_bytes": 0
        }
      }
    },
    "frp-event-parser-2026.01.27": {
      "uuid": "wNouMttASIOkOIbCS3yWvA",
      "primaries": {
        "store": {
          "size_in_bytes": 544636,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 1102101,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw005-scheduler-2026.01.27": {
      "uuid": "bNEBlHt7QY2ozcWvtvF3tg",
      "primaries": {
        "store": {
          "size_in_bytes": 83692871,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 176215918,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw004-tie_server-2026.01.27": {
      "uuid": "6jiyL8urSjC_zqAEUDPGFg",
      "primaries": {
        "store": {
          "size_in_bytes": 51762056912,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 103580506766,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw005-dxl_broker-2026.01.27": {
      "uuid": "dxUg__TpSwy5B7rPZMXomg",
      "primaries": {
        "store": {
          "size_in_bytes": 613658421,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 1247446703,
          "reserved_in_bytes": 0
        }
      }
    },
    "dlp2-stats-eventstats-2026.01.27": {
      "uuid": "Xq4FUaTIQryByClEYtMITQ",
      "primaries": {
        "store": {
          "size_in_bytes": 15772021340,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 31632272265,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-mcafee-dxl_broker_hub-2026.01.27": {
      "uuid": "a8r6KB9WQKWxDoZA0YbksQ",
      "primaries": {
        "store": {
          "size_in_bytes": 8822963,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 17719654,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-mcafee-cds_server-2026.01.27": {
      "uuid": "95nD7OHJR6uoXj3s5t0AjA",
      "primaries": {
        "store": {
          "size_in_bytes": 85234506,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 177621583,
          "reserved_in_bytes": 0
        }
      }
    },
    "usw2-epo-usw002-gold_master-2026.01.27": {
      "uuid": "MsOiVgsSTUqEwkDZY9BziA",
      "primaries": {
        "store": {
          "size_in_bytes": 20118582,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 43691320,
          "reserved_in_bytes": 0
        }
      }
    },
    "kafka-dynamodb-service-2026.01.27": {
      "uuid": "UTWdETezT4i_Cl2cJpPfUQ",
      "primaries": {
        "store": {
          "size_in_bytes": 33764226762,
          "reserved_in_bytes": 0
        }
      },
      "total": {
        "store": {
          "size_in_bytes": 67542427946,
          "reserved_in_bytes": 0
        }
      }
    }
  }
}
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
