---
title : "Lab 3 - S3 and CloudFront"
weight : 13
---

### Objectives

The objective of this lab is move the static portions of the solution from the WebServer to a S3 bucket served using Amazon CloudFront. This lab includes:

-   Copying your image assets to your asset S3 bucket.
-   Updating your application to store static assets such as thumbnails and images on S3.

:::alert{type="warning" header="Important"}
You must **have completed Labs 1 and 2** successfully before proceeding with lab 3.
:::

![static/img/](/static/img/lab3.gif)

### Lab Guide

1.  Login to your AWS Console and ensure you are in the region you have been using for the previous labs.


#### Storing images on S3

First, you need to create a bucket to store your images. That will be your data bucket.

2.  Select the **S3** service and click **Create Bucket**.

![/static/img/lab3/](/static/img/lab3/1.png)

![/static/img/lab3/](/static/img/lab3/2.png)

3.  Enter _tsgallery.123456789012_ as the **Bucket name** with 123456789012 replaced by the **AWS Account Number**.
    -   This name will not be used publicly, but the bucket name must be globally unique.
    -   Make a note of the bucket name e.g. by cut and pasting to Notepad as you will need it in later labs.

![/static/img/lab3/](/static/img/lab3/3.png)

4.  Ensure the **Region** matches the region you are using for these labs.
    
5.  At the very bottom of the page click **Create bucket**.
    
![/static/img/lab3/](/static/img/lab3/5.png)

![/static/img/lab3/](/static/img/lab3/5.2.png)

Next you need to copy the image assets to the S3 data bucket and update the code to use the CloudFront locations.

#### Copy the images and Update the server

There are several ways you can access your web server to run commands. The simplest way is to use the Systems Manager service but if you wish to use an ssh or PuTTY tool with the keypair you downloaded earlier that is also possible. Here we describe the Systems Manager approach.

6.  Select the **Systems Manager** service.

![/static/img/lab3/](/static/img/lab3/6.png)

7.  Click **Session Manager** on the left side menu.

![/static/img/lab3/](/static/img/lab3/7.png)

8.  Click **Start session**.

![/static/img/lab3/](/static/img/lab3/8.png)

9.  Pick the **tsgallery.webserver** instance and click **Start Session**.

![/static/img/lab3/](/static/img/lab3/9.png)

-   Systems Manager will start a command line session on your browser.

![/static/img/lab3/](/static/img/lab3/9.2.png)

-   To update the server, you will run an update script.

The update script will do the following:

:::alert{header="Update Script"}
-   Copy the images to S3
-   Remove the images from the server
-   Update the server config to reference S3 bucket
-   Update the filesystem server code to use S3
-   Restart the server
:::

10.  Run the following commands to update the server. You will need the bucket name you noted in step 4.

:::code{language=bash showLineNumbers=true showCopyAction=true}
sudo su -
cd /opt/ts_gallery
sh update.sh
:::

11.  When prompted, select the option to **Update S3** by typing in 1 and hitting enter. You will need to enter the name of your bucket. The script will tell you “images have now been deleted from the server.

To confirm the files are not on the server, return to the browser window with the photo gallery open and refresh it. You should see the site load with broken images“.

Once you validate images are not working, press enter again in the session. The script will appear to stop at the "Add the AWS-SDK package" stage. This stage will take a minute or two to run as it is also updating the server's development tools.

:::alert
If you check your gallery again, you’ll note it works without pictures, this will be fixed after setting up the **Cloud Front Distribution** on next steps.
:::

![/static/img/lab3/](/static/img/lab3/11.png)

:::alert{header="Update problems?"}
If you have any issues with the update in this lab, or in any of the following labs, you can run the update shell script again. The script will allow you to rollback any of the updates and will only offer options that are valid for your current server state.
:::


#### Serving using CloudFront

Lastly, you need to setup a CloudFront distribution to serve the static assets and route the API requests to the server.

12.  Select the **CloudFront** service and click **Create a CloudFront distribution**.

![/static/img/lab3/](/static/img/lab3/12.png)

