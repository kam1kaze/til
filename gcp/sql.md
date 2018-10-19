## Change intance flag (mysql global variable)

Get current values 
```
> gcloud sql instances describe $INSTANCE --format='table(settings.databaseFlags:format="table(name,value)")'
NAME                           VALUE
slow_query_log                 on
log_output                     FILE
long_query_time                1
log_queries_not_using_indexes  off
innodb_print_all_deadlocks     on
```

Get gcloud sql flags list
```
> gcloud sql flags list
NAME                             TYPE                   DATABASE_VERSION               ALLOWED_VALUES
character_set_server             STRING                 MYSQL_5_5,MYSQL_5_6,MYSQL_5_7  latin1,utf8,utf8mb4
default_time_zone                MYSQL_TIMEZONE_OFFSET  MYSQL_5_5,MYSQL_5_6,MYSQL_5_7
event_scheduler                  BOOLEAN                MYSQL_5_5,MYSQL_5_6,MYSQL_5_7
ft_max_word_len                  INTEGER                MYSQL_5_5,MYSQL_5_6,MYSQL_5_7
ft_min_word_len                  INTEGER                MYSQL_5_5,MYSQL_5_6,MYSQL_5_7
...
```

Set a new value
```
❯ gcloud sql instances patch $INSTANCE --database-flags=long_query_time=1
The following message will be used for the patch API method.
{"project": "test-1234", "name": "main-4321", "settings": {"databaseFlags": [{"name": "long_query_time", "value": "1"}]}}
WARNING: This patch modifies a value that requires your instance to be
 restarted. Submitting this patch will immediately restart your
instance if it's running.

Do you want to continue (Y/n)?  y

Patching Cloud SQL instance...done.
Updated [https://www.googleapis.com/sql/v1beta4/projects/test-1234/instances/main-4321].
```
