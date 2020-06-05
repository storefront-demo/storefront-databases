###### storefront-databases

# Set Up databases for Storefront

*This project is part of the 'IBM Cloud Native Reference Architecture' suite, available at
http://cloudnativereference.dev*

## Table of Contents

* [Introduction](#introduction)
* [Pre-requisites](#pre-requisites)
    + [Openshift](#openshift)
    + [Local environment](#local-environment)
* [Setup Storefront Databases](#setup-storefront-databases)
    + [Get the repo](#get-the-repo)
    + [Deploy on Openshift](#deploy-on-openshift)
    + [Deploy locally](#deploy-locally)
* [Conclusion](#conclusion)

## Introduction

This project will demonstrate how to deploy all the databases that are required by the storefront application. This will show you how to deploy MySQL, Elasticsearch, CouchDB, and MariaDB databases.

## Pre-requisites

### Openshift

* [RedHat Openshift Cluster](https://cloud.ibm.com/kubernetes/catalog/openshiftcluster)

* Command line (CLI) tools
  + [oc](https://www.okd.io/download.html)
  + [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
  + [appsody](https://appsody.dev/docs/getting-started/installation)

### Local environment

* Docker Desktop
    + [Docker for Mac](https://docs.docker.com/docker-for-mac/)
    + [Docker for Windows](https://docs.docker.com/docker-for-windows/)

## Setup Storefront Databases

### Get the repo

- Clone the repository:

```bash
git clone https://github.com/ibm-garage-ref-storefront/storefront-databases.git
cd storefront-databases
```

### Deploy on Openshift

- Run the below command to deploy the databases on the openshift cluster

```bash
./setup_databases.sh storefront-dev
```

where `storefront-dev` is the project name.

### Deploy locally

#### Deploy the MySQL database

```bash
# Start a MySQL Container with a database user, a password, and create a new database
docker run --name inventorymysql \
    -e MYSQL_ROOT_PASSWORD=admin123 \
    -e MYSQL_USER=dbuser \
    -e MYSQL_PASSWORD=password \
    -e MYSQL_DATABASE=inventorydb \
    -p 3306:3306 \
    -d mysql:5.7.14
```

If it is successfully deployed, you will see something like below.

```bash
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
84a511e74794        mysql:5.7.14        "docker-entrypoint.s…"   13 seconds ago      Up 12 seconds       0.0.0.0:3306->3306/tcp   inventorymysql
```

- Now let us populate the MySQL with data.

Firstly, `ssh` into the MySQL container.

```bash
docker exec -it inventorymysql bash
```

- Now, run the below command for table creation.

```bash
mysql -udbuser -ppassword
```

- This will take you to something like below.

```bash
root@d88a6e5973de:/# mysql -udbuser -ppassword
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.14 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

- Go to `scripts > mysql_data.sql`. Copy the contents from [mysql_data.sql](https://cloudnativereference.dev/deployments/scripts/mysql_data.sql) and paste the contents in the console.

- You can exit from the console using `exit`.

```bash
mysql> exit
Bye
```

- To come out of the container, enter `exit`.

```bash
root@d88a6e5973de:/# exit
```

#### Deploy the Elasticsearch database

```bash
# Start an Elasticsearch Container
docker run --name catalogelasticsearch \
      -e "discovery.type=single-node" \
      -p 9200:9200 \
      -p 9300:9300 \
      -d docker.elastic.co/elasticsearch/elasticsearch:6.3.2
```

If it is successfully deployed, you will see something like below.

```bash
$ docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                            NAMES
cc5af0ec4f75        docker.elastic.co/elasticsearch/elasticsearch:6.3.2   "/usr/local/bin/dock…"   6 minutes ago       Up 6 minutes        0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   catalogelasticsearch
7a958fa9cae3        mysql:5.7.14                                          "docker-entrypoint.s…"   About an hour ago   Up About an hour    0.0.0.0:3306->3306/tcp                           inventorymysql
```

#### Deploy the CouchDB database

```bash
# Start a CouchDB Container with a database user, a password, and create a new database
docker run --name customercouchdb -p 5985:5984 -e COUCHDB_USER=admin -e COUCHDB_PASSWORD=passw0rd -d couchdb:2.1.2
```

If it is successfully deployed, you will see something like below.

```bash
$ docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                            NAMES
3be611140a1a        couchdb:2.1.2                                         "tini -- /docker-ent…"   26 seconds ago      Up 23 seconds       4369/tcp, 9100/tcp, 0.0.0.0:5985->5984/tcp       customercouchdb
cc5af0ec4f75        docker.elastic.co/elasticsearch/elasticsearch:6.3.2   "/usr/local/bin/dock…"   15 hours ago        Up 15 hours         0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   catalogelasticsearch
7a958fa9cae3        mysql:5.7.14                                          "docker-entrypoint.s…"   16 hours ago        Up 16 hours         0.0.0.0:3306->3306/tcp                           inventorymysql
```

- Create the database.

```bash
# Create the database
$ curl -X PUT http://admin:passw0rd@localhost:5985/customers
{"ok":true}
```

- Populate customer data.

```bash
$ curl -X PUT http://admin:passw0rd@localhost:5985/customers/"001" -d "{\"username\": \"foo\", \"password\": \"bar\", \"firstName\": \"foo\", \"lastName\": \"bar\", \"email\": \"foo@bar.com\"}"
{"ok":true,"id":"001","rev":"1-fa7c8a9627d7d3f2df3e961a72a06660"}

$ curl -X PUT http://admin:passw0rd@localhost:5985/customers/"002" -d "{\"username\": \"user\", \"password\": \"password\", \"firstName\": \"user\", \"lastName\": \"name\", \"email\": \"user@name.com\"}"
{"ok":true,"id":"002","rev":"1-946189ae30d3dd6b495436ac788c7075"}
```

#### Deploy the MariaDB database

```bash
# Start a MariaDB Container with a database user, a password, and create a new database
docker run --name ordersmysql \
    -e MYSQL_ROOT_PASSWORD=admin123 \
    -e MYSQL_USER=dbuser \
    -e MYSQL_PASSWORD=password \
    -e MYSQL_DATABASE=ordersdb \
    -p 3307:3306 \
    -d mariadb
```

If it is successfully deployed, you will see something like below.

```bash
$ docker ps
CONTAINER ID        IMAGE                                                 COMMAND                  CREATED             STATUS              PORTS                                            NAMES
a0b8b6357607        mariadb                                               "docker-entrypoint.s…"   7 seconds ago       Up 4 seconds        0.0.0.0:3307->3306/tcp                           ordersmysql
3be611140a1a        couchdb:2.1.2                                         "tini -- /docker-ent…"   29 minutes ago      Up 29 minutes       4369/tcp, 9100/tcp, 0.0.0.0:5985->5984/tcp       customercouchdb
cc5af0ec4f75        docker.elastic.co/elasticsearch/elasticsearch:6.3.2   "/usr/local/bin/dock…"   15 hours ago        Up 15 hours         0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   catalogelasticsearch
7a958fa9cae3        mysql:5.7.14                                          "docker-entrypoint.s…"   16 hours ago        Up 16 hours         0.0.0.0:3306->3306/tcp                           inventorymysql
```

## Conclusion

You have successfully deployed all databases that are part of storefront application.
