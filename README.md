# twoge_eks
Deploy twoge web application with Postgres DB using Kubernetes EKS cluster

### Git Repo
```
https://github.com/chandradeoarya/twoge/tree/k8s
```
#### Git clone Repo
```
 git clone -b k8s git@github.com:chandradeoarya/twoge.git
```

### Create Dockerfile
```
FROM python:alpine

ENV  DB_USER=twoge
ENV  DB_PASSWORD=twogedb1234
ENV  DB_HOST=twoge-database-server
ENV  DB_PORT=5432
ENV  DB_DATABASE=twoge_db

RUN apk update && \
    apk add --no-cache build-base libffi-dev openssl-dev
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
EXPOSE 8080
CMD python app.py
```

#### Build Dockerfile
```
docker build -t asanni2022/twoge-eks .
```

### Create Namespase.yml file
```
apiVersion: v1
kind: Namespace
metadata:
  name: twoge-ns
  labels:
    name: twoge-ns
```

### Create Resourcequota.yml file
```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: twoge-quota
  namespace: twoge-ns
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

#### Create Configmap.yml file
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-configmap
data:
  database_url: "postgres-service"
  database_port: "5432"
```

### Create secrets.tml file
```
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secret
type: Opaque
data:
  db_name: dHdvZ2VfZGI=
  username: dHdvZ2U=
  password: dHdvZ2VkYjEyMzQ=
```

### Create twoge web deployment yml file
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: twoge-dep
  labels:
    app: twoge-k8s
spec:
  selector:
    matchLabels:
      app: twoge-k8s
  replicas: 1
  template:
    metadata:
      labels:
        app: twoge-k8s
    spec:
      containers:
        - name: twoge-container
          image: asanni2022/twoge-eks
          ports:
            - containerPort: 8080
          env:
            - name: DB_DATABASE
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: db_name
            - name: DB_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: username
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-secret
                  key: password
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: postgres-configmap
                  key: database_url
            - name: DB_PORT
              valueFrom:
                configMapKeyRef:
                  name: postgres-configmap
                  key: database_port
```

### Create twoge web service yml file
```
apiVersion: v1
kind: Service
metadata:
  name: twoge-k8s-service
spec:
  type: NodePort
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30036
  selector:
    app: twoge-k8s
```

### Create twoge Postgres deployment yml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres
        env:
          - name: POSTGRES_DB
            valueFrom:
              secretKeyRef:
                name: postgres-secret
                key: db_name
          - name: POSTGRES_USER
            valueFrom:
              secretKeyRef:
                name: postgres-secret
                key: username
          - name: POSTGRES_PASSWORD
            valueFrom:
              secretKeyRef:
                name: postgres-secret
                key: password
```

### create twoge Postgres service yml file
```
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  type: ClusterIP
  ports:
  - port: 5432
```

### Run Commands
```
kubectl apply -f Namespace.yaml                                 # create namespace
kubectl apply -f ResourceQuota.yml                              # create resourcequota
kubectl describe namespaces twoge-ns                            # describe twoge-ns namespace
kubectl apply -f . --namespace twoge-ns                         # attach all resource to namespace
kubectl get resourcequotas --namespace twoge-ns                 # apply resourcequota twoge-ns resourcequota
kubectl config set-context --current --namespace=twoge-ns       # Set namespace preference
kubectl get pods --namespace twoge-ns                           # get pods associated with twoge-ns namespace

```

###  Validate
```
minikube service twoge-k8s-service -n twoge-ns --url
```
![Ns](https://github.com/asanni2022/twoge_eks/assets/104282577/0ca7d64d-cf23-4d0d-add7-2d3d55d7de50)


![twoge2](https://github.com/asanni2022/twoge_eks/assets/104282577/eb5f5fe7-afca-4ae7-8cec-321125499434)

![twoge1](https://github.com/asanni2022/twoge_eks/assets/104282577/64f7cf4c-3304-4248-ba8b-27b3b76efaac)

```
kubectl describe pods/twoge-dep-77fdb57d87-kz6m5
```
![twoge describe](https://github.com/asanni2022/twoge_eks/assets/104282577/f38cc66e-8b22-48a4-93d8-94166fdc2cdc)







