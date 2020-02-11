# Выполнено ДЗ №

 - [x] Основное ДЗ
 - [ ] Задание со *

## В процессе сделано:
 - CustomResourceDefinition
Создаем crd.yml для разворачивания манифеста cr.yml и запускаем по очереди
```
kubectl get crd
NAME                   CREATED AT
mysqls.otus.homework   2020-02-09T20:58:52Z

kubectl get mysqls.otus.homework
NAME             AGE
mysql-instance   55s

kubectl describe mysqls.otus.homework mysql-instance
Name:         mysql-instance
Namespace:    default
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"otus.homework/v1","kind":"MySQL","metadata":{"annotations":{},"name":"mysql-instance","namespace":"default"},"spec":{"datab...
API Version:  otus.homework/v1
Kind:         MySQL
Metadata:
  Creation Timestamp:  2020-02-09T20:58:55Z
  Generation:          1
  Resource Version:    2495
  Self Link:           /apis/otus.homework/v1/namespaces/default/mysqls/mysql-instance
  UID:                 7c5289c2-e0d6-4938-92fb-2505a8bd0d19
Spec:
  Database:      otus-database
  Image:         mysql:5.7
  Password:      otuspassword
  storage_size:  1Gi
usless_data:     useless info
Events:          <none>
```
 - Validation
Описали схему CR и поправили 2 манифеста, добавили обязательные параметры для разворачивания mysql.

 - Деплой оператора
Создали 4 манифеста и применили. Проверили, что все работает.
```
kubectl get pvc
NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
backup-mysql-instance-pvc   Bound    pvc-6edc59f5-7115-47e7-9333-8a1228b7ea8a   1Gi        RWO            standard       69s
mysql-instance-pvc          Bound    pvc-ab945a10-b6fc-4ec9-9339-75ba73fb68ea   1Gi        RWO            standard       69s
```

 - Проверим, что все работает
Заполняем базу данных и проверяем
```
kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+--------------+
| id | name         |
+----+--------------+
|  1 | some data-11 |
|  2 | some data-21 |
+----+--------------+
```
После удаления mysql-instance видим что pv не запущен
```
kubectl get jobs.batch
NAME                         COMPLETIONS   DURATION   AGE
backup-mysql-instance-job    1/1           3s         51s
restore-mysql-instance-job   0/1           13m        13m
```
Смотрим в логи и видим выполнение задания
```
kubectl describe jobs
Name:           backup-mysql-instance-job
Namespace:      default
Selector:       controller-uid=db0fe55d-acd3-4092-98db-cd1e46bbc8bb
Labels:         usage=backup-mysql-instance-job
...
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  14m   job-controller  Created pod: backup-mysql-instance-job-w942j


Name:           restore-mysql-instance-job
Namespace:      default
Selector:       controller-uid=af60c97f-88a1-437d-ac02-f398de3102e7
Labels:         controller-uid=af60c97f-88a1-437d-ac02-f398de3102e7
                job-name=restore-mysql-instance-job
...
Events:
  Type     Reason                Age                From            Message
  ----     ------                ----               ----            -------
  Normal   SuccessfulCreate      26m                job-controller  Created pod: restore-mysql-instance-job-d2x2r
  Normal   SuccessfulDelete      20m                job-controller  Deleted pod: restore-mysql-instance-job-d2x2r
  Warning  BackoffLimitExceeded  20m (x2 over 20m)  job-controller  Job has reached the specified backoff limit
```
Снова применяем манифест cr.yml и проверяем записи в БД
```
kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+--------------+
| id | name         |
+----+--------------+
|  1 | some data-11 |
|  2 | some data-21 |
+----+--------------+
```

## Как запустить проект:
 - kubectl apply -f deploy/<name>.yml
 
## Как проверить работоспособность:
 - export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}")
 - kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database
 
 - kubectl get pv
 - kubectl get crd
 - kubectl get mysqls.otus.homework
 - kubectl describe mysqls.otus.homework mysql-instance

## PR checklist:
 - [x] Выставлен label с номером домашнего задания
