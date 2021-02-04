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

Set new values, unfortunately `--database-flags` param is reset all previously defined flags.
```
❯ gcloud sql instances patch $INSTANCE --database-flags=long_query_time=1,log_output=FILE,long_query_time=1,log_queries_not_using_indexes=off,innodb_print_all_deadlocks=on,slow_query_log=on
The following message will be used for the patch API method.
{"project": "test-1234", "name": "main-4321", "settings": {"databaseFlags": [{"name": "innodb_print_all_deadlocks", "value": "on"}, {"name": "log_output", "value": "FILE"}, {"name": "log_queries_not_using_indexes", "value": "off"}, {"name": "long_query_time", "value": "1"}, {"name": "slow_query_log", "value": "on"}]}}
WARNING: This patch modifies a value that requires your instance to be
 restarted. Submitting this patch will immediately restart your
instance if it's running.

Do you want to continue (Y/n)?  y

Patching Cloud SQL instance...done.
Updated [https://www.googleapis.com/sql/v1beta4/projects/test-1234/instances/main-4321].
```

## Connect locally to cloudsql
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

### Migrate data between 2 CloudSQL instances with minimal downtime

1. [Create Read Replica](https://cloud.google.com/sql/docs/mysql/replication/create-replica) so we have new instance with full data of original.
2. Create MySQL user on the master instance with necessary privileges:

```
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'replica_tmp'@'%' identified by 'some_password';
```

3. [Stop replication](https://cloud.google.com/sql/docs/mysql/replication/manage-replicas#stopped)
4. Execute `show slave status` on read replica and save output, example:

```
> show slave status\G
***************************[ 1. row ]***************************
Slave_IO_State                |
Master_Host                   | 10.250.0.3
Master_User                   | cloudsqlreplica
Master_Port                   | 3306
Connect_Retry                 | 60
Master_Log_File               | mysql-bin.001535
Read_Master_Log_Pos           | 67200176
Relay_Log_File                | relay-log.000002
Relay_Log_Pos                 | 2729244
Relay_Master_Log_File         | mysql-bin.001535
Slave_IO_Running              | No
Slave_SQL_Running             | No
Replicate_Do_DB               |
Replicate_Ignore_DB           |
Replicate_Do_Table            |
Replicate_Ignore_Table        |
Replicate_Wild_Do_Table       |
Replicate_Wild_Ignore_Table   |
Last_Errno                    | 0
Last_Error                    |
Skip_Counter                  | 0
Exec_Master_Log_Pos           | 67200176
Relay_Log_Space               | 2729445
Until_Condition               | None
Until_Log_File                |
Until_Log_Pos                 | 0
Master_SSL_Allowed            | Yes
Master_SSL_CA_File            | master_server_ca.pem
Master_SSL_CA_Path            | /mysql/datadir
Master_SSL_Cert               | replica_cert.pem
Master_SSL_Cipher             |
Master_SSL_Key                | replica_pkey.pem
Seconds_Behind_Master         | <null>
Master_SSL_Verify_Server_Cert | No
Last_IO_Errno                 | 0
Last_IO_Error                 |
Last_SQL_Errno                | 0
Last_SQL_Error                |
Replicate_Ignore_Server_Ids   |
Master_Server_Id              | 2875592407
Master_UUID                   | 7651b30f-d1d8-11e8-8db5-42010a8400bf
Master_Info_File              | mysql.slave_master_info
SQL_Delay                     | 0
SQL_Remaining_Delay           | <null>
Slave_SQL_Running_State       |
Master_Retry_Count            | 86400
Master_Bind                   |
Last_IO_Error_Timestamp       |
Last_SQL_Error_Timestamp      |
Master_SSL_Crl                |
Master_SSL_Crlpath            |
Retrieved_Gtid_Set            | 7651b30f-d1d8-11e8-8db5-42010a8400bf:165651206-165654171
1 row in set
Time: 0.106s
```

5. [Promoting a read replica](https://cloud.google.com/sql/docs/mysql/replication/manage-replicas#promote-replica) stops replication and converts the instance to a standalone Cloud SQL primary instance with read and write capabilities. 
6. Make neccessary changes on new instance, for example ALTER tables or something
7. Install https://github.com/danfengcao/binlog2sql somewhere between both insances (master and replica)
8. Run the following command:

```
MASTER_HOST=10.250.0.3
MASTER_PORT=3306
REPL_HOST=10.250.0.19
MYSQL_USER=replica_tmp
MYSQL_PASSWORD=some_password
MASTER_LOG_FILE='get Master_Log_File value from `show slave status` saved previously'
MASTER_LOG_POS='get Exec_Master_Log_Pos value from `show slave status` saved previously'
MYSQL_DATABASE="database_to_sync, we have to exclude queries to `mysql` db that requires SUPER privileges" 
python3 ./binlog2sql.py \
  -h "$MASTER_HOST" \
  -P "$MASTER_PORT" \
  -u "$MYSQL_USER" \
  -p "$MYSQL_PASSWORD" \
  --start-file "$MASTER_LOG_FILE" \
  --start-position "$MASTER_LOG_POS" \
  -d "$MYSQL_DATABASE" \
  --stop-never | pv | mysql -h "$REPL_HOST" -u "$MYSQL_USER" -p"$MYSQL_PASSWORD"
```

9. Wait till pv speed slows down to make sure that relica node caughts up with master node
