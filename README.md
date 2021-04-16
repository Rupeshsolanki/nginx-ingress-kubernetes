# Install Ingress in kubernetes using helm charts #

## Add ingress-nginx helm ##

```sh
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```
## Install chart ##
```helm install [RELEASE_NAME] ingress-nginx/ingress-nginx
```
## Check the created resources ##
``` kubectl get all ```

## Deploy Main Application ##
``` vim nginx-deploy-main.yaml```
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx-deploy-main
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx-main
  template:
    metadata:
      labels:
        run: nginx-main
    spec:
      containers:
      - image: nginx
        name: nginx
```
## Deploy  Blue application ##
``` vim nginx-deploy-blue ```
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx-deploy-blue
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx-blue
  template:
    metadata:
      labels:
        run: nginx-blue
    spec:
      volumes:
      - name: webdata
        emptyDir: {}
      initContainers:
      - name: web-content
        image: busybox
        volumeMounts:
        - name: webdata
          mountPath: "/webdata"
        command: ["/bin/sh", "-c", 'echo "<h1>I am <font color=blue>BLUE</font></h1>" > /webdata/index.html']
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: webdata
          mountPath: "/usr/share/nginx/html"
```
## Deploy Green Application ##
``` vim nginx-deploy-blue ```
```sh
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    run: nginx
  name: nginx-deploy-green
spec:
  replicas: 1
  selector:
    matchLabels:
      run: nginx-green
  template:
    metadata:
      labels:
        run: nginx-green
    spec:
      volumes:
      - name: webdata
        emptyDir: {}
      initContainers:
      - name: web-content
        image: busybox
        volumeMounts:
        - name: webdata
          mountPath: "/webdata"
        command: ["/bin/sh", "-c", 'echo "<h1>I am <font color=green>GREEN</font></h1>" > /webdata/index.html']
      containers:
      - image: nginx
        name: nginx
        volumeMounts:
        - name: webdata
          mountPath: "/usr/share/nginx/html"
```
## Create POD ##
```kubectl create -f nginx-deploy-main.yaml -f nginx-deploy-blue.yaml -f nginx-deploy-green.yaml```

## Created Service for main, blue and green pods ##

```sh
kubectl expose deploy nginx-deploy-main --port 80

kubectl expose deploy nginx-deploy-blue --port 80

kubectl expose deploy nginx-deploy-green --port 80

```
## Check Resources are created or not ##
``` kubectl get all```


# Creating Ingress #


## Ingress PATH based routing ##
```sh
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  name: ingress-resource-3
spec:
  rules:
  - host: nginx.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-deploy-main
            port:
              number: 80
      - path: /blue
        pathType: Prefix
        backend:
          service:
            name: nginx-deploy-blue
            port:
              number: 80
      - path: /green
        pathType: Prefix
        backend:
          service:
            name: nginx-deploy-green
            port:
              number: 80
```
## Ingress HOST based routing ##
```sh
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-resource-2
spec:
  rules:
  - host: nginx.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-deploy-main
            port:
              number: 80
  - host: blue.nginx.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-deploy-blue
            port:
              number: 80
  - host: green.nginx.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-deploy-green
            port:
              number: 80
```

## Check Ingress ##
``` kubectl get ing ```

``` kubectl describe <ingress-name> ```