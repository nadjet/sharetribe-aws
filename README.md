#How I setup Sharetribe with Amazon AWS, MySQL and my own domain name

## Disclaimers

1. I am just a developer: servers, linux, cloud computing or Ruby on Rails are **not** my area of specialty. 
2. In this document I try to write as faithfully as I can remember the steps I followed in order to make Sharetribe run with my own domain name using AWS Cloud Services. Unfortunately it is all from memory as I did not keep a log of what I have been doing. I do hope it might help some people who are struggling like I did.

Because of these two disclaimers, I apologize for not having a thorough understanding of each and every step I have gone through, and which might show in some of my rather sloppy instructions and explanations. I welcome comments and improvements.


## Sources of information

1. Sharetribe installation instructions on [Sharetribe github](https://github.com/sharetribe/sharetribe),
2. [Deploying on Amazon AWS (Free-Tier) with EC2, RDS & Sharetribe](https://gist.github.com/pcm211/10950bf5447a51fdcd1c). Please note that some instructions are not correct anymore: for example "bundle exec rake db:schema:load" is "bundle exec rake db:structure:load", as specified in the Sharetribe Github specified above. Also version of Ruby is 2.3.4 as of today's Sharetribe Github (not 2.2.4 as specified).

##Amazon AWS Steps

1. I created an EC2 instance by choosing AMI "Ubuntu Server 14.04 LTS (HVM), SSD Volume Type" (eligible for Free Tier) with T2 Micro (also eligible for Free Tier). I chose this type of server because I followed installation instructions from [this Github repository](https://gist.github.com/pcm211/10950bf5447a51fdcd1c) about "Deploying on Amazon AWS (Free-Tier) with EC2, RDS & Sharetribe", and its use of "apt-get" commands required the use of a certain kind of OS (I had selected a different server before, for which "apt-get" commands did not work).


Once created (and the key saved in a .pem file), you can access the EC2 instance on the command line by clicking "Connect" and copying and pasting the displayed "ssh" command (provided you are in the same directory as where you saved your .pem file on your machine).

3. I created in Security Group Inbound properties a "TCP Custom Rule" on Port 3000 (Source="Anywhere"). You can also add a "TCP Custom Rule" on Port 8000 for testing the connection. 

3. I created an Elastic IP that I associated to my EC2 Instance.

4. I created and started an RDS (Relational Database Service) Database of type MySQL so I can have my Database saved on AWS.  Please remember the password you specify for that DB. I created a single RDS DB that will hold the different DBs that will be created for Sharetribe (development, test).

5. I created another Security Group Input property of type "MYSQL/Aurora" on Port 3000 (Source="Anywhere").

6. I associated the Security Groups created in the previous steps with both the RDS Database and the EC2 instance (according to some source I consulted, specifying the same Security Groups for both RDS and EC2 is important).

7. To test the connection with RDS Database, you can use MySQL Workbench, as in (this video)[https://youtu.be/in7KWCA1ufg]. 


##Domain Name provider Steps

1. Buy domain name. There are many domain name providers. I used 1&1.

2. Configure the DNS Server in your Domain Name provider dashboard by associating your domain name to the AWS Elastic IP in the A Registry. Sharetribe provides [instructions on how to do this for various Domain Name Providers](https://help.sharetribe.com/dns-and-domain-setup).

3. I transferred my domain name to AWS "Route 53". I followed the instructions [here](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/MigratingDNS.html), more specifically the smaller set of steps when domain is not in use. Transferring the domain requires declaring a record set that must be of type A whose name must be your domain name **with** a subdomain. At first I omitted the subdomain and, after creating the marketplace, it forwarded to an invalid url where subdomain was domain (so address was "domain.domain.net" if domain address was "domain.net").

##Sharetribe Steps

1. Follow the instructions on the [Sharetribe github](https://github.com/sharetribe/sharetribe). Before working with AWS and connecting to my own domain name, I did manage to install Sharetetribe in a couple of hours and use it on localhost by following instructions on Github.

2. Make sure you use the specified versions of Ruby and Node (you can specify the correct version of Node to use with the npm package). I had problems because I was using an incorrect version of Ruby (instead of the one specified in Sharetribe GemFile). 

I also had problems because I was using the incorrect version of Node: I got a "vendor-bundle" not found message as reported [here](https://github.com/sharetribe/sharetribe/issues/2096). In order to solve this problem and generate the appropriate custom style files I had to enter:

```bundle exec rake assets:precompile"

as specified (here)[https://github.com/sharetribe/sharetribe/issues/1949]. But for this to work I had to have the correct version of Node running.

##Final stages

1. I followed the instructions in [Deploying on Amazon AWS (Free-Tier) with EC2, RDS & Sharetribe](https://gist.github.com/pcm211/10950bf5447a51fdcd1c).
2. In ```config/database.yml", I specified as "host" the **Endpoint** specified in RDS Dashboard, in addition to username and password (for which I used the same as RDS Database).
3. In ```config.config.yml", I only specified the domain without subdomain or port number.
4. When testing, and to make sure you start again from scratch, you can reboot your instance by typing "sudo reboot now" before connecting back in. This was particularly useful when I got an out of memory error once.


##What next

1. I need to figure out what is AWS S3 for.
2. Setup so that I don't need to type in port number 3000 after url.
3. How to migrate from development to production.