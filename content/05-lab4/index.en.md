---
title : "Lab 4 - Databases"
weight : 14
---

### Objectives

The objective of this lab is move your database off the single instance to highly available and fault tolerant serverless database. This lab includes:

-   Creating an Aurora Serverless database
-   Exporting data from your single instance
-   Importing data into Aurora Server-less
-   Updating the app code to use the new database

:::alert{type="warning" header="Important"}
You must **have completed the previous labs** successfully before proceeding with lab 4.
:::

![static/img/](/static/img/lab4.gif)

### Lab Guide

1.  Login to your AWS Console and ensure you are in the region you have been using for the previous labs.

#### Create a Database subnet group

An RDS database needs to be located in a database subnet group. A database subnet group is simply a set of VPC subnets that the database will use to host the various nodes and endpoints.

Before you can create a subnet group, you need to find the Subnet IDs for your private subnets.

2.  Navigate to the **VPC** service and click **Subnets** from the left side menu.
    
    Enter :code[tsgallery.subnets.private]{showCopyAction=true} into the **filter** box to filter to only your private subnets and make a note of the two **Subnet ID**s e.g. by copying and pasting them into Notepad.

![/static/img/lab4/](/static/img/lab4/2.1.png)

![/static/img/lab4/](/static/img/lab4/2.2.png)

3.  Navigate to the **RDS** service and click **Subnet groups** from the left side menu.

![/static/img/lab4/](/static/img/lab4/3.1.png)

![/static/img/lab4/](/static/img/lab4/3.2.png)

4.  Click **Create DB Subnet Group**.
    
5.  Enter :code[tsgallery.dbsubnetgroup]{showCopyAction=true} as the subnet **Name** and :code[TSGallery DB Subnet Group]{showCopyAction=true} as the **Description**.

6.  Select your **tsgallery.vpc** from the **VPC** dropdown box.
    
![/static/img/lab4/](/static/img/lab4/6.png)

7.  Select **Availability Zones** **a** and **b** in the drop down menu. Then using the **Subnets** dropdown, choose the two private subnets you identified earlier. If you followed the table in lab 1, your private subnets will have the CIDR ranges of 10.11.32.0/20 and 10.11.48.0/20.
    
8.  Click **Create**
    
![/static/img/lab4/](/static/img/lab4/8.png)

9.  On the Subnet Group Listing, open the Subnet Group editor by clicking the subnet group name.

![/static/img/lab4/](/static/img/lab4/9.png)

10.  In the **Tags** section, click **Manage tags**.

![/static/img/lab4/](/static/img/lab4/10.png)

11.  Click **Add new tag**. Enter the Key :code[Name]{showCopyAction=true} with the value of :code[tsgallery.dbsubnetgroup]{showCopyAction=true}. Click **Save**.

![/static/img/lab4/](/static/img/lab4/11.png)

#### Create a private security group

12.  Navigate to the **VPC** service and click **Security Groups** from the left side menu.

![/static/img/lab4/](/static/img/lab4/12.png)

13.  Click **Create security group**.

-   Enter :code[tsgallery.securitygroups.database]{showCopyAction=true} as the **Security group name** and **Description**.
-   Delete the existing text in the **VPC** section and select the _tsgallery.vpc_ from the dropdown.
-   Click **Add rule** to add a new **Inbound rule**. Select **MYSQL/Aurora** as the **Type**. In the **Source** box, ensure **Custom** is selected and then place your cursor in the text box to select the _tsgallery.securitygroups.webserver_ security group.  
-   Click **Add new tag**. Enter the Key :code[Name]{showCopyAction=true} with the value of :code[tsgallery.securitygroups.database]{showCopyAction=true}.
-   Click **Create security group**.
    
![/static/img/lab4/](/static/img/lab4/13.png)

#### Add an Aurora Database

Now you will create an Aurora database.

14.  Navigate to the **RDS** service, click **Databases** from the left side menu.
    
15.  Click **Create database**.
    
![/static/img/lab4/](/static/img/lab4/15.png)

16.  Ensure **Standard Create** is selected.

![/static/img/lab4/](/static/img/lab4/16.png)

17.  Ensure **Amazon Aurora** is the selected **Engine type**.
    
18.  Ensure **Amazon Aurora with MySQL compatibility** is selected. Leave the default value under **Available versions**. Under the **Templates** section, select **Dev/Test**.
    
![/static/img/lab4/](/static/img/lab4/18.png)

19.  In the **Settings** section, enter :code[tsgallery-dbcluster]{showCopyAction=true} as the **DB cluster identifier** and check the **Auto generate a password** option. In the **Instance Configuration** section, select the **Burstable classess** instance and leave the default value of **db.t3.small** fo the instance type.

![/static/img/lab4/](/static/img/lab4/19.png)

20.  In the **Availability & durability** section, leave the default value.
    
21.  In the **Connectivity** section, select the **tsgallery.vpc** from the **Virtual Private Cloud (VPC)** dropdown box and ensure the **DB Subnet group** is set to **tsgallery.dbsubnetgroup**.
    
22.  Under **Existing VPC security groups** select the **tsgallery.securitygroups.database** to add it, then deselect the **Default** security group in blue.
    
![/static/img/lab4/](/static/img/lab4/22.png)

Under **Monitoring**, uncheck the **Enable Enhanced monitoring**. 

![/static/img/lab4/](/static/img/lab4/22.1.png)

:::alert{header="Info"}
With Enhanced Monitoring, you can monitor the operating system of your DB instance in real time. When you want to see how different processes or threads use the CPU, Enhanced Monitoring metrics are useful.
:::

23.  Under **Additional configuration**, uncheck the **Enable deletion protection**. In a production environment, you should enable this protection, however for these labs, unchecking this will make the clean up easier.

