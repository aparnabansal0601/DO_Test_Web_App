
# Tutorial: Deploying a Static Website on DigitalOcean Managed Kubernetes Cluster


Containerize & Deploy a static website on DigitalOcean's Managed Kubernetes cluster. This repository provides a step-by-step tutorial, complete with code examples and configuration files, to help you get started with containerization and Kubernetes on DigitalOcean

# Prerequisite
 - Any machine connected to internet.
 - Docker installed and running. If not already, refer [here](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
 - DockerHub id. if not already, please refer [here](https://docs.docker.com/accounts/create-account/)
 - git client to clone the repository.
 - static website files. Feel free to use to sample in this repo (https://github.com/aparnabansal0601/DO_Test_Web_App.git)
 - DigitalOcean login id with relevant permissions.


# Step 01 - Static Website Containerization with Docker

 - **Clone this repository locally**

```shell
git clone https://github.com/vinayvtiwari/simple-web-app-do.git
cd DO_Test_Web_App
```
output:
 ```shell
 apbansal@Aparna-MacBook-Pro ~ % git clone https://github.com/aparnabansal0601/DO_Test_Web_App.git
Cloning into 'DO_Test_Web_App'...
remote: Enumerating objects: 34, done.
remote: Counting objects: 100% (34/34), done.
remote: Compressing objects: 100% (30/30), done.
remote: Total 34 (delta 7), reused 0 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (34/34), 132.57 KiB | 1.56 MiB/s, done.
Resolving deltas: 100% (7/7), done.
apbansal@Aparna-MacBook-Pro ~ % 
apbansal@Aparna-MacBook-Pro ~ % cd simple-web-app-do/
apbansal@Aparna-MacBook-Pro DO_Test_Web_App % ls
deployment.yaml	Dockerfile	index.html	LICENSE		README.md	service.yaml
apbansal@Aparna-MacBook-Pro ~ %

```
- **cd into the website-files directory and examine the content.**

```shell
apbansal@Aparna-MacBook-Pro DO_Test_Web_App % ls -l
total 112
-rw-r--r--@ 1 jigyashu  staff    341 18 Dec 12:23 deployment.yaml
-rw-r--r--  1 jigyashu  staff    137 18 Dec 09:12 Dockerfile
-rw-r--r--  1 jigyashu  staff    750 18 Dec 09:12 index.html
-rw-r--r--  1 jigyashu  staff  35149 18 Dec 09:12 LICENSE
-rw-r--r--  1 jigyashu  staff   2823 18 Dec 09:12 README.md
-rw-r--r--@ 1 jigyashu  staff    287 18 Dec 12:11 service.yaml
```
Except Dockerfile , all the remaining files are website used to serve static content. Lets understand Dockerfile.

Dockerfile --> A Dockerfile is a text file that contains instructions for building a Docker image. It's a blueprint for creating a Docker image, specifying  the base image, dependencies, and commands to run.

```shell
apbansal@Aparna-MacBook-Pro DO_Test_Web_App % cat Dockerfile
FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY index.html /usr/share/nginx/html/
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

- **Understanding the lines**  
This Dockerfile creates a lightweight Nginx container that serves your index.html webpage.
It removes Nginx’s default page, copies your webpage inside, opens port 80, and then runs Nginx so your site can be accessed in a browser.


- **Building our own image**  
Docker build command is used to build the image. To tag the image We use -t. We did not specify the docker file, because the .(dot) at the end means, the Dockerfile is in the current folder. we using the name in the format of **[DOCKERHUB_USERNAME/IMAGE_NAME_YOU_WANT_TO_KEEP]**. It is required because next we will also push it to the dockerhub  so that it can be dowloaded and utilized within the kubernetes environment.

```shell
docker buildx build \
  --platform linux/amd64 \
  -t apbansal0601/tech-intro:latest \
  --push .
```

output:
```shell
apbansal@Aparna-MacBook-Pro ~ % DO_Test_Web_App % docker buildx build \
  --platform linux/amd64 \
  -t apbansal0601/tech-intro:latest \
  --push .
[+] Building 9.0s (9/9) FINISHED                                                                                                                                                                                                docker:desktop-linux
 => [internal] load build definition from Dockerfile                                                                                                                                                                                            0.0s
 => => transferring dockerfile: 176B                                                                                                                                                                                                            0.0s
 => [internal] load metadata for docker.io/library/nginx:alpine                                                                                                                                                                                 1.1s
 => [internal] load .dockerignore                                                                                                                                                                                                               0.0s
 => => transferring context: 2B                                                                                                                                                                                                                 0.0s
 => [1/3] FROM docker.io/library/nginx:alpine@sha256:fd9f8ce722ab13edb2e47ebdd16b843939280457bf1567a6cd155203f9ce98d8                                                                                                                           0.0s
 => => resolve docker.io/library/nginx:alpine@sha256:fd9f8ce722ab13edb2e47ebdd16b843939280457bf1567a6cd155203f9ce98d8                                                                                                                           0.0s
 => [internal] load build context                                                                                                                                                                                                               0.0s
 => => transferring context: 32B                                                                                                                                                                                                                0.0s
 => CACHED [2/3] RUN rm -rf /usr/share/nginx/html/*                                                                                                                                                                                             0.0s
 => CACHED [3/3] COPY index.html /usr/share/nginx/html/                                                                                                                                                                                         0.0s
 => exporting to image                                                                                                                                                                                                                          7.7s
 => => exporting layers                                                                                                                                                                                                                         0.0s
 => => exporting manifest sha256:8fff8c3507e486c902f6e974ac627a0af2957ac6f73aa6ec167f94f866298bcd                                                                                                                                               0.0s
 => => exporting config sha256:5c94e85aeb53267dfa9dc6fd5eca272eb64b4bd891915ac7510ea0a3d2984875                                                                                                                                                 0.0s
 => => exporting attestation manifest sha256:498bbd80a869d9f8c924fa63101af2378ccf9991f45587e9319fa6f2467e6382                                                                                                                                   0.0s
 => => exporting manifest list sha256:da29a824ac87d32f9569f1b9660683da59a9b973cd3686fe22464509ff758142                                                                                                                                          0.0s
 => => naming to docker.io/apbansal0601/tech-intro:latest                                                                                                                                                                                       0.0s
 => => pushing layers                                                                                                                                                                                                                           4.0s
 => => pushing manifest for docker.io/apbansal0601/tech-intro:latest@sha256:da29a824ac87d32f9569f1b9660683da59a9b973cd3686fe22464509ff758142                                                                                                    3.6s
 => [auth] apbansal0601/tech-intro:pull,push token for registry-1.docker.io                                                                                                                                                                     0.0s

```
- **Check whether the image is build using the below docker command**

```shell
sudo docker images
```
output:
```shell

IMAGE                            ID             DISK USAGE   CONTENT SIZE   EXTRA
apbansal0601/do-web-app:latest   d7f03d266fae       81.1MB           23MB        
apbansal0601/tech-intro:latest   da29a824ac87         23MB           23MB   
```

# Step 02 -  Publishing Your Docker Image to Docker Hub
- **Authenticate to DockerHub using docker id**

```shell
docker login -u <dockerhub username>
```

output:
```shell
apbansal@Aparna-MacBook-Pro ~ % docker login -u apbansal0601

i Info → A Personal Access Token (PAT) can be used instead.
         To create a PAT, visit https://app.docker.com/settings


Password:

WARNING! Your credentials are stored unencrypted in '/home/apbansal0601/.docker/config.json'.
Configure a credential helper to remove this warning. See
https://docs.docker.com/go/credential-store/

Login Succeeded
```

- **Push the image to the DockerHub repository**

```shell
docker push apbansal0601/tech-intro:latest
```

output:
```shell
apbansal@Aparna-MacBook-Pro ~ % docker push apbansal0601/tech-intro:latest
The push refers to repository [docker.io/apbansal0601/tech-intro]
a96335b001f0: Mounted from apbansal0601/tech-intro
d1e3e4dd1aaa: Mounted from apbansal0601/tech-intro
ccc5aac17fc4: Mounted from apbansal0601/tech-intro
8d83f6b79143: Mounted from apbansal0601/tech-intro
9e3c6e8c1e25: Mounted from apbansal0601/tech-intro
9aad78ecf380: Mounted from apbansal0601/tech-intro
bd903131a05e: Mounted from apbansal0601/tech-intro
ea680fbff095: Mounted from apbansal0601/tech-intro
latest: digest: sha256:59e17eea6ca66392f78ce0dedf2edaa6ff35e049aca3c699d9bc1c5566ded450 size: 1988
apbansal@Aparna-MacBook-Pro ~ %$
```
The image should now be visible in the docker hub under your account. As the access level is public, it can now be utilized by anyone.

<img width="1717" height="601" alt="image" src="https://github.com/user-attachments/assets/6e4f1950-c1ea-4068-8fe2-109c04e01be0" />



# Step 03 - Creating a Managed Digital Ocean Kubernetes Cluster(DOKS)

- **Login to DigitalOcean [here](https://cloud.digitalocean.com/login)**

- **Click Create Button on top right and select Kubernetes.**
  
  ![image](https://github.com/user-attachments/assets/122a739d-9302-4d02-9de7-4e125cc2dd58)

- **Select the latest version. Select appropriate datacenter region & keep the VPC Setting as is.**

  ![image](https://github.com/user-attachments/assets/726e0271-49b4-4648-b0ee-d0c7bc225252)

- Under Cluster Capacity, Provide a Node pool name and change number of Nodes to 3. There is "Set Node Pool to Autoscale" which will increase the number on Nodes, based on utilization. There is also a high avaialbility option for controlplane, which will run control plane components in HA mode. Because this is not production we will it unchecked.

  ![image](https://github.com/user-attachments/assets/51660cce-cd07-4fcb-a926-86cc25b1c61d)

- **Under Finalize section, provide a name for your cluster or leave it at default. Click Create Cluster.**
  
 <img width="1728" height="1044" alt="image" src="https://github.com/user-attachments/assets/51438be9-e1b2-4060-b464-4d9886fb1f61" />


- **After some time (5-10min), Along with the Cluster, you will also see your two nodes also up and running.**

  <img width="1728" height="1049" alt="image" src="https://github.com/user-attachments/assets/2264dfb9-5236-4059-b49b-847269e91934" />


- **Under the section "Connecting and managing this cluster". make a note of the below command, where the long string at the end is the cluster id. For security reason, i have provided a wrong id.** 

```shell
doctl kubernetes cluster kubeconfig save <cluster id from the Digital Ocean Control Panel>
```

output:
```shell
doctl kubernetes cluster kubeconfig save 12345678-1234-5678-9012-123456789012
```
   <img width="1727" height="818" alt="image" src="https://github.com/user-attachments/assets/c4562d9b-4914-4f72-a0f4-985c6323e150" />


**Kubernetes Cluster is now ready to run your own container images.**

# Step 04 - Install & configure Digital Ocean CLI(DOCTL), API Token & Kubernetes CLI (Kubectl)

- **This step is required to connect the local linux instance to DOs kubernetes cluster. Run the below command.**

```shell
brew install doctl
```
output:
```shell
apbansal@Aparna-MacBook-Pro ~ % brew install doctl

==> Fetching downloads for: doctl
✔︎ Bottle Manifest doctl (1.148.0)                                                                                                                                                                                       [Downloaded    7.4KB/  7.4KB]
✔︎ Bottle doctl (1.148.0)                                                                                                                                                                                                [Downloaded    8.5MB/  8.5MB]
==> Pouring doctl--1.148.0.arm64_sequoia.bottle.tar.gz
🍺  /opt/homebrew/Cellar/doctl/1.148.0: 10 files, 25.5MB
==> Running `brew cleanup doctl`...
Disable this behaviour by setting `HOMEBREW_NO_INSTALL_CLEANUP=1`.
Hide these hints with `HOMEBREW_NO_ENV_HINTS=1` (see `man brew`).
==> Caveats
zsh completions have been installed to:
  /opt/homebrew/share/zsh/site-functions
apbansal@Aparna-MacBook-Pro ~ % 
apbansal@Aparna-MacBook-Pro ~ % 
apbansal@Aparna-MacBook-Pro ~ % doctl version

doctl version 1.148.0-release
```


- **create a API Token from the DigitalOcean Control Panel. On the left hand panel, select API**

  ![image](https://github.com/user-attachments/assets/9e38c30d-d1ae-4b4d-a56b-1bd3bf3c214c)

- **Click on generate new token.**

  ![image](https://github.com/user-attachments/assets/1d202f72-d1a0-4c7e-a2b1-0b7764d21373)

- **Give a meaningful name. Also select the expiry period. For simplicty. under scope, select full access. Click generate token**

  ![image](https://github.com/user-attachments/assets/67a1b4aa-29df-441b-9171-da8193eba729)

- **Copy the token and keep it in a notepad. We will need this to authenticate doctl.**

  ![image](https://github.com/user-attachments/assets/9ecbe2fc-538c-4ba9-9f8d-43bcc053a50e)

- **Use the API token to grant doctl access to your DigitalOcean account. Pass in the token string when prompted by doctl auth init, and give this authentication context a name.**

```shell
doctl auth init
```
output:
```shell
apbansal@Aparna-MacBook-Pro DO_Test_Web_App % doctl auth init
Please authenticate doctl for use with your DigitalOcean account. You can generate a token in the control panel at https://cloud.digitalocean.com/account/api/tokens

❯ Enter your access token:  ●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●

Validating token... ✔

apbansal@Aparna-MacBook-Pro DO_Test_Web_App % doctl account get
User Email                    Team       Droplet Limit    Email Verified    User UUID                               Status
aparnabansal0601@gmail.com    My Team    3                true              65cb7959-f5d3-4720-a3d3-3113f74768da    active
```
- **To install kubectl**

```shell
brew install kubectl
```
output:
```shell
apbansal@Aparna-MacBook-Pro ~ % brew install kubectl

==> Fetching downloads for: kubernetes-cli
✔︎ Bottle Manifest kubernetes-cli (1.35.0)                                                                                                                                                                               [Downloaded    7.5KB/  7.5KB]
✔︎ Bottle kubernetes-cli (1.35.0)                                                                                                                                                                                        [Downloaded   17.9MB/ 17.9MB]
==> Pouring kubernetes-cli--1.35.0.arm64_sequoia.bottle.tar.gz
🍺  /opt/homebrew/Cellar/kubernetes-cli/1.35.0: 260 files, 61.9MB
==> Running `brew cleanup kubernetes-cli`...
Disable this behaviour by setting `HOMEBREW_NO_INSTALL_CLEANUP=1`.
Hide these hints with `HOMEBREW_NO_ENV_HINTS=1` (see `man brew`).
==> Caveats
zsh completions have been installed to:
  /opt/homebrew/share/zsh/site-functions
apbansal@Aparna-MacBook-Pro ~ % 
apbansal@Aparna-MacBook-Pro ~ % 
apbansal@Aparna-MacBook-Pro ~ % kubectl version --client

Client Version: v1.35.0
Kustomize Version: v5.7.1
```

- **Set the kubectl context to point to DOs cluster. Please note that the clusterid i am using is wrong. Please use the one you see on the control manager. Also run the get nodes command as shown below, the output will confirm that you are now connected.**

```shell
doctl kubernetes cluster kubeconfig save <your cluster id>
```
output:
```shell  
apbansal@Aparna-MacBook-Pro ~ % doctl kubernetes cluster kubeconfig save 12345678-1234-5678-9012-123456789012
Notice: Adding cluster credentials to kubeconfig file found in "/home/aparnabansal0601@gmail.com/.kube/config"

```
# Step 05 - Kubernetes Fundamentals: Deploying Pods, ReplicaSets, Declarative definitions, Deployments, and LoadBalancer Service Type

- **Before proceeding further, lets understand some basic concept of Kubernetes.**

PODs: In Kubernetes, a Pod is the smallest and most basic execution unit that can be created and managed. It's a logical host for one or more containers. The container image that we created and uploaded to docker hub will run in the POD.

ReplicaSets: A Kubernetes ReplicaSet (RS) ensures a specified number of replicas (identical Pods) are running at any given time. It's a way to maintain a desired state for an application. This makes sure that the define number of pods are always running.

Declarative Definitions: Declarative definitions in Kubernetes often use YAML files to define resources.  These YAML files specify the desired state of resources like Pods, ReplicaSets, Deployments, and more.

Deployment: Kubernetes Deployments manage the rollout of new versions or configurations of an application. They provide a declarative way to describe the desired state of an application. Within a deployment, we can define the PODs, Replica Set, and many more.

Load Balancer Service Type: In Kubernetes, a LoadBalancer service type exposes a service to the outside world by provisioning a load balancer from a cloud provider. In our case, it is DigitalOcean.

Apply manifests:
```shell
kubectl apply -f deployment.yaml

```
```shell
 kubectl get pods -n webapp
```
output:
```shell
NAME                               READY   STATUS    RESTARTS   AGE
tech-intro-page-6b94c665b7-nklq8   1/1     Running   0          26h
tech-intro-page-6b94c665b7-pgtn6   1/1     Running   0          26h
```
As the PODs are running, lets now make it public facing by deploying a load balancer service. In the below output, we are asking DigitalOcean to deploy a load balancer. This load balancer will serve the application running on PODs to the outside world.After running the command, if you notice, the External IP section for the LoadBalancer Service type is pending. It will appear once the load balancer is deployed within the DigitalOcean Account. 

```shell
kubectl apply -f service.yaml
```
```shell
kubectl get svc
```
output:
```shell
NAMESPACE     NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
default       kubernetes           ClusterIP      10.109.0.1      <none>        443/TCP                  28h
kube-system   cilium-agent         ClusterIP      None            <none>        9964/TCP                 28h
kube-system   hubble-metrics       ClusterIP      None            <none>        9965/TCP                 28h
kube-system   hubble-peer          ClusterIP      10.109.18.188   <none>        443/TCP                  28h
kube-system   hubble-relay         ClusterIP      10.109.21.186   <none>        80/TCP                   28h
kube-system   hubble-ui            ClusterIP      10.109.17.225   <none>        80/TCP                   28h
kube-system   kube-dns             ClusterIP      10.109.0.10     <none>        53/UDP,53/TCP,9153/TCP   28h
webapp        tech-intro-service   LoadBalancer   10.109.20.138   24.199.67.5   80:30793/TCP             27h
```


- **Using the Load Balancer IP, you can now browse the Website we deployed on the POD.**

  <img width="1721" height="446" alt="image" src="https://github.com/user-attachments/assets/98d07c32-a2fa-4b47-83dd-313d456fa024" />


  



