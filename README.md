# Setup
First, run docker-compose.yml file. It will run 3 mysql instances

1) in master create users for replication to slave1 and slave2 :
   - ```export MYSQL_PWD=111; mysql -u root -e 'CREATE USER "mydb_slave_user1"@"%" IDENTIFIED BY "mydb_slave_pwd"; GRANT REPLICATION SLAVE ON *.* TO "mydb_slave_user1"@"%"; FLUSH PRIVILEGES;'```
   - ```export MYSQL_PWD=111; mysql -u root -e 'CREATE USER "mydb_slave_user2"@"%" IDENTIFIED BY "mydb_slave_pwd"; GRANT REPLICATION SLAVE ON *.* TO "mydb_slave_user2"@"%"; FLUSH PRIVILEGES;'```
2) get master status. We need log file name and log file pos
   - ```export MYSQL_PWD=111; mysql -u root -e "SHOW MASTER STATUS"```
3) Set server id for slave1 : slave 1:
   - ```export MYSQL_PWD=111; mysql -u root -e "SET GLOBAL server_id = 2"```
4) Make slave 1 to replicate master:
    - ```export MYSQL_PWD=111; mysql -u root -e "CHANGE MASTER TO MASTER_HOST='mysql_master',MASTER_USER='mydb_slave_user1',MASTER_PASSWORD='mydb_slave_pwd',MASTER_LOG_FILE='binlog.000003',MASTER_LOG_POS=157; START SLAVE;"``` 
5) Check slave 1 status:
    - ```export MYSQL_PWD=111; mysql -u root -e 'SHOW SLAVE STATUS \G'```
6) Do similar thing for slave 2:
   - ```export MYSQL_PWD=111; mysql -u root -e "SET GLOBAL server_id = 3"``` 
   - ```export MYSQL_PWD=111; mysql -u root -e "CHANGE MASTER TO MASTER_HOST='mysql_master',MASTER_USER='mydb_slave_user2',MASTER_PASSWORD='mydb_slave_pwd',MASTER_LOG_FILE='binlog.000003',MASTER_LOG_POS=157; START SLAVE;"```
   - ```export MYSQL_PWD=111; mysql -u root -e 'SHOW SLAVE STATUS \G'```
7) Slave status example:
``` 
*************************** 1. row ***************************
Slave_IO_State: Waiting for source to send event
Master_Host: mysql_master
Master_User: mydb_slave_user1
Master_Port: 3306
Connect_Retry: 60
Master_Log_File: 1.000004
Read_Master_Log_Pos: 7777
Relay_Log_File: 2b2ea8b1c506-relay-bin.000002
Relay_Log_Pos: 7938
Relay_Master_Log_File: 1.000004
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Replicate_Do_DB:
Replicate_Ignore_DB:
Replicate_Do_Table:
Replicate_Ignore_Table:
Replicate_Wild_Do_Table:
Replicate_Wild_Ignore_Table:
Last_Errno: 0
Last_Error:
Skip_Counter: 0
Exec_Master_Log_Pos: 7777
Relay_Log_Space: 8155
Until_Condition: None
Until_Log_File:
Until_Log_Pos: 0
Master_SSL_Allowed: No
Master_SSL_CA_File:
Master_SSL_CA_Path:
Master_SSL_Cert:
Master_SSL_Cipher:
Master_SSL_Key:
Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
Last_IO_Errno: 0
Last_IO_Error:
Last_SQL_Errno: 0
Last_SQL_Error:
Replicate_Ignore_Server_Ids:
Master_Server_Id: 1
Master_UUID: 63c7ee3d-5669-11ee-87ea-0242ac140002
Master_Info_File: mysql.slave_master_info
SQL_Delay: 0
SQL_Remaining_Delay: NULL
Slave_SQL_Running_State: Replica has read all relay log; waiting for more updates
Master_Retry_Count: 86400
Master_Bind:
Last_IO_Error_Timestamp:
Last_SQL_Error_Timestamp:
Master_SSL_Crl:
Master_SSL_Crlpath:
Retrieved_Gtid_Set:
Executed_Gtid_Set:
Auto_Position: 0
Replicate_Rewrite_DB:
Channel_Name:
Master_TLS_Version:
Master_public_key_path:
Get_master_public_key: 0
Network_Namespace:
```

# Insert Data and Test Replication fail scenario
1) Create table in master
   - ```CREATE TABLE students (id INT NOT NULL AUTO_INCREMENT,name VARCHAR(100),surname VARCHAR(100),PRIMARY KEY (id));```
2) insert data in master
    - ```INSERT INTO students (name, surname) VALUES  ('Max', 'Nazaruk');```
3) Check that data is replicated in slaves
    - ```SELECT * FROM students;```
   ![Screenshot 2023-09-19 110147.png](..%2F..%2FPictures%2FScreenshot%202023-09-19%20110147.png)
4) remove middle column in slave 2:
   ![Screenshot 2023-09-19 110105.png](..%2F..%2FPictures%2FScreenshot%202023-09-19%20110105.png)
5) insert more data into master
6) You can check slave data again and observe that replication fail after column drop.
