**Title:** Aurora Multi-AZ & Read Replica\
**Name:** Robert Mathisen\
**Date:** 3/9/2021 \
\
\
**Procedure:** <br/>
\
**1) Create an EC2 Instance and install MySQL using a bootstrap script.** <br/>
Create a t2.micro EC2 Instance using the Linux 2 AMI. \
Enter the following script into the User Data to install MySQL: \
#!/bin/bash -ex \
yum install mysql -y \
Security Group - enable SSH \
Create a Key Pair\
\
**2) Create a Security Group allowing Inbound traffic on Port 3306 (MySQL/Aurora).** <br/>
Create a Security Group which allows Inbound traffic on Port 3306 (MySQL/Aurora) from Custom source 0.0.0.0/0 \

**3) Create Aurora database with replication and Multi-AZ deployment and attach the created Security Group to the VPC.** <br/>
Create an db.t2.small Aurora Database with Single-master Replication and Multi-AZ deployment on the default VPC, and assign it the Security Group created in Step 2. \
Keep note of the Username & Password as you will need it in Steps 5 & 6! <br/>

**4) Connect the EC2 Instance to the Aurora database.** <br/>
In the Security Group of the Writer (Master) cluster, change the Source IP address of your from 0.0.0.0/0 to the Private IPv4 of your RDS EC2 Instance with a prefix length of /32 (ex: 172.31.81.193/32) <br/>

**5) SSH into the Aurora database and create a table with a few records.** <br/>
SSH into the RDS EC2 Instance using the Public IPv4 & Key Pair created in Step 1. \
Switch to the Root User using sudo -s \
Log into the RDS Instance using the syntax mysql -h (Hostname) -u (username) -p \
  The Hostname is the Endpoint of the Writer (Master) Cluster, and use the Username that was created when creating the Aurora Database in Step 3. Then, you will be prompted for the password. \
  To List all Databases use show databases; \
  Create a database named aurora_db using Create database aurora_db; \
  To use this db, type use aurora_db; \
  Create a table and insert rows (records) into this table. \
  Use a SELECT statement to view the records in the table. \
  To exit, type exit. <br/>
  
**6) Force a Failover on the Writer (Master) cluster. <br/>
Determine whether or not the Read Replica is promoted to the Writer cluster, and the Writer cluster is demoted to the Read Replica cluster.** <br/>
Force a Failover on the Master cluster. If Multi-AZ is working properly, the Read Replica will be promoted to the new Master. This Failover may take a few min, but the Master will become the Reader, and the Reader will become the Master. Keep an eye on the DB Identifier to notice this change! \
Once this change occurs, test the new Master by confirming whether or not the records are in the database. \
Copy the Endpoint of the new Master, and log in, just like we did in Step 5. \
Log into the RDS Instance using the syntax mysql -h (Hostname) -u (username) -p (Make sure you use the new Hostname!!) \
show databases; \
use aurora_db; \
show tables; \
select * from (tablename);
  
**7) Clean up!! (~5 min)** \
Terminate RDS EC2 Instance \
Delete the Reader cluster \
Delete the Writer cluster. To do this, select whether or not you want to create a final snapshot, acknowledge and confirm the deletion.
