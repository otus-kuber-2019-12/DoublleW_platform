# Выполнено ДЗ № 9

 - [x] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:
 - Подготовлен кластер в GKE и развернут HipsterShop в пространстве имен microservices-demo на ноде default-pool.
 - В пространстве имен observability разворачиваем EFK стек в пуле infra-pool при этом добавляем переменные в манифест elasticsearch.values.yaml для пула infra-pool, чтобы привязать развертывание к 3 нодам в этом пуле.
```
kubectl create -f namespace.yaml

helm upgrade --install elasticsearch elastic/elasticsearch --namespace observability -f elasticsearch.values.yaml

kubectl get pods -n observability -o wide -l chart=elasticsearch
NAME                     READY   STATUS    RESTARTS   AGE   IP          NODE                                            NOMINATED NODE   READINESS GATES
elasticsearch-master-0   1/1     Running   0          28m   10.60.3.2   gke-cluster-otus-hw9-infra-pool-375c01f8-bb36   <none>           <none>
elasticsearch-master-1   1/1     Running   0          28m   10.60.1.8   gke-cluster-otus-hw9-infra-pool-375c01f8-j23q   <none>           <none>
elasticsearch-master-2   1/1     Running   0          28m   10.60.2.2   gke-cluster-otus-hw9-infra-pool-375c01f8-5bf8   <none>           <none>
```
 - Установка nginx-ingress | Самостоятельное задание
Создаем namespace.yaml и описываем создание namespace в пуле infra-pool используя PodNodeSelector и дописываем values.yaml для разворачивания 3 реплик в infra-pool.
Запускаем heml и устанавливаем  nginx-ingress.
```
kubectl create -f nginx-ingress/namespace.yaml

helm install nginx-ingress-infra stable/nginx-ingress -f nginx-ingress/values.yaml --wait \
  --namespace=nginx-ingress-infra

kubectl get pods -n nginx-ingress-infra -o wide
NAME                                                   READY   STATUS    RESTARTS   AGE   IP          NODE                                            NOMINATED NODE   READINESS GATES
nginx-ingress-infra-controller-5785b8b8f9-946hk        1/1     Running   0          36m   10.60.1.3   gke-cluster-otus-hw9-infra-pool-375c01f8-j23q   <none>           <none>
nginx-ingress-infra-controller-5785b8b8f9-9v8hp        1/1     Running   0          36m   10.60.1.4   gke-cluster-otus-hw9-infra-pool-375c01f8-j23q   <none>           <none>
nginx-ingress-infra-controller-5785b8b8f9-z7bch        1/1     Running   0          36m   10.60.1.6   gke-cluster-otus-hw9-infra-pool-375c01f8-j23q   <none>           <none>
nginx-ingress-infra-default-backend-7f457b76b9-gxnjc   1/1     Running   0          36m   10.60.1.7   gke-cluster-otus-hw9-infra-pool-375c01f8-j23q   <none>           <none>
nginx-ingress-infra-default-backend-7f457b76b9-lxz5v   1/1     Running   0          36m   10.60.1.2   gke-cluster-otus-hw9-infra-pool-375c01f8-j23q   <none>           <none>
nginx-ingress-infra-default-backend-7f457b76b9-x4z6n   1/1     Running   0          36m   10.60.1.5   gke-cluster-otus-hw9-infra-pool-375c01f8-j23q   <none>           <none>
```
Устанавливаем cert-manager так-же 3 реплики для получения сертификата внешних адресов.
```
kubectl create -f cert-manager/namespace.yaml

kubectl apply --dry-run -o yaml --validate=false \
    -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.13/deploy/manifests/00-crds.yaml \
    | sed 's/namespace\: cert-manager/namespace\: cert-manager-infra/g' \
    | kubectl apply -f -
```
Устанавливаем cert-manager
```
helm upgrade --install cert-manager-infra jetstack/cert-manager -f cert-manager/values.yaml --wait \
  --namespace=cert-manager-infra 

kubectl get pods -n cert-manager-infra -o wide
NAME                                             READY   STATUS    RESTARTS   AGE     IP          NODE                                            NOMINATED NODE   READINESS GATES
cert-manager-infra-6868d485f9-4mnhx              1/1     Running   0          2m12s   10.60.3.5   gke-cluster-otus-hw9-infra-pool-375c01f8-bb36   <none>           <none>
cert-manager-infra-6868d485f9-c79vn              1/1     Running   0          2m12s   10.60.2.4   gke-cluster-otus-hw9-infra-pool-375c01f8-5bf8   <none>           <none>
cert-manager-infra-6868d485f9-xshcm              1/1     Running   0          2m12s   10.60.3.4   gke-cluster-otus-hw9-infra-pool-375c01f8-bb36   <none>           <none>
cert-manager-infra-cainjector-857cd4f9dc-fkrvr   1/1     Running   0          2m12s   10.60.2.3   gke-cluster-otus-hw9-infra-pool-375c01f8-5bf8   <none>           <none>
cert-manager-infra-cainjector-857cd4f9dc-jd424   1/1     Running   0          2m12s   10.60.2.5   gke-cluster-otus-hw9-infra-pool-375c01f8-5bf8   <none>           <none>
cert-manager-infra-cainjector-857cd4f9dc-qhpr9   1/1     Running   0          2m12s   10.60.3.3   gke-cluster-otus-hw9-infra-pool-375c01f8-bb36   <none>           <none>
cert-manager-infra-webhook-657bf674cf-5929s      1/1     Running   0          2m12s   10.60.1.9   gke-cluster-otus-hw9-infra-pool-375c01f8-j23q   <none>           <none>
cert-manager-infra-webhook-657bf674cf-8792g      1/1     Running   0          2m12s   10.60.2.6   gke-cluster-otus-hw9-infra-pool-375c01f8-5bf8   <none>           <none>
cert-manager-infra-webhook-657bf674cf-wt88b      1/1     Running   0          2m12s   10.60.3.6   gke-cluster-otus-hw9-infra-pool-375c01f8-bb36   <none>           <none>
```
!Подсказка!
Если удалить namespace, то установленные CRD останутся, избавится от них можно слудующим способом
```
helm template nginx-ingress-infra stable/nginx-ingress -f nginx-ingress/values.yaml --namespace nginx-ingress-infra | kubectl delete -f -

helm template cert-manager-infra jetstack/cert-manager -f cert-manager/values.yaml --namespace cert-manager-infra | kubectl delete -f -
```
Это помогает при возникновении ошибки
```
Error: rendered manifests contain a resource that already exists. Unable to continue with install: existing resource conflict: kind: ClusterRole, namespace: , name: nginx-ingress-infra
```
Разворачиваем базовый ACME Issuer
```
kubectl apply -f cert-manager/ci-letsencript.yaml --force
```
Регестрируем доменное имя на любом бесплатном ddns хостинге.
 - Установка EFK стека | Kibana
