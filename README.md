### Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 2» Баранов Сергей

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к двум приложениям снаружи кластера по разным путям.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).

2. Установленный локальный kubectl.

3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Инструкция](https://microk8s.io/docs/getting-started) по установке MicroK8S.

2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Service.

3. [Описание](https://kubernetes.io/docs/concepts/services-networking/ingress/) Ingress.

4. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment приложений backend и frontend

1. Создать Deployment приложения _frontend_ из образа nginx с количеством реплик 3 шт.

создан манифест [deployment-front.yaml]

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-front
  labels:
    app: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```

2. Создать Deployment приложения _backend_ из образа multitool. 


создан манифест [deployment-back.yaml]

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-back
  labels:
    app: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: multitool
        image: wbitt/network-multitool
        ports:
        - containerPort: 80
          name: http
        - containerPort: 443
          name: https
```


3. Добавить Service, которые обеспечат доступ к обоим приложениям внутри кластера. 


[service-front.yaml]

```
apiVersion: v1
kind: Service
metadata:
  name: front
spec:
  ports:
    - name: http
      port: 80
  selector:
    app: frontend
```

[service-back.yaml]

```
apiVersion: v1
kind: Service
metadata:
  name: back
spec:
  ports:
    - name: miltitool
      port: 80
  selector:
    app: backend
```
```
root@baranov:/home/baranovsa/kube-1.5# kubectl apply -f deployment-front.yaml
deployment.apps/deployment-front created
root@baranov:/home/baranovsa/kube-1.5# kubectl apply -f deployment-back.yaml
deployment.apps/deployment-back created
root@baranov:/home/baranovsa/kube-1.5#
root@baranov:/home/baranovsa/kube-1.5# kubectl get pods 
NAME                                READY   STATUS    RESTARTS   AGE
deployment-back-7d779c456d-99zfc    1/1     Running   0          32m
deployment-front-868d6b748f-nprms   1/1     Running   0          32m
deployment-front-868d6b748f-vdjcf   1/1     Running   0          32m
deployment-front-868d6b748f-xhmgb   1/1     Running   0          32m
root@baranov:/home/baranovsa/kube-1.5#
root@baranov:/home/baranovsa/kube-1.5# kubectl apply -f service-front.yaml
service/front created
root@baranov:/home/baranovsa/kube-1.5#
root@baranov:/home/baranovsa/kube-1.5# kubectl apply -f service-back.yaml
service/back created
root@baranov:/home/baranovsa/kube-1.5# kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
back         ClusterIP   10.97.157.75   <none>        80/TCP    32m
front        ClusterIP   10.99.50.173   <none>        80/TCP    32m
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   12d
root@baranov:/home/baranovsa/kube-1.5#
```

4. Продемонстрировать, что приложения видят друг друга с помощью Service.

```
root@baranov:/home/baranovsa/kube-1.5# kubectl exec --stdin --tty deployment-back-7d779c456d-99zfc -- /bin/bash
deployment-back-7d779c456d-99zfc:/# curl front
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
deployment-back-7d779c456d-99zfc:/# 
deployment-back-7d779c456d-99zfc:/# curl back
WBITT Network MultiTool (with NGINX) - deployment-back-7d779c456d-99zfc - 10.244.0.37 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
deployment-back-7d779c456d-99zfc:/# 
```

5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.

[deployment-back]()

[deployment-front]()

[service-back]()

[service-front]()


```
root@baranov:/home/baranovsa/kube-1.5# kubectl exec --stdin --tty deployment-back-7d779c456d-99zfc -- /bin/bash
deployment-back-7d779c456d-99zfc:/# curl front
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
deployment-back-7d779c456d-99zfc:/# 
deployment-back-7d779c456d-99zfc:/# curl back
WBITT Network MultiTool (with NGINX) - deployment-back-7d779c456d-99zfc - 10.244.0.37 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
deployment-back-7d779c456d-99zfc:/# 
```

------


### Задание 2. Создать Ingress и обеспечить доступ к приложениям снаружи кластера

1. Включить Ingress-controller в MicroK8S.

```
root@baranov:/home/baranovsa/kube-1.5# microk8s.enable ingress
Infer repository core for addon ingress
Enabling Ingress
ingressclass.networking.k8s.io/public created
ingressclass.networking.k8s.io/nginx created
namespace/ingress created
serviceaccount/nginx-ingress-microk8s-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-microk8s-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-microk8s-role created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-microk8s created
configmap/nginx-load-balancer-microk8s-conf created
configmap/nginx-ingress-tcp-microk8s-conf created
configmap/nginx-ingress-udp-microk8s-conf created
daemonset.apps/nginx-ingress-microk8s-controller created
Ingress is enabled
root@baranov:/home/baranovsa/kube-1.5#

```

2. Создать Ingress, обеспечивающий доступ снаружи по IP-адресу кластера MicroK8S так, чтобы при запросе только по адресу открывался _frontend_ а при добавлении /api - _backend_.

Создан [ingress.yaml]

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-01
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    - host: example.com
      http:
        paths:
          - pathType: Prefix
            backend:
              service:
                name: front
                port:
                  number: 80
            path: /
          - pathType: Prefix
            backend:
              service:
                name: back
                port:
                  number: 80
            path: /api
```

```
oot@baranov:/home/baranovsa/kube-1.5# kubectl apply -f ingress.yaml
ingress.networking.k8s.io/ingress-01 created
root@baranov:/home/baranovsa/kube-1.5#
root@baranov:/home/baranovsa/kube-1.5# kubectl get ingress
NAME         CLASS   HOSTS         ADDRESS        PORTS   AGE
ingress-01   nginx   example.com   192.168.49.2   80      33s
root@baranov:/home/baranovsa/kube-1.5#
root@baranov:/home/baranovsa/kube-1.5# kubectl get svc -A
NAMESPACE       NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
default         back                                 ClusterIP   10.97.157.75     <none>        80/TCP                       36m
default         front                                ClusterIP   10.99.50.173     <none>        80/TCP                       36m
default         kubernetes                           ClusterIP   10.96.0.1        <none>        443/TCP                      12d
ingress-nginx   ingress-nginx-controller             NodePort    10.101.236.7     <none>        80:30187/TCP,443:31243/TCP   7m54s
ingress-nginx   ingress-nginx-controller-admission   ClusterIP   10.106.177.139   <none>        443/TCP                      7m54s
kube-system     kube-dns                             ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP,9153/TCP       12d
root@baranov:/home/baranovsa/kube-1.5#

```

3. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.

```
root@baranov:/home/baranovsa# curl example.com
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@baranov:/home/baranovsa# 
root@baranov:/home/baranovsa# 
root@baranov:/home/baranovsa# 
root@baranov:/home/baranovsa# curl example.com/api
WBITT Network MultiTool (with NGINX) - deployment-back-7d779c456d-99zfc - 10.244.0.37 - HTTP: 80 , HTTPS: 443 . (Formerly praqma/network-multitool)
root@baranov:/home/baranovsa#
```

4. Предоставить манифесты и скриншоты или вывод команды п.2.


Манифест [ingress.yaml]()

Вывод команд:

```
root@baranov:/home/baranovsa# curl example.com
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@baranov:/home/baranovsa# 
root@baranov:/home/baranovsa# 
root@baranov:/home/baranovsa# 
root@baranov:/home/baranovsa# curl example.com/api
WBITT Network MultiTool (with NGINX) - deployment-back-7d779c456d-99zfc - 10.244.0.37 - HTTP: 80 , HTT>
root@baranov:/home/baranovsa#
```

------

### Правила приема работы

1. Домашняя работа оформляется в своем Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

2. Файл README.md должен содержать скриншоты вывода необходимых команд `kubectl` и скриншоты результатов.

3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.

------
