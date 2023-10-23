- 和基础的安装大同小异，要注意的是初始化有安全强度设置
```
fd@cfd-virtual-machine:~/Desktop$ mysql
ERROR 1045 (28000): Access denied for user 'cfd'@'localhost' (using password: NO)
cfd@cfd-virtual-machine:~/Desktop$ 
cfd@cfd-virtual-machine:~/Desktop$ 
cfd@cfd-virtual-machine:~/Desktop$ sudo su
[sudo] password for cfd: 
root@cfd-virtual-machine:/home/cfd/Desktop# mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 8.0.33-0ubuntu0.22.04.2 (Ubuntu)

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password by '123456';
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
mysql> 
mysql> 
mysql> mysql root -p
    -> ;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'mysql root -p' at line 1
mysql> 
mysql>  SHOW VARIABLES LIKE ‘validate_password%’;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '‘validate_password%’' at line 1
mysql> 
mysql> 
mysql> set global validate_password_policy=LOW;
ERROR 1193 (HY000): Unknown system variable 'validate_password_policy'
mysql> 
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
7 rows in set (0.00 sec)

mysql> set global validate_password_policy=0
    -> set global validate_password_length=1;
ERROR 1193 (HY000): Unknown system variable 'validate_password_policy'
mysql> 
mysql> set global validate_password_policy=0
    -> set global validate_password.length=1;
ERROR 1193 (HY000): Unknown system variable 'validate_password_policy'
mysql> set global validate_password.length=1;^C
mysql> set global validate_password_policy=0
    -> SELECT user,authentication_string,plugin,host FROM mysql.user;^C
mysql> set global validate_password.policy=0;
Query OK, 0 rows affected (0.00 sec)

mysql> set global validate_password.length=6
    -> ;
Query OK, 0 rows affected (0.00 sec)

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password by '123456';
Query OK, 0 rows affected (0.00 sec)

mysql> mysql_secure_installation
    -> ;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'mysql_secure_installation' at line 1
mysql> exit
Bye
root@cfd-virtual-machine:/home/cfd/Desktop# mysql_secure_installation

Securing the MySQL server deployment.

Enter password for user root: 
The 'validate_password' component is installed on the server.
The subsequent steps will run with the existing configuration
of the component.
Using existing password for root.

Estimated strength of the password: 50 
Change the password for root ? ((Press y|Y for Yes, any other key for No) : y

New password: 

Re-enter new password: 

Estimated strength of the password: 50 
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : n

 ... skipping.
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : n

 ... skipping.
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.

All done! 
```