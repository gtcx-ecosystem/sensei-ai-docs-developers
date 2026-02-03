# Error Codes

## Error Code Reference and Solutions

Comprehensive list of Sensei error codes with explanations and solutions.

---

### Error Code Format

```text
E[CATEGORY][NUMBER]

Categories:
1xxx - Connection errors
2xxx - Schema/mapping errors
3xxx - Transformation errors
4xxx - Validation errors
5xxx - Execution errors
6xxx - System errors
7xxx - API errors
```

---

### Connection Errors (E1xxx)

| Code  | Message                   | Cause                    | Solution                        |
| ----- | ------------------------- | ------------------------ | ------------------------------- |
| E1001 | Connection refused        | Database not reachable   | Check host, port, firewall      |
| E1002 | Authentication failed     | Invalid credentials      | Verify username/password        |
| E1003 | SSL/TLS error             | Certificate issue        | Check SSL configuration         |
| E1004 | Connection timeout        | Network latency          | Increase timeout, check network |
| E1005 | Database not found        | Invalid database name    | Verify database exists          |
| E1006 | Permission denied         | Insufficient privileges  | Grant required permissions      |
| E1007 | Too many connections      | Connection limit reached | Reduce workers, increase limit  |
| E1008 | Connection pool exhausted | All connections in use   | Wait or increase pool size      |
| E1009 | Invalid connection string | Malformed connection URL | Check connection format         |
| E1010 | Host not resolved         | DNS resolution failed    | Check hostname, DNS settings    |

---

### Schema/Mapping Errors (E2xxx)

| Code  | Message                 | Cause                       | Solution                        |
| ----- | ----------------------- | --------------------------- | ------------------------------- |
| E2001 | Table not found         | Source table doesn't exist  | Verify table name and schema    |
| E2002 | Column not found        | Column missing              | Check column exists             |
| E2003 | Unsupported data type   | Type cannot be mapped       | Configure custom mapping        |
| E2004 | Schema mismatch         | Source/target incompatible  | Review schema differences       |
| E2005 | Primary key missing     | Table has no PK             | Add PK or configure alternative |
| E2006 | Foreign key violation   | Referenced row missing      | Check referential integrity     |
| E2007 | Constraint conflict     | Constraint prevents mapping | Modify target constraints       |
| E2008 | Index creation failed   | Index cannot be created     | Check index definition          |
| E2009 | Sequence sync failed    | Sequence value mismatch     | Manually sync sequence          |
| E2010 | Schema analysis timeout | Analysis taking too long    | Increase timeout, reduce scope  |

---

### Transformation Errors (E3xxx)

| Code  | Message                       | Cause                      | Solution                        |
| ----- | ----------------------------- | -------------------------- | ------------------------------- |
| E3001 | Type conversion failed        | Cannot convert value       | Check source data, mapping      |
| E3002 | Null value in non-null column | Null not allowed           | Configure default or fix source |
| E3003 | Value truncation              | Value too long for target  | Increase target size            |
| E3004 | Numeric overflow              | Value exceeds target range | Use larger numeric type         |
| E3005 | Date parsing error            | Invalid date format        | Configure date format           |
| E3006 | Encoding error                | Character encoding issue   | Specify source encoding         |
| E3007 | Custom transform failed       | User transform errored     | Debug custom function           |
| E3008 | Expression evaluation failed  | Invalid expression         | Check transformation rule       |
| E3009 | Lookup failed                 | Reference value not found  | Verify lookup table             |
| E3010 | Regex pattern error           | Invalid regex              | Fix regex pattern               |

---

### Validation Errors (E4xxx)

