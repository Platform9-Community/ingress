# Jenkins deployment on Platform9 Managed Kubernetes Freedom Plan

Jenkins is a well known, widely adopted Continuous Integration platform in enterprises. 

## Deployment of Jenkins on one of your Platform9 Managed Kubernetes clusters
Here we are going to deploy Jenkins on top of platform9 managed kubernetes freedom tier. The Jenkins docker image provided here has Openjdk8, Maven, Go and NodeJS preinstalled with commonly used plugins. It can be further customized once Jenkins is up and running. Ensure that bare metal cluster has metallb load-balancer pre configured before deploying jenkins. It is also required for exposing the NodeJS app that will get deployed with Jenkins pipeline at the end.

## Jenkins configuration
Before deploying Jenkins label one node with a specific key value pair so that Jenkins pod gets scheduled on this node. 

Select the node with enough resources for Jenkins to run on. Label it in the following manner. 

```bash
$ kubectl label nodes <node-name> jenkins=allow
```

## Store Dockerhub and GitHub credentials
Storing DockerHub username and password of your DockerHub repository into Jenkins is required so that pipeline can upload images to your dockerhub repository. Additionally store your GitHub credentials to pull the cicd repository. To do this in Jenkins UI, on the home page click Credentials -> Jenkins -> Global credentials (unrestricted) -> Add credentials. Fill out the dockerhub username, password and set ID as 'dockerhub'.

![add-cred-dhub](https://github.com/KoolKubernetes/cicd/blob/master/jenkins/images/add_cred_dhub.png)