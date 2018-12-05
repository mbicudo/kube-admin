
Create MySQL PV and PVC
$ kubectl apply -f jpetstore-mysql-pv.yaml

Create MySQL deployment:
$ kubectl apply -f jpetstore-mysql.yaml

Create the MySQL Service
$ kubectl apply -f jpetstore-mysql-svc.yaml

After deploying, check the endpoint IP of the MySQL Service:
$ kubectl describe service mysql-jpet-service |grep Endpoints
Endpoints:         172.17.0.9:3306

Connect to MySQL (using the service's endpoint IP) to create the initial DB
$ kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-jpet-client -- mysql -h 172.17.0.9 -u jpetstore -pjpetstore

Deploy JPetStore App and Service:
$ kubectl apply -f jpetstore-app.yaml


