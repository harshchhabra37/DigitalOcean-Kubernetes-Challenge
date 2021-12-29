# DigitalOcean-Kubernetes-Challenge

Attempting Challenge: <b>Deploy an internal container registry</b> for DigitalOcean's Kubernetes program.<br>

## Installing Prerequisites 
We have to first install Chocolately to be able to install kubectl (Reference: https://chocolatey.org/install#individual):

    Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

Now needed things to install, using Chocolately:

    choco install kubernetes-helm
    choco install kubernetes-cli
    kubectl version --client

## Steps

<h3>Step 1</h3>: We have to first install `doctl` - DigitalOcean CLI tool (Reference: https://docs.digitalocean.com/reference/doctl/how-to/install)

<h3>Step 2</h3>: Now we have to create a Kubernetes cluster
         (We can do it either by using command line doctl or through DigitalOcean web platform)

```
doctl kubernetes cluster create do-k8s-challenge
```
>OUTPUT

![image](https://user-images.githubusercontent.com/60788180/147603136-38d7b69b-863c-40b1-a02b-62642eb2ce31.png)

It take few minutes in creating the cluster, after that we get this output.
After 5 to 10 minutes our cluster is created successfully and we get this output.

![image](https://user-images.githubusercontent.com/60788180/147611042-951029bf-2c47-429c-8374-c49970813dcd.png)

Through DigitalOcean web platform we can set Cluster settings according to us.

![image](https://user-images.githubusercontent.com/60788180/147611113-bcb6375f-a84d-4b8a-b210-30b02057bd12.png)

We can also download Config file.

![image](https://user-images.githubusercontent.com/60788180/147611150-30aabaf5-76ca-4a4e-ac37-2cedb4955199.png)


<h3>Step 3</h3>: Harbor installation steps

Let's start the challenge. We are using Bitnami [helm chart](https://bitnami.com/stack/harbor/helm) to deploy Harbor

Add bitnami repo to Helm

```
helm repo add bitnami https://charts.bitnami.com/bitnami
```
> OUTPUT
```
PS C:\Users\hp\Desktop\DigitalOcean-Kubernetes-Challenge> helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
```
Before installing helm chart we need to edit the yaml file, so we have to create a yaml file first.
```
helm show values bitnami/harbor > harbor-values.yaml
```

open `harbor-values.yaml` in a editor

change the external URL value to `externalURL: https://hub.chhabraharsh37.me` 

![image](https://user-images.githubusercontent.com/60788180/147604327-95e4008c-0065-41c6-a41f-284bc8387059.png)


set admin password `harborAdminPassword: "<YOUR PASSWORD>", we can set it as "Harbor123"`

![image](https://user-images.githubusercontent.com/60788180/147607174-9d7ae317-14d7-4125-a2a4-71ad876b4b6b.png)

and commonName `commonName: 'hub.chhabraharsh37.me'`

![image](https://user-images.githubusercontent.com/60788180/147604482-6c3426d7-85ab-49a1-8ae8-af8abd7c712b.png)



Install Harbor via helm
```
helm install harbor bitnami/harbor --values harbor-values.yaml -n harbor --create-namespace
```
>OUTPUT


![image](https://user-images.githubusercontent.com/60788180/147611541-2afd56d3-3661-4ff0-a29f-2537df25a036.png)

Check pods status - (everything is running now)

![image](https://user-images.githubusercontent.com/60788180/147611706-dbfbe3ba-5c34-4185-bdd5-d002207578c7.png)

Get External-IP of Harbor loadbalancer

![image](https://user-images.githubusercontent.com/60788180/147611781-ebb89c9b-bba9-4c5e-93de-e244043c921e.png)


we have to use the external IP to login to Harbor
- UserName: admin
- Password: <PASSWORD_entered_on_yaml>

![image](https://user-images.githubusercontent.com/60788180/147614249-97ea6b1f-c9ab-41f1-a4ff-67a6abbd5b2e.png)

![image](https://user-images.githubusercontent.com/60788180/147614855-47dcc7cf-b378-4d97-8014-7bbc3f764ebb.png)


Create a new project and make it public, so pulling image doesn't need authentication.

![image](https://user-images.githubusercontent.com/60788180/147613986-cf1c3a43-8119-4189-9267-aee3ed48bf49.png)

Now let create a new Docker image using this command

        docker run httpd:alpine

Now lets try to push a container(httpd) image to Harbor

![image](https://user-images.githubusercontent.com/60788180/147612078-43535349-78c9-40c2-a25b-8789ef0aeb4a.png)

first we need to tag our container, so we do this 

```
docker tag httpd:alpine hub.chhabraharsh37.me/k8s/https:latest
```
>OUTPUT
```
docker tag httpd:alpine hub.chhabraharsh37.me/k8s/https:latest
```
![image](https://user-images.githubusercontent.com/60788180/147612222-fff9c319-0a10-4faa-a43f-4c19fac2d2b9.png)


Login to `hub.chhabraharsh37.me` on Docker

*NOTE: if DNS is not resolved edit hosts file and add the IP of Harbor*

```
docker login hub.chhabraharsh37.me
```
>OUTPUT
```
PS C:\Users\hp\Desktop\DigitalOcean-Kubernetes-Challenge> docker login hub.chhabraharsh37.me
Username: admin
Password: 
Login Succeeded
```
![image](https://user-images.githubusercontent.com/60788180/147616238-5bde388e-5a02-484b-ae85-735c131cab57.png)


Now we can push our image to our hub

```docker
docker push hub.chhabraharsh37.me/k8s/https
```
>OUTPUT (Successfully pushed)
```
PS C:\Users\hp\Desktop\DigitalOcean-Kubernetes-Challenge> docker push hub.chhabraharsh37.me/k8s/https
Using default tag: latest
The push refers to repository [hub.chhabraharsh37.me/k8s/https]
59bf1c3509f3: Pushed
085d9bf42ab2: Pushed
907dfa98a03e: Pushed
242407d61b32: Pushed
aa5e87b8047c: Pushed
f1ff32cc6322: Pushed
latest: digest: sha256:c7b8040505e2e63eafc82d37148b687ff488bf6d25fc24c8bf01d71f5b457531 size: 1572
```
Image available at Harbor 

![image](https://user-images.githubusercontent.com/60788180/147614929-08cb8a9e-0ed0-498a-ad33-bed86dda03ff.png)


Lets deploy this app using [httpd-deployment.yaml](httpd-deployment.yaml) file

```
kubectl apply -f httpd-deployemnt.yaml
```
>OUTPUT
```
PS C:\Users\hp\Desktop\DigitalOcean-Kubernetes-Challenge> kubectl apply -f httpd-deployment.yaml
service/httpd-service unchanged
deployment.apps/httpd-deployment created
PS C:\Users\hp\Desktop\DigitalOcean-Kubernetes-Challenge> kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
httpd-deployment-6559f66ffd-4pqlm   1/1     Running   0          9m58s
```
In this project we created our Kubernetes internal container registry (Harbor) and pushed our local image to container registry. 
Finally our K8s deployment was able to pull the image from Harbor and spin-up a pod successfully.

## Helpful links 
1. Installing Chocolatey: https://chocolatey.org/install#individual
2. Kubectl: https://kubernetes.io/docs/tasks/tools/#kubectl
3. Doctl: https://docs.digitalocean.com/reference/doctl/how-to/install
4. Helm: https://helm.sh/docs/intro/install
5. Harbor: https://goharbor.io
6. Docker Desktop: https://docs.docker.com/desktop
7. Link to this challenge!: https://www.digitalocean.com/community/pages/kubernetes-challenge
