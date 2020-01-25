# Выполнено ДЗ № 4

 - [X] Основное ДЗ
 - [ ] Задание со *
 
## В процессе сделано:

Из ДЗ №1 в манифест web-pod.yaml добавили в описание пода readinessProbe с ошибкой номера порта, в итоге под не запустился

```
Warning  Unhealthy  32s   kubelet, minikube  Readiness probe failed: Get http://172.17.0.6:80/index.html: dial tcp 172.17.0.6:80:connect: connection refused
```

Добавили проверку livenessProbe, т.к. readinessProbe у нас с ошибкой проверки порта, то по условиям livenessProbe pod будет перезапускаться постоянно пытаясь пройти проверку readinessProbe.

```
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  44s               default-scheduler  Successfully assigned default/web to minikube
  Normal   Pulling    43s               kubelet, minikube  Pulling image "busybox:1.31.0"
  Normal   Pulled     40s               kubelet, minikube  Successfully pulled image "busybox:1.31.0"
  Normal   Created    40s               kubelet, minikube  Created container init-web
  Normal   Started    39s               kubelet, minikube  Started container init-web
  Normal   Pulling    38s               kubelet, minikube  Pulling image "doubllew/public1:web-v2"
  Normal   Pulled     33s               kubelet, minikube  Successfully pulled image "doubllew/public1:web-v2"
  Normal   Created    33s               kubelet, minikube  Created container web
  Normal   Started    32s               kubelet, minikube  Started container web
  Warning  Unhealthy  2s (x4 over 32s)  kubelet, minikube  Readiness probe failed: Get http://172.17.0.6:80/index.html: dial tcp 172.17.0.6:80: connect: connection refused

NAME   READY   STATUS    RESTARTS   AGE
web    0/1     Running   0          2m4s

```

#Вопрос для самопроверки:

1. В данном контексте вэб сервис или иной сервис может работать, но pod не принимает соединения и не обрабатывает запросы
2. Ситуации когда такая проверка имеет смысл, может возникнуть когда проверяемый сервис падает и происходит перезапуск pod

#Создание Deployment
Создали новый манифест Deployment, перекопировали блок конфигурации из web-deploy.yaml с ошибкой readinessProbe. В итоге после деплоя под не запустился и в условии появилась ошибка

```
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      False   MinimumReplicasUnavailable
  Progressing    True    ReplicaSetUpdated
```

Исправили ошибку readinessProbe и увеличили число реплик до 3. В итоге новый манифест деплоя заменил старый и условия выполняются

```
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable

NAME   READY   UP-TO-DATE   AVAILABLE   AGE
web    3/3     3            3           6m50s
```

#Создание Deployment. Самостоятельная работа

Добавили блок RollingUpdate в манифест web-deploy.yaml

maxUnavailable: 0
maxSurge: 0
```
The Deployment "web" is invalid: spec.strategy.rollingUpdate.maxUnavailable: Invalid value: intstr.IntOrString{Type:0, IntVal:0, StrVal:""}: may not be 0 when `maxSurge` is 0
```

maxUnavailable: 100%
maxSurge: 100%
```
0s          Normal    ScalingReplicaSet   deployment/web              Scaled up replica set web-6d5556764d to 3
0s          Normal    SuccessfulCreate    replicaset/web-6d5556764d   Created pod: web-6d5556764d-wxx9s
0s          Normal    SuccessfulCreate    replicaset/web-6d5556764d   Created pod: web-6d5556764d-4xw5h
0s          Normal    Scheduled           pod/web-6d5556764d-4xw5h    Successfully assigned default/web-6d5556764d-4xw5h to minikube
0s          Normal    SuccessfulCreate    replicaset/web-6d5556764d   Created pod: web-6d5556764d-q82pz
0s          Normal    Scheduled           pod/web-6d5556764d-wxx9s    Successfully assigned default/web-6d5556764d-wxx9s to minikube
0s          Normal    Scheduled           pod/web-6d5556764d-q82pz    Successfully assigned default/web-6d5556764d-q82pz to minikube
0s          Normal    Pulled              pod/web-6d5556764d-wxx9s    Container image "busybox:1.31.0" already present on machine
0s          Normal    Pulled              pod/web-6d5556764d-4xw5h    Container image "busybox:1.31.0" already present on machine
0s          Normal    Pulled              pod/web-6d5556764d-q82pz    Container image "busybox:1.31.0" already present on machine
0s          Normal    Created             pod/web-6d5556764d-wxx9s    Created container init-web
0s          Normal    Created             pod/web-6d5556764d-4xw5h    Created container init-web
0s          Normal    Created             pod/web-6d5556764d-q82pz    Created container init-web
0s          Normal    Started             pod/web-6d5556764d-wxx9s    Started container init-web
0s          Normal    Started             pod/web-6d5556764d-q82pz    Started container init-web
0s          Normal    Started             pod/web-6d5556764d-4xw5h    Started container init-web
0s          Normal    Pulled              pod/web-6d5556764d-4xw5h    Container image "doubllew/public1:web-v2" already present on machine
0s          Normal    Pulled              pod/web-6d5556764d-q82pz    Container image "doubllew/public1:web-v2" already present on machine
0s          Normal    Pulled              pod/web-6d5556764d-wxx9s    Container image "doubllew/public1:web-v2" already present on machine
0s          Normal    Created             pod/web-6d5556764d-4xw5h    Created container web
0s          Normal    Created             pod/web-6d5556764d-wxx9s    Created container web
0s          Normal    Created             pod/web-6d5556764d-q82pz    Created container web
0s          Normal    Started             pod/web-6d5556764d-4xw5h    Started container web
0s          Normal    Started             pod/web-6d5556764d-q82pz    Started container web
0s          Normal    Started             pod/web-6d5556764d-wxx9s    Started container web

```

