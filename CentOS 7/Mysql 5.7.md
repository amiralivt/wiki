# Mysql 5.7

## Install MySQL 5.7
Enable the MySQL 5.7 repository with the following command:
```sh
sudo yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

Install MySQL as any other package using yum:
```sh
sudo yum install mysql-community-server
```

start the MySQL service and enable it:
```sh
sudo systemctl enable mysqld
sudo systemctl start mysqld
```

You can find the password by running the following command:
```sh
sudo grep 'temporary password' /var/log/mysqld.log
```

Securing MySQL:
```sh
sudo mysql_secure_installation
```

Login to the MySQL shell by executing the following command:
```sh
mysql -u root -p
```

Create a database and user name:
```sql
CREATE DATABASE db_name CHARACTER SET utf8 COLLATE utf8_general_ci;
GRANT ALL ON db_name.* TO 'user_name'@'localhost' IDENTIFIED BY 'password';
FLUSH PRIVILEGES;
EXIT;
```
