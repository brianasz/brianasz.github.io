---
layout: post
title: A quick overview of mysql 5.7 general query log
---

I had to take a look at Mysql General Query log and this is what I learned:

# What's the general query log? #
Is a log of what mysqld is doing and this includes:
  1. Client's connections and disconnections.
  2. Each query statement received from clients.

# What is looged? #
All the client is sending to mysqld, this also includes the protocol used by the client to establish the connection and the connection_type can be any of the following: 
  * TCP/IP
  * SSL/TLS
  * Socket
  * Named Pipe
  * Shared Memory

# Interesting facts #
1. The general query log is disabled by default.
2. It is possible to enable/disabled the general log file at runtime by using the general_log and general_log_file system variables.
3. If the log destination is NONE, the server doesn't write any queries even if the general log is enabled. FILE log destination must be selected for the server to log queries.
4. Server restarts and log flushing don't cause a new general query log file to be generated.
5. To rename the general query log file you need to disable the log, rename the log file and then enable the log again. This doesn't requires a restart.
6. Plain text passwords in SQL statements are rewritten by the server to prevent showing them in plain text. This configuration can be overwritten by starting the server with the ```--log-raw``` option.
7. Query statements that can't be parsed are not written to the general log because they can't be known to be plain text password free. This configuration can also be overwritten by starting the server with the ```--log-raw``` option.
8. In terms of performance, it is best to log to FILE instead of TABLE.


# How to activate the general query log? #
By using the following:
```--general_log[={0|1}]``` 
And to specify a log file name:
```--general_log_file=file_name. Note: If a name is not specified, the default one is: host_name.log```
**Note:** Use the log_output system variable to specify the log destination. The options are FILE and TABLE.

# Performance Impact of enabling the general query log #
Enabling general logs is always going to have a performance penalty but is hard to have an accurate assessment of what the penalty is going to be without executing performance test. The best scenario is to disable general logs, the next best scenario is to enable general logs and writing them into FILE. [This](https://fromdual.com/general_query_log_vs_mysql_performance#:~:text=Using%20the%20general%20log%20enabled,the%20response%20time%20by%2059%25.) article shows an insight of the impact of enabling general logs using different configurations, it is worth taking a look at it.

## Conclusion ##
Enabling general logs shouldn't be a default action on a production DB because enabling them comes with a performance penalty. General logs are really useful for debugging and troubleshooting and they should only be enbled for such action for a limited time of period. 
I had to enable this on a Google Cloud SQL instance and Google's Console makes this process really easy by using flags, there is no need to log into the mysql instance, you can just add flags on the GCP console and apply the configuration.

## References ##
[The General Query Log](https://dev.mysql.com/doc/refman/5.7/en/query-log.html)