

```
Prerequisites
First, make sure Oracle jdk is installed. I recommend java 1.8.0
To uninstall effectively your current jdk, perform this:
$ sudo apt-get remove openjdk*
$ sudo apt-get remove --auto-remove openjdk*
$ sudo apt-get purge openjdk*
$ sudo apt-get purge --auto-remove openjdk*

To install java 1.8.0, open Terminal Ctrl+Alt+T and run the command:
$  sudo add-apt-repository ppa:webupd8team/java // adds PPA repository
$  sudo apt-get update // updates package list
$  sudo apt-get install openjdk-8-jdk // installs openjdk
```
## Part 1 : create a User and a Group using IAM

- We log into our `AWS` management console using `$ https://console.aws.amazon.com.`<br>

I'm using `MFA` to secure my root account access coupled with `Google Authenticator` on my `Android` smartphone.<br>

We can bypass this step and login normally to `AWS` Management Console.<br>

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-1.jpg](https://i.postimg.cc/L5F2KQwp/isaac-arnault-AWS-1.jpg)](https://postimg.cc/nj26q2nR)

</p>
</details>

We go to Services > IAM > Users > Add user<br>

<li>User name : user-1</li><br>

<li>Access type : Programmatic access</li><br>

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-16.png](https://i.postimg.cc/Mpdv5JTN/isaac-arnault-AWS-16.png)](https://postimg.cc/fVSzWFYf)

</p>
</details>

<b> Next : Permissions > Create group</b><br>

<li>Group name : Developers</li><br>

<b>Administrator Access > Create group</b><br>

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-17.png](https://i.postimg.cc/cJC65ktH/isaac-arnault-AWS-17.png)](https://postimg.cc/Ty8RK99M)

</p>
</details>

<b>Next : Tags</b><br>

<li>Key: dev-1 | Value: name of the developer</li><br>

<b>Create user</b><br>

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-18.png](https://i.postimg.cc/sXpzn5mx/isaac-arnault-AWS-18.png)](https://postimg.cc/hzPNvzHR)

</p>
</details>

<b>Download .csv</b> (you're going to use these credentials later on in this tutorial)<br>

- We write down our Access key ID and Secret access key > close the window<br>

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-28.png](https://i.postimg.cc/WzPg3281/isaac-arnault-AWS-28.png)](https://postimg.cc/FdD7CXwM)

</p>
</details>

- Now in Groups we should have one group named Developers which should list <b>user-1</b>.

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-20.png](https://i.postimg.cc/TPfZch1q/isaac-arnault-AWS-20.png)](https://postimg.cc/dhNHss8L)

</p>
</details>

<hr>

## Part 2 : deploy one EC2 instance using a custom script<br>

Services > EC2<br>

- In "Create Instance" section, click on "Launch Instance"<br>

We're going to choose 2 instances <br>

- We select Amazon Linux 2 AMI (HVM), SSD Volume Type<br>

- Instance type: choose t2.micro (Free tier eligible). Instance comes with 1vCPU and 1 GiB (memory).<br>

<b>Next: Configure instance details</b>

We choose to deploy one instances and we provision the<b> Advanced details </b> section with the following script:<br>

<details>
<summary>🔵 See script</summary>
  
<p>  
  
#!/bin/bash<br>
yum install httpd php php-mysql -y<br>
cd /var/www/html<br>
wget https://wordpress.org/wordpress-5.1.1.tar.gz<br>
tar -xzf wordpress-5.1.1.tar.gz<br>
cp -r wordpress/* /var/www/html/<br>
rm -rf wordpress<br>
rm -rf wordpress-5.1.1.tar.gz<br>
chmod -R 755 wp-content<br>
chown -R apache:apache wp-content<br>
service httpd start<br>
chkconfig httpd on<br>

</p>
</details>

- We leave all fields as they're by default, we just Enable termination protection.<br>

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-46.png](https://i.postimg.cc/qBCJ7j2F/isaac-arnault-AWS-46.png)](https://postimg.cc/3WY5SjSj)

</p>
</details>

<b>Next : Add Storage</b><br>

- We leave all fields as they're by default.<br>

<b>Next : Add Tags</b><br>
We use as Key : Name and Value : WP-Server.<br>
 
<b>Next : Configure Security Group</b><br>

- We create a new security group > Security group name: WP-SG > Description : WordPress Server Security Group<br>

Make sure you Add Rule (add 4 new rules):<br>

1. HTTP- This will enable access to `Port 80`.<br>
2. SSH - This will enable access to `Port 22` using source 0.0.0.0/0<br>
3. SSH - This will enable access to `Port 22` using source::/0<br>
4. HTTPS - This will enable access to `Port 443`.<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-60.png](https://i.postimg.cc/VLBjN5H0/isaac-arnault-AWS-60.png)](https://postimg.cc/RNh3sCmv)

</p>
</details>

Review and launch > Launch > Create New Key Pair > Key Pair Name : EC2KP > Download Key Pair.

<b>Launch Instances > View Instances</b><br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-55.png](https://i.postimg.cc/7Y6GHpxy/isaac-arnault-AWS-55.png)](https://postimg.cc/V5xL4Zr7)

</p>
</details>

- At this point of the tutorial, we should have one running EC2 instance, a User and a Group created via IAM.

<hr>

## Part 3 : create a RDS instance

- Go to Services > Database > RDS<br>

- We'll click on "Create database" and select "MySQL" then "Next" <br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-47.png](https://i.postimg.cc/wTqsRCFv/isaac-arnault-AWS-47.png)](https://postimg.cc/9zkfvN65)

</p>
</details>

- Choose use case - select Dev/Test - MySQL<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-50.png](https://i.postimg.cc/x8t8Pdgd/isaac-arnault-AWS-50.png)](https://postimg.cc/dZZJswbM)

</p>
</details>

- Specify DB details : we'll leave all fields as they're by default, except :<br>
  
<b>DB instance class</b> : select db.t2.micro which comes with 1 vCPU and 1 GiB RAM. Obviously, if you need more computational resources, feel free to scale it up.<br>

<b>DB instance identifier</b> : wp_mySQL

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-50.png](https://i.postimg.cc/5tBCCfcX/isaac-arnault-AWS-50.png)](https://postimg.cc/v1mTRFZy)

</p>
</details>

<b>Configure advanced settings</b>

- Database options : Database name<br>

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-50.png](https://i.postimg.cc/5tBCCfcX/isaac-arnault-AWS-50.png)](https://postimg.cc/v1mTRFZy)

</p>
</details>

- Backup : Backup retention period<br>

Set it to 0 days <br>

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-52.png](https://i.postimg.cc/sxsdg9YB/isaac-arnault-AWS-52.png)](https://postimg.cc/FdWnPSnm)

</p>
</details>

- Deletion protection : we enable deletion protection, then we "Create database".<br>

<details>
<summary>🔴 See output</summary>
<p>

[![isaac-arnault-AWS-53.png](https://i.postimg.cc/N0HW8Pb9/isaac-arnault-AWS-53.png)](https://postimg.cc/8FTKN42N)

</p>
</details>

To verify that our RDS instance is deployed, we go to RDS > atabases<br>

<details>
<summary>🔴 See output</summary>
<p>

[![isaac-arnault-AWS-54.png](https://i.postimg.cc/Y0nnB1bB/isaac-arnault-AWS-54.png)](https://postimg.cc/rR4CSRBf)

</p>
</details>

## Part 4 : make both security groups talk to each other<br>

We'll now enable the communication between our originate Security Group and the one created once we set an `Amazon RDS` instance.<br>

Go to EC2  > Security Groups. At this stage of the tutorial you should see 3 security groups : one is default, the second one is the one created by the `Amazon RDS` instance which starts wich Group Name is `rds-launch-wizard`, and the last one is the one we manually created.

<details>
<summary>🔴 See output</summary>
<p>
  
[![isaac-arnault-AWS-56.png](https://i.postimg.cc/65Wvg87z/isaac-arnault-AWS-56.png)](https://postimg.cc/Ths1WYJW)

</p>
</details>

- Select `rds-launch-wizard` and go to "Inbound". Click "Edit" > Add Rule > Select MYSQL/Aurora and enter GroupID of WP-SG, then "Save". See below output.<br>

<details>
<summary>🔴 See output</summary>
<p>
  
[![isaac-arnault-AWS-57.png](https://i.postimg.cc/T16fF5C9/isaac-arnault-AWS-57.png)](https://postimg.cc/62YkGqs2)

</p>
</details>

Once this is done, move on Part 5 to deploy your Wordpress site.


## Part 5 : deploy your Wordpress site

# 5.1: tune your WP server on your web browser

Connect to `IPv4 Public IP` of your `EC2` instance in your web browser. You should see the below screen.<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-58.png](https://i.postimg.cc/G2k4SykL/isaac-arnault-AWS-58.png)](https://postimg.cc/8Jzk7sK9)

</p>
</details>

Click "Let's go" and fill up the provided fields.<br>

Replace "localhost" in "Database host" filed by your `RDS` endpoint. Go to RDS > Databses > select your database and copy "Endpoint url". Paste it into "Database Host" field.<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-59.png](https://i.postimg.cc/3RvD2qLc/isaac-arnault-AWS-59.png)](https://postimg.cc/CBw1gcpG)

</p>
</details>

Before you "Run the installation", connect to your `EC2` instance.<br>

# 5.2: connect to your EC2 instance

We should remember that we've downloaded an EC2KP.pem file earlier. We will now move this file to a newly created directory.<br>

Ctrl + Alt + T to open a new CLI window<br>

`$ cd Desktop > $ mkdir SSH` - Creates an SSH directory to store our Key Pair (credentials).<br>

`$ cd Downloads` > `$ sudo mv /home/zaki/Downloads/EC2KP.pem /home/zaki/Desktop>SSH`<br>

- Go to your SSH directory and check that the file persists there : `$ cd Desktop/SSH` > ls<br>

- We change the permissions to .pem file, ie: `$ chmod 400 EC2KP.pem`.<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-23.png](https://i.postimg.cc/4xbCDphh/isaac-arnault-AWS-23.png)](https://postimg.cc/zyBPWb7J)

</p>
</details>

- We will now connect to our EC2 instance using our `CLI`<br>

- Use : `$ ssh ec2-user@your-ipv4-public-address -i EC2KP.pem`.<br>

- Type "yes" when prompted by the `CLI`<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-24.png](https://i.postimg.cc/jj5X0d3V/isaac-arnault-AWS-24.png)](https://postimg.cc/qNPn20dQ)

</p>
</details>

- Go in root mode : `$ sudo su` and use `$ aws s3 ls`. The last command should return "Unable to locate credentials. We can configure credentials by running "aws configure".<br>

To use your provided credentials use : `$ aws configure` <br>

Remember that we wrote down our `Access Key ID` and `Secret access key` when creating our EC2 Instances. We use the provided credentials.<br>

- We connect to our `EC2` instance using the following command:

`$ ssh ec2-user@your-ipv4-address -i EC2KP.pem`

- We provide Access Key ID > AWS Secret Access Key > Default region name (use the Availability Zone of our EC2 instance, ie : us-east-1) > default output format : we can use "text" or "json". In this tutorial we use "json".<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-36.png](https://i.postimg.cc/G3f9qwg2/isaac-arnault-AWS-36.png)](https://postimg.cc/CdjFxtgT)

</p>
</details>

<hr>
<b>Important</b><br>
If buckets do not show up, we can go to Users > Security credentials > Create a new access key. Or we can create a new EC2 instance and restart the procedure in our `AWS` CLI.<br>

When you Create access key, you'll have to download a file "access.Keys.csv".<br>
<br>

When connected to SSH, we should perform the following commands :

`$ sudo su` - to elevate our priviledges<br>

`$ cd /var/www/html` - to enter html directory<br>

`$ nano wp-config.php` - to create and set up a `Wordpress` conf file<br>

At this step, we should go back to our web browser and copy/paste the provided script upon `Wordpress` installation. We paste it into our config.php file. See below :<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-61.png](https://i.postimg.cc/j5frHpGm/isaac-arnault-AWS-61.png)](https://postimg.cc/vgGjMN8t)

</p>
</details>

`Ctrl + x` and y - to exit from our config.php file and save the file<br>

Now we can go back to our web browser to complete the installation. Click "Run the installation" and fill out the form.

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-62.png](https://i.postimg.cc/5N42J0ct/isaac-arnault-AWS-62.png)](https://postimg.cc/nMWJYnh8)

</p>
</details>

We should be able to log into our Wordpress dashboard if everything went fine.<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-63.png](https://i.postimg.cc/QdRDFNz1/isaac-arnault-AWS-63.png)](https://postimg.cc/Jscvg8Mz)

</p>
</details>

<details>
<summary>🔵 See dashboard</summary>
<p>  
  
[![isaac-arnault-AWS-64.png](https://i.postimg.cc/28GFjVV9/isaac-arnault-AWS-64.png)](https://postimg.cc/9r4qJXPy)

</p>
</details>

Go to appearance and activate your desired theme.

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-65.png](https://i.postimg.cc/SQ6cPK8s/isaac-arnault-AWS-65.png)](https://postimg.cc/Rq0364g5)

</p>
</details>

<hr>

