## Using sysbench

### Set variables
```
user=sbtest
pass=SomeRandomPass
host=somehost
port=4008
test=oltp_read_write
```

### Prepare MySQL server
```
mysql <<MYSQL
CREATE SCHEMA sbtest;
CREATE USER sbtest@'%' IDENTIFIED BY '${pass}';
GRANT ALL PRIVILEGES ON sbtest.* to ${user}@'%';
MYSQL
```

### Run test
```
for cmd in prepare run cleanup; do
  docker run \
    --rm=true \
    --name=sb-prepare \
    severalnines/sysbench \
    sysbench \
    --db-driver=mysql \
    --report-interval=2 \
    --table-size=1000000 \
    --tables=24 \
    --threads=64 \
    --mysql-host=$host \
    --mysql-port=$port \
    --mysql-user=$user \
    --mysql-password=$pass \
    --time=100 \
    $test \
    $cmd
done
```

### Cleanup
```
mysql <<MYSQL
DROP USER sbtest@'%';
DROP DATABASE sbtest;
MYSQL
```
