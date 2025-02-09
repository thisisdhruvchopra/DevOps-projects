# Jenkins Master-Slave Setup on AWS EC2

## Prerequisites
- Two AWS EC2 instances (one for Jenkins Master, one for Jenkins Slave)
- Security group allowing SSH (port 22) and Jenkins (port 8080)
- Java installed on both instances

## Step 1: Install Java

### On Both Master and Slave Nodes:
```sh
sudo apt update && sudo apt install -y openjdk-11-jdk
java -version
```

## Step 2: Install Jenkins on Master
```sh
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
echo "deb http://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt update
sudo apt install -y jenkins
sudo systemctl enable --now jenkins
```

### Check Jenkins Status:
```sh
sudo systemctl status jenkins
```

### Retrieve Initial Admin Password:
```sh
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

## Step 3: Configure SSH Access for Passwordless Authentication

### On Jenkins Master:
```sh
ssh-keygen -t rsa -b 4096 -N "" -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub
```

### On Jenkins Slave:
```sh
echo "<PASTE_MASTER_PUBLIC_KEY>" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### Test SSH Access:
```sh
ssh <slave-user>@<slave-ip>
```

## Step 4: Install Jenkins Agent on Slave

### On Slave Node:
```sh
sudo useradd -m jenkins
sudo passwd jenkins
su - jenkins
mkdir ~/.jenkins
exit
```

## Step 5: Connect Slave to Master

### On Jenkins Master UI:
1. Go to **Manage Jenkins** → **Manage Nodes and Clouds** → **New Node**
2. Provide a **Node Name**, select **Permanent Agent**, and configure:
   - **Remote Root Directory**: `/home/jenkins`
   - **Launch method**: "Launch agents via SSH"
   - **Host**: `<Slave-IP>`
   - **Credentials**: Add SSH private key of master
   - **Save & Launch**

### Verify Connection
- Check logs to see if the node is connected successfully.
- Run a test job on the slave node.

## Conclusion
You have successfully set up a Jenkins Master-Slave architecture on AWS EC2 instances with passwordless SSH authentication.

