## **IMPLEMENTATION OF A CLIENT SERVER ARCHITECTURE USING MYSQL DATABASE MANAGEMENT SYSTEM (DBMS).**

## CLIENT-SERVER ARCHITECTURE WITH MYSQL

- Client-Server refers to an architecture in which two or more computers are connected together over a network to send and receive requests between one another. In their communication, each machine has its own role: the machine sending requests is usually referred as "Client" and the machine responding (serving) is called "Server". A simple diagram of Web Client-Server architecture is presented below:

![alt text](<Images/illustration 1.png>)

- In the example above, a machine that is trying to access a Web site using a Web browser or simply ‘curl’ command is a client and it sends HTTP requests to a Web server (Apache, Nginx, IIS or any other) over the Internet. If we extend this concept further and add a Database Server to our architecture, we can get this picture:

![alt text](<Images/illustration 2.png>)

- The Web Server has a role of a "Client" that connects and reads/writes to/from a Database (DB) Server (MySQL, MongoDB, Oracle, SQL Server or any other), and the communication between them happens over a Local Network (it can also be an Internet connection, but it is a common practice to place Web Server and DB Server close to each other in a local network).
Essentially, it is sending requests to the remote server, and in turn, would be expecting some kind of response from the remote server. 


 ## **TO DEMONSTRATE A BASIC CLIENT-SERVER USING MYSQL RELATIONAL DATABASE MANAGEMENT SYSTEM (RDBMS), FOLLOW THE BELOW INSTRUCTIONS:**


- Create and configure two Linux-based virtual servers (EC2 instances in AWS).

   Server A name:  ```mysql-server``` and Server B name:  ```mysql-client```

![alt text](Images/AWS.png)

- SSH into your terminal and Let’s take a very quick example and see Client-Server communicatation in action. Open up your Ubuntu or Windows terminal and run the curl command:

   ```curl -Iv www.bing.com```

**Note: If your Ubuntu does not have ‘curl’, you can install it by running ```sudo apt install curl```.  In this example, your terminal will be the client, while [www.bing.com](https://www.bing.com) will be the server.**

See the response from the remote server in the below output. You can also see that the requests from the URL are being served by a computer with an IP address 204.79.197.200 on port 80.
![alt text](Images/curl.png)

- On mysql-server Linux Server install MySQL Server software:

   ```sudo apt update```

![alt text](Images/1.png)

- Then install the mysql-server package:

   ```sudo apt install mysql-server```

![alt text](Images/2.png)

- Start that the server by using the systemctl command:

   ```sudo systemctl start mysql.service```
- Ensure that the server is running using the command below (If you see active **(running)**, then MySQL is working.):

   ```sudo systemctl status mysql```

![alt text](Images/3.png)

## SETTING IT UP
- Run:   ```sudo mysql```
- For the password I'll be using 'Prince4Real', you can use any password of your choice then run paste the command below:

   ```ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';```

![alt text](Images/4.png)

- Run a MySQL secure installation

   ```sudo mysql_secure_installation```

![alt text](Images/5.png)

- In the MySQL server create a user and a **database named first_db** and a **user named first_user**, but you can replace these names with different values.

- First, connect to the MySQL console using the root account:

   ```sudo mysql -p```
- Create a new database by running this command from your MySQL console:

   ```CREATE DATABASE example_first_db;```

- Create a new user and grant full privileges on the database we have just created.

   ```CREATE USER 'first_user'@'%' IDENTIFIED WITH mysql_native_password BY 'MyRoot@2026#';```
- Give this user permission over the example_database database:

   ```GRANT ALL ON first_db.* TO 'first_user'@'%';```

![alt text](Images/6.png)

- Test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:

   ```mysql -u first_user -p```

![alt text](Images/7.png)

- After logging in to the MySQL console, confirm that you have access to the example_database database: ```mysql> SHOW DATABASES;```
This will give you the following output:

![alt text](Images/8.png)

- Exit MySQL and restart the mySQL service using:

   ```sudo systemctl restart mysql```

   ```sudo systemctl status mysql.service```

![alt text](Images/9.png)

- You might need to configure MySQL server to allow connections from remote hosts.

   ```sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf```

![alt text](Images/10.png)

- By default, both of your EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. Use mysql server's local IP address to connect from mysql client. MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups.

![alt text](Images/11.png)

- **NOTE: Mysql client private ip address is used above instead of 0.0.0.0 for extra Security**

## **SET UP MYSQL CLIENT**

- ssh into mysql-client instance
- On mysql client Linux Server install MySQL client software.

   ```sudo apt update && sudo apt ugrade```

![alt text](Images/12.png)
- install the mysql-client package: 

  ```sudo apt install mysql-client -y```

![alt text](Images/13.png)

- From mysql client instance connect remotely to mysql server Database using:

   ```sudo mysql -u example_user -h <mysqlserver private ip> -p```
- Type this in and the database should be visible: s```show databases;```

![alt text](Images/14.png)

















