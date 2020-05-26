# MariaDB Sharding using Docker
Below versions.
  - Ubuntu 18.04
  - Docker 19.03
  - MariaDB 10.4

### Installation
Install the server and apt update.
```sh
$ sudo apt update
$ sudo apt-get update
$ sudo apt-get upgrade
```

Install the dependencies.
```sh
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

Add docker repository on apt
```sh
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"

# Check docker was installed
$ apt-cache policy docker-ce
```

apt update again.
```sh
$ sudo apt update
$ sudo apt-get update
$ sudo apt-get upgrade
```

### Install Docker

```sh
$ sudo apt install docker-ce
$ systemctl status docker
$ docker version
```

### Download Docker images
Verify docker images.
https://hub.docker.com/_/mariadb?tab=description&page=1
```sh
$ sudo docker pull mariadb:10.4
$ sudo docker image ls
```

### Run Docker container 
sudo docker run -d -p [host_port]:[docker_port] -e MYSQL_ROOT_PASSWORD=[your_pwd] --name [name] [image]:[tag]
```sh
$ sudo docker run -d -p 33061:3306 -e MYSQL_ROOT_PASSWORD=wpdlwl --name maria_1 mariadb:10.4
$ sudo docker run -d -p 33062:3306 -e MYSQL_ROOT_PASSWORD=wpdlwl --name maria_2 mariadb:10.4
$ sudo docker run -d -p 33063:3306 -e MYSQL_ROOT_PASSWORD=wpdlwl --name maria_3 mariadb:10.4

$ sudo docker ps -a
```

### Grep IP Adress
```sh
$ ifconfig
> 192.168.10.89

$ sudo docker inspect maria_1 | grep IPAddress
> 172.17.0.2

$ sudo docker inspect maria_2 | grep IPAddress
> 172.17.0.3

$ sudo docker inspect maria_3 | grep IPAddress
> 172.17.0.4
```

### Install spider engine on MariaDB
##### Maria1

```sh
$ sudo docker exec -it maria_1 bash
```

Install spider engine.
```sh
$ apt-get update
$ apt-get install vim
$ apt-get install mysql-plugin-spider

$ mysql -u root -p < /usr/share/mysql/install_spider.sql

$ mysql -u root -p
```

Check and grant.
```sh
mysql> show engines;

mysql> create user 'jayg'@'192.168.10.81' identified by 'wpdlwl';
mysql> grant all privileges on *.* to 'jayg'@'192.168.10.81' with grant option;
mysql> flush privileges;
```

Create two servers.
```sh
mysql> create server backend_1
foreign data wrapper mysql
Options(
    HOST '192.168.10.89',
    Database 'backend',
    User 'spider_test',
    Password 'test123',
    Port 33062
);

mysql> create server backend_2
foreign data wrapper mysql
Options(
    HOST '192.168.10.89',
    Database 'backend',
    User 'spider_test',
    Password 'test123',
    Port 33063
);
```

Create sysbench table.
```sh
mysql> CREATE SCHEMA `backend` DEFAULT CHARACTER SET utf8 ;

mysql> create table backend.sbtest(
    id int(10) not null auto_increment,
    k int(10) not null default '0',
    c char(120) not null default '',
    pad char(60) not null default '',
    primary key (id),
    key k (k)
) engine=spider comment='database "backend",table "sbtest"'
Partition by key(id)(
    Partition pr1 comment='srv "backend_1"',
    Partition pr2 comment='srv "backend_2"'
);


mysql> select * from mysql.servers;
mysql> Select * from mysql.spider_tables;
```

##### maria_2
```sh
$ sudo docker exec -it maria_2 bash

$ mysql -u root -p 
```

```sh
mysql> CREATE SCHEMA `backend` DEFAULT CHARACTER SET utf8 ;

mysql> create table backend.sbtest(
    id int(10) not null auto_increment,
    k int(10) not null default '0',
    c char(120) not null default '',
    pad char(60) not null default '',
    primary key (id),
    key k (k)
) engine=innodb  

mysql> show engines;

mysql> create user 'spider_test'@'172.17.0.2' identified by 'test123';
mysql> grant all privileges on *.* to 'spider_test'@'172.17.0.2' with grant option;
mysql> flush privileges;
```

##### maria_3
```sh
$ sudo docker exec -it maria_3 bash

$ mysql -u root -p 
```

```sh
mysql> CREATE SCHEMA `backend` DEFAULT CHARACTER SET utf8 ;

mysql> create table backend.sbtest(
    id int(10) not null auto_increment,
    k int(10) not null default '0',
    c char(120) not null default '',
    pad char(60) not null default '',
    primary key (id),
    key k (k)
) engine=innodb  

mysql> show engines;

mysql> create user 'spider_test'@'172.17.0.2' identified by 'test123';
mysql> grant all privileges on *.* to 'spider_test'@'172.17.0.2' with grant option;
mysql> flush privileges;
```

### Restart Docker container
```sh
$ sudo docker stop maria_1
$ sudo docker start maria_1

$ sudo docker stop maria_2
$ sudo docker start maria_2

$ sudo docker stop maria_3
$ sudo docker start maria_3
```

### TEST
##### maria_1
```sh
mysql> INSERT INTO backend.sbtest SELECT * FROM backend.sample
```

### References
| Site | README |
| ------ | ------ |
| Docker | https://soyoung-new-challenge.tistory.com/52 |
| Docker | https://docs.docker.com/engine/reference/run/ |
| Sharding | https://sarc.io/index.php/mariadb/1433-mariadb-spider-engine-sharing-1 |
| Sharding | https://formin97.tistory.com/239 |
