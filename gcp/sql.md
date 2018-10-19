## Change intance flag (mysql global variable)


Get instance name and set the variable
```
❯ gcloud sql instances list
NAME       DATABASE_VERSION  LOCATION        TIER              PRIMARY_ADDRESS  PRIVATE_ADDRESS  STATUS
test-4321  MYSQL_5_7         europe-west1-b  db-n1-standard-2  35.205.230.98    -                RUNNABLE

❯ INSTANCE=test-4321
```

Get current values
```
❯ gcloud sql instances describe $INSTANCE --format='table(settings.databaseFlags:format="table(name,value)")'
NAME                           VALUE
slow_query_log                 on
log_output                     FILE
long_query_time                1
log_queries_not_using_indexes  off
innodb_print_all_deadlocks     on
```

Get gcloud sql flags list
```
❯ gcloud sql flags list
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

## Connect localy to cloudsql
### Using cloud-sql-proxy

Get instance name
```
❯ gcloud sql instances list
NAME       DATABASE_VERSION  LOCATION        TIER              PRIMARY_ADDRESS  PRIVATE_ADDRESS  STATUS
test-4321  MYSQL_5_7         europe-west1-b  db-n1-standard-2  35.205.230.98    -                RUNNABLE

❯ INSTANCE=test-4321
```

Run cloud proxy using docker
```
❯ CONNECTION_NAME=$(gcloud sql instances describe $INSTANCE --format='value(connectionName)')
❯ docker run --rm -d --name "cloudsql" \
  -v $HOME/.config/gcloud/:/root/.config/gcloud \
  -p 127.0.0.1:3307:3306 \
  gcr.io/cloudsql-docker/gce-proxy \
  /cloud_sql_proxy -instances="$CONNECTION_NAME=tcp:0.0.0.0:3306"
```

Connect to the server
```
mysql -h 127.0.0.1 -P3307 -u $USER -p$PASSWORD
```

### Using whitelisted ip

```
❯ INSTANCE=
❯ USER=
❯ gcloud sql connect $INSTANCE -u $USER
Whitelisting your IP for incoming connection for 5 minutes...done.
Connecting to database with SQL user [admin].Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13636
Server version: 5.7.14-google-log (Google)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
