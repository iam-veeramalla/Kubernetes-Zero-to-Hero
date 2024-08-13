#Kubernetes Multi Node Cluster Set-up Step By Step
# https://kind.sigs.k8s.io/docs/user/quick-start/#installation realease notes/ look for realease version node according today


#Installation of Kubectl on Ubuntu
sudo apt update
#Install Kubectl CLI
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
#The Curl command is a CLI tool for transferring data from a URL
curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
#Download the Kubectl checksum file
echo "$(cat kubectl.sha256) kubectl" | sha256sum --check
#The above command should return the below output
kubectl: OK
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
#Open a new terminal to load the Kubectl CLI tool and then run the below command
kubectl version --client

Kind create cluster --image kindest/node:v1.29.4@sha256:3abb816a5b1061fb15c6e9e60856ec40d56b7b52bcea5f5f1350bc6e2320b6f8 --name cka setup-1  (whatever your wanted give you can)
kubectl cluster-info --context kind-cka-setup--1   (If there is multiple cluster setup then we have to choose this and go inside the cluster setup)

