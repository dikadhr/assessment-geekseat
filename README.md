# assessment-geekseat
A. Simple Statefull Aplication (Wordpress + Mysql) (application flow attached app flow.png)
B. 
C. How I do the implementation (how-to notes)

i used GKE instance to implement the test and pull images container from my dockerhub https://hub.docker.com/u/dikadhr
1. encode the secrets and save to secrets.yaml
### secrets.yaml
```
---
apiVersion: v1
kind: Secret
metadata:
  name: webapp-secrets
type: Opaque
data:
  mysql_password: YWRtaW5hZG1pbgo=
  admin_password: YWRtaW5hZG1pbgo=

```
2. create db(i used mysql:5.7), declare to webapp-db-deployment.yaml and create webapp service as a LoadBalancer

### webapp-db-deployment.yaml
PersistantVolume
```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-a-disk
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

```
Deployment
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: db-a-deployment
  labels:
    app: db-a
spec:
  replicas: 1
  selector:
    matchLabels:
      app: db-a
  template:
    metadata:
      labels:
        app: db-a
    spec:
      containers:
        - name: db-a
          image: dikadhr/mysql:assessment
          ports:
            - containerPort: 3306
          volumeMounts:
            - mountPath: "/var/lib/mysql"
              subPath: "mysql"
              name: mysqldb
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: webapp-secrets
                  key: mysql_password
      volumes:
        - name: mysqldb
          persistentVolumeClaim:
            claimName: db-a-disk

```
Service
```
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-db-service
spec:
  selector:
    app: db-a
  ports:
  - protocol: TCP
    port: 3306
    targetPort: 3306

```
3. create webapp(i used wordpress:5.2) and declare to webapp-deployment.yaml

###webapp-deployment.yaml
PersistantVolume
```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: webapp-disk
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

```
Deployment
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webapp-deployment
  labels:
    app: webapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webapp
  template:
    metadata:
      labels:
        app: webapp
    spec:
      containers:
        - name: webapp
          image: dikadhr/wordpress:assessment
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: "/var/www/html"
              name: webapp-disk
          env:
            - name: WORDPRESS_DB_HOST
              value: webapp-db-service
            - name: WORDPRESS_DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: webapp-secrets
                  key: mysql_password
      volumes:
        - name: webapp-disk
          persistentVolumeClaim:
            claimName: webapp-disk

```
Service
```
---
apiVersion: v1
kind: Service
metadata:
  name: webapp-service
spec:
  selector:
    app: webapp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer

```
4. Developers can easily and securely access the cluster with credential (assessment-test-geekseat-.......json)
```
gcloud container clusters get-credentials <GKE cluster name> --region us-central1 --project assessment-test-geekseat

```
