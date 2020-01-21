---
layout: post
title:  " Rancher : One place for all Kubernetes Clusters"
date:   2019-05-03
categories: Rancher Kubernetes Clusters
---



[Source](https://medium.com/faun/rancher-one-place-for-all-kubernetes-clusters-51586d72858a "Permalink to Rancher : One place for all Kubernetes Clusters - Faun")

# Rancher : One place for all Kubernetes Clusters - Faun

![Saiyam Pathak][1]

Rancher is open-source software for delivering Kubernetes-as-a-Service.

Managing kubernetes clusters running in different cloud providers was never an easy task until Rancher came. So what exactly is Rancher , Rancher is an open source platform where you can create the cluster in different clouds or import an existing one as well. Today I will tell you how to spin up a kubernetes cluster in Google cloud, AWS Cloud and how to import a cluster from Oracle Cloud. All these three clusters you will be able to see and manage from one place itself which is nothing but Rancher Dashboard. Rancher have wide variety of tools and the company is coming up with more and more cool open source projects including the k3os.io that they recently launched . I will show you the creation of kubernetes cluster from rancher and how easy monitoring and deployments can be done via Rancher Dashboard.

Apart from Deploying the cluster to different cloud vendors , Rancher in march 2019 launched there own RKE(Rancher Kubernetes Engine) which is an extremely simple, lightning fast Kubernetes installer that works everywhere. So it eliminates the pain of installing the kubernetes cluster on baremetal servers or VM's and it provides lot of customization flexibility as well.

RKE installation : In this I will explain how to install rancher kuberntes cluster on 3 VM's. So the first thing you need is three machines which you can use to spin up RKE cluster. I am taking 3 EC2 Instances with ubuntu18.04 as the boot image. So I have a separate VM from where I will be performing all the installations to these three nodes where one will be the master and other two will be worker nodes. I have followed the [official documentation][2] for installation but for some steps are tweaked a bit for more easy stuff.  
\- Step 1: Downloading the RKE binary  
wget   
\- Step 2: mv rke_linux-amd64 rke  
\- Step 3: export PATH=/home/cloud_user/rke:$PATH  
\- Step 4: By this point I already have three ubuntu EC2 machines provisioned with docker installed (make sure you run "**usermod -aG docker ubuntu**" to make docker accessible by ubuntu user as well) on them and the private key file which I used while creating those instances . What you need to do is in your current VM create a file, copy the contents of the key, change key permissions and try to login to one of your EC2 instance .

As you can see I have also done the same. Now you run the following command :   
**rke config — name cluster.yml** (you can use ./rke if path not set). As soon as you hit enter it will start asking you different parameter values based on which it will create the cluster.yml file. These parameters are basically the nodes characteristics that you define and rke automatically creates cluster.yml for you. You can create that by yourself as well by following the documentation from Rancher.

Above are the parameters I passed and based on that it generated cluster.yml file. Basically its the three nodes configuration and some other cluster related config which I have chosen as default .

Above is how the Node will look like in cluster.yml file based on the parameters we passed. Now that you have the cluster.yml file ready you can move on to the next step.

\- Step 5: Run "rke up" to make the cluster up (it assumes that you have a file cluster.yml at same location) or if you have file other than cluster.yml than you can run :   
**rke up — config abc.yml**

THATS IT …!!! You will see the cluster spinning up and displaying various INFO Logs for doing all the work to spin up the cluster and connecting the nodes together. This also lets you see what is happening behind the scenes so that you can feel all the steps for Cluster creation. If not then just look our for "building kubernetes cluster successfully".

Few Logs

After this rke also creates a kubeconfig file which you can use to interact with the cluster ([install kubectl][3] before that) with the name 'kube_config_cluster_yml' and if you used any other name for the yml file then it will be 'kube_config_test_yml', so you can use this config file to interact with the cluster.

All setup

R**ancher Installation** : Now I will show you how you can install Rancher and create/import clusters from Rancher Dashboard. I will be using the same VM which I used for RKE installation. I will be running Rancher as a docker container on port 80 .

Command : **docker run -d — restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher**

DONE..!! Rancher will be up and running now.   
Login to Rancher  and set password,URl to use.

[1]: https://miro.medium.com/fit/c/96/96/0*nyqdP5wSFJOkdRbv.jpg
[2]: https://rancher.com/docs/rke/latest/en/installation/
[3]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux

  
