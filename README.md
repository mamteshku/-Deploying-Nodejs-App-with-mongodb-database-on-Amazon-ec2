# These are the steps

1. Create an AWS account
2. Launch an EC2 instance
3. SSH into your instance
4. Install Node.js
5. Install Git and clone repository from GitHub
6. Start the node.js app
7. Keep the app running
8. Move the app to port 80 (not really)
9. Configure Nginx to autostart on instance reboot

# Step 1  create a new ec2 instance from Amazon Web Service.

To create a new instance, you should have an active account on AWS.After logging to Aws
select an ec2 option from Services
Then click on Launch Instance button after that you’ll land to below page.

Then select one image from list of options. Please select an image according to your requirement and whichever is suitable for you. So in my case I’ll select Amazon Linux.

Note: After that maybe some installation command not work if you select an image other than Amazon Linux ,Centos.

Next Choose instance type, Let’s choose t2.micro which is eligible for the free tier, so if your account is less than 12 months old you can run your server for free. Thanks to Amazon!

Next Configure instance details, this is more complicated step but we can ignore this for now.

Next Add storage, Default size is 8 Gb but you can update size according to your requirement , but for now 8gb is enough

Next Add Tags, add key-value pair for instance, but for now we’ll skip this step .Tags is useful if you more instance it’s better to search by Tag

Next Configure Security Group,In my opinion this steps is more important ,Where we inbound(expose our server port) and outbound(restrict to access other server)

In our case , to connect ec2 instance wee need open ssh port 22 and to access our site publicly we need expose http port 80 (when you visit any website by default it connect to port 80 )with selecting option anywhere for source

Inbound & Outbound Source 
1. Anywhere from anywhere we can access this port
2. Custom only provided IP access this port
3. My IP only access this port within same server

But you can expose any port according to your requirement , such as for ftp open port 21, for https open port 443

Next Click review and launch, then you’ll see options selected in all steps

Next Click on Launch, then it’ll ask to create new key pair , which will used to connect our server using ssh with this key.

Download this key pair, and click on Launch

# Step 2# Setup ssh connection to connect EC2 instance

After creating instance , Go to that instance. In this page there is connect button click on that connect button then you’ll see this modal

So follow the steps which you’ll see when you open a modal.

Steps :

> chmod 400 <pem file path>
> ssh -i <pem file path> <user>@<public DNS>

eg. ssh -i "sunilaws.pem" ec2-user@ec2-54-218-8-133.us-west-2.compute.amazonaws.com

Note: In my case, user for my ec2 server is ec2-user because I’ve selected Amazon Linux Image.So In your case user will be different if you have selected an Amazon Machine Image other than Amazon Linux. Here is the list of al default user for Amazon Image . See here
Step 3# Install Node Js on ec2 instance

# To set up Node.js on your Linux instance

    Connect to your Linux instance as ec2-user using SSH.
    Install the current version of node version manager (nvm) by typing the following at the command line to install version 33.8.

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash

We will use nvm to install Node.js because nvm can install multiple versions of Node.js and allow you to switch between them. See the nvm repo on GitHub for the current version to install.

 3. Activate nvm by typing the following at the command line.

. ~/.nvm/nvm.sh

 4. Use nvm to install the version of Node.js you intend to use by typing the following at the command line.

nvm install 7.9.0

 5. Test that Node.js is installed and running correctly by typing the following at the command line.

node -e "console.log('Running Node.js ' + process.version)"

This should display the following message that confirms the installed version of Node.js running.

Running Node.js v7.9.0

For more info, click on this link
# Step 4# Install Mongodb on ec2 instance

We’ve selected Amazon Image , we’ll install mongodb using yum command

Steps :

    Configure the package management System(yum)
    Create a /etc/yum.repos.d/mongodb-org-3.6.repo file so that you can install MongoDB directly, using yum.
    Use the following repository file:

[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/amazon/2013.03/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc

2. Install the mongodb package

sudo yum install -y mongodb-org

3. Start mongodb server

sudo service mongod start

4. Check mongodb server started by running below command

mongo

For more info , Please got to this link
# Step 5 # Setup up Nodejs App(Express)

In this step, we’re going to setup a new sample express app with mongodb connection using mongodb client library.
According to your requirement, you can use any mongo library available in npm, such as mongoose,mongojs etc.

First connect to your ec2 instance using command provided in Step 1

ssh -i <pem file path> <user>@<public DNS>

Then

mkdir SampleExpressApp
npm init

Create new file app.js with following code

var express = require("express");
var app = express();
var MongoClient = require("mongodb").MongoClient;

app.get("/", function(req, res) {
  res.send("Hello World!");
});

app.get("/users", function() {
  MongoClient.connect("mongodb://localhost:27017/test", function(err, db) {
    if (err) next
    db
      .collection("users")
      .find()
      .toArray(function(err, result) {
        if (err) throw err;

        res.json(result)
      });
  });
});

app.listen(3000,function(){
    console.log('Express app start on port 3000')
});

Then install mongodb & express dependancy

npm install mongodb --save
npm install express --save

Then start server

node app.js

Now Express App start on port 3000

Then open below url in browser

http://<your public DNS>:3000

But if you close this terminal or if you perform Ctrl+C server will stop.

So to start server in the background there are multiple NPM library but we’ll use forever to start server. Here is the different option
https://expressjs.com/en/advanced/pm.html

Install forever globally using npm

npm install -g forever

Start server using forever

forever start app.js

To see list of forever process

 forever list

To see Express Server logs

tail -f <logfile path>

If you want to store logs in predefined file, then start server with following command

forever start app.js -l /path/to/log/file

To access this server publicly, you’ve to open port 3000 from security group by adding into inbound rule, as we open port in Step1

After opening 3000 port publicly , hit below url

http://<your pblic DNs>:3000
eg. http://ec2-0-0-0-0.us-west-2.compute.amazonaws.com:3000

But To access your app on public domain (port 80) you’ve to forward port 80 to 3000.

We’ve 2 way to forward port 3000 to 80, you can choose any one. But I’ll prefer to to select option of nginx

    iptables
    nginx

iptables

iptables -A INPUT -i eth0 -p tcp --dport 80 -j ACCEPT

iptables -A INPUT -i eth0 -p tcp --dport 3000 -j ACCEPT

iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 80 -j REDIRECT --to-port 3000

nginx

Install nginx

sudo yum install nginx -y

Start nginx Server

sudo service nginx start

To see nginx started , Enter your public DNS URL in browser

Next step , To forward port 3000 to 80

Edit below nginx configuration file

vi /etc/nginx/nginx.config

And change below code

location / {
    root html;
    index index.html index.htm;
  }

with following one

location / {
    proxy_set_header  X-Real-IP  $remote_addr;
    proxy_set_header  Host       $http_host;
    proxy_pass        http://127.0.0.1:3000;
  }

Restart nginx for the new config to take effect.

Now visit your server’s public DNS URL, It should show “hello word” in response rather than nginx welcome page

If still not working, then check nginx.config file including another configuration file
