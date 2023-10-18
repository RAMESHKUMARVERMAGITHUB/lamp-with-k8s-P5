#  ✅ Project Title: Deploy lamp(php with mysql) project Using Kubernetes
this project create simple lamp application that run using  Kubernetes Cluster


## ✅ Step 1: Set Up Kubernetes Cluster
Create an Kubernetes cluster if you don't already have one.You can create Kubernetes cluster in aws eks, azure aks or your vm(using minikube)

## ✅ Step 2: Containerize Your Application
 Dockerize your application by creating a Dockerfile that defines how your application should be packaged as a container. Build and push the Docker image to docker hub or docker registry.

## ✅ Step 3: Configure application yml

 ☑️ Here configure yml file  for `` service `` and `` deploymnet ``
 
 ```
apiVersion: v1
kind: Service
metadata:
  name: lamp-app-service
spec:
  ports:
    - port: 80
  selector:
    app: lamp-app
  type: LoadBalancer

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: lamp-app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: lamp-app
  template:
    metadata:
      labels:
        app: lamp-app
    spec:
      containers:
        - name: lamp-app
          image: shihab24/lamp:latest
          volumeMounts:
            - name: storage
              mountPath: /var/www/html/uploads
          ports:
            - containerPort: 80
      volumes:
       - name: storage
         persistentVolumeClaim:
           claimName: lamp-pvc

```
☑️ Configure volume mount for so that applaction upload file can store in single path  .

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: lamp-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /temp/storage/lamp/uploads
---

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: lamp-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual

```

## ✅ Step 4: Mysql using k8s cluster

 ☑️ Create Mysql configuration
 
 ```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

---

apiVersion: v1
kind: Service
metadata:
  name: mysql-service
spec:
  ports:
    - port: 3306
  selector:
    app: mysql
  type: ClusterIP

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:latest
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: root
          ports:
            - containerPort: 3306

```
☑️ Configure the sync policy to automatically apply changes when the Git repository is updated.

## ✅ Step 5: Run Application
☑️ First run mysql using `` kubectl apply  kubernetes/mysql.yml `` command
☑️ Run volume `` kubectl apply  kubernetes/persistentvolumeclaim.yaml `` command
☑️ Then run application using `` kubectl apply  kubernetes/app.yml `` command

## ✅ Step 6: Update application
☑️ Authenticate you docker hub or docker registry .
☑️ Build and push your Docker image to your docker hub or registry.
☑️ Update build image tag this in deployment kubernetes yaml.
☑️ Then run application using `` kubectl apply  kubernetes/app.yml `` command

## ✅ Conclsion: 
We used  dockerized application on an Kubernetes cluster. This gave us a strong base for our deployment. After that, we update application by update image tag and push this in docker registry and also edit kubernetes yml for this image tag and run this yml.
