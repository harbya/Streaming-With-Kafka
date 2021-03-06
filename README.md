

### "Streaming Data With Kafka Part 2"
author: "Harby Ariza"




### First thing start kafka-connect:

```
cd /opt/kafka/bin

/connect-distributed.sh -daemon /opt/kafka/config/connect-distributed.properties 

```

### List the availables plugings: 

```

curl  http://localhost:8083/connector-plugins | jq
```

### The we copy the kafka-connect-jdbc driver from the confluent instalation:

```
cp /opt/confluent/confluent-4.1.0/share/java/kafka-connect-jdbc/kafka-connect-jdbc-4.1.0.jar /opt/kafka/libs

```
### Then restart zookeeper and kafka-server plus the kafka-connect 

```
cp /home/hariza/Downloads/mysql-connector-java-5.1.46/mysql-connector-java-5.1.46-bin.jar 
cp postgresql-9.4-1206-jdbc41.jar ../lib
cp /home/db2/sqllib/java/db2jcc4.jar 
cp $JAVA_HOME/lib/ext
easy_install ibm_db  


List the availables plugings: 

List the availables plugings: 

curl  http://localhost:8083/connector-plugins | jq 


```
### Lets have a look to the bank.csv file 

```
cat bank.csv

S_no,age,job,marital,education,default,balance,housing,loan,contact,day,month,duration,campaign,pdays,previous,poutcome,y
1,30,unemployed,married,primary,no,1787,no,no,cellular,19,oct,79,1,-1,0,unknown,no
2,33,services,married,secondary,no,4789,yes,yes,cellular,11,may,220,1,339,4,failure,no
3,35,management,single,tertiary,no,1350,yes,no,cellular,16,apr,185,1,330,1,failure,no
4,30,management,married,tertiary,no,1476,yes,yes,unknown,3,jun,199,4,-1,0,unknown,no
5,59,blue-collar,married,secondary,no,0,yes,no,unknown,5,may,226,1,-1,0,unknown,no
6,35,management,single,tertiary,no,747,no,no,cellular,23,feb,141,2,176,3,failure,no
7,36,self-employed,married,tertiary,no,307,yes,no,cellular,14,may,341,1,330,2,other,no
8,39,technician,married,secondary,no,147,yes,no,cellular,6,may,151,2,-1,0,unknown,no
9,41,entrepreneur,married,tertiary,no,221,yes,no,unknown,14,may,57,2,-1,0,unknown,no
10,43,services,married,primary,no,-88,yes,yes,cellular,17,apr,313,1,147,2,failure,no
11,39,services,married,secondary,no,9374,yes,no,unknown,20,may,273,1,-1,0,unknown,no
12,43,admin.,married,secondary,no,264,yes,no,cellular,17,apr,113,2,-1,0,unknown,no
13,36,technician,married,tertiary,no,1109,no,no,cellular,13,aug,328,2,-1,0,unknown,no
14,20,student,single,secondary,no,502,no,no,cellular,30,apr,261,1,-1,0,unknown,yes
15,31,blue-collar,married,secondary,no,360,yes,yes,cellular,29,jan,89,1,241,1,failure,no

```


### Donwload DB2 Express-C for Linux x64 Version  11.1 

<https://www.ibm.com/analytics/in/en/technology/db2/db2-trials.html>

<https://www-01.ibm.com/marketing/iwm/iwm/web/pickUrxNew.do?source=swg-db2expressc>

DB2 Express-C for Linux x64 Version  11.1 


### Untar the file by executing the command below:

```
tar -xvf v11.1_linuxx64_expc.tar 

/opt/db2c/expc $ ./db2setup

```

## Create the new SOURCE database:

```

db2 "create database source"


```

### Now let's create the new **customer_details** table :

```

db2 "connect to source"
db2 "CREATE TABLE customer_details (
  s_no int DEFAULT NULL,
  age int DEFAULT NULL,
  job varchar(20) DEFAULT NULL,
  marital varchar(10) DEFAULT NULL,
  education varchar(20) DEFAULT NULL,
  mdefault varchar(10) DEFAULT NULL,
  balance bigint DEFAULT NULL,
  housing varchar(20) DEFAULT NULL,
  loan varchar(20) DEFAULT NULL,
  contact varchar(20) DEFAULT NULL,
  day varchar(20) DEFAULT NULL,
  month varchar(20) DEFAULT NULL,
  duration int DEFAULT NULL,
  campaign int DEFAULT NULL,
  pdays int DEFAULT NULL,
  previous int DEFAULT NULL,
  poutcome varchar(20) DEFAULT NULL,
  y varchar(5) DEFAULT NULL,
  inserted_time timestamp ) "
  
```

### Creating the JDBC Source connector:

```

curl -XPOST localhost:8083/connectors -H "Content-Type: application/json"         -d "{
              \"name\": \"db2-bank-source-connector\",
             \"config\": {
               \"connector.class\": \"io.confluent.connect.jdbc.JdbcSourceConnector\",
               \"tasks.max\": \"1\",
               \"connection.user\": \"db2\",
               \"connection.password\": \"hariza11\",
               \"topic.prefix\": \"source.\",
               \"mode\": \"timestamp\",
               \"table.whitelist\": \"CUSTOMER_DETAILS\",
               \"timestamp.column.name\": \"INSERTED_TIME\",
               \"validate.non.null\": \"false\",
               \"connection.url\": \"jdbc:db2://127.0.0.1:50000/source\"
              }
           }" | jq

```