![/static/img/lab3/](/static/img/lab3/12.2.png)

13.  In the **Origin Settings** section enter the website URI, (your EC2 instance's Public IPv4 DNS), into the **Origin Domain** and make sure that **Enable Origin Shield** is set as **No**.

:::alert{type="warning" header="Important"}
Do not select the S3 bucket from the drop down. We will configure that later.
:::

![/static/img/lab3/](/static/img/lab3/13.png)

14.  In the **Default cache behaviour** section, under **Allowed HTTP Methods** select **GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE**. Ensure **Cache policy and origin request policy** is selected and select **CachingDisabled** from the **Cache Policy** dropdown.

![/static/img/lab3/](/static/img/lab3/14.png)

15.  Scroll down and locate the **Settings** section. In the **Default root object** text box, specify :code[index.html]{showCopyAction=true} and click **Create Distribution**.

![/static/img/lab3/](/static/img/lab3/15.png)

At this stage, the distribution only has the server in it. you need to add the images from S3.

16.  If not already in the distribution you created, open the distribution by clicking on the **ID** in the distribution list.

![/static/img/lab3/](/static/img/lab3/16.png)

17.  Select the **Origins** tab. Click **Create Origin**.

![/static/img/lab3/](/static/img/lab3/17.png)

18.  In the drop down list, for **Origin Domain** select your S3 bucket. The name will appear as your bucket name followed by **.s3.amazon.com**. For example, **tsgallery.542323877777.s3.amazonaws.com**.
    
19.  In the **Origin access** section, select **Legacy access identities** then click on **Create new OIA** and then **Create**. In the **Bucket policy** section, select **Yes, update the bucket policy**.
    
20.  Leave the other settings as default and click **Create origin**.

![/static/img/lab3/](/static/img/lab3/20.png)

21.  Next you need to create the behavior that links to the new origin. Click the **Behaviors** tab, then click **Create Behavior**.

![/static/img/lab3/](/static/img/lab3/21.png)

22.  Enter :code[images/uploads/*.*]{showCopyAction=true} as the **Path Pattern** and select your S3 bucket (similar to _S3-tsagallery.54232877777_) for the **Origin or Origin Group**. Leave the rest of the values as default and click **Create behavior**.

![/static/img/lab3/](/static/img/lab3/22.png)

23.  Click **Distributions** along the top of the page to go back to the distribution list. Wait until the **Status** changes from **In Progress** to **Enabled** which may take **up to 5 minutes**.

![/static/img/lab3/](/static/img/lab3/23.png)

![/static/img/lab3/](/static/img/lab3/23.2.png)

24.  In the Cloud Front main page, copy the value of your Distribution **Domain Name**, some thing similar to _d2fr9z4g1t1ubt.cloudfront.net_. This is the new URI for your photo gallery. Open this in a browser to confirm everything is working properly. If you receive an error stating that your connection is not private, or you get an AccessDenied message, S3 is still updating the it's internal DNS. Your setup is correct, you will simply need to wait until the update completes.

![/static/img/lab3/](/static/img/lab3/24.png)

### Wrap-up

In this lab you achieved a lot. You have taken a big step towards modernizing your application and set a solid basis for the next labs. In review, in this lab you have:

-   Copied your image assets to your asset S3 bucket.
-   Updated your application to store static assets such as thumbnails and images into S3.


### Next...

:::alert{type="info" header="Note"}
Let you instructor know you have completed the lab, then wait for the instructor to confirm the next step.
:::

### Optional Challenges

If you finish early, then here are a couple of challenges to try and complete for yourself:

25.  **Use Versioning to protect images**

Enable versioning on the S3 bucket to protect your images in the event of changes or deletions. See [Enable versioning on buckets](https://docs.aws.amazon.com/AmazonS3/latest/userguide/manage-versioning-examples.html) for guidance.

26.  **Cut storage costs on old images**

Create an S3 lifecycle policy to reduce storage costs for images over 3 months old by moving them automatically to Standard-IA storage class. See [Setting lifecycle configuration on a bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/how-to-set-lifecycle-configuration-intro.html) for guidance.