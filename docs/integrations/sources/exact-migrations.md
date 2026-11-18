# Exact Migration Guide

## Upgrading to 1.0.0

### What changed

- **Primary keys of sync streams.** All `sync_*` streams now use `Division` + `ID` as primary key instead of `Timestamp`.
  `Timestamp` is a row version that changes every time a record is edited, so in **Incremental | Append + Deduped**
  tables every edited record was kept once per version. `sync_subscription_subscriptions` uses `Division` + `EntryID`
  (this endpoint has no `ID`).
- **Primary key of `openingbalance_current_year_after_entry`.** Now `Division` + `ReportingYear` + `GLAccount`. The
  previous key (`ID`) does not exist on this endpoint.
- **Full refresh only.** Endpoints without a `Modified` field no longer offer incremental sync:
  `financial_gl_account_classification_mappings`, `financial_reporting_balance`, `financial_receivables_list` and
  `openingbalance_current_year_after_entry`. Exact rejected the `$orderby=Modified` the connector sent for them.

### Who is affected

| Sync mode of the stream                 | Action                                                   |
|:----------------------------------------|:---------------------------------------------------------|
| Full Refresh (Overwrite or Append)      | None                                                     |
| Incremental, Append                     | None                                                     |
| Incremental, Append + Deduped           | Refresh the stream after upgrading                       |
| Incremental on a stream now full-only   | Switch the stream to Full Refresh                        |

To check whether a deduped table holds duplicate versions before upgrading:

```sql
SELECT "Division", "ID", COUNT(*)
FROM sync_financial_transaction_lines
GROUP BY 1, 2
HAVING COUNT(*) > 1;
```

### Steps

1. Upgrade the connector to 1.0.0.
2. In each connection, refresh the source schema and accept the changes.
3. Refresh (clear and resync) the impacted streams that use **Append + Deduped**.
4. Update downstream models or tests that rely on `Timestamp` being unique.
