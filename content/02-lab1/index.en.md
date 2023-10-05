---
title : "Lab 1 - VPC Setup"
weight : 11
---

### Key Objectives

The objective of this lab is to get network and permissions set up ready to launch a web site later.

![static/img/](/static/img/lab1.gif)

### Lab Guide

#### Login to your AWS Console.


1.  In the top right corner of the console, check if the region you are connected to is the same the instructor has set for the labs.
    
    :::alert{type="warning" header="Important"}
    Do not change the region during the labs. You will use this region for all remaining labs.
    :::
    
![static/img/1.png](/static/img/lab1/1.png)

#### Setup the VPC and Subnets

To select any AWS service you can use either the `search box` or `Services` menu at the top of the management console.

2.  Select the **VPC** service.

![static/img/1.png](/static/img/lab1/2.png)

3.  In the left side menu Select **Your VPCs** then click **Create VPC**.

![/static/img/Migrate-1Day-Images/Lab1/](/static/img/lab1/3.png)

4.  For **Name tag** enter :code[tsgallery.vpc]{showCopyAction=true} and for **IPv4 CIDR block** enter :code[10.11.0.0/16]{showCopyAction=true}.

![/static/img/Migrate-1Day-Images/Lab1/](/static/img/lab1/4.png)

5.  Click **Create VPC**.

![/static/img/Migrate-1Day-Images/Lab1/](/static/img/lab1/5.png)

6.  Click the **Actions** button, then **Edit VPC settings**.

![/static/img/Migrate-1Day-Images/Lab1/](/static/img/lab1/6.png)

7.  Check the **Enable DNS hostnames** checkbox and click **Save**.

![/static/img/Migrate-1Day-Images/Lab1/](/static/img/lab1/7.png)

8.  In the left side menu select **Subnets**.
    
9.  Click **Create subnet**.
    
![/static/img/Migrate-1Day-Images/Lab1/](/static/img/lab1/9.png)

Use the values from the below table bellow for each subnet:       

| Subnet name | IPv4 CIDR block | Availability Zone |
| --- | --- | --- |
| :code[tsgallery.subnets.public.a]{showCopyAction=true} | :code[10.11.0.0/20]{showCopyAction=true} | \*a |
| :code[tsgallery.subnets.public.b]{showCopyAction=true}| :code[10.11.16.0/20]{showCopyAction=true} | \*b |
| :code[tsgallery.subnets.private.a]{showCopyAction=true} | :code[10.11.32.0/20]{showCopyAction=true} | \*a |
| :code[tsgallery.subnets.private.b]{showCopyAction=true} | :code[10.11.48.0/20]{showCopyAction=true} | \*b |

-   For `VPC ID` select `tsgallery.vpc`.
    
-   For `Subnet name`, `Availability Zone` and `IPv4 CIDR block` use the values of the first line in the table above.
    
-   Click in the **Add new subnet** button at the botton of the page and repeat the process for the remaining 3 subnets.
    
-   After that, you should see a list of 4 subnets (2 publics and 2 privates) ready to be created in acordance with the table above.
    
-   Click **Create subnet**.
    

![/static/img/lab1/](/static/img/lab1/9.2.png)

10.  For the public subnets, there are two further steps:

-   Select public subnet **a**, click **Actions** then **Edit subnet settings**.

![/static/img/lab1/](/static/img/lab1/10.png)

-   Tick the **Enable auto-assign public IPv4 address** checkbox, scrolldown the page and click **Save**.
    
-   Repeat the procedure for the **Public Subnet b**.
    

![/static/img/lab1/](/static/img/lab1/10.2.png)

:::alert{type="info" header="Note"}
_**\*Only the public subnets need to have Auto-assign IPv4**_ set.

To be accessed from the Internet directly, objects e.g. EC2 Instances need a public IP address as well as having to be in a Public Subnet. You can set a public subnet to automatically assign a public IP address as well as a private IP address by changing this subnet setting "Auto-assign IPv4"
:::

#### Adding an Internet gateway

11.  Next, Click **Internet Gateways** in the left side menu.
    
12.  Click **Create internet gateway**. Enter :code[tsgallery.internetgateway]{showCopyAction=true} as the **Name tag**, then click **Create internet gateway**.

![/static/img/lab1/](/static/img/lab1/12.1.png)

![/static/img/lab1/](/static/img/lab1/12.2.png)

13.  After the Internet gateway is created, the console will show a green success box at the top of the window. Click the **Attach to a VPC** button in the top green bar.

![/static/img/lab1/](/static/img/lab1/13.png)

14.  Select the VPC created in step 4. Click **Attach internet gateway**.

![/static/img/lab1/](/static/img/lab1/14.png)

#### Securing your private subnets

15.  Click **Route Tables** from the left side menu.
    