#Создание Service || ClusterIP
Создали манифест web-svc-cip.yaml где описали доступ к pod по 80 порту и применили его
```
kubectl get services
NAME          TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
kubernetes    ClusterIP   10.96.0.1     <none>        443/TCP   137m
web-svc-cip   ClusterIP   10.96.13.17   <none>        80/TCP    4m51s

$ minikube ssh
$ curl http://10.96.13.17/index.html -Ivvv
> HEAD /index.html HTTP/1.1
> Host: 10.96.13.17
> User-Agent: curl/7.66.0
> Accept: */*
> 
< HTTP/1.1 200 OK
HTTP/1.1 200 OK
```

В таблице arp и addr не отображается CLUSTER-IP
```
$ sudo iptables --list -nv -t nat
Chain KUBE-SERVICES
    1    60 KUBE-SVC-WKCOG6KH24K26XRJ  tcp  --  *      *       0.0.0.0/0            10.96.13.17          /* default/web-svc-cip: cluster IP */ tcp dpt:80

Chain KUBE-SVC-WKCOG6KH24K26XRJ (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 KUBE-SEP-4EHRVUY3IKVUBA4M  all  --  *      *       0.0.0.0/0            0.0.0.0/0            statistic mode random probability 0.33333333349
    0     0 KUBE-SEP-S353CMO5VXPY3Z3E  all  --  *      *       0.0.0.0/0            0.0.0.0/0            statistic mode random probability 0.50000000000
```

#Включение IPVS
Отредактировали конфигурационный файл kube-proxy, применили настройки, проверили цепочку iptables.
Почистили правила iptables в самой ВМ minikube, посмотрели результат
```
[root@minikube ~]# ipvsadm --list -n
TCP  10.96.13.17:80 rr
  -> 172.17.0.4:8000              Masq    1      0          0         
  -> 172.17.0.5:8000              Masq    1      0          0         
  -> 172.17.0.6:8000              Masq    1      0          0
```
И ping проходит до нашего pod
```
ip addr show kube-ipvs0
56: kube-ipvs0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default 
    link/ether 9e:ec:f8:2b:c1:e3 brd ff:ff:ff:ff:ff:ff
...
    inet 10.96.13.17/32 brd 10.96.13.17 scope global kube-ipvs0
       valid_lft forever preferred_lft forever
```


# Установка MetalLB
Устанавливаем из манифеста (на официальном сайте есть обновленный манифест), настроили балансировщик создав манифест metallb-config.yaml (в д/з ошибка, addresses не нужно брать в кавычки иначе не поднимется LoadBalancer) и применяем.
Создаем манифест web-svc-lb.yaml (в описании д/з ошибка, type должен находится в блоке spec) и применяем, смотрим результат.
```
$ kubectl describe svc web-svc-lb 
Name:                     web-svc-lb
Namespace:                default
Labels:                   <none>
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"web-svc-lb","namespace":"default"},"spec":{"ports":[{"port":80,"p...
Selector:                 app=web
Type:                     LoadBalancer
IP:                       10.96.252.212
LoadBalancer Ingress:     172.17.255.1
Port:                     <unset>  80/TCP
TargetPort:               8000/TCP
NodePort:                 <unset>  30903/TCP
Endpoints:                172.17.0.4:8000,172.17.0.5:8000,172.17.0.6:8000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason        Age   From                Message
  ----    ------        ----  ----                -------
  Normal  IPAllocated   110s  metallb-controller  Assigned IP "172.17.255.1"
  Normal  nodeAssigned  110s  metallb-speaker     announcing from node "minikube"
```
Добавим статический маршрут из нашего хоста к балансировщику.
```
curl http://172.17.255.1/ -Iv
*   Trying 172.17.255.1...
* TCP_NODELAY set
* Connected to 172.17.255.1 (172.17.255.1) port 80 (#0)
> HEAD / HTTP/1.1
> Host: 172.17.255.1
> User-Agent: curl/7.58.0
> Accept: */*
> 
< HTTP/1.1 200 OK
HTTP/1.1 200 OK
```

