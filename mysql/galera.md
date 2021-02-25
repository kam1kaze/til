How to calculate `gcache.size`

```
SELECT ROUND(SUM(bytes)/1024/1024*60) AS megabytes_per_hour 
FROM  (
    SELECT SUM(VARIABLE_VALUE) * -1 AS bytes        
    FROM information_schema.global_status        
    WHERE VARIABLE_NAME IN ('wsrep_received_bytes', 'wsrep_replicated_bytes')    
    UNION ALL  SELECT sleep(60) AS bytes 
    UNION ALL  SELECT SUM(VARIABLE_VALUE) AS bytes        
    FROM information_schema.global_status        
    WHERE VARIABLE_NAME IN ('wsrep_received_bytes',           
                            'wsrep_replicated_bytes')    
) AS COUNTED
```

```
Example:
+--------------------+
| megabytes_per_hour |
+--------------------+
| 14474.0            |
+--------------------+
1 row in set
Time: 60.171s
```
