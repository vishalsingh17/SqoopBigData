# Sqoop

[Official Documentation](https://sqoop.apache.org/docs/1.4.7/SqoopUserGuide.html)

Before existence of Hadoop, there was no concept of big data. At that point the data was stored in relational database like MySQL, Oracle etc. With the introduction of big data, the data needed to be dealt with more concise and in a effective way. Thus Sqoop came into existence. Sqoop stands for SQL + Hadoop. 

Sqoop helped to transfer the large volume of data stored in the RDBMS to Hadoop. It was next to impossible to manually transfer this data, hence Sqoop was developed.  Sqoop is the tool which is used to perform data transfer operations from RDBMS to Hadoop server. It helped us to transfer bulk data from one data point to the other.

### Features of Sqoop

- Sqoop helps us connect the result from the SQL queries into HDFS
- Sqoop helps us to load the processed data directly into the hive or HBase.
- It performs the security operation of data with the help of Kerberos.
    
    **Kerberos** provides a centralized authentication server whose function is to authenticate users to servers and servers to users. In Kerberos Authentication server and database is used for client authentication. Kerberos runs as a third-party trusted server known as the Key Distribution Center (KDC). Each user and service on the network is a principal.
    
    The main components of Kerberos are: 
    
    - **Authentication Server (AS):** The Authentication Server performs the initial authentication and ticket for Ticket Granting Service.
    - **Database:** The Authentication Server verifies the access rights of users in the database.
    - **Ticket Granting Server (TGS):** The Ticket Granting Server issues the ticket for the Server
    
    **Kerberos Overview:**
    
    ![https://media.geeksforgeeks.org/wp-content/uploads/20190711134228/Capture6663.jpg](https://media.geeksforgeeks.org/wp-content/uploads/20190711134228/Capture6663.jpg)
    
    - **Step-1:** User login and request services on the host. Thus user requests for ticket-granting service.
    - **Step-2:** Authentication Server verifies user’s access right using database and then gives ticket-granting-ticket and session key. Results are encrypted using the Password of the user.
    - **Step-3:** The decryption of the message is done using the password then send the ticket to Ticket Granting Server. The Ticket contains authenticators like user names and network addresses.
    - **Step-4:** Ticket Granting Server decrypts the ticket sent by User and authenticator verifies the request then creates the ticket for requesting services from the Server.
    - **Step-5:** The user sends the Ticket and Authenticator to the Server.
    - **Step-6:** The server verifies the Ticket and authenticators then generate access to the service. After this User can access the services.
- With the help of Sqoop, we can perform compression of processed data.
- Sqoop is highly powerful and efficient in nature.

### Operations performed in Sqoop

1. Import
2. Export

![image1.png](Sqoop%20e3cbb172071042cc88734becefb19854/image1.png)

The operations is Sqoop are user friendly. Sqoop uses CLI to process commands given by the user.  Sqoop also used Java APIs to interact with the users. Sqoop only performs the import and export of data based on the commands given by the users. Sqoop can’t do the aggregation of data. 

### Sqoop installation on EC2

- Install docker in EC2

```bash
sudo apt-get update

sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

sudo mkdir -m 0755 -p /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo docker run hello-world
```

Once docker is installed we will use Cloudera docker image to run our Sqoop

```bash
pwd

docker run --hostname=quickstart.cloudera --privileged=true -t -i -v /home/ubuntu:/Src --publish-all=true -p 8888 cloudera/quickstart:latest /usr/bin/docker-quickstart
```

Set the docker permission

```bash
sudo usermod -aG docker ubuntu
newgrp docker
```

Now let’s try to run SQL in the terminal

```bash
mysql -uroot -pcloudera

show databases;

use retail_db;

show tables;

select * from <table_name>
```

Lets open another terminal and try to see if we Sqoop works fine

```bash
docker run --hostname=quickstart.cloudera --privileged=true -t -i -v /home/ubuntu:/Src --publish-all=true -p 8888 cloudera/quickstart:latest /usr/bin/docker-quickstart

# check version
sqoop version  

# list databases
sqoop list-databases --connect jdbc:mysql://localhost:3306 --username root --password cloudera

# list databases
sqoop list-databases \
--connect jdbc:mysql://localhost:3306 \
--username root \
--password cloudera

# list tables
sqoop list-tables \
--connect jdbc:mysql://localhost:3306/retail_db \
--username root \
--password cloudera

# get the table "categories" from "retail_db" database
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table categories \
--username root \
--password cloudera
```

There are 4 things that we can see in the output

- Data is selected with “Select” command
- Min & Max query is applied
- Default no of split = 4
- Mapper & Reducer task gets executed

In the SQL shell if we run the below query we can see the range is 58

```sql
SELECT MIN(`category_id`), MAX(`category_id`) FROM `categories`
```

Now, if we want to see my data in the HDFS we can run the following command

```bash
hdfs dfs -ls /user/

hdfs dfs -ls /user/root/

hdfs dfs -ls /user/root/categories/

# getting inside the file
hdfs dfs -cat /user/root/categories/part-m-00000

# get no of rows
hdfs dfs -cat /user/root/categories/part-m-00000 | wc -l
```

To execute SQL query in Sqoop we can use “eval”

```bash
# see all the databases
sqoop eval \
--connect jdbc:mysql://localhost:3306/ \
--username root \
--password cloudera \
--query "show databases"
```

```bash
# see all the tables
sqoop eval \
--connect jdbc:mysql://localhost:3306/retail_db \
--username root \
--password cloudera \
--query "show tables"
```

### Boundary Query

Boundary query is used to optimised query by using fields like min, max, between, split by etc kind of fields.

```bash
sqoop import --connect jdbc:mysql://localhost:3306/retail_db \
--table categories \
--username root \
--password cloudera \
--boundary-query "select min(category_id), max(category_id) from categories" \
--split-by category_id
```

Again if we want to see the data we can do it using the following commands

```bash
hdfs dfs -ls /user/

hdfs dfs -ls /user/root/

hdfs dfs -ls /user/root/categories/
 
hdfs dfs -cat /user/root/categories/part-m-00000
```

### Importing table without Primary Key

If we look at the “categories” table that we have been using is having “categories_id” as a primary key. 

```sql
show databases;

use retail_db;

show tables;

describe categories;
```

- Sqoop by default uses 4 concurrent map tasks to import data from SQL to Hadoop.
- While performing the parallel imports Sqoop needs a criteria by which it can split the workload. Sqoop uses the splitting column to split the workload.
- By default, Sqoop will identify the primary key column (if present) in the table to use it as a splitting column
- The low and high values of splitting column are retrieved from databases and the map task operate on evenly sized components of total range.

If a table is not having a primary key it can be imported by 2 ways:

- By using split-by argument
- By using -m1, i.e., using only 1 mapper task. In this the table will not get split into parts.

Let’s create a table in retail_db

Now let’s import the table into Sqoop

```bash
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table customers \
--username root \
--password cloudera \
--m 10
```

```bash
hdfs dfs -ls /

hdfs dfs -ls /user

hdfs dfs -ls /user/root

hdfs dfs -ls /user/root/customers

hdfs dfs -cat /user/root/customers/part-m-00008
```

Let’s try another example using 1 mapper

```bash
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table products \
--username root \
--password cloudera \
--m 1
```

```bash
hdfs dfs -ls /

hdfs dfs -ls /user

hdfs dfs -ls /user/root

hdfs dfs -ls /user/root/products

hdfs dfs -cat /user/root/products/part-m-00000

# delete a dir in hdfs
hfds dfs -rm -r /user/root/products
```

### Protecting the password

Till now we are passing the password as an argument. We will look how we can protect our password and we don’t to pass it as an argument. There are two ways to do it.

- Using standard input via keyword
- Saving the password in a file and specify the path to the file using - - password-file parameter. The file containing the password should be either on the HDFS or LFS (Local File System)

Using standard input via keyword

```bash
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table categories \
--username root \
--P \
--split-by category_id
```

Saving the password in a file and specify the path to the file using - - password-file parameter. The file containing the password should be either on the HDFS or LFS (Local File System)

```bash
# shows all the files present in the local file system
ls -ltr

# create a new file having the password
echo -n "cloudera" > sqoop.pwd

# let's verify
ls -ltr

# see the file
cat sqoop.pwd

# delete the dir
hdfs dfs -rm -r /user/root/categories

# import the table into sqoop with error
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table categories \
--username root \
--password-file sqoop.pwd \
--split-by category_id

# using file:///
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table categories \
--username root \
--password-file file:///sqoop.pwd \
--split-by category_id
```

This we did using the Local File System.

Now let’s put the the password file in HDFS and use it from there.

```bash
# picking the file from edge node to hdfs
hdfs dfs put file:///sqoop.pwd /user/cloudera

# import table
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table categories \
--username root \
--password-file /user/cloudera/sqoop.pwd \
--split-by category_id
```

### Speed up the import

We can speed the import using - -direct flag

```bash
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table customers \
--username root \
--password-file file:///sqoop.pwd
```

It took around 17 sec to import the data import Sqoop. Now let’s try direct parameter

```bash
# delete dir
hdfs dfs -rm -r /user/root/customers

# import the table
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table customers \
--username root \
--password-file file:///sqoop.pwd \
--direct
```

Using direct parameter it took around 16 sec to import the data import Sqoop.

Limitations of - - direct:

- Sqoop can only perform direct mode imports from DBs like MySQL, PostgreSQL, Oracle and Netezza.
- Binary formats like Sequence File or Avro won’t work with direct mode import.
- In case of MySQL, using direct parameter with import query, Sqoop will take advantage of MySQL’s native utility like **mysqldump** and **mysqlimport,** rather than using the JDBC interface for transferring data.

### Importing the data to target directory

So the by default path of importing the data is /user/root. We can give our custom path        by using target-dir parameter. target-dir parameter specify the directory on HDFS where Sqoop should import your data. Sqoop will import import the data in the form of part files in the specified directory.

```bash
# import the data in sqoop
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table customers \
--username root \
--password-file file:///sqoop.pwd \
--target-dir /user/data/customers
```

### Deleting target directory and import

This deletes your existing directory with all files within it & creates a new directory.

```bash
# import the data in sqoop
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table customers \
--username root \
--password-file file:///sqoop.pwd \
--target-dir /user/data/customers
```

To overcome the error we can run the below command

```bash
# import the data in sqoop
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table customers \
--username root \
--password-file file:///sqoop.pwd \
--delete-target-dir --target-dir /user/data/customers
```

### Add data to an existing file in directory

```bash
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table customers \
--username root \
--password-file file:///sqoop.pwd \
--target-dir /user/data/customers \
--append
```

Its not that the data will be added to the existing part file. New part files will get created for the appended data.