| Code  | Message                    | Cause                       | Solution                     |
| ----- | -------------------------- | --------------------------- | ---------------------------- |
| E4001 | Row count mismatch         | Source/target counts differ | Check for filtering, errors  |
| E4002 | Checksum mismatch          | Data integrity issue        | Investigate specific records |
| E4003 | Null rate deviation        | Null percentage changed     | Check transformations        |
| E4004 | Distribution anomaly       | Value distribution changed  | Review transformation logic  |
| E4005 | Foreign key invalid        | Referenced row missing      | Check load order             |
| E4006 | Unique constraint violated | Duplicate values            | Deduplicate or fix source    |
| E4007 | Check constraint failed    | Value fails constraint      | Fix value or constraint      |
| E4008 | Semantic validation failed | Business rule violated      | Review business logic        |
| E4009 | Time-Travel test failed    | Behavioral difference       | Investigate query results    |
| E4010 | Validation timeout         | Validation taking too long  | Increase timeout, sample     |

---

### Execution Errors (E5xxx)

| Code  | Message                   | Cause                      | Solution                      |
| ----- | ------------------------- | -------------------------- | ----------------------------- |
| E5001 | Migration already running | Duplicate execution        | Wait for current to complete  |
| E5002 | Migration not found       | Invalid migration ID       | Check migration exists        |
| E5003 | Checkpoint not found      | Resume point missing       | Start from beginning          |
| E5004 | Worker failed             | Worker process crashed     | Check worker logs             |
| E5005 | Batch insert failed       | Bulk insert error          | Reduce batch size             |
| E5006 | Lock timeout              | Table locked               | Wait or reduce concurrency    |
| E5007 | Disk space exhausted      | Storage full               | Free space or add storage     |
| E5008 | Memory exhausted          | Out of memory              | Increase memory, reduce batch |
| E5009 | Rate limit exceeded       | Too many operations        | Slow down, increase limits    |
| E5010 | CDC lag too high          | Replication falling behind | Scale up, check source        |

---

### System Errors (E6xxx)

| Code  | Message                | Cause                      | Solution                   |
| ----- | ---------------------- | -------------------------- | -------------------------- |
| E6001 | Internal server error  | Unexpected error           | Contact support with logs  |
| E6002 | Service unavailable    | System maintenance         | Retry later                |
| E6003 | Configuration error    | Invalid configuration      | Check configuration        |
| E6004 | License error          | License issue              | Check license status       |
| E6005 | Feature not available  | Plan doesn't include       | Upgrade plan               |
| E6006 | Dependency unavailable | Required service down      | Check service status       |
| E6007 | Version incompatible   | Component version mismatch | Upgrade components         |
| E6008 | Resource exhausted     | System limit reached       | Contact support            |
| E6009 | Backup failed          | Backup operation error     | Check storage, permissions |
| E6010 | Recovery failed        | Restore operation error    | Check backup integrity     |

---

### API Errors (E7xxx)

| Code  | Message                 | Cause                    | Solution               |
| ----- | ----------------------- | ------------------------ | ---------------------- |
| E7001 | Invalid API key         | Key invalid or expired   | Generate new key       |
| E7002 | Unauthorized            | Permission denied        | Check key permissions  |
| E7003 | Invalid request         | Malformed request        | Check request format   |
| E7004 | Resource not found      | Entity doesn't exist     | Check ID               |
| E7005 | Validation error        | Invalid input            | Check input parameters |
| E7006 | Rate limited            | Too many requests        | Slow down requests     |
| E7007 | Conflict                | Resource state conflict  | Refresh and retry      |
| E7008 | Payload too large       | Request body too big     | Reduce payload size    |
| E7009 | Unsupported media type  | Wrong content type       | Use application/json   |
| E7010 | Webhook delivery failed | Cannot reach webhook URL | Check URL, firewall    |

---

### Getting Help

If an error persists after trying the suggested solution:

1. **Check logs** for additional details
2. **Search documentation** for related issues
3. **Ask the community** at [community.sensei.ai](https://community.sensei.ai)
4. **Contact support** with error code, migration ID, and logs

→ [Troubleshooting](../../enterprise/resources/support/troubleshooting.md) → [Support](../../enterprise/resources/support/)