#Создание Ingress
Устанавливаем манифест из репозитория kubernetes и применяем.
Создаем манифест с LoadBalancer для ingress-nginx и применяем.
Проверяем работу сервиса
```
kubectl describe -f nginx-lb.yaml 
Name:                     ingress-nginx
Namespace:                ingress-nginx
Labels:                   app.kubernetes.io/name=ingress-nginx
                          app.kubernetes.io/part-of=ingress-nginx
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app.kubernetes.io/name":"ingress-nginx","app.kubernetes.io/par...
Selector:                 app.kubernetes.io/name=ingress-nginx,app.kubernetes.io/part-of=ingress-nginx
Type:                     LoadBalancer
IP:                       10.96.145.128
LoadBalancer Ingress:     172.17.255.2
Port:                     http  80/TCP
TargetPort:               http/TCP
NodePort:                 http  32527/TCP
Endpoints:                172.17.0.10:80
Port:                     https  443/TCP
TargetPort:               https/TCP
NodePort:                 https  30522/TCP
Endpoints:                172.17.0.10:443
Session Affinity:         None
External Traffic Policy:  Local
HealthCheck NodePort:     30831
Events:
  Type    Reason        Age                From                Message
  ----    ------        ----               ----                -------
  Normal  IPAllocated   22m                metallb-controller  Assigned IP "172.17.255.2"
  Normal  nodeAssigned  14m (x3 over 22m)  metallb-speaker     announcing from node "minikube"
  
ping 172.17.255.2
PING 172.17.255.2 (172.17.255.2) 56(84) bytes of data.
64 bytes from 172.17.255.2: icmp_seq=1 ttl=64 time=0.495 ms
64 bytes from 172.17.255.2: icmp_seq=2 ttl=64 time=2.03 ms

curl http://172.17.255.2 -Iv
* Rebuilt URL to: http://172.17.255.2/
*   Trying 172.17.255.2...
* TCP_NODELAY set
* Connected to 172.17.255.2 (172.17.255.2) port 80 (#0)
> HEAD / HTTP/1.1
> Host: 172.17.255.2
> User-Agent: curl/7.58.0
> Accept: */*
> 
< HTTP/1.1 404 Not Found
HTTP/1.1 404 Not Found
```

#Подключение приложение Web к Ingress
Создаем манифест с сервисом web-svc без ip адреса и применяем его
```
kubectl describe -f web-svc-headless.yaml 
Name:              web-svc
Namespace:         default
Labels:            <none>
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"name":"web-svc","namespace":"default"},"spec":{"clusterIP":"None","ports...
Selector:          app=web
Type:              ClusterIP
IP:                None
Port:              <unset>  80/TCP
TargetPort:        8000/TCP
Endpoints:         172.17.0.6:8000,172.17.0.7:8000,172.17.0.8:8000
Session Affinity:  None
Events:            <none>
```
Создаем манифест с правилом по которому ingress будет отрабатывать и применяем.
```
kubectl describe -f web-ingress.yaml 
Name:             web
Namespace:        default
Address:          172.17.255.2
Default backend:  default-http-backend:80 (<none>)
Rules:
  Host  Path  Backends
  ----  ----  --------
  *     
        /web   web-svc:8000 (172.17.0.6:8000,172.17.0.7:8000,172.17.0.8:8000)
Annotations:
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"networking.k8s.io/v1beta1","kind":"Ingress","metadata":{"annotations":{"nginx.ingress.kubernetes.io/rewrite-target":"/"},"name":"web","namespace":"default"},"spec":{"rules":[{"http":{"paths":[{"backend":{"serviceName":"web-svc","servicePort":8000},"path":"/web"}]}}]}}

  nginx.ingress.kubernetes.io/rewrite-target:  /
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  74s   nginx-ingress-controller  Ingress default/web
  Normal  UPDATE  25s   nginx-ingress-controller  Ingress default/web



curl http://172.17.255.2/web/index.html -Iv
*   Trying 172.17.255.2...
* TCP_NODELAY set
* Connected to 172.17.255.2 (172.17.255.2) port 80 (#0)
> HEAD /web/index.html HTTP/1.1
> Host: 172.17.255.2
> User-Agent: curl/7.58.0
> Accept: */*
> 
< HTTP/1.1 200 OK
HTTP/1.1 200 OK
```

## Как запустить проект:
 - Например, зайти в директорию с манифестами и запустить kubectl
 kubectl apply -f <file_name>.yaml
 
## Как проверить работоспособность:
 - перейти по ссылке http://172.17.255.1
 - перейти по ссылке http://172.17.255.2/web/index.html

## PR checklist:
 - [X] Выставлен label с номером домашнего задания