Устанавливаем сразу с получением сертификата на доменное имя kibana.o-k-hw9.dns-cloud.net
Изменяем kibana.values.yaml
```
nodeSelector:
  cloud.google.com/gke-nodepool: infra-pool

tolerations:
  - key: node-role
    operator: Equal
    value: infra
    effect: NoSchedule

ingress:
  enabled: true
  annotations: 
    kubernetes.io/ingress.class: "nginx"
    kubernetes.io/tls-acme: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    cert-manager.io/acme-challenge-type: "http01"
  path: /
  hosts:
    - kibana.o-k-hw9.dns-cloud.net
  tls: 
    - secretName: kibana.o-k-hw9.dns-cloud.net-tls
      hosts:
        - kibana.o-k-hw9.dns-cloud.net
```
и применяем манифест
```
helm upgrade --install kibana elastic/kibana --namespace observability -f kibana.values.yaml
```
Смотрим на полученный сертификат
```
curl https://kibana.o-k-hw9.dns-cloud.net -Iv
* Rebuilt URL to: https://kibana.o-k-hw9.dns-cloud.net/
*   Trying 35.228.95.154...
* TCP_NODELAY set
* Connected to kibana.o-k-hw9.dns-cloud.net (35.228.95.154) port 443 (#0)
.....
* Server certificate:
*  subject: CN=kibana.o-k-hw9.dns-cloud.net
*  start date: Mar 22 23:20:19 2020 GMT
*  expire date: Jun 20 23:20:19 2020 GMT
*  subjectAltName: host "kibana.o-k-hw9.dns-cloud.net" matched cert's "kibana.o-k-hw9.dns-cloud.net"
*  issuer: C=US; O=Let's Encrypt; CN=Let's Encrypt Authority X3
*  SSL certificate verify ok.
.....
> Host: kibana.o-k-hw9.dns-cloud.net
> User-Agent: curl/7.58.0
> Accept: */*
.....
```
Устанавливаем fluent-bit
```
helm upgrade --install fluent-bit stable/fluent-bit --namespace observability
```
Создадим файл fluent-bit.values.yaml и допишем сервис elasticsearch-master и уберем дублирующие поля.
Применим манифест
```
helm upgrade --install fluent-bit stable/fluent-bit --namespace observability -f fluent-bit.values-def.yaml

kubectl logs fluent-bit-fzpl8 -n observability --tail 2
[2020/03/23 00:56:20] [ info] [filter_kube] API server connectivity OK
[2020/03/23 00:56:20] [ info] [sp] stream processor started
```
#####kibana-def.png

 - Мониторинг ElasticSearch
