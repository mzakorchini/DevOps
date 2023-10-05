---
title : "Lab 2 - EC2 Servers"
weight : 12
---

### Objectives
The objective of this lab is to build a solid base you can use to modernize your application. This lab includes:

-   Create a new Amazon EC2 instance to host a web server.
-   Install your web server, database and other support tools onto your web server.
-   Test your web server.

:::alert{type="warning" header="Important"}
You must **have completed Lab 1** successfully before proceeding with lab 2.
:::

![static/img/](/static/img/lab2.gif)

#### Lab Guide
1.  Login to your AWS Console.
2.  Ensure you have selected the region you were using in the previous lab.
    
#### Creating a SSH Certificate

3.  Navigate to the EC2 service.

![static/img/lab2/](/static/img/lab2/3.png)

4.  Select **Key Pairs** from the left side menu. Then click **Create Key Pair**.

![static/img/lab2/](/static/img/lab2/4.png)

5.  Give your new key the name :code[tsa-ap-southeast-2]{showCopyAction=true}. Select **pem** as the file format unless you already use PuTTY. Click **Create key pair**.

![static/img/lab2/](/static/img/lab2/5.png)

6.  Your browser will download a file called `tsa-ap-southeast-2.pem`. Keep this file in case you have later problems connecting to your instance.

#### Creating a EC2 Instance

7.  Select **Instances** from the left side menu.

![static/img/lab2/](/static/img/lab2/7.png)

8.  Click **Launch Instances**.
    
9.  In the **Launch an instance** page, under the **Name and tags** section, enter :code[tsgallery.webserver]{showCopyAction=true} in the **Name** field.

![static/img/lab2/](/static/img/lab2/9.png)

10.  In the **Application and OS Images (Amazon Machine Image)** section, ensure **Amazon Linux 2** is selected as the Amazon Machine Image (AMI). Select the drop down under **Architecture** and ensure **64-bit (Arm)** is selected. By selecting the Arm option, your EC2 instance will be running on the AWS Graviton 2 Processor.

