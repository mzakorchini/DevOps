---
title : "Lab 5 - Elastic Beanstalk"
weight : 15
---

### Objectives

The objective of this lab is to host your server in a highly available way using Elastic Beanstalk. This lab includes:

-   Configuring a project on Amazon Elastic Beanstalk
-   Deploying your application into a highly available setup
-   Updating CloudFront to use the new servers
-   Shutdown your original server as you no longer need it.

:::alert{type="warning" header="Important"}
You must **have completed all previous labs** successfully before proceeding with lab 5.
:::

![static/img/](/static/img/lab5.gif)

### Lab Guide

1.  Login to your AWS Console and ensure you are in the region you have been using for the previous labs.
    
2.  If you still have a Systems Manager or ssh session open to the web server you can re-use it. If not, **repeat the step 41 of Lab 4**.
    

#### Update and package your server code for Elastic Beanstalk

In order to host our server on Elastic Beanstalk, we need to add some supporting files and build a zip package.

The update script will do the following:

-   Create the .ebextensions folder
-   Configure NodeJS
-   Configure to disable npm install / npm rebuild - This is not normally needed, however, one of the modules you use has issues with Elastic Beanstalk. In this case, you simply provide a Zip file with all of the modules as part of the deployment.
-   Zip server files for deployment
-   Upload deployment package to S3

3.  Run the following commands to update the server.
    
:::code{language=bash showLineNumbers=true showCopyAction=true}
sudo su -
cd /opt/ts_gallery
sh update.sh
:::
    
-   When prompted, select the option to **Update Elastic Beanstalk**.

![/static/img/lab5/](/static/img/lab5/3.png)

4.  On S3 / Buckets, Click on your bucket name.

![/static/img/lab5/](/static/img/lab5/4.png)

5.  On the Objects tab, check the **ts\_gallery.zip** file and click **Download** to save the file locally on your computer. You are going to use this file later to update the Elastic Beanstalk environment.

![/static/img/lab5/](/static/img/lab5/5.png)

#### Update the Web server role to enable Elastic Beanstalk

6.  Select the **IAM** service and click **Roles**. Enter :code[tsgallery.roles.serverrole]{showCopyAction=true} into the search box and click on the **tsgallery.roles.serverrole** role name.

![/static/img/lab5/](/static/img/lab5/6.png)

7.  On the **Permissions** tab, select **Add permissions** and click **Attach policies**.

![/static/img/lab5/](/static/img/lab5/7.png)

8.  You need to add 3 policies. Enter :code[AWSElasticBeanstalk]{showCopyAction=true} into the **Filter policies** box. Select the **AWSElasticBeanstalkWebTier**, **AWSElasticBeanstalkMulticontainerDocker** and **AWSElasticBeanstalkWorkerTier**. Click **Attach policies**.

![/static/img/lab5/](/static/img/lab5/8.png)

#### Deploy your server to Elastic Beanstalk

9.  Navigate to the **Elastic Beanstalk** service and click **Create application**.

![/static/img/lab5/](/static/img/lab5/9.png)

10.  In the **Application Information** section. Enter :code[TSGalleryBeanstalk]{showCopyAction=true} as the **ApplicationName**. Leave the **Application tags** section blank.

![/static/img/lab5/](/static/img/lab5/10.png)

11.  In the **Platform** section. Select **Node.js** in the **Platform** dropdown. The **Platform branch** and **Platform version** should auto-select the latest versions of NodeJS for you.
    
![/static/img/lab5/](/static/img/lab5/11.png)

12.  In the **Application code** section. Select **Sample application** from the options.
    
:::alert{header="Note"}
We will select the Sample Application at this point. Later in this lab we will update our Beanstalk deployment and watch it deploy the website using the ts\_gallery.zip file we downloaded earlier.
:::

13.  In the **Presets** section, select **High availability** as the **Configuration presets**. Click **Next**.

![/static/img/lab5/](/static/img/lab5/13.png)

14.  In the **Configure service access** section. Select **Create and use new service role**, leave the **Service role name** as the given default name. Select the _tsa-ap-southeast-2_ as the **EC2 key pair** and select _tsgallery.roles.serverrole_ as the **EC2 instance profile**. Click **Next**

![/static/img/lab5/](/static/img/lab5/14.png)

15.  In the **Set up networking, database, and tags** section. Select the _tsgallery.vpc_ as the VPC. Select **Activated** on the Public IP address and select the two public subnets under the **Instance settings**. Under **Database**, leave everything default. Click **Next**.

![/static/img/lab5/](/static/img/lab5/15.1.png)

:::alert{header="Note"}
In a production environment, your instances should be in a private subnet. This would be a good case for having three tiers of subnets rather than just two. For example, your public subnets, then private subnets for instances and another set of private subnets for databases. However, to support this, you would also need to consider what services your worker instances are accessing. For example, you may need to add a NAT Gateway, or Amazon RDS VPC endpoints to allow access to the RDS instances. In this exercise, we will simply host the instances in the public subnet and secure them with Security Groups.
:::

![/static/img/lab5/](/static/img/lab5/15.2.png)

16.  In the **Configure instance traffic and scaling** section. Under **EC2 security groups**, select _tsgallery.securitygroups.webserver_.

![/static/img/lab5/](/static/img/lab5/16.1.png)