Устанавливаем prometheus-operator в namespace observability
```
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/release-0.35/example/prometheus-operator-crd/monitoring.coreos.com_alertmanagers.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/release-0.35/example/prometheus-operator-crd/monitoring.coreos.com_podmonitors.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/release-0.35/example/prometheus-operator-crd/monitoring.coreos.com_prometheuses.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/release-0.35/example/prometheus-operator-crd/monitoring.coreos.com_prometheusrules.yaml
kubectl apply -f https://raw.githubusercontent.com/coreos/prometheus-operator/release-0.35/example/prometheus-operator-crd/monitoring.coreos.com_servicemonitors.yaml

helm upgrade --install prometheus-operator stable/prometheus-operator --set prometheusOperator.createCustomResource=false --namespace observability -f prometheus-operator.values.yaml

kubectl get pods -n observability --sort-by=.metadata.name | grep prometh
alertmanager-prometheus-operator-alertmanager-0           2/2     Running   0          34s
prometheus-operator-grafana-cb7cdbb74-h7tz2               2/2     Running   0          40s
prometheus-operator-kube-state-metrics-57d47947fd-cj78r   1/1     Running   0          40s
prometheus-operator-operator-7f965958fd-gclcw             2/2     Running   0          40s
prometheus-operator-prometheus-node-exporter-8cw58        1/1     Running   0          40s
prometheus-operator-prometheus-node-exporter-mlxpv        1/1     Running   0          40s
prometheus-operator-prometheus-node-exporter-qzzf4        1/1     Running   0          40s
prometheus-operator-prometheus-node-exporter-vxf2t        1/1     Running   0          40s
prometheus-prometheus-operator-prometheus-0               3/3     Running   1          23s
```
Устанавливаем elasticsearch-exporter
```
helm upgrade --install elasticsearch-exporter stable/elasticsearch-exporter --set es.uri=http://elasticsearch-master:9200 --set serviceMonitor.enabled=true --namespace observability

kubectl get pods elasticsearch-exporter-f65df58f8-p4mvs -n observability -o wide
NAME                                     READY   STATUS    RESTARTS   AGE   IP           NODE                                            NOMINATED NODE   READINESS GATES
elasticsearch-exporter-f65df58f8-p4mvs   1/1     Running   0          19h   10.60.2.11   gke-cluster-otus-hw9-infra-pool-375c01f8-5bf8   <none>           <none>
```
Все dashboards для grafana берем из оф. ресурса - https://grafana.com/grafana/dashboards?dataSource=prometheus&direction=asc&orderBy=name
#####grafana-es.png

Делаем drain одной из нод infra-pool
При удалении подов с еще 1 ноды, поды на 1 оставшейся ноде перешли в состояние "Not Ready", перестали собираться метрики, вернул ноды в рабочее состояние.

 - EFK | nginx ingress
Создаем fluent-bit.values.yaml с парамантрами для чтения логов nginx-ingress, для nginx-ingress в values.yaml дописываем параметры для того чтобы он смог отдавать свои логи и разворачиваем
```
helm upgrade --install fluent-bit-observability stable/fluent-bit --namespace observability -f fluent-bit.values.yaml

helm upgrade --install nginx-ingress-infra stable/nginx-ingress -f nginx-ingress/values.yaml --wait \
    --set controller.metrics.enabled=true,controller.metrics.serviceMonitor.enabled=true \
    --namespace=nginx-ingress-infra 
```
#####kibana-nginx.png
Cоздаем в kibana визуализацию, показывающую общее количество запросов к nginx-ingress. 
#####kibana-nginx_f.png

 - Loki
Установливаем Loki и Promtail в namespace observability, дописываем манифест loki.values.yaml и разворачиваем
```
helm upgrade --install loki loki/loki-stack -f loki.values.yaml \
    --set fluent-bit.enabled=false,promtail.enabled=true,grafana.enabled=false,prometheus.enabled=false,prometheus.alertmanager.persistentVolume.enabled=false,prometheus.server.persistentVolume.enabled=false \
    --namespace=observability 
```

 - Loki | Datasource
В grafana смотрим что loki видит nginx-ingress и показывает его логи
#####grafana-loki.png

 - Loki | Визуализация
Создаем в grafana свою новую панель
#####grafana-nginx_man.png

## Как запустить проект:
 - Описание выше
 
## Как проверить работоспособность:
 - Посмотреть результат:
   https://kibana.o-k-hw9.dns-cloud.net
   https://grafana.o-k-hw9.dns-cloud.net (пароль в манифесте)
   
## PR checklist:
 - [x] Выставлен label с номером домашнего задания
