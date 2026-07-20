AWS DMS Quick Reference


# AWS DMS — Quick Reference


< 10 GB → t3.large → Single Task 10–100 GB → r5.large → Multi-Task 100 GB–1 TB → r5.2xlarge → Multi-Task

    1 TB → r5.4xlarge+ → Multi-Instance


## Architecture Patterns

**Pattern A (Simple):** Single task → small DBs, dev/test  
**Pattern B (Recommended):** Split by Large / High-DML / LOB / Reference tables  
**Pattern C (Enterprise):** Multiple replication instances for >1 TB  

## Full Load + CDC (3 Phases)

1. **Full Load** — Drop indexes, disable Multi-AZ on target, parallel load, low-traffic window  
2. **Cached Changes** — Apply buffered CDC, recreate indexes, re-enable Multi-AZ, validate  
3. **Ongoing CDC** — `BatchApplyEnabled=true`, monitor latency, cutover at CDCLatency ≈ 0  

## Source CDC Prerequisites

| Source | Key Requirements |
|--------|-----------------|
| Oracle | ARCHIVELOG ON, supplemental logging |
| SQL Server | MS-CDC enabled, SQL Agent running |
| PostgreSQL | `wal_level=logical`, `heartbeatEnable=true` |
| MySQL | `binlog_format=ROW`, `binlog_row_image=FULL` |

## Essential Task Settings

Settings
MySQL/Aurora	parallelLoadThreads=4-8, disable FK checks
PostgreSQL	maxFileSize=512, executeTimeout=100
Redshift	fileTransferUploadStreams=10, compUpdate=false
S3	dataFormat=parquet, datePartitionEnabled=true
Common Mistakes

    T3 for production CDC → Use R5/C5
    Single task for all tables → Split by workload
    Full LOB mode unnecessarily → Use Limited LOB
    Multi-AZ on target during full load → Disable temporarily
    Not monitoring CDCLatencyTarget → Monitor continuously
    Starting during peak traffic → Use low-traffic window

Pre-Migration Checklist

    Instance sized (r5.xlarge+ for prod)
    Security groups: RI → Source/Target
    Source CDC prerequisites met
    SCT run, action items resolved
    LOB strategy defined
    Large tables → parallel load configured
    CloudWatch alarms on CDCLatency
    Rollback plan documented
