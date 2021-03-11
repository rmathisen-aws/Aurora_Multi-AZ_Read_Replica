**Title:** Aurora Multi-AZ & Read Replica\
**Name:** Robert Mathisen\
**Date:** 3/9/2021 \
\
**Summary:** \
Created an Aurora database with replication and Multi-AZ deployment. Forced a Failover on the Master instance, and confirmed the Read Replica is promoted to Master. \
<br/>
**Reason & Reflection:** \
Already having background knowledge on databases, I thought it would be interesting to learn about the process of configuring and launching an Aurora database. It was nice to be able to write some very familiar SQL statements in the CLI. But what I found to be the most interesting and rewarding is being able to implement a solution where I'm able to protect a client's data in the event of a failover. \
<br/>

**Process:** <br/>
\
**1) Create an EC2 Instance and install MySQL using a bootstrap script.** <br/>
Create a t2.micro EC2 Instance using the Linux 2 AMI. \
Enter the following script into the User Data to install MySQL: \
&nbsp;&nbsp;&nbsp;&nbsp;*#!/bin/bash -ex* \
&nbsp;&nbsp;&nbsp;&nbsp;*yum install mysql -y* \
Security Group - enable SSH \
Create a Key Pair\
\
**2) Create a Security Group for the Aurora database.** <br/>
Navigate to EC2 → Network & Security → Security Groups \
Create a Security Group which allows Inbound traffic on Port 3306 (MySQL/Aurora) from Custom source 0.0.0.0/0 \
\
**3) Create Aurora database with replication and Multi-AZ deployment** <br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**and attach the created Security Group to the VPC.** <br/>
RDS → Databases → Create database \
Create an db.t2.small Aurora Database with Single-master Replication and Multi-AZ deployment on the default VPC, <br/>
&nbsp;&nbsp;&nbsp;&nbsp;and assign it the Security Group created in Step 2. \
Keep note of the Username & Password as you will need it in Steps 5 & 6! <br/>
\
**4) Connect the EC2 Instance to the Aurora database.** <br/>
In the Security Group of the Writer (Master) instance, change the Source IP address of your from 0.0.0.0/0 <br/>
&nbsp;&nbsp;&nbsp;&nbsp;to the Private IPv4 of your RDS EC2 Instance with a prefix length of /32 &nbsp;&nbsp;&nbsp;&nbsp; (ex: 172.31.81.193/32) <br/>
\
**5) SSH into the Aurora database and create a table with a few records.** <br/>
SSH into the RDS EC2 Instance using the Public IPv4 & Key Pair created in Step 1. \
Switch to the Root User: *sudo -s* \
\
Log into the RDS Instance: *mysql -h `<Hostname>` -u `<username>` -p* \
&nbsp;&nbsp;&nbsp;&nbsp;The Hostname is the Endpoint of the Writer (Master) instance. <br/>
&nbsp;&nbsp;&nbsp;&nbsp;Use the Username that was created when creating the Aurora Database in Step 3. <br/>
&nbsp;&nbsp;&nbsp;&nbsp;Then, you will be prompted for the password. \
\
List all Databases: *show databases;* \
Create a database: *Create database aurora_db;* \
To use this db: *use aurora_db;* \
Create a Table: *CREATE TABLE `<tablename>` (* \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *field_id* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *INT* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *AUTO_INCREMENT,* \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *field_name1* &nbsp;&nbsp;&nbsp;&nbsp; *VARCHAR(255)* &nbsp;&nbsp;&nbsp;&nbsp; *NOT NULL,* \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *field_name2* &nbsp;&nbsp;&nbsp;&nbsp; *VARCHAR(255),* \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *PRIMARY KEY (field_id));*

Insert Rows (Records) into this table: \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *INSERT INTO `<tablename>`(field_name1, field_name2) VALUES ('value1', 'value2');*

Use a Select statement to view the records in the table: \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; *SELECT * FROM `<tablename>`;*

To Exit: *exit* <br/>
\
**6) Force a Failover on the Writer (Master) instance and test the new Writer instance. <br/>**
Force a Failover on the Master instance. <br/>
&nbsp;&nbsp;&nbsp;&nbsp;If Multi-AZ is working properly, the Read Replica will be promoted to the new Master. <br/>
&nbsp;&nbsp;&nbsp;&nbsp;This Failover may take a few min, but the Master will become the Reader, and the Reader will become the Master. <br/>
&nbsp;&nbsp;&nbsp;&nbsp;Keep an eye on the DB Identifier to notice this change! <br/>
\
Once this change occurs, test the new Master by confirming whether or not the records are in the database. \
&nbsp;&nbsp;&nbsp;&nbsp;Copy the Endpoint of the new Master, and log in, just like we did in Step 5. \
&nbsp;&nbsp;&nbsp;&nbsp;Log into the RDS Instance: *mysql -h `<Hostname>` -u `<username>` -p* \
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(Make sure you use the new Hostname!!) \
&nbsp;&nbsp;&nbsp;&nbsp;*show databases;* \
&nbsp;&nbsp;&nbsp;&nbsp;*USE aurora_db;* \
&nbsp;&nbsp;&nbsp;&nbsp;*show tables;* \
&nbsp;&nbsp;&nbsp;&nbsp;*SELECT * FROM `<tablename>`;* <br/>
\
**7) Clean up!! (~5 min)** \
Terminate EC2 instance. \
Delete the Reader instance. \
Delete the Writer instance. To do this, select whether or not you want to create a final snapshot, acknowledge, and confirm the deletion.
