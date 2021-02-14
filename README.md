# Udacity-DevOps-Capstone-Rolling-Deployment
This is the capstone project for Udacity DevOps Nanodegree. Here, we will be using rolling deployment to deploy Docker images on Kubernetes Cluster with Jenkins. Follow along the README file to run these files on Jenkins pipeline.

## Creating AWS (Amazon Web Services) account
For running Kuberntes on Amazon EKS, and running Jenkins pipeline on Amazon EC2 instance, you will need to create a AWS account. Follow this link to create your own AWS account: [Sign-up for Amazon Web Services](https://portal.aws.amazon.com/billing/signup#/start)

## Requirements for running Jenkins Pipeline
For running Jenkins Pipeline, you will need to install Jenkins on your local machine, or (in this case as I used) an Amazon EC2 instance. 
1. (Optional) Create your own EC2 instance, by login into Amazon AWS console. Here's a series of steps required to set up your own EC2 instance: https://portal.aws.amazon.com/billing/signup#/start 
Use a Ubuntu 18 with T2.micro since this is included in Free Tier from Amazon.
Once launched, create a security group for the vm. In the left sidebar, under Network and Security, select "Security Groups." Under name, use: 'jenkins', description: "basic Jenkins security group," VPC should have the default one used. Click Add Rule: Custom TCP Rule, Protocol: TCP, Port Range 8080, Source 0.0.0.0/0 Then add the SSH rule: Protocol: SSH, Port range: 22, From source, use the dropdown and select "My IP."

2. Now connect to your EC2 istance by following steps, written on following link: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Connect-using-EC2-Instance-Connect.html

3. Now once connected, we've to install Jenkins on your EC2 instance (or on your local machine if you want to deploy your Jenkins pipeline on your local machine). Here are commands for installing Jenkins on Ubuntu EC2 instance: 
```
$ apt update
$ apt upgrade
$ apt install default-jdk
$ wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
$ sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
$ sudo apt update
$ sudo apt install jenkins
$ sudo systemctl status jenkins
```
After running the above commands, you should status of Jenkins instance, running on your EC2 instance or local machine.

4. Jenkins by defualt runs on port 8080. So, Visit Jenkins on its default port, 8080, with your server IP address or domain name included like this: http://your_server_ip_or_domain:8080 (localhoast:8080 for local machine)

5. Now run the following command, and copy the password generated. Put this password in Jenkins login screen.
```
$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

6. The next screen gives you the choice of installing recommended plugins, or selecting specific plugins - choose the Install suggested plugins option, which quickly begins the installation process. 

7. When installation is complete, you are prompted to set up the first admin user. Create the admin user and make note of both the user and password to use in the future.

## Install BlueOcean on Jenkins
1. "Blue Ocean" and other required plugins need to be installed. Logged in as an admin, go to the top left, click 'Jenkins', then 'manage Jenkins', and select 'Manage Plugins'.

2. Use the "Available" tab, filter by "Blue Ocean," select the first option ("BlueOcean aggregator") and install without a restart.

3. Filter once again for "pipeline-aws" and install, this time selecting "Download now and install after restart."

4. Once all plugins are installed, Jenkins will restart. If it hasn't restarted, run the following in the VM:
```
sudo systemctl restart jenkins
```
5. Verify everything is working for Blue Ocean by logging in. An "Open Blue Ocean" link should show up in the sidebar. Click it, and it will take you to the "Blue Ocean" screen, where we will have to add a project.

## Learning Objectives

- Spinning up & Creating Jenkins
- Configuring Jenkins with the needed plugins and batteries (docker+kubectl)
- Pushing Docker images to an private ECR Instance (fast and easy)
- Pull Docker images from the private ECR Instance
- Deploy those images through k8s

### Application

The application is a simple static site running in an nginx container exposing
port 80.

## Deploy the Infrastructure

To begin with the CI/CD Pipeline, you need to have a EC2 Instance with Jenkins,
aws-cli, Docker & kubectl deployed.

The first parts of the infrastructure is the network (Public & Private Subnets,
VPC and the ControlPlane) through
`./create_stack.sh UdacityCapStone-Infrastructure network.yml network-params.json`.

After the network stack is up and running deploy the K8S cluster (EKS) through
`./create_stack.sh UdacityCapStone-K8S cluster.yml cluster-params.json`
This may take up to 15mins.

To enable nodes in the cluster, where the pods get deployed through Kubernetes,
run the k8s-workers script through
`./create_stack.sh UdacityCapStone-K8S-Workers cluster-workers.yml cluster-params.json`
The nodes only join, if they get the correct permissions assigned. This has to
be done within the K8S Cluster through a ConfigMap. Run
`kubectl apply -f aws-auth-cm.yaml`. Make sure to have your kubectl already
configured with the EKS
(`aws eks --region region-code update-kubeconfig --name cluster_name`). To make
sure everything was successfull run `kubectl get svc` which should show the
kubernetes services.

Additionally, check `kubectl get nodes` which should show the desired state of
worker nodes.

Another component is missing: The ECR (Container Registry). Run
`./create_stack UdacityCapStone-ECR container-registry.yml container-registry-params.json`
which spins up a container registry.

To have a proper deployment double-check the parameters in environment in the
`Jenkinsfile`

## Rolling update

I've chosen the rolling update within kubernetes, which means it updates the
pods without any downtime. The rolling update updates one pod after another. If
the first pod isn't successful K8s rolls it back and spins up another instance
of the previous pod. Therefore no downtime.

Indicated is the rolling update within the `k8s/deployment.yml`-file through the
deploy strategy type. The CI/CD pipeline (`Jenkinsfile`) performs an rolling
update command to update the image to the latest image created during the build.

## Thanks and have a good day ! - Prabhjot Singh Bharaj 