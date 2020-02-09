# Выполнено ДЗ №

 - [ ] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:
Домашнее задание выполнялось в облаке GCP с развернутым kubernetes кластером.
 - Устанавливаем готовые Helm charts
Разворачиваем кластер в Google Cloud Platform, устанавливаем Helm 3, устанавливаем репозиторий stable в Helm 3.
 - nginx-ingress
Создаем namespace и устанавливаем nginx-ingress через helm
 - cert-manager
Создаем namespace и устанавливаем cert-manager (по описанию на странице https://cert-manager.io/docs/installation/kubernetes/)
```
kubectl get svc --all-namespaces 
NAMESPACE       NAME                            TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)                      AGE
cert-manager    cert-manager                    ClusterIP      10.0.10.210   <none>         9402/TCP                     39m
cert-manager    cert-manager-webhook            ClusterIP      10.0.12.125   <none>         443/TCP                      39m
default         kubernetes                      ClusterIP      10.0.0.1      <none>         443/TCP                      2d
kube-system     default-http-backend            NodePort       10.0.9.48     <none>         80:30573/TCP                 47h
kube-system     heapster                        ClusterIP      10.0.1.192    <none>         80/TCP                       47h
kube-system     kube-dns                        ClusterIP      10.0.0.10     <none>         53/UDP,53/TCP                47h
kube-system     metrics-server                  ClusterIP      10.0.2.226    <none>         443/TCP                      47h
nginx-ingress   nginx-ingress-controller        LoadBalancer   10.0.6.89     34.76.36.154   80:32295/TCP,443:30835/TCP   64m
nginx-ingress   nginx-ingress-default-backend   ClusterIP      10.0.4.38     <none>         80/TCP                       64m
```

 - cert-manager. Самостоятельное задание
Для работы службы cert-manager, необходимо сконфигурировать поставщика сертификатов и настроить 1 из ресурсов Issuer или ClusterIssuer. 
Описываем манифест, где развернем элемент ClusterIssuer в производственной среде (перед этим можно зарегестрировать имя на бесплатном ddns сервере)
```
kubectl describe -f ci-letsencript.yaml 
Name:         letsencrypt-prod
Namespace:    
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"cert-manager.io/v1alpha2","kind":"ClusterIssuer","metadata":{"annotations":{},"name":"letsencrypt-prod"},"spec":{"acme":{"e...
API Version:  cert-manager.io/v1alpha2
Kind:         ClusterIssuer
Metadata:
  Creation Timestamp:  2020-01-29T23:40:29Z
  Generation:          1
  Resource Version:    671800
  Self Link:           /apis/cert-manager.io/v1alpha2/clusterissuers/letsencrypt-prod
  UID:                 e7ed05a4-13c5-4b6a-8b79-70f73ec8b880
Spec:
  Acme:
    Email:  v.selyutin@revo.ru
    Private Key Secret Ref:
      Name:  letsencrypt-prod
    Server:  https://acme-v02.api.letsencrypt.org/directory
    Solvers:
      http01:
        Ingress:
          Class:  nginx
Status:
  Acme:
    Last Registered Email:  v.selyutin@revo.ru
    Uri:                    https://acme-v02.api.letsencrypt.org/acme/acct/76944463
  Conditions:
    Last Transition Time:  2020-01-29T23:40:30Z
    Message:               The ACME account was registered with the ACME server
    Reason:                ACMEAccountRegistered
    Status:                True
    Type:                  Ready
Events:                    <none>
```
Подключаем выдачу сертификатов к nginx-ingress-controller и объявляем имя нашего хоста и смотрим что сертификат выдан
```
kubectl describe -f ci-lb.yaml 
Name:             echo-ingress
Namespace:        default
Address:          
Default backend:  default-http-backend:80 (10.32.0.6:8080)
TLS:
  echo-tls terminates prod.o-k-hw6.dns-cloud.net
Rules:
  Host                        Path  Backends
  ----                        ----  --------
  prod.o-k-hw6.dns-cloud.net  
                                 nginx-ingress-controller:80 (<none>)
Annotations:
  kubernetes.io/ingress.class:                       nginx
  cert-manager.io/cluster-issuer:                    letsencrypt-prod
  kubectl.kubernetes.io/last-applied-configuration:  {"apiVersion":"networking.k8s.io/v1beta1","kind":"Ingress","metadata":{"annotations":{"cert-manager.io/cluster-issuer":"letsencrypt-prod","kubernetes.io/ingress.class":"nginx"},"name":"echo-ingress","namespace":"default"},"spec":{"rules":[{"host":"prod.o-k-hw6.dns-cloud.net","http":{"paths":[{"backend":{"serviceName":"nginx-ingress-controller","servicePort":80}}]}}],"tls":[{"hosts":["prod.o-k-hw6.dns-cloud.net"],"secretName":"echo-tls"}]}}

Events:
  Type    Reason             Age   From                      Message
  ----    ------             ----  ----                      -------
  Normal  CREATE             10s   nginx-ingress-controller  Ingress default/echo-ingress
  Normal  CreateCertificate  10s   cert-manager              Successfully created Certificate "echo-tls"
```
Проверяем
```
curl https://prod.o-k-hw6.dns-cloud.net -v
* Rebuilt URL to: https://prod.o-k-hw6.dns-cloud.net/
*   Trying 34.76.36.154...
* TCP_NODELAY set
* Connected to prod.o-k-hw6.dns-cloud.net (34.76.36.154) port 443 (#0)
...
* Server certificate:
*  subject: CN=prod.o-k-hw6.dns-cloud.net
*  start date: Jan 29 22:41:25 2020 GMT
*  expire date: Apr 28 22:41:25 2020 GMT
*  subjectAltName: host "prod.o-k-hw6.dns-cloud.net" matched cert's "prod.o-k-hw6.dns-cloud.net"
*  issuer: C=US; O=Let's Encrypt; CN=Let's Encrypt Authority X3
*  SSL certificate verify ok.
...
```

 - chartmuseum
Устанавливаем chartmuseum, берем за основу values.yaml из github репозитория и дописываем ingress ресурс с корректным именем хоста (аннотацию по sert-manager исправляем на описанную ранее в самостоятельной работе иначе выдаст ошибку о неправильной аннотации) и добавляем автоматическую генерацию Let's Encrypt сертификата.
Проверяем
```
curl https://chartmuseum.o-k-hw6.dns-cloud.net -v
* Rebuilt URL to: https://chartmuseum.o-k-hw6.dns-cloud.net/
*   Trying 34.76.36.154...
* TCP_NODELAY set
* Connected to chartmuseum.o-k-hw6.dns-cloud.net (34.76.36.154) port 443 (#0)
...
* SSL connection using TLSv1.2 / ECDHE-RSA-AES256-GCM-SHA384
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=chartmuseum.o-k-hw6.dns-cloud.net
*  start date: Feb  2 16:09:18 2020 GMT
*  expire date: May  2 16:09:18 2020 GMT
*  subjectAltName: host "chartmuseum.o-k-hw6.dns-cloud.net" matched cert's "chartmuseum.o-k-hw6.dns-cloud.net"
*  issuer: C=US; O=Let's Encrypt; CN=Let's Encrypt Authority X3
*  SSL certificate verify ok.
...
> Accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/2 200 
...

helm ls -n chartmuseum
NAME       	NAMESPACE  	REVISION	UPDATED                                	STATUS  	CHART            	APP VERSION
chartmuseum	chartmuseum	1       	2020-02-02 20:08:47.506936208 +0300 MSK	deployed	chartmuseum-2.3.2	0.8.2      

kubectl get secrets -n chartmuseum
NAME                                TYPE                                  DATA   AGE
chartmuseum-chartmuseum             Opaque                                0      7m22s
chartmuseum.o-k-hw6.dns-cloud.net   kubernetes.io/tls                     3      7m20s
default-token-jdprn                 kubernetes.io/service-account-token   3      13m
sh.helm.release.v1.chartmuseum.v1   helm.sh/release.v1                    1      7m22s
```

 - harbor. Самостоятельное задание
Добавляем из репозиторий harbor файл values.yaml для helm, изменяем и разворачиваем с версией 1.1.2
```
helm upgrade --install harbor harbor/harbor --wait \
>   --namespace=harbor \
>   --version=1.1.2 \
>   -f values.yaml
Release "harbor" does not exist. Installing it now.
NAME: harbor
LAST DEPLOYED: Mon Feb  3 04:07:14 2020
NAMESPACE: harbor
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Please wait for several minutes for Harbor deployment to complete.
Then you should be able to visit the Harbor portal at https://core.harbor.domain. 
For more details, please visit https://github.com/goharbor/harbor.
```
Проверяем сертификата и подключение
```
curl https://harbor.o-k-hw6.dns-cloud.net -Ivv
* Rebuilt URL to: https://harbor.o-k-hw6.dns-cloud.net/
*   Trying 34.76.36.154...
* TCP_NODELAY set
* Connected to harbor.o-k-hw6.dns-cloud.net (34.76.36.154) port 443 (#0)
...
* Server certificate:
*  subject: CN=harbor.o-k-hw6.dns-cloud.net
*  start date: Feb  3 00:07:21 2020 GMT
*  expire date: May  3 00:07:21 2020 GMT
*  subjectAltName: host "harbor.o-k-hw6.dns-cloud.net" matched cert's "harbor.o-k-hw6.dns-cloud.net"
*  issuer: C=US; O=Let's Encrypt; CN=Let's Encrypt Authority X3
*  SSL certificate verify ok.
...
> Accept: */*
> 
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/2 200 
HTTP/2 200 
...
```

 - Создаем свой helm chart
Создаем helm chart, убераем все лишнее и в templates переносим шаблон. Запускам и пробрасываем порты
```
kubectl port-forward --namespace hipster-shop $(kubectl get pod --namespace hipster-shop --selector="app=frontend" --output jsonpath='{.items[0].metadata.name}') 8080:8080
```
Подключаемся http://120.0.0.1:8080 и видим что приложение работает.
Создаем отделный helm chart для микросервиса frontend из манифеста all-hipster-shop.yaml и добавляем манифест ingress с доменным именем. Применяем и смотрим результат
```
curl https://shop.o-k-hw6.dns-cloud.net/ -Ivv
*   Trying 34.76.36.154...
* TCP_NODELAY set
* Connected to shop.o-k-hw6.dns-cloud.net (34.76.36.154) port 443 (#0)
...
* Server certificate:
*  subject: CN=shop.o-k-hw6.dns-cloud.net
*  start date: Feb  3 21:55:22 2020 GMT
*  expire date: May  3 21:55:22 2020 GMT
*  subjectAltName: host "shop.o-k-hw6.dns-cloud.net" matched cert's "shop.o-k-hw6.dns-cloud.net"
*  issuer: C=US; O=Let's Encrypt; CN=Let's Encrypt Authority X3
*  SSL certificate verify ok.
...
* Connection state changed (MAX_CONCURRENT_STREAMS updated)!
< HTTP/2 200 
HTTP/2 200 
...
```
Делаем шаблон некоторых параметров и описываем их в values.yaml, добавляем условие проверки типа сервиса.
Включаем в зависимость frontend в helm chart hipster-shop и обновляем hipster-shop, ресурсы frontend вновь созданы.

 - Kubecfg
Выносим из all-hipster-shop.yaml в директорию kubecfg paymentservice и shippingservice. Ооновляем hipster-shop, микросервисы paymentservice и shippingservice не работают.
Устанавливаем kubecfg в систему и делаем исполняемым.
Описываем в services.jsonnet шаблон включающий описание service и deployment, устанавливаем и проверяем, что корзина в на сайте снова заработала.

 - Kustomize | Самостоятельное задание
Описали манифест по разворачиванию 1 микросервиса из all-hipster-shop.yaml.yaml, отдельный deploy длф stage и production
Зазвернуть манифест можно указав kustomize.yaml:
kubectl apply -k kubernetes-templating/kustomize/overrides/<hipster-shop-stage/hipster-shop-stage>/kustomize.yaml

## Как запустить проект:
 - kubectl apply -k kubernetes-templating/kustomize/overrides/<hipster-shop-stage/hipster-shop-stage>/kustomize.yaml
 - helm upgrade --install hipster-shop hipster-shop --namespace hipster-shop
 - helm upgrade --install harbor harbor/harbor --wait \
  --namespace=harbor \
  --version=1.1.2 \
  -f values.yaml
 - helm upgrade --install chartmuseum stable/chartmuseum --wait \
  --namespace=chartmuseum \
  --version=2.3.2 \
  -f values.yaml
  
## Как проверить работоспособность:
 - chartmuseum.o-k-hw6.dns-cloud.net
 - prod.o-k-hw6.dns-cloud.net 
 - harbor.o-k-hw6.dns-cloud.net/
 - harbor.o-k-hw6.dns-cloud.net/api/
 - harbor.o-k-hw6.dns-cloud.net/service/
 - harbor.o-k-hw6.dns-cloud.net/v2/
 - harbor.o-k-hw6.dns-cloud.net/chartrepo/
 - harbor.o-k-hw6.dns-cloud.net/c/
 - shop.o-k-hw6.dns-cloud.net/ 
 
## PR checklist:
 - [x] Выставлен label с номером домашнего задания