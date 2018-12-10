# PetStore on Kubernetes

## TLDN

## Pre-requisite
Tested on 
* CentOS 7.5
* Docker 18.06.1
* kubectl 1.13


## MySQL Deployment & Service

This deployment is based on 
* [mddb/myjpetstore]: https://hub.docker.com/r/mddb/myjpetstore/
  * Based on centos7 & iBatis JPetStore
* [mysql]: https://hub.docker.com/_/mysql/ 5.6 


Create MySQL PV and PVC
  ```
  $ kubectl apply -f jpetstore-mysql-pv.yaml
  ```

Create MySQL deployment:
  ```
  $ kubectl apply -f jpetstore-mysql.yaml
  ```

Create the MySQL Service
  ```
  $ kubectl apply -f jpetstore-mysql-svc.yaml
  ```

Connect to MySQL (using the service's endpoint IP) to create the initial DB
  ```
  $ kubectl run -i --rm --image=mysql:5.6 --restart=Never mysql-jpet-client -- mysql -h mysql-jpet-service -D jpetstore -u jpetstore -pjpetstore < jpetstore.sql
  ```

### JPetStore
Create MySQL PV and PVC
Deploy JPetStore App and Service:
  ```
  $ kubectl apply -f jpetstore-app.yaml
  $ kubectl apply -f jpetstore-svc.yaml
  ```

### kube-dns or coredns not working?

After deploying, check the endpoint IP of the MySQL Service:
  ```
  $ kubectl describe service mysql-jpet-service |grep Endpoints
  Endpoints:         172.17.0.9:3306
  
And edit jpetstore-app.yaml accordingly.