### Command to list the connectors:

```
curl  http://localhost:8083/connectors | jq

```


### Command to list the existing topics:


```
/opt/kafka/bin/kafka-topics.sh --list --zookeeper localhost:2181

```

Command to remove topics:

```

/kafka-topics.sh --delete --zookeeper localhost:2181 --topic source.BANK_DETAILS

```

### Creating the Sink Connector for the Postgres Database:

```

curl -XPOST localhost:8083/connectors        -H "Content-Type: application/json"           -d "{
             \"name\": \"postgres-bank-sink-connector\",
             \"config\": {
               \"connector.class\": \"io.confluent.connect.jdbc.JdbcSinkConnector\",
               \"tasks.max\": \"1\",
               \"connection.user\": \"postgres\",
               \"connection.password\": \"hariza11\",
               \"topics\": \"source.CUSTOMER_DETAILS\",
               \"table.name.format\": \"customer_details\",
               \"auto.create\": \"true\",
               \"connection.url\": \"jdbc:postgresql://127.0.0.1:5432/target\"
              }
           }" | jq



```

### Creating the JDBC Sink connector for the mysql database;

```

curl -XPOST localhost:8083/connectors        -H "Content-Type: application/json"         -d "{
             \"name\": \"mysql-bank-sink-connector\",
             \"config\": {
               \"connector.class\": \"io.confluent.connect.jdbc.JdbcSinkConnector\",
               \"tasks.max\": \"1\",
               \"connection.user\": \"root\",
               \"connection.password\": \"hariza11\",
               \"topics\": \"source.CUSTOMER_DETAILS\",
               \"table.name.format\": \"customer_details\",
               \"auto.create\": \"true\",
               \"connection.url\": \"jdbc:mysql://127.0.0.1:3306/target\"
              }
           }" | jq



```

```

curl -XPOST localhost:8083/connectors        -H "Content-Type: application/json"           -d "{
             \"name\": \"postgres-bank-sink-connector-2\",
             \"config\": {
               \"connector.class\": \"io.confluent.connect.jdbc.JdbcSinkConnector\",
               \"tasks.max\": \"1\",
               \"connection.user\": \"postgres\",
               \"connection.password\": \"hariza11\",
               \"topics\": \"CUSTOMER_DETAILS_BY_MARITAL_STATUS\",
               \"table.name.format\": \"customer_details_by_marital_status\",
               \"auto.create\": \"true\",
               \"connection.url\": \"jdbc:postgresql://127.0.0.1:5432/target\"
              }
           }" | jq

```

### Let's have a look a the db2insertdata.py program:

```

#!/usr/bin/env python

import fileinput
import datetime
import time
import sys
import ibm_db
ibm_db_conn  = ibm_db.connect("DRIVER={IBM DB2 ODBC DRIVER};DATABASE=source;HOSTNAME=localhost;PORT=50000;PROTOCOL=TCPIP;UID=db2;PWD=hariza11;", "", "")
import ibm_db_dbi
conn = ibm_db_dbi.Connection(ibm_db_conn)

cursor = conn.cursor()

for line in fileinput.input('bank_details.txt'):
	cl = line.split(',')
        insert=("insert into db2.bank_details values ("+cl[0]+","+cl[1]+",'"+cl[2]+"','"+cl[3]+"','"+cl[4]+"','"+cl[5]+"',"+cl[6]+",'"+cl[7]+"','"+cl[8]+"','"+cl[9]+"','"+cl[10]+"','"+cl[11]+"',"+cl[12]+","+cl[13]+","+cl[14]+","+cl[15]+",'"+cl[16]+"','"+cl[17].rstrip('\n\r')+"',"+"now());")
        print("insert into db2.bank_details values ("+cl[0]+","+cl[1]+",'"+cl[2]+"','"+cl[3]+"','"+cl[4]+"','"+cl[5]+"',"+cl[6]+",'"+cl[7]+"','"+cl[8]+"','"+cl[9]+"','"+cl[10]+"','"+cl[11]+"',"+cl[12]+","+cl[13]+","+cl[14]+","+cl[15]+",'"+cl[16]+"','"+cl[17].rstrip('\n\r')+"',"+"now());")
	query = (insert)
	cursor.execute(query)
        time.sleep(2)
        query=("commit;")
	cursor.execute(query)


cursor.close()

```


```

./connect-distributed /opt/confluent/confluent-4.1.0/etc/schema-registry/connect-avro-distributed.properties
./schema-registry-start /opt/confluent/confluent-4.1.0/etc/schema-registry/schema-registry.properties


```

```


create stream customer_details (s_no int ,age int ,job string,marital string,education string,mdefault string,balance bigint,housing string ,loan string ,contact string,day string,month string,duration int,campaign int,pdays int,previous int ,poutcome string,y string) with (kafka_topic='source.CUSTOMER_DETAILS',value_format='avro',key='s_no');

create stream customer_details_by_marital_status as select s_no,age,job,marital from customer_details where marital = 'single';




```