Under **Load balancer network settings**, verify that the Visibility is set to _Public_ and the two public subnets are selected.

![/static/img/lab5/](/static/img/lab5/16.2.png)

Scroll to the **Processes** section, select _default_ and click **Action > Edit**.

![/static/img/lab5/](/static/img/lab5/16.3.png)

An **Add process** windows will appear. Expand the **Sessions** and select **Enabled** under the Session stickiness. Click **Save**. Click **Next**

![/static/img/lab5/](/static/img/lab5/16.4.png)

17.  In the **Configure updates, monitoring, and logging** section. Under **Monitoring**, select **Basic** for the System. Under **Managed platform updates**, deseclect **Activated**. Leave everything default and click **Next**.

![/static/img/lab5/](/static/img/lab5/17.png)

18.  Review thw Elastic Beanstalk configuration and click **Submit**.

![/static/img/lab5/](/static/img/lab5/18.png)

19.  The **Environment overview** will appear, wait for a few minutes until the **Health** turn into **Green**.

![/static/img/lab5/](/static/img/lab5/19.1.png)

![/static/img/lab5/](/static/img/lab5/19.2.png)
    
20. Verify that the evironment is fully deployed by clicking the **Domain** url. A new browser tab will open and a **Congratulations** page will display confirming a successful environment creation.

![/static/img/lab5/](/static/img/lab5/20.1.png)

![/static/img/lab5/](/static/img/lab5/20.2.png)


#### Updating CloudFront

21.  Navigate to the **CloudFront** service and click the distribution ID to open the distribution.

![/static/img/lab5/](/static/img/lab5/21.png)

22.  On the **Origins** tab, select the origin with an **Origin ID** that ends with **compute.amazonaws.com**. Click **Edit**.

![/static/img/lab5/](/static/img/lab5/22.png)

23.  Delete the contents of the **Origin Domain** box. Once the box is empty, a dropdown list will appear. Select the Load balancer listed under the heading **Elastic Load Balancers**. 

![/static/img/lab5/](/static/img/lab5/23.png)

Confirm that the _Protocol_ **HTTP only** is selected. At the bottom, click **Save changes**.

![/static/img/lab5/](/static/img/lab5/23.1.png)

#### Terminate the web server

Now that you are running a high availability server using Elastic Beanstalk, you no longer need your original server.

24.  Navigate to the **EC2** service, click **Instances** from the left hand menu and then select the original web server called **tsgallery.webserver**. Under the **Instance State** dropdown click **Terminate Instance**.

![/static/img/lab5/](/static/img/lab5/24.png)

25.  Click **Terminate** to confirm termination of the instance.

![/static/img/lab5/](/static/img/lab5/25.png)

26.  Once the web server **Instance State** changes to **terminated** it is no longer running.

#### Update your Application

27.  Go back to Cloud Front and open in a new tab your Distribution domain name. You're going to see a sample aplication like bellow:

![/static/img/lab5/](/static/img/lab5/27.png)

28.  Now, you will update your Elastic Beanstalk with the application code you downloaded earlier. Go to the Elastic Beanstalk Service. In the Left side menu, click on **Environments**, then in the list click the **environment name** link.

![/static/img/lab5/](/static/img/lab5/28.png)

29.  In the Environment overview windows, click the **Upload and deploy** button.

![/static/img/lab5/](/static/img/lab5/29.png)

30.  Click the **Choose file** button and locate your local **ts\_gallery.zip** file.
    
31.  Add :code[ts_gallery_v1]{showCopyAction=true} as your Version label. Under the **Deployment preferences** select **All at once** as the Deployment policy. Click **Deploy**.

![/static/img/lab5/](/static/img/lab5/31.png)

:::alert{header="Note"}
If you encounter the following error message:

![/static/img/lab5/](/static/img/lab5/31.1.png)

Go to the [S3](https://s3.console.aws.amazon.com/s3/buckets?region=ap-southeast-2) service.

![/static/img/lab5/](/static/img/lab5/31.2.png)

Click on the Elastic Beanstalk bucket (bucket that start with _elasticbeanstalk_).

![/static/img/lab5/](/static/img/lab5/31.3.png)

Select the **Permissions** tab.

![/static/img/lab5/](/static/img/lab5/31.4.png)

Under the **Object Ownership** section. Click on **Edit**.

![/static/img/lab5/](/static/img/lab5/31.5.png)

Under the **Object Ownership** section. Select **ACLs enabled**, tick the **I acknowledge that ACLs will be restored**. Select **Object writer**. Click **Save Changes**.

![/static/img/lab5/](/static/img/lab5/31.6.png)

Repeat steps **28** to **31** but give the _Version label_ name :code[ts_gallery_v2]{showCopyAction=true}.

![/static/img/lab5/](/static/img/lab5/31.7.png)

:::

32.  After 2 minutes, check again the Cloud Front distribution URL. You will see that your application has been refreshed.

![/static/img/lab5/](/static/img/lab5/32.png)

### Wrap-up

In this lab you have achieved the goal of migrating the web application to Elastic Beanstalk. Your photo gallery is now running as a single tenanted, high availability application. The steps completed in this lab will support you on migrating many of your applications to the cloud.

### Next...

Congratulations! You have completed all the labs for this course. If you used your own AWS Account along these labs, don't forget to clean up the resources in order to avoid costs.