# SqoopBigData

### Installing Docker into Machine (Cloud instance or VM)
```
sudo apt-get update
```

```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

```
sudo mkdir -m 0755 -p /etc/apt/keyrings
```

```
 curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
sudo apt-get update
```

```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Changing the docker permission
```
sudo usermod -aG docker <username>
```

```
newgrp docker
```

### Run the cloudera image
```
docker run --hostname=quickstart.cloudera --privileged=true -t -i -v /home/ubuntu:/Src --publish-all=true -p 8888 cloudera/quickstart:latest /usr/bin/docker-quickstart
```

### For chechking Sqoop version
```
sqoop version
```

### Opening the SQL terminal
```
mysql -uroot -pcloudera
```
###### Now you can run the required queries
```
show databases;
```

## Sqoop Commands

### List the databases
```
sqoop list-databases --connect jdbc:mysql://localhost:3306 --username root --password cloudera
```

### Listing tables
```
sqoop list-tables \
--connect jdbc:mysql://localhost:3306/retail_db \
--username root \
--password cloudera
```

### importing categories table from MySQL db
```
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

### Verify our data in HDFS
```
hdfs dfs -ls /

hdfs dfs -ls /user

hdfs dfs -ls /user/root

hdfs dfs -ls /user/root/categories
```

### To view the specific part file
```
hdfs dfs -cat /user/root/categories/part-m-00000
```

### To view whole data
```
hdfs dfs -cat /user/root/categories/part*
```

### To run SQL queries in Sqoop commands
```
sqoop eval \
--connect jdbc:mysql://localhost:3306/ \
--username root \
--password cloudera \
--query "show databases"
```

```
# see all the tables
sqoop eval \
--connect jdbc:mysql://localhost:3306/retail_db \
--username root \
--password cloudera \
--query "show tables"
```

### Split-by parameter
To split the column based on the user's input we can use --split-by
```
sqoop import \
--connect jdbc:mysql://localhost:3306/retail_db \
--table categories \
--username root \
--password cloudera \
--boundary-query "select min(category_id), max(category_id) from categories" \
--split-by category_id
```