![/static/img/lab4/](/static/img/lab4/23.png)

24.  Click **Create database**.
    
25.  To get the auto generated password, click the **View credential details** button. Make a copy of these values e.g. cut and paste into Notepad, as you cannot recover them once you close this screen.
    
![/static/img/lab4/](/static/img/lab4/25.1.png)

![/static/img/lab4/](/static/img/lab4/25.2.png)

26.  Click on the **DB identifier** to open the cluster details. Under **Connectivity & security** copy the **Endpoint** and **Port** values for use later e.g. **copy and paste into Notepad**.

:::alert{header="Note"}
Database creation will take around **10-15 minutes**. If the **Endpoint** and **Port** are empty, you need to wait for the database creation to finish before proceeding to the next step.
:::

![/static/img/lab4/](/static/img/lab4/26.png)

27.  Select the **Tags** tab and click **Add tags**.

![/static/img/lab4/](/static/img/lab4/27.png)

28.  Enter the Key :code[Name]{showCopyAction=true} with the value of :code[tsgallery.dbcluster]{showCopyAction=true}. Click **Add**.

![/static/img/lab4/](/static/img/lab4/28.png)

#### Use secret manager

29.  Navigate to the **Secrets Manager** service and click **Store a new secret**.

![/static/img/lab4/](/static/img/lab4/29.1.png)

![/static/img/lab4/](/static/img/lab4/29.2.png)

30.  Ensure **Credentials for other database** is selected and enter the **User name** and **Password** you were given in step 25.

![/static/img/lab4/](/static/img/lab4/30.png)

31.  Select **MySQL** for the **Select which database this secret will access** option. Enter the Endpoint and Port from step 26 as the **Server address** and the **Port**. Enter :code[tsgallerydata]{showCopyAction=true} as the **Database name**. Click **Next**.

![/static/img/lab4/](/static/img/lab4/31.png)

32.  Enter :code[tsgallery.secrets.dbcluster]{showCopyAction=true} as the **Secret name** and the **Description**. Under **Tags**, enter the Key :code[Name]{showCopyAction=true} with the value of :code[tsgallery.secrets.dbcluster]{showCopyAction=true}. Click **Next** in the button of the page.

![/static/img/lab4/](/static/img/lab4/32.png)

33.  You will not be enabling automatic key rotation for this lab. In a production environment, the key rotation should be enabled and set with a value that makes sense for your situation. Click **Next** and click **Store** to save the secret.

#### Give your server access to Secrets Manager

34.  So that your server can read the secret, you need to update the server role. Select the **IAM** service and click **Roles**.
    
35.  Enter :code[tsgallery.roles.serverrole]{showCopyAction=true} into the **Search** box and click on the role to open it.

![/static/img/lab4/](/static/img/lab4/35.png)

36.  Click **Add permissions** and select **Create inline policy**. Then click **Choose a service**.

![/static/img/lab4/](/static/img/lab4/36.png)

37.  Use the Search box to find **Secrets Manager**.
    
38.  Click **Select actions** and check the box next to **Read**.
    
39.  Under **Resources** select **All resources**.
    
![/static/img/lab4/](/static/img/lab4/39.png)

40.  Click **Review policy**, then enter :code[tsgallery.policies.readsecrets]{showCopyAction=true} as the **Name**. Click **Create policy**.

![/static/img/lab4/](/static/img/lab4/40.png)

#### Connecting back to your webserver

If you still have a Systems Manager or ssh session open to the web server you can re-use it. If not, repeat the process of opening a session:

41.  Select the **Systems Manager** service

-   Click **Session Manager** on the left side menu.
-   Click **Start session**.
-   Pick the **tsgallery.webserver** instance and click **Start Session**.
-   Systems Manager will open up a window on the webserver instance where you can type commands.
    
#### Move the existing data and Update the server

To migrate your data and update the server, you will run an update script.

:::alert{header="Update Script"}
The update script will do the following:

```
- Setup new schema in Aurora
- Copy data from local MySQL server
- Update the server config to reference the secret and remove hard coded database credentials
- Update the filesystem server code to use secret to get credentials
- Restart the server
- Shutdown MySQL server on local host as it's no longer needed
```
:::

42.  Run the following commands to update the server. You will need your your secret name.

:::code{language=bash showLineNumbers=true showCopyAction=true}
sudo su -
cd /opt/ts_gallery
sh update.sh
:::

When prompted, select the option to **Update RDS**. You will need your secret name, if it's not the the same as in this guide.

![/static/img/lab4/](/static/img/lab4/42.png)

If you have any issues with the update in this lab, or in any of the following labs, you can run the update shell script again. The script will allow you to rollback any of the updates and will only offer options that are valid for your current server state.

43.  Confirm the application is still working by reloading the CloudFront distribution instance URL in your browser.

### Wrap-up

In this lab you have setup a servlerless database cluster, migrated your data into the new cluster and updated the server to use the new cluster.

### Next...

:::alert{type="info" header="Note"}
Let you instructor know you have completed the lab, then wait for the instructor to confirm the next step.
:::

### Optional Challenges

If you finish early, then here are a couple of challenges to try and complete for yourself:

44.  **Create On-Demand Backup of the Database**

Create an on-demand backup of the database using the AWS Backup service. See [Create an on-demand backup](https://docs.aws.amazon.com/aws-backup/latest/devguide/create-on-demand-backup.html)  for guidance.

45.  **Set up regular hourly backups of the Database**

Use AWS Backup to create a new backup plan with a backup rule specifying hourly backups and with the `tsgallery-dbcluster` resource assigned via its Name tag. See [Creating a backup plan](https://docs.aws.amazon.com/aws-backup/latest/devguide/creating-a-backup-plan.html)  for guidance.