16.  Select the route table with the **VPC ID** equal to _tsgallery.vpc_.
    

![/static/img/lab1/](/static/img/lab1/16.png)

17.  Hover your mouse over the blank name and click the pencil icon in the blank name section to add a name. Enter :code[tsgallery.routetable.private]{showCopyAction=true} in the popup box and click the tick icon.

![/static/img/lab1/](/static/img/lab1/17.png)

This is the default route table which will be used by all subnets when first created. As a security measure, we have left if without internet access.


#### Adding a public route

In order to give internet access to your public subnets you need to create a new route table with internet access through a route in the route table pointing to your internet gateway, and then associate it with your public subnets.

18.  Click **Create route table**.

![/static/img/lab1/](/static/img/lab1/18.png)

19.  Enter :code[tsgallery.routetable.public]{showCopyAction=true} as the **Name tag** and select the **tsgallery.vpc** from the VPC dropdown list, then click **Create route table**.

![/static/img/lab1/](/static/img/lab1/19.png)

20.  Click the **Routes** tab to show the current routes. Click **Edit routes** to open the route editor. (if you don’t see your new Route Table details in this page, on the left side menu click on **Route Tables** and select the new Route Table, them proceed with previous instructions).

![/static/img/lab1/](/static/img/lab1/20.png)

21.  Click **Add route**. Enter :code[0.0.0.0/0]{showCopyAction=true} as **Destination**, select **Internet Gateway** from the **Target** dropdown, then select the internet gateway you created on step 11.

![/static/img/lab1/](/static/img/lab1/21.png)

22.  Click **Save changes**.
    
23.  Select the **Subnet Associations** tab and click the first **Edit subnet associations** button.
    
![/static/img/lab1/](/static/img/lab1/23.png)

24.  Enter the word :code[public]{showCopyAction=true} into the **Filter** box and hit enter. This will list just the public subnets. Select both subnets and click **Save associations**.

![/static/img/lab1/](/static/img/lab1/24.png)

#### Setup some AWS IAM roles

Now you will setup an IAM role that will be required by your application to interact with AWS Services.

26.  Using the Service Search bar, go to **IAM**. On the left side menu click **Roles**.

![/static/img/lab1/](/static/img/lab1/26.1.png)

![/static/img/lab1/](/static/img/lab1/26.2.png)

27.  Click **Create role** and choose **EC2** as this is the service that will use this role. Then click **Next**

![/static/img/lab1/](/static/img/lab1/27.png)

28.  In the **Filter policies** box enter :code[AmazonS3FullAccess]{showCopyAction=true} in the **Filter policies** box, hit enter, and tick the box next to the policy in the listing. **Do not** click Next yet as you will add another IAM Policy to this IAM Role in the next step.

![/static/img/lab1/](/static/img/lab1/28.png)

29.  On the same **Add permissions** page, clear the AmazonS3FullAccess filter from the previous step and change the text in the **Filter policies** box to :code[AmazonSSMManagedInstanceCore]{showCopyAction=true}, hit enter, and tick the box next to the policy in the listing. Then click **Next**

![/static/img/lab1/](/static/img/lab1/29.png)

30.  Enter :code[tsgallery.roles.serverrole]{showCopyAction=true} as the **Role name**.

![/static/img/lab1/](/static/img/lab1/30.png)

31.  Finally, scroll down on the **Name, review, and create** page and double check that both **AmazonS3FullAccess** and **AmazonSSMManagedInstanceCore**, are listed as policies for the role and then click **Create role**.

The SSMManagedInstanceCore is required to allow Systems Manager to open sessions directly on an EC2 instance and for its Patch Manager to apply security updates automatically for you

![/static/img/lab1/](/static/img/lab1/31.png)

### Wrap-up

In this lab you have created a VPC with four subnets. This is the bare minimum you need to do to host a server on AWS. An IAM role for the Amazon EC2 service was also created. Over the next labs, you will be migrating your photo gallery into a new VPC and making it highly available.

### Next...

:::alert{type="info" header="Note"}
Let you instructor know you have completed the lab in the main training chat, and wait for the instructor to confirm the next step.
:::

### Optional Challenges

If you finish early, then here are a couple of challenges to try and complete for yourself:

33.  **SSH access from Corporate Network**

Set up the AWS end of a secure network connection to a corporate network, `172.16.0.0/12`, to allow ssh access to a bastion host in each public subnet. See [Site-to-Site VPN - Getting Started](https://docs.aws.amazon.com/vpn/latest/s2svpn/SetUpVPNConnections.html#vpn-create-target-gateway) for guidance.

34.  **Outgoing Internet access**

Set up a NAT gateway and adjust private subnet routes to allow outgoing-only Internet access from the private sub nets. See [Access the Internet from a private subnet](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html#public-nat-internet-access)  for guidance.