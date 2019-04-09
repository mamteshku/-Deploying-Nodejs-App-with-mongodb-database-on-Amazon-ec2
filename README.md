# Deploying a Node App on Amazon EC2

1. Launch an EC2 instance and SSH into it
You can refer to my article Launching an Amazon EC2 instance to launch an EC2 instance and SSH into it.
When configuring the security group make sure you open the port on which your Node application will run. In my case, the port is 3000.

2.2. Install Node on EC2 Instance

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash

. ~/.nvm/nvm.sh

nvm install 8.10

3. Copy code on your EC2 instance and install dependencies
    1. Using Github, BitBucket or Gitlab
      sudo yum install -y git
      
      git clone https://github.com/mamteshku/node-UserManagement.git
      
4. Start server to run forever
  1. Install pm2 by running the following command
    npm install -g pm2
    
  2. Set up pm2 to start the server automatically on server restart.
    pm2 start app.js
    pm2 save
    pm2 startup
