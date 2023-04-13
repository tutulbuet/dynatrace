# dynatrace Presentation - Part 1
#Assumptions:
- Security is not a concern for this demo (Hence didn't consider creating a custom VPC, subnets-private/public, routing table, internet gatway, NAT gatway etc.)
- Assumed any bug fix for sock-shop is not scoped in this demo as only task is to host the application in kubernetes environment and enable Dynatrace monitoring 


## Launch EC2 Instance 
Login to AWS console and go to EC2 service > Launch Instance

I selected Ubuntu as the OS, chose t2.xlarge as it comes with 4vCPU and 16GB memory. As I will be using minikube, I need 2vCPU as minimum and for sock-shop application recommended is 4 vCPU. 

Then allowed all traffic in the security group as security is not what we are concern here in this demo lab. Selected 40GB of volume. Named it as "myminikube" and launched the instance

## Update apt for ubuntu
sudo apt update

## Install docker
sudo apt -y install docker.io

## Download and Install kubectl for Linux

 ### Install kubectl binary with curl on Linux
Download the latest release with the command:

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"


#### Download the kubectl checksum file (optional but I did it to verify to avoid any issue):

curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

#### Validate the kubectl binary against the checksum file:


If valid, the output is:
kubectl: OK


### Install kubectl

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version --client --output=yaml    



## Download and install  Minikube for Linux
To install the latest minikube stable release on x86-64 Linux using binary download:

curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube


### Starting cluster 
Note: I used docker driver.  Also as I used 4 CPUs and used memory 7815. Minikube needs minimum 2 vCPU. However Sock Shop's recommendation is 4 vCPUs and memory 8192, however that is for VM. docker driver doesn't allow more than 7815. 

minikube start - memory 7815 - cpu 4 --force 

### verify minikube
minikube version
minikube status
kubectl get nodes
<img width="783" alt="Screen Shot 2023-04-13 at 12 36 48 pm" src="https://user-images.githubusercontent.com/8380801/231637077-a0d43c63-7a76-4a49-ace7-b54d0b61c3a0.png">



## Deploy the application sock-shop
#### Clone the microservices-demo repo
git clone https://github.com/microservices-demo/microservices-demo
cd microservices-demo/deploy/kubernetes
Note: check the deployment files and apply the appropriate deployment.

kubectl -f apply complete-demo.yaml

### Verify the deployment
kubectl get all -n sock-shop

<img width="1509" alt="Screen Shot 2023-04-13 at 12 57 16 pm" src="https://user-images.githubusercontent.com/8380801/231636693-be1b3e70-393c-4fc8-8711-ab2aaecab917.png"><img width="1505" alt="Screen Shot 2023-04-13 at 12 57 38 pm" src="https://user-images.githubusercontent.com/8380801/231636699-516df3b1-3889-4b2b-ab2d-cdec22262bf7.png">

Expose the front end service in minikube to find out the external ip and the nodeport to access the application
<img width="1106" alt="Screen Shot 2023-04-13 at 1 40 26 pm" src="https://user-images.githubusercontent.com/8380801/231643423-2e961586-0d6c-4146-91ea-61ffe16dda24.png">

Check the URL to see if we get HTTP code 200 OK to confirm the web service is accessible
<img width="1344" alt="Screen Shot 2023-04-13 at 1 46 15 pm" src="https://user-images.githubusercontent.com/8380801/231644309-32d00557-5536-4f21-869c-7b02fb2d911c.png">

  
## Deploy Dynatrace 

login to dynatrace trial account. 
- go to "Deploy Dynatrace" (the green button at the top right corner) 
- go to Start Installation 
- Select Kubernetes 
- Fill out the form with name and generate the tokens. 

Note: Once the form is filled, the custom resource file named dynakube.yaml will be generated which I downloaded in my local machine and subsequently copied it in the EC2 for later use. This file is useful as it automatically populated the tokens and also updated the apiURL with my unique dynatrace ID jyi05749.

Execute the following commands in the terminal:

kubectl create namespace dynatrace

kubectl apply -f https://github.com/Dynatrace/dynatrace-operator/releases/download/v0.10.4/kubernetes.yaml

kubectl -n dynatrace wait pod --for=condition=ready --selector=app.kubernetes.io/name=dynatrace-operator,app.kubernetes.io/component=webhook --timeout=300s

kubectl apply -f dynakube.yaml

Note: I could have saved the deployment file to my local machine and run the deployment there. However, as I used the same namespace as the deployment file, I just applied it from the link.

#Start using Dynatrace for monitoring
<img width="1503" alt="Screen Shot 2023-04-13 at 1 28 16 pm" src="https://user-images.githubusercontent.com/8380801/231641742-8faaa7b8-9c86-4093-aa0c-453c571a88ed.png">




# Dynatrace Presentation - Part 2

Steps that I would take if I see a dip in quality:
1. Raise a ticket to track issue of the quality dip 

2. Gather data: First, I would collect data on the dip in quality, including the number of issues identified, the types of issues, and any other relevant information.

3. Analyze the data: Next, I would analyze the data to identify any patterns or trends that may be contributing to the dip in quality.

4. Identify root causes: Based on my analysis, I would identify any potential root causes that could be contributing to the dip in quality.

4. Implement solutions: Once I have identified any potential root causes, I would then work with the appropriate teams to develop and implement solutions to address the issues.

5. Monitor results: Finally, I would track and monitor the results of the solutions implemented to ensure that the quality of the production environment is improving.

6. Communicate findings: I would then report these findings in my reports, including the data I collected, the analysis I conducted, the root causes identified, the solutions implemented, and the results achieved. This will allow my team and other stakeholders to stay informed and up-to-date on the progress of the quality improvement efforts.



