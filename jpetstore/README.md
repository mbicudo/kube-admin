# PetStore on Kubernetes
=================

## Pre-requisites
* Tested on 
* CentOS 7.5
* Docker 18.06.1
* minikube 0.30
* kubectl-1.12.2-0


## MySQL Deployment & Service

This deployment is based on 
* [mddb/myjpetstore] (https://hub.docker.com/r/mddb/myjpetstore/)
  * Based on centos7 & iBatis JPetStore
* [mysql] 5.6 (https://hub.docker.com/_/mysql/)


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

After deploying, check the endpoint IP of the MySQL Service:
  ```
  $ kubectl describe service mysql-jpet-service |grep Endpoints
  Endpoints:         172.17.0.9:3306
  ```

Connect to MySQL (using the service's endpoint IP) to create the initial DB
  ```
  $ kubectl run -i --rm --image=mysql:5.6 --restart=Never mysql-jpet-client -- mysql -h 172.17.0.9 -u jpetstore -pjpetstore < jpetstore.sql
  ```

- JPetStore
Create MySQL PV and PVC
Deploy JPetStore App and Service:
  ```
  $ kubectl apply -f jpetstore-app.yaml
  $ kubectl apply -f jpetstore-svc.yaml
  ```

I should have used DNS for the mysql but since kube-dns got wrecked and I stil have to fix it I had to modify YAML file with MySQL Service ClusterIP and patch or modify the jpetstore deployment YAML accordingly.
  ```
  $ 
  ```

