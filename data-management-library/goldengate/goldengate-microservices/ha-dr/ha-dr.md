# GoldenGate Microservices Active-Active 

## Introduction
This lab will introduce you to Oracle GoldenGate for Microservices Workshop Architecture and High Availability / Disaster Recovery using Active-Active Technology

*Estimated Lab Time*:  60 minutes

### Lab Architecture
![](./images/ggmicroservicesarchitecture.png " ")
## Bi-Directional Replication, AutoCDR, Rapid Deployment and Intro to the Admin Client
## Introduction

Since we’ve already done multiple labs, this one will take what we used so far to script this using DB container reset scripts, SQL scripts to setup AutoCDR in the database, OGGCA silent deployment scripts and GG REST API scripts to do a rapid deployment.

## Objectives

- Rapid Deployment using:
  - OGGCA silent deployment scripts (remove and recreate deployments).
  - REST API to setup bi-directional GoldenGate replication between two databases.
  - SQL Scripts to setup up auto conflict detection and resolution in the database.

## Required Artifacts

- VNC Client for the deployment.
- Browser to check the deployment.
- Swingbench to apply transactions.

## **STEP 1**: Run a script to perform a rapid deployment.

1. Open up a terminal window and change directory to Lab6 and Review script build_all_bi_di.sh.

                [oracle@OGG181DB183 ~]$ cd ~/OGG181_WHKSHP/Lab6
```
<copy>cd ~/OGG181_WHKSHP/Lab6</copy>
```

                [oracle@OGG181DB183 Lab6]$ cat build_all_bi_di.sh 
```
<copy>cat build_all_bi_di.sh </copy>
```

**This script performs the following:**

Drops the existing container databases.

Clones two container databases from a base container.

Deletes the two deployments (Atlanta and SanFran).  This will remove any current lab setups.

Creates the two deployments again.

Creates new credentials for both deployments.

Adds Schema supplemental logging to both container databases for the SOE schema.

Adds checkpoint tables on both container databases.

Adds the Extract, Distribution Path and Replicat for both deployments.  This includes the correct parameters for the Extract and Replicats.

2. Run the **build_all_bi_di.sh** script

[oracle@OGG181DB183 Lab6]$ ./build_all_bi_di.sh 
```
<copy>./build_all_bi_di.sh</copy>
```

While it's running note the messages displayed that informs what has been added to the services.You should see the below message to be sure that all the steps are completed.

![](./images/600/Lab600_image6001.PNG) 


## **STEP 2**: Add AutoCDR to tables in the database.

When more than one replica of a table allows changes to the table, a conflict can occur when a change is made to the same row in two different databases at nearly the same time. Oracle GoldenGate replicates changes using the row LCRs. 
It detects a conflict by comparing the old values in the row LCR for the initial change from the origin database with the current values of the corresponding table row at the destination database identified by the key columns. 
If any column value does not match, then there is a conflict.
After a conflict is detected, Oracle GoldenGate can resolve the conflict by overwriting values in the row with some values from the row LCR, ignoring the values in the row LCR, or computing a delta to update the row values.

Automatic conflict detection and resolution does not require application changes for the following reasons:

    •	Oracle Database automatically creates and maintains invisible timestamp columns.

    •	Inserts, updates, and deletes use the delete tombstone log table to determine if a row was deleted.

    •	LOB column conflicts can be detected.

    •	Oracle Database automatically configures supplemental logging on required columns.


This step runs the ADD_AUTO_CDR procedure in the DBMS_GOLDENGATE_ADM package in the database.

1. In the terminal window change directory to Lab6 and Review script **setup_autocdr.sh**.

                [oracle@OGG181DB183 bin]$ cd ~/OGG181_WHKSHP/Lab6
```
<copy>cd ~/OGG181_WHKSHP/Lab6</copy>
```

                [oracle@OGG181DB183 Lab6]$ cat setup_autocdr.sh 
```
<copy>cat setup_autocdr.sh </copy>
```

This script performs the following:

    1.	Logs into the database.

    2.	Changes session to a container.

    3.	Executes the ADD_AUTO_CDR procedure in the DBMS_GOLDENGATE_ADM package.  This sets up the timestamp conflict detection and resolution.  You have to do this for any table you want to enable for CDR.  That’s why it’s best to have this scripted for multiple tables.

2. Run the script setup_autocdr.sh.

                [oracle@OGG181DB183 Lab6]$ ./setup_autocdr.sh 
```
<copy> ./setup_autocdr.sh </copy>
```
                Setup AutoCDR tables in database

                SQL*Plus: Release 18.0.0.0.0 - Production on Thu Feb 7 22:44:15 2019
                Version 18.3.0.0.0

                Copyright (c) 1982, 2018, Oracle.  All rights reserved.


                Connected to:
                Oracle Database 18c Enterprise Edition Release 18.0.0.0.0 - Production
                Version 18.3.0.0.0

                SQL> 
                Session altered.

                SQL> 
                PL/SQL procedure successfully completed.
                .
                .
                .
                SQL> 
                PL/SQL procedure successfully completed.

                SQL> Disconnected from Oracle Database 18c Enterprise Edition Release 18.0.0.0.0 - Production
                Version 18.3.0.0.0

                Done setting up AutoCDR


                [oracle@OGG181DB183 Lab6]$ 

## **STEP 3**: Start Replication

1. Run the start_replication.sh script to start the replication processes for the Atlanta capture and the SanFran delivery.

                [oracle@OGG181DB183 Lab6]$ ./start_replication.sh Welcome1 16001 EXTSOE1 16002 SOE2SOE1 17001 IREP2
