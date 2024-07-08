# SummerCamp Week Two Challenge 

Ops Manager installation files.
Step 1: Setting Up the VMs

Ensure your three VMs are up and running. For the purposes of this walkthrough, so like last week we can call them:
VM1: Ops Manager Instance 1
VM2: Ops Manager Instance 2
VM3: Load Balancer

Step 2: Installing and Configuring Ops Manager
VM1 and VM2: Install Ops Manager
Download and Install Ops Manager:

# Update package list and install dependencies
```
sudo apt-get update
sudo apt-get install -y openjdk-8-jre-headless wget
```

# Download Ops Manager
```
wget https://downloads.mongodb.com/on-prem-mms/deb/mongodb-mms_VERSION.deb
```

# Install Ops Manager
```
sudo dpkg -i mongodb-mms_VERSION.deb
```

Configure Ops Manager:
Edit the configuration file (/opt/mongodb/mms/conf/conf-mms.properties) to set up your Ops Manager instance:
```
sudo nano /opt/mongodb/mms/conf/conf-mms.properties
```
Key configurations to set:
```
mms.centralUrl=http://192.168.1.101:8080
#mms.fromEmailAddr=opsmanager@example.com
#mms.replyToEmailAddr=noreply@example.com
```

Start Ops Manager:
```
sudo service mongodb-mms start
```

Rinse and repeat the above steps for VM2.

VM3: Let's install the load balancer 
Install Nginx (as an example load balancer):

```
sudo apt-get update
sudo apt-get install -y nginx
```
Configure Nginx:

Edit the Nginx configuration file to load balance traffic between the two Ops Manager instances:

sudo nano /etc/nginx/sites-available/default
Nginx configuration: um I think?
```
upstream opsmanager {
    server <vm1_ip>:8080;
    server <vm2_ip>:8080;
}

server {
    listen 8080;

    location / {
        proxy_pass http://opsmanager;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
Restart Nginx:
```
sudo service nginx restart
```
Step 3: Deploying Application Database on VMs
Install MongoDB on each VM:

# Add MongoDB repository
```
wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
```

# Install MongoDB
```
sudo apt-get update
sudo apt-get install -y mongodb-org
```
Start MongoDB:
```
sudo service mongod start
```
Configure MongoDB Replica Set:
Connect to MongoDB and initiate the replica set:

```
mongo --host 192.168.1.101:27017

rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "192.168.1.101:27017" },
    { _id: 1, host: "192.168.1.102:27017" },
    { _id: 2, host: "192.168.1.103:27017" }
  ]
})
```
Step 4: Install Ops Manager Agent and Ensure Connectivity
Install the Ops Manager Agent on Each VM:
```
wget https://downloads.mongodb.com/on-prem-mms/deb/mongodb-mms-monitoring-agent_VERSION_amd64.deb
sudo dpkg -i mongodb-mms-monitoring-agent_VERSION_amd64.deb
```
```
wget https://downloads.mongodb.com/on-prem-mms/deb/mongodb-mms-backup-agent_VERSION_amd64.deb
sudo dpkg -i mongodb-mms-backup-agent_VERSION_amd64.deb
```
Configure the Monitoring and Backup Agents:
```
sudo nano /etc/mongodb-mms/monitoring-agent.config
sudo nano /etc/mongodb-mms/backup-agent.config
```
Set the mmsGroupId, mmsApiKey, and mmsBaseUrl in these configuration files.

Start the Agents:

```
sudo service mongodb-mms-monitoring-agent start
sudo service mongodb-mms-backup-agent start
```
