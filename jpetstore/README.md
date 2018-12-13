# PetStore on Kubernetes

## TLDN
  ```
  git clone https://github.com/mbicudo/kube-admin/
  cd jpetstore
  kubectl apply -f jpetstore-mysql-pv.yaml
  kubectl apply -f jpetstore-mysql.yaml
  kubectl apply -f jpetstore-mysql-svc.yaml
  kubectl run -i --rm --image=mysql:5.6 --restart=Never mysql-jpet-client -- mysql -h mysql-jpet-service -D jpetstore -u jpetstore -pjpetstore < jpetstore.sql
  kubectl apply -f jpetstore-app.yaml
  kubectl apply -f jpetstore-svc.yaml
  ```


## Pre-requisite
Tested on:
* CentOS 7.5
* Docker 18.06.1
* kubectl 1.13

### Assigning Pods and Node Labels

I am using node labels to assign pods to different nodes, but you can obviously assign the app and mysql to the same node.
Just make sure your nodes have the labels for mysql and jpetstore.
  ```
  kubectl label nodes yournode.hostname  app=jpetstore
  kubectl label nodes yournode.hostname  run=mysql
  ```
To remove labels:
  ```
  kubectl label nodes yournode.hostname  app-
  kubectl label nodes yournode.hostname  run-
  ```
  
## MySQL Deployment & Service

This deployment is based on 
* [mddb/myjpetstore]
  * Based on centos7 & iBatis JPetStore
* [mysql] 5.6 

[mddb/myjpetstore]: https://hub.docker.com/r/mddb/myjpetstore/
[mysql]: https://hub.docker.com/_/mysql/

Create MySQL PV and PVC:
  ```
  kubectl apply -f jpetstore-mysql-pv.yaml
  ```

Create MySQL deployment:
  ```
  kubectl apply -f jpetstore-mysql.yaml
  ```

Create the MySQL Service
  ```
  kubectl apply -f jpetstore-mysql-svc.yaml
  ```

Connect to MySQL Service to confirm it is working fine:
  ```
  $ kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-jpet-client -- mysql -h mysql-jpet-service -u jpetstore -pjpetstore
  mysql>
  ```

You shoudl be fine if you see 'mysql>'. Then create the initial DB:
  ```
  kubectl run -i --rm --image=mysql:5.6 --restart=Never mysql-jpet-client -- mysql -h mysql-jpet-service -D jpetstore -u jpetstore -pjpetstore < jpetstore.sql
  ```

### JPetStore
Deploy JPetStore App and Service:
  ```
  kubectl apply -f jpetstore-app.yaml
  kubectl apply -f jpetstore-svc.yaml
  ```

### kube-dns or coredns not working?

After deploying, check the endpoint IP of the MySQL Service:
  ```
  kubectl describe service mysql-jpet-service |grep Endpoints
  Endpoints:         172.17.0.9:3306
  ```

And edit jpetstore-app.yaml accordingly. By doing this, you will by pass DNS and use the IP directly.

## Loadbalance (optional)

The JPetSore service can be easily acccessed as ClusterIP or NodePort.

If you want to scale your pods and have them load-balanced, then you will need either a cloud LB or a kubernetes ingress controller. The official and most common ingress controller is nginx. 

First, install nginx ingress-controller.Attention: there are at least two nginx ingress controllers: one maintained by nginx, another by kubernetes community. To install the nginx maintained, see [nginx install instructions]. You will see that you can install in two flavours, Deployment or DaemonSet. Use the latter, and you should get 1 nginx pod at every node.

```
[root@node1]# kubectl get pods -n nginx-ingress -o wide
NAME                  READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
nginx-ingress-nwzcz   1/1     Running   0          9h    10.244.0.27   n0.mlab.loc   <none>           <none>
nginx-ingress-s6nps   1/1     Running   1          9h    10.244.3.18   n2.mlab.loc   <none>           <none>
nginx-ingress-tdggg   1/1     Running   1          9h    10.244.2.12   n1.mlab.loc   <none>           <none>
```

After confirming that nginx is working as expected (see install notes), add your jpetstore ingress:
  ```
  kubectl apply -f jpetstore-ingress.yaml
  ```

Ingress rules are based on hostname (like apache virtualHosts). To access it, you have to point jpetstore.mlab.loc to ANY of your nodes (if you are using nginx DaemonSet). Do this editting /etc/host or passing a resolve parameter to curl:
  ```
  curl --resolve jpetstore.mlab.loc:192.168.2.228 http://jpetstore.mlab.loc/
  ```

[nginx install instructions]: https://github.com/nginxinc/kubernetes-ingress/tree/master/deployments
