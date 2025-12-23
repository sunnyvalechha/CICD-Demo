How Build and Deployment Process works before CICD pipeline of Jenkins?

Step 1: One or multiple Developement team start planning and working on application.
Step 2: They push their code to Central repository (Github)
Step 3: After all the peices of code pushed into github we will Build the code, as the code is raw code and not in usable format that means we have to create the .exe of code.
Step 4: Test the code, when the build is created we will test the code if the build code is working fine.
Step 5: Deploy, Deploy application to server

Above all step done manually and might get error in any stage so the developers have to follow all process again.

Challenges of manual work:
1. Dev write new code every day and follow the same process every day.
2. Multiple environment can be there like Test and Prod.
3. Time consuming
4. Error prone

Multiple Environments present:
1. Dev Environment: where developers write code.
2. QA / Test Environment: 
3. UAT Environment: User Acceptance testing Environment
4. Pilot Environment or Pre production Environment: Performance test, how application is performing.
5. Production Environment: Go Live

**Now, Entry of DevOps**

Development + Operations 


==========================================
CICD Demo:

* Aws t2.medium, Aws instance - Jenkins-server (setup jenkins on this machine)
* Setup Aws cli, Git, Maven on same jenkins machine

#Aws CLI:
apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version

#Jenkins
sudo apt install fontconfig openjdk-21-jre -y
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins

# Maven:
apt-get install wget -y
cd /opt
wget https://dlcdn.apache.org/maven/maven-3/3.9.12/binaries/apache-maven-3.9.12-bin.tar.gz
tar -xvzf apache-maven-3.8.1-bin.tar.gz	# rm -rf tar file
vi /etc/profile.d/maven.sh
	export M2_HOME=/opt/apache-maven-3.9.12
	export PATH=$PATH:$M2_HOME/bin
sudo chmod 644 /etc/profile.d/maven.sh
source /etc/profile.d/maven.sh
bash -c 'echo "source /etc/profile.d/maven.sh" >> /etc/bash.bashrc'
exec bash
mvn --version

# Install git
yum install git -y

# Assign shell to jenkins user
vi /etc/passwd
change shell from /bin/false to /bin/bash

# Install Docker
apt-get install docker.io -y
systemctl start docker
systemctl enable docker
sudo groupadd docker
sudo usermod -aG docker jenkins
sudo chmod 777 /var/run/docker.sock

vi /etc/sudoers
jenkins ALL=(ALL) NOPASSWD: ALL

Jenkins pass - 92300ce0b05a4d3383aea6e1b41bb5f5

# Plugins install in jenkins
* Maven Integration 
* Docker pipeline

# Eks cluster setup from different machine. (jump machine is t2.micro)

# Docker hub sign in

# Setup Helm in Jenkins server machine:
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4
chmod 700 get_helm.sh
./get_helm.sh

# Attach Iam role to jenkins machine which created for eks cluster. (create admin access role & attach to both machines)

# No connectivity between jenkins & kubernetes:
- copy k8s config file from eks jump machine to jenkins server
- Login to eks jump machine > login as root > ls -lrtha > .kube dir > copy content of config file >
- In Jenkins server cd /var/lib/jenkins > mkdir .kube > vi config > paste the content
- Verify >> helm list
 
Solution:
EKS - kubectl edit configmap aws-auth -n kube-system

apiVersion: v1
data:
  mapRoles: |
    - rolearn: arn:aws:iam::913024026100:role/eksctl-sunny-eks-nodegroup-ng-e77b-NodeInstanceRole-kMnX9OPb3N9J
      groups:
      - system:bootstrappers
      - system:nodes
      username: system:node:{{EC2PrivateDNSName}}
    # Jenkins CI role for EKS access
    - rolearn: arn:aws:iam::913024026100:role/jenkins-eks-integration
      username: jenkins
      groups:
        - system:masters
kind: ConfigMap
metadata:
  creationTimestamp: "2025-12-23T07:01:36Z"
  name: aws-auth
  namespace: kube-system
  resourceVersion: "23364"
  uid: ffd05215-56c3-473a-8478-d7a05a6cca65

Jenkins machine - jenkins aws eks update-kubeconfig --region ap-south-1 --name sunny-eks

su - jenkins | helm list

helm repo list
helm repo add stable https://charts.helm.sh/stable
helm search repo stable | less
helm pull stable/mysql
ls -lrth > mysql-1.6.9.tgz 	# chart downloaded as package
tar xvf mysql-1.6.9.tgz
cd mysql
ls -lrth templates		# all this list will created once run this chart
helm package mysql
==========================================
Dev: Project development team
Ops: Build & Deploy team