:::alert{header=Tip}
The AWS Graviton 2 Processor based instances offer great performance for the cost and are an ideal option in those cases where the server software does not require a specific processor. For more information visit [https://aws.amazon.com/ec2/graviton/](https://aws.amazon.com/ec2/graviton/) 
:::

![static/img/lab2/](/static/img/lab2/10.png)

11.  Scroll down to the **Instance type** section and select the **m6g.medium** instance type.

:::alert{header=Note}
If you do not see the m6g.medium instance type, go back to the previous step and ensure you selected the **64-bit Arm** architecutre.
:::

![static/img/lab2/](/static/img/lab2/11.png)

12.  In the **Key pair (login)** section, select your key pair, e.g. tsa-ap-southeast-2, from the dropdown list.

![static/img/lab2/](/static/img/lab2/12.png)

13.  In the **Network settings** section, click **Edit**, and set the following values:

| Field | Value |
| --- | --- |
| Network | tsgallery.vpc |
| Subnet | tsgallery.subnets.public.a |
| Auto-assign Public IP | Ensure this is enabled |

![static/img/lab2/](/static/img/lab2/13.png)

13.  Continuing in the **Network settings** section, under the **Firewall (security groups)** subsection, ensure **Create security group** is selected. Enter :code[tsgallery.securitygroups.webserver]{showCopyAction=true} for both **Security group name** and **Description**.

14.  Change the **Source type** for rule 1 (ssh) to **My IP** as only you will need access during the lab. Your IP should automatically appear next to the selection. Click **Add security group rule** and change the **Type** to _HTTP_. Change the **Source type** for rule 2 (HTTP) to **Anywhere**
    
![static/img/lab2/](/static/img/lab2/14.png)

15.  In the **Configure storage** section, leave the default values.
    
16.  Expand the **Advanced details** section. In the **IAM instance profile** drop down, select the IAM role you created in Lab 1 called `tsgallery.roles.serverrole`.
    
![static/img/lab2/](/static/img/lab2/16.png)

17.  Scroll to the bottom of the **Advanced details** section and copy and paste the following block of code into the **User data** box.

:::code{showCopyAction=true}
#!/bin/bash 
echo ==== Starting UserData Script ====
curl -k -o /root/setup.sh http://labassets.awstechshift.cloud/prod/migrate/2/setup.sh
chmod +x /root/setup.sh
sudo -i /root/setup.sh
echo ==== Finished UserData Script ====
:::

:::alert{type="info" header="Info"}
User data is a script that is run on the new instance when it first boots up. You can use the user data to install the components needed by your server.

In your user data, you download the setup script, update the permission and then run it as root.

The setup script itself does the following:

-   Update the server
-   Download the files needed for your server
-   Install MySQL and run the initialization SQL script
-   Download and configure NodeJS
-   Download Forever utility
-   Install and configure NGinx
-   Copy all the server files to the correct locations
-   Start the server

Here is an example setup.sh:

```
#!/bin/bash
echo ==== Starting Setup.sh Script ====
echo $HOME
echo ----------------------------------

stageid=D6B6CABA-F1CF-4693-A410-510642EB8FA4

yum update -y

curl -k -o ~/ts_gallery.zip http://labassets.awstechshift.cloud/prod/migrate/2/ts_gallery.zip
curl -k -o ~/setup.sql http://labassets.awstechshift.cloud/prod/migrate/2/setup.sql
curl -k -o ~/nginx.conf http://labassets.awstechshift.cloud/prod/migrate/2/nginx.conf

yum install git -y

yum install mariadb-server -y
systemctl enable mariadb
systemctl start mariadb
mysql -u root < ~/setup.sql

touch ~/.bashrc
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
source ~/.bashrc
nvm install 11.10.1
npm install -g forever

unzip ~/ts_gallery.zip -d /opt/ts_gallery
cd /opt/ts_gallery
npm install
chmod -R 555 /opt/ts_gallery
chmod -R 777 /opt/ts_gallery/uploads
chmod -R 666 /opt/ts_gallery/public/images/uploads

cd /opt/ts_gallery/utils
cp config.js config.install.bak.js
installId=$(uuidgen)
sed -i "4i\      installId: '$installId'," config.js

amazon-linux-extras install nginx1.12 -y
rm -f /etc/nginx/nginx.conf
mv ~/nginx.conf /etc/nginx/nginx.conf

systemctl start nginx
systemctl enable nginx

cd /opt/ts_gallery
forever start ./bin/www.js

echo ==== Finished Setup.sh Script ====
```
:::

![static/img/lab2/](/static/img/lab2/17.png)

18.  Locate the **Summary** section on the right side of the page and review the settings. Ensure **Number of instances** is set to 1 and click **Launch instance**.

![static/img/lab2/](/static/img/lab2/18.png)

19.  After launching the instance, click the instance id link to view the instance state.

![static/img/lab2/](/static/img/lab2/19.png)

#### Accessing your site
20.  You will now need to wait until the **Instance State** switches to **Running**. At this point the server will run your user data script. This should only take a minute or so to run.
    
21.  Select your instance. On the **Details** tab, locate the **Public IPv4 DNS**. Copy this address using the copy button at the start of the address.
    
:::alert{header=Note}
If you click directly the “open adress” link, it will try to open the application using _**https**_ protocol, but only _**http**_ is enabled in the Security Group, so it will not work.
:::

![static/img/lab2/](/static/img/lab2/21.png)

22.  Open this URI in your browser. If the site does not load, the user data is probably still running. Just wait 2 to 3 minutes and refresh the browser.

![static/img/lab2/](/static/img/lab2/22.png)

23.  You can use the photo gallery application to view photos. To access the site administration page, click **Current User: Guest** in the top right corner. Enter the username :code[admin]{showCopyAction=true} and the password :code[awstechshift]{showCopyAction=true}

#### Adding a Tag to our Security Group

Because you created a security group using the instance wizard, it was created without any tags. Best practices state that you should tag all resources, so you will need to go back and update the security group to add the project tag.

24.  In the EC2 service, select **Security Groups** from the left side menu. Then select the security group with the name _tsgallery.securitygroups.webserver_. Select the **Tags** tab and click **Manage tags**.

![static/img/lab2/](/static/img/lab2/24.png)

25.  Click **Add new tag**. Enter the Key :code[Name]{showCopyAction=true} with the value of :code[tsgallery.securitygroups.webserver]{showCopyAction=true}. Click **Save changes**.

![static/img/lab2/](/static/img/lab2/25.png)

### Wrap-up

In this lab you did a basic lift-and-shift migration. Your site is now running on AWS.

### Next...

:::alert{type="info" header="Note"}
Let you instructor know you have completed the lab in the main training chat, and wait for the instructor to confirm the next step.
:::

### Optional Challenges

If you finish early, then here are a couple of challenges to try and complete for yourself:

26.  **Create Auto Scaling Launch Template**

Create a launch template suitable for use by an auto-scaling group See [Create a launch template from an instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-launch-templates.html#create-launch-template-from-instance)  for guidance.

27.  **Set up Auto Scaling Group**

Create an auto-scaling group to automatically restart a webserver in case of failure, using the launch template you just created and a minimum and maximum size of 1. After the new webserver has launched, try out the auto scaling by terminating the new instance and watching for another to be created in its place. See [Creating an Auto Scaling group using a launch template](https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-asg-launch-template.html)  for guidance.