```
<copy>./start_replication.sh Welcome1 16001 EXTSOE1 16002 SOE2SOE1 17001 IREP2</copy>
```

                % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                                Dload  Upload   Total   Spent    Left  Speed
                100   853  100   702  100   151   8865   1907 --:--:-- --:--:-- --:--:--  9000
                {
                .
                .
                .
                        {
                        "$schema": "ogg:message",
                        "code": "OGG-15426",
                        "issued": "2019-02-07T22:51:05Z",
                        "severity": "INFO",
                        "title": "EXTRACT EXTSOE1 started",
                        "type": "http://docs.oracle.com/goldengate/c1810/gg-winux/GMESG/oggus.htm#OGG-15426"
                        }
                ]
                }
                [oracle@OGG181DB183 Lab6]$ 

2. Next, run the start_replication.sh script again to start the replication processes for the SanFran capture and the Atlanta delivery.  Use the CREATE SCN value from OGGOOW182 as the last value of the script (See example above).  This is for the startup of the Replicat on the Atlanta deployment.

                [oracle@OGG181DB183 Lab6]$ ./start_replication.sh Welcome1 17001 EXTSOE2 17002 SOE2SOE2 16001 IREP1
```
<copy> ./start_replication.sh Welcome1 17001 EXTSOE2 17002 SOE2SOE2 16001 IREP1</copy>
```

                % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                                Dload  Upload   Total   Spent    Left  Speed
                100   853  100   702  100   151   8354   1797 --:--:-- --:--:-- --:--:--  8357
                {
                .
                .
                .
                        {
                        "$schema": "ogg:message",
                        "code": "OGG-15426",
                        "issued": "2019-02-07T22:53:42Z",
                        "severity": "INFO",
                        "title": "EXTRACT EXTSOE2 started",
                        "type": "http://docs.oracle.com/goldengate/c1810/gg-winux/GMESG/oggus.htm#OGG-15426"
                        }
                ]
                }
                [oracle@OGG181DB183 Lab6]$ 

## **STEP 4**: Run transactions and check conflicts with Performance Metric Service

In this step we’ll use a script to invoke Swingbench to apply data to both databases at the same time and then check them using the Performance Metric Service.

1. In the terminal window review script start_swingbench.sh.

                [oracle@OGG181DB183 Lab6]$ cat start_swingbench.sh 
```
<copycat start_swingbench.sh </copy>
```
>
                #!/bin/bash
                cd ~/OGG181_WHKSHP/Lab6/Build
```
<copy>cd ~/OGG181_WHKSHP/Lab6/Build</copy>
```

                ./start_swingbench_181.sh &
                ./start_swingbench_182.sh &
```
<copy>./start_swingbench_181.sh</copy>
```
```
<copy>./start_swingbench_182.sh</copy>
```

                [oracle@OGG181DB183 Lab6]$ 

This script runs the swingbench jobs you ran in the other labs, but this time it will run two jobs in the background and each job applies data to one or the other databases.

2. Run start_swingbench.sh.  It will take a few seconds to start up and run for 10 mins.

                [oracle@OGG181DB183 Lab6]$ ./start_swingbench.sh 
```
<copy>./start_swingbench.sh </copy>
```
  
                [oracle@OGG181DB183 Lab6]$ Author  :	 Dominic Giles
                Author  :	 Dominic Giles
                Version :	 2.6.0.1046

                Version :	 2.6.0.1046
                Results will be written to results.xml.

                Results will be written to results.xml.
                Hit Return to Terminate Run...
                Hit Return to Terminate Run...

                Time		Users

                Time		Users
                00:10:47	[0/2]
                00:10:47	[0/2]

3. From the browser, log in to the Service Manager using the Administrator account **"oggadmin"** the password should be **"Welcome1"**.

![](./images/600/Lab600_image110.PNG) 

4. Next click on the link to the Performance Metrics Server for Atlanta.

![](./images/600/select

    Click on the Replicat icon.

![](./images/600/repl_atl.PNG) 

6. We’ll take a longer look at the Metric Service in another lab, so for now just click on the “Database Statistics” tab.

![](./images/600/sel_db_stats_atl.PNG) 

On this screen you’ll see the number of operations performed and their types and also the number of conflicts detected, and the number of conflicts resolved.  This is done automatically by the AutoCDR configuration.

![](./images/600/disp_db_stats_atl.PNG) 

If you want, you can check the Replicat of the other deployment and you’ll see a similar display.

## Summary

Oracle GoldenGate offers high-performance, fault-tolerant, easy-to-use, and flexible real- time data streaming platform for big data environments. It easily extends customers’ real-time data
integration architectures to big data systems without impacting the performance of the source systems and enables timely business insight for better decision making.

You may now *proceed to the next lab*

## Learn More

* [GoldenGate Microservices](https://docs.oracle.com/goldengate/c1230/gg-winux/GGCON/getting-started-oracle-goldengate.htm#GGCON-GUID-5DB7A5A1-EF00-4709-A14E-FF0ADC18E842")

## Acknowledgements
* **Author** - Brian Elliott, Data Integration, November 2020
* **Contributors** - Zia Khan
* **Last Updated By/Date** - Brian Elliott, November 2020

## Need Help?
Please submit feedback or ask for help using our [LiveLabs Support Forum](https://community.oracle.com/tech/developers/categories/livelabsdiscussions). Please click the **Log In** button and login using your Oracle Account. Click the **Ask A Question** button to the left to start a *New Discussion* or *Ask a Question*.  Please include your workshop name and lab name.  You can also include screenshots and attach files.  Engage directly with the author of the workshop.

If you do not have an Oracle Account, click [here](https://profile.oracle.com/myprofile/account/create-account.jspx) to create one.
