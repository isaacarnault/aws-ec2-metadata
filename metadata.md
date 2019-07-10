## Part 1 : log into your AWS Management Console

- Log into your `AWS` management console using `$ https://console.aws.amazon.com.`<br>

I'm using `MFA` to secure my root account access coupled with `Google Authenticator` on my `Android` smartphone.<br>

You can bypass this step and login normally.<br>

<details>
<summary>🔴 See output</summary>
<p>  

[![isaac-arnault-AWS-1.jpg](https://i.postimg.cc/L5F2KQwp/isaac-arnault-AWS-1.jpg)](https://postimg.cc/nj26q2nR)

</p>
</details>

## Part 2 : create an EC2 instance

Services > EC2<br>

- In "Create Instance" section, click on "Launch Instance"<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS2.png](https://i.postimg.cc/nVSG28yg/isaac-arnault-AWS2.png)](https://postimg.cc/6TRZ6P5f)

</p>
</details>

- Select Amazon Linux 2 AMI (HVM), SSD Volume Type<br>

- Instance type: choose t2.micro (Free tier eligible). Instance comes with 1vCPU and 1 GiB (memory).<br>

<b>Next: Configure instance details</b><br>

- Leave all fields as they're by default, just Enable termination protection.<br>

- Important : click on Advanced Details (here we'll provide our bootstrapped script).<br>

Use the following Script<br>

<details>
<summary>🔵 See script</summary>
<p>  
  
#!/bin/bash<br><br>

yum update -y<br>
yum install httpd -y<br>
service httpd start<br>
chkconfig httpd on<br>
cd /var/www/html<br>
echo "<html><h1>This is a web server from an EC2 bootstrapped instance!</h1></html>" > index.html<br>
aws s3 mb s3://yourbucketname<br>
aws s3 cp index.html s3://yourbucketname

</p>
</details>

Please note that the last line of the script will push all files related to your web server in a s3 bucket.<br>
<details>
<summary>🔴 See output</summary>
<p>  
  
[![Isaac-Arnault-AWS-31.png](https://i.postimg.cc/CdyyPF8k/Isaac-Arnault-AWS-31.png)](https://postimg.cc/Cn46dpmx)

</p>
</details>

<b>Next : Add Storage</b><br>

- Leave default configuration then click on Next: Add Tags. You can leave tags blanks, here I'm using some tags for my own needs.<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS4.png](https://i.postimg.cc/TY8qFjPJ/isaac-arnault-AWS4.png)](https://postimg.cc/8sH6r6M7)
 
</p>
</details>

<b>Next : Configure Security Group</b><br>

- Create a new security group > Security group name: dev-master > Description : Security Group Master > Add Rule : add HTTP protocole to allow listening on Port 80 > Review and launch > Launch > Create New Key Pair > Key Pair Name : EC2KP > Download Key Pair (EC2KP.pem should appear in your Downloads).

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-21.png](https://i.postimg.cc/XYWd37JH/isaac-arnault-AWS-21.png)](https://postimg.cc/PP6PQH1Y)

</p>
</details>

<b>Launch Instances > View Instances</b><br>

- Rename your instance to "EC2 Bootstrapped Instance - Web Server".<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![Isaac-Arnault-AWS-30.png](https://i.postimg.cc/YCGkzsm1/Isaac-Arnault-AWS-30.png)](https://postimg.cc/G4CZczK2)

</p>
</details>

- At this point of the tutorial, you should have one running `EC2` instance. You can check if it was deployed correctly.<br>

Just use the `IPv4 public address` provided by your `EC2` instance in your web browser.<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-36.png](https://i.postimg.cc/3JpKLykb/isaac-arnault-AWS-36.png)](https://postimg.cc/vcYFTZLW)

</p>
</details>

<hr>

## Part 3 : use your Command Line Interface to connect to your EC2 instance

`$ cd Desktop > $ mkdir SSH`<br>

`$ cd Downloads` > `$ sudo mv /home/zaki/Downloads/EC2KP.pem /home/zaki/Desktop>SSH`<br>

- Go to your SSH directory and check that the file persists there : `$ cd Desktop/SSH` > ls<br>

- Change the permissions to .pem file, ie: `$ chmod 400 EC2KP.pem`.<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-23.png](https://i.postimg.cc/4xbCDphh/isaac-arnault-AWS-23.png)](https://postimg.cc/zyBPWb7J)

</p>
</details>

<b>Connect to your EC2 instance using your `CLI`</b><br>

Use : `$ ssh ec2-user@your-ipv4-public-address -i EC2KP.pem`.<br>

Type "yes" when prompted by the `CLI`<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-24.png](https://i.postimg.cc/jj5X0d3V/isaac-arnault-AWS-24.png)](https://postimg.cc/qNPn20dQ)

</p>
</details>

- Go in root mode : `$ sudo su` and use `$ aws s3 ls`. The last command may return "Unable to locate credentials". You can configure credentials by running "aws configure".<br>

To use your provided credentials use : `$ aws configure` <br>

Remember that you wrote down your `Access Key ID` and `Secret access key` when creating your EC2 Instance. Use the provided credentials (go to your Downloads and check for the credentials.csv file).<br>

- Provide Access Key ID > AWS Secret Access Key > Default region name (use the Availability Zone of your EC2 instance, ie : us-east-1) > default output format : you can use "text" or "json". In this tutorial I'm using "json".<br>

`$ aws s3 ls` displays my available buckets. If your buckets do not show up, go to Users > Security credentials > Create a new access key or create a new EC2 instance and restart the procedure in your CLI.<br>

`$ aws s3 ls` - Outputs that I have 2 available buckets.<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-34.png](https://i.postimg.cc/5tQ1FZqF/isaac-arnault-AWS-34.png)](https://postimg.cc/TyxBzNsR)

</p>
</details>

## Part 4 : retrieve metadata of your EC2 bootstraped instance

`$ curl http://169.254.169.254/latest/user-data` - This should render the script you've provided for bootstraping the `EC2` instance.<br>

<details>
<summary>🔴 See output</summary>
<p>  
  
[![isaac-arnault-AWS-33.png](https://i.postimg.cc/MpTzzcn5/isaac-arnault-AWS-33.png)](https://postimg.cc/WhQBn16q)

</p>
</details>

At this point of the tutorial you may want to save the script in a `.txt` file on your s3 bucket.<br>

`$ curl http://169.254.169.254/latest/user-data > bootstrap.txt`<br>

<details>
<summary>🔴 See output</summary>
<p>    

[![isaac-arnault-AWS-35.png](https://i.postimg.cc/k4N0Q9VD/isaac-arnault-AWS-35.png)](https://postimg.cc/Hcxvmq9C)

</p>
</details>

To retrieve metadata related to your `EC2` instance, use the following command :

`$ curl http://169.254.169.254/latest/meta-data/`<br>

<details>
<summary>🔴 See output</summary>
<p>    

[![isaac-arnault-AWS-36.png](https://i.postimg.cc/3JztxVRq/isaac-arnault-AWS-36.png)](https://postimg.cc/3dCXLL2j)

</p>
</details>

You can curl specific resource of your metadata :<br>

`$ curl http://169.254.169.254/latest/meta-data/public-ipv4` - Returns your `IPv4 public address` for instance.

<details>
<summary>🔴 See output</summary>
<p>    

[![isaac-arnault-AWS-36.png](https://i.postimg.cc/3JztxVRq/isaac-arnault-AWS-36.png)](https://postimg.cc/3dCXLL2j)

</p>
</details>

You may also want to save this metadata into a `.txt` file :<br>

`$ curl http://169.254.169.254/latest/meta-data/public-ipv4 > myipv4.txt`<br>

<details>
<summary>🔴 See output</summary>
<p>    

[![isaac-arnault-AWS-37.png](https://i.postimg.cc/25GdhKkr/isaac-arnault-AWS-37.png)](https://postimg.cc/3yyDH90c)

</p>
</details>

<hr>

That's all for now guys. Feel free to fork it and to spread the word about it. Thanks.
