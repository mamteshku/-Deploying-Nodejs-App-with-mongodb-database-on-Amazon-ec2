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

Now that we are connected to the instance, we can start installing the required packages in the environment for your node.js app to work. We will be using NVM (Node Version Manager) to install node.js as it provides a neat way to manage multiple node.js versions on the instance if required. Run the following code in the terminal to get NVM and install it.
  
  curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.8/install.sh | bash

We will use nvm to install Node.js because nvm can install multiple versions of Node.js and allow you to switch between them. See the nvm repo on GitHub for the current version to install.

Activate nvm by typing the following at the command line.
  . ~/.nvm/nvm.sh

Use nvm to install the version of Node.js you intend to use by typing the following at the command line.

  nvm install 7.9.0

Test that Node.js is installed and running correctly by typing the following at the command line.

  node -e "console.log('Running Node.js ' + process.version)"
  
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


# Move the app to port 80 (not really)

The URL for our app contains the port number 3000 which if you have surfed the web a lot will appear a bit out of the place because you don’t usually see such numbers in the URLs, reason being the websites are generally served on the standard port 80 used for HTTP requests and browsers assume it by default.

Currently our node server is running on a publicly open port which is not a particularly brilliant idea as we are directly exposing the application server to internet traffic and also reducing performance by not taking advantages of load balancing or serving static content efficiently.

A better way to manage this is to use Nginx as a reverse proxy server in front of all other application servers. It will act as an efficient web server routing multiple requests on port 80 to different application servers based on the requested domain or even route high volume internet traffic to the same domain on port 80 to multiple instances of the application server listening on different private ports to perform load balancing in case of application server overloading or crashing. Nginx also takes care of SSL encryption instead of letting Node do all the work.

Let’s install Nginx first:
  sudo yum install nginx
and verify the installed version with 
  nginx -v.

Enter the public DNS or the public IP of your EC2 instance in the browser now without a port and you should see the default Nginx welcome page.

Now, we are not really going to move the app to listen on port 80, the app will still be running on port 3000. We will install Nginx in front of the application server, run it on port 80 so that it can intercept all internet traffic and route it to port 3000 where the required application server will be listening if the HTTP headers match (more details in the next article on serving the site on HTTPS). A nice explanation of why we should be doing this is available here.

To use port 80 to route requests to the application server on port 3000, we will edit the nginx configuration file — nginx.conf as follows.

Open the nginx.conf file with an editor in the terminal:
  sudo vi /etc/nginx/nginx.conf
  
If there’s no server block listening on port 80, add one or change it if it already exists to look like this:

server {
   listen         80 default_server;
   listen         [::]:80 default_server;
   server_name    localhost;
   root           /usr/share/nginx/html;
   location / {
       proxy_pass http://127.0.0.1:3000;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection 'upgrade';
       proxy_set_header Host $host;
       proxy_cache_bypass $http_upgrade;
   }
}

Since we have a single server block listening on port 80, all traffic will be routed from here itself regardless of the value of the server_name directive. The location directive has been changed to point to the resource from where the content has to be served, in our case it is the node.js application server running on the private localhost IP 127.0.0.1 and port 3000. Ignore the other directives for now.

Save the configuration and restart the server with the following command:

  sudo service nginx restart
  
You will now have all HTTP traffic available on port 80 forwarded to port 3000. You should close port 3000 to public access by removing it from the security group settings. Visit the URL again in the browser without the port and you will be able to see the app content now instead of the default Nginx welcome page.

#Configure Nginx to autostart on instance reboot
There’s just one last thing to take care of before we wrap up. We need to configure Nginx to start automatically if the instance is rebooted. Execute the following command to do this:

  sudo chkconfig nginx on

You can close the session, reboot the instance and visit the URL again. You should see the app being served on port 80.

