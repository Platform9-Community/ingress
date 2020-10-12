# NGINX ingress controller deployment on Platform9 Managed Kubernetes Freedom Plan
 
Here we are going to deploy the well known NGINX ingress controller on top of the Platform9 Managed Kubernetes freedom tier. This deployment is going to require a PMK/FT 4.3 cluster (Kubernetes 1.16+) with metallb loadbalancer on baremetal servers/VMs running on Ubuntu 18.04 or Centos 7.6.

# Deployment

Clone the ingress repository in KoolKubernetes organization and apply it on your cluster. 
```bash
$ git clone https://github.com/KoolKubernetes/ingress.git


$ kubectl apply -f ingress/nginx/nginx-v0.33.0.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created
```

Validate the ingress controller pods are running. Notice that NGINX ingress has got the load balancer IP from metallb.
```bash
$ kubectl get ns ingress-nginx
NAME            STATUS   AGE
ingress-nginx   Active   80s

$ kubectl get all -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-9dt62        0/1     Completed   0          82s
pod/ingress-nginx-admission-patch-fzlvg         0/1     Completed   0          82s
pod/ingress-nginx-controller-58f68f5ccc-c9gmt   1/1     Running     0          93s


NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)                      AGE
service/ingress-nginx-controller             LoadBalancer   10.21.21.80    10.128.231.209   80:32755/TCP,443:32569/TCP   93s
service/ingress-nginx-controller-admission   ClusterIP      10.21.109.77   <none>           443/TCP                      93s


NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           93s

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-58f68f5ccc   1         1         1       93s



NAME                                       COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   1/1           6s         92s
job.batch/ingress-nginx-admission-patch    1/1           8s         92s
```
Clone the cicd repo to test the ingress controller with our favorite pf9reactapp.

```bash
$ git clone https://github.com/KoolKubernetes/cicd.git
```

Deploy the app with ingress resource to leverage NGINX ingress controller.
```bash
$ kubectl apply -f cicd/jenkins/webapp01/k8s/ingress-nginx-app.yaml
namespace/p9-react-app created
deployment.apps/p9-react-app created
service/p9-react-app created
ingress.networking.k8s.io/pf9app-routing created
```

Validate the app is running in p9-react-app namespace with an ingress resource.

```bash
$ kubectl get all -n p9-react-app
NAME                                READY   STATUS    RESTARTS   AGE
pod/p9-react-app-67d4df6bb6-5f944   1/1     Running   0          17s
pod/p9-react-app-67d4df6bb6-t8p45   1/1     Running   0          17s


NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/p9-react-app   ClusterIP   10.21.39.141   <none>        80/TCP    18s


NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/p9-react-app   2/2     2            2           18s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/p9-react-app-67d4df6bb6   2         2         2       18s

$ kubectl get ingress -n p9-react-app
NAME             HOSTS                    ADDRESS         PORTS   AGE
pf9app-routing   pf9app.platform9.horse   10.128.229.21   80      40m
```

The ingress resource gets created as shown below.
```bash
$ kubectl describe ingress -n p9-react-app
Name:             pf9app-routing
Namespace:        p9-react-app
Address:          10.128.229.21
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host                    Path  Backends
  ----                    ----  --------
  pf9app.platform9.horse
                          /   p9-react-app:80 (10.20.20.22:80,10.20.96.22:80)
Annotations:
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"networking.k8s.io/v1beta1","kind":"Ingress","metadata":{"annotations":{},"name":"pf9app-routing","namespace":"p9-react-app"},"spec":{"rules":[{"host":"pf9app.platform9.horse","http":{"paths":[{"backend":{"serviceName":"p9-react-app","servicePort":80},"path":"/"}]}}]}}

Events:
  Type    Reason  Age                  From                      Message
  ----    ------  ----                 ----                      -------
  Normal  CREATE  45m                  nginx-ingress-controller  Ingress p9-react-app/pf9app-routing

```

The Service p9-react-app within Kubernetes namespace p9-react-app is now exposed outside via hostname pf9app.platform9.horse. One should make a local hosts file entry to visit this URL from the LAN. Modify the Manifest to set the hostname of your choice before deploying the app manifest on the cluster.


![add-cred-dhub](https://github.com/KoolKubernetes/ingress/blob/master/nginx/images/app-ingress.png)



Ingress controller should be running on a node which has enough resources. It can be further configured to add authentication, TLS termination so on and so forth.



# Reference

https://kubernetes.github.io/ingress-nginx/examples/

https://kubernetes.github.io/ingress-nginx/
