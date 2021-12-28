# DigitalOcean-Kubernetes-Challenge

Attempting Challenge: Deploy an internal container registry for DigitalOcean's Kubernetes program.<br>
Start Date: 18-11-2021

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%201.svg)](https://www.digitalocean.com/?refcode=a9cc9b42d247&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)


# DigitalOcean Kubernetes Challenge
This repo is for [DigitalOcean Kubernetes Challenge](https://www.digitalocean.com/community/pages/kubernetes-challenge)

I choose **Deploy an internal container registry**

 

## Challenge runbook

Step 1: Install `doctl` - DigitalOcean CLI tool

Step: 2: Create a Kubernetes cluster

```
doctl kubernetes cluster create do-k8s-challenge
```
>OUTPUT

![image](https://user-images.githubusercontent.com/60788180/147603136-38d7b69b-863c-40b1-a02b-62642eb2ce31.png)
It take few minutes in creating the cluster, after that we get this output.
After 5 to 10 minutes our cluster is created successfully and we get this output 
![image](https://user-images.githubusercontent.com/60788180/147606766-645664f9-deb0-4da9-a3ef-2ebbdc8b0b88.png)



Step 3: Harbor installation steps

Our challenge starts here. I'm using Bitnami [helm chart](https://bitnami.com/stack/harbor/helm) to deploy Harbor

Add bitnami repo to Helm

```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
> OUTPUT
```
PS C:\Users\hp\Desktop\DigitalOcean-Kubernetes-Challenge> helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
```
Before installing helm chart we need to edit the yaml file, so create a yaml file 
```
helm show values bitnami/harbor > harbor-values.yaml
```

open `harbor-values.yaml` in a editor

change the exterlname URL value to `externalURL: https://hub.chhabraharsh37.me` 
![image](https://user-images.githubusercontent.com/60788180/147604327-95e4008c-0065-41c6-a41f-284bc8387059.png)


set admin password `harborAdminPassword: "<YOUR PASSWORD>" or set as default "Harbor123"`
![image](https://user-images.githubusercontent.com/60788180/147607174-9d7ae317-14d7-4125-a2a4-71ad876b4b6b.png)

and commonName `commonName: 'hub.chhabraharsh37.me'`
![image](https://user-images.githubusercontent.com/60788180/147604482-6c3426d7-85ab-49a1-8ae8-af8abd7c712b.png)



Install Harbor via helm
```
helm install harbor bitnami/harbor --values harbor-values.yaml -n harbor --create-namespace
```
>OUTPUT


![image](https://user-images.githubusercontent.com/60788180/147607269-f679326f-96dd-4a1c-87ad-4a1f552beab9.png)

Check pods status - (everything is running now)

![image](https://user-images.githubusercontent.com/60788180/147607348-7ce05517-e76a-427e-ab63-4f34ff5e832f.png)


Get External-IP of Harbor loadbalancer
![image](https://user-images.githubusercontent.com/60788180/147604962-01e9cd93-2478-406d-bf65-89f9e8d65703.png)


use the external IP to login to Harbor
- UserName: admin
- Password: <PASSWORD_entered_on_yaml>

![Harbor Login](images/Harbor_1_login.png)

Create a new project and make it public, so pulling image doesn't need authentication.

![Create new project](images/Harbor_2_NewProject.png)

Now lets try to push a container(httpd) image to Harbor

![image](https://user-images.githubusercontent.com/60788180/147609152-08195b3d-9da3-43ee-8359-c451209537b6.png)


first we need to tag our container 

```
docker tag httpd:alpine hub.chhabraharsh37.me/k8s/https:latest
```
>OUTPUT
```
PS C:\LAB\harbor> docker tag httpd:2.4.52-alpine hub.jagan-sekaran.me/k8s/https:latest
PS C:\LAB\harbor>
PS C:\LAB\harbor> docker images
![image](https://user-images.githubusercontent.com/60788180/147609502-02bb1da1-919d-4595-bb6d-928a9eb1c56d.png)

```
Login to `hub.jagan-sekaran.me` on Docker

*NOTE: if DNS is not resolved edit hosts file and add the IP of Harbor*

```
docker login hub.jagan-sekaran.me
```
>OUTPUT
```
PS C:\Users\hp\Desktop\DigitalOcean-Kubernetes-Challenge>> docker login hub.chhabraharsh37.me
Username: admin
Password: 
Login Succeeded
```
Now we can push our image to our hub

