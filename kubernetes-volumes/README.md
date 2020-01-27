# Выполнено ДЗ №

 - [x] Основное ДЗ
 - [x] Задание со *
 
## В процессе сделано:
 - Применение StatefulSet
 - Применили манифест с разворачиванием pod MinIO, созданием PVC и на нем же PV с помощью дефолотногоStorageClass.
 - Создали Headless Service для внешнего доступа в minio
```
kubectl get statefulsets
NAME    READY   AGE
minio   1/1     10m


kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
minio-0   1/1     Running   0          11m


kubectl get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-minio-0   Bound    pvc-ef7b745c-ce49-4d04-8bc9-49741d3fa669   10Gi       RWO            standard       11m


kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pvc-ef7b745c-ce49-4d04-8bc9-49741d3fa669   10Gi       RWO            Delete           Bound    default/data-minio-0   standard                11m


kubectl describe pvc 
Name:          data-minio-0
Namespace:     default
StorageClass:  standard
Status:        Bound
Volume:        pvc-ef7b745c-ce49-4d04-8bc9-49741d3fa669
Labels:        app=minio
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: kubernetes.io/host-path
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      10Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Mounted By:    minio-0
Events:
  Type    Reason                 Age   From                         Message
  ----    ------                 ----  ----                         -------
  Normal  ProvisioningSucceeded  12m   persistentvolume-controller  Successfully provisioned volume pvc-ef7b745c-ce49-4d04-8bc9-49741d3fa669 using kubernetes.io/host-path


kubectl describe pv 
Name:            pvc-ef7b745c-ce49-4d04-8bc9-49741d3fa669
Labels:          <none>
Annotations:     kubernetes.io/createdby: hostpath-dynamic-provisioner
                 pv.kubernetes.io/bound-by-controller: yes
                 pv.kubernetes.io/provisioned-by: kubernetes.io/host-path
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    standard
Status:          Bound
Claim:           default/data-minio-0
Reclaim Policy:  Delete
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        10Gi
Node Affinity:   <none>
Message:         
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /tmp/hostpath_pv/40242fba-a8e6-47d8-b093-98c24470f5fc
    HostPathType:  
Events:            <none>
```



#Задание со *
 - В манифесте StatefulSet логин и пароль передаются в открытом виде, создадим секрет и изменим манифест
```
kubectl create secret generic minio-db-secret --from-literal=MINIO_ACCESS_KEY=minio --from-literal=MINIO_SECRET_KEY=minio123


kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-svgw4   kubernetes.io/service-account-token   3      4h46m
minio-db-secret       Opaque                                2      47s


kubectl get secret minio-db-secret -o yaml
apiVersion: v1
data:
  MINIO_ACCESS_KEY: bWluaW8=
  MINIO_SECRET_KEY: bWluaW8xMjM=
kind: Secret
metadata:
  creationTimestamp: "2020-01-26T17:46:26Z"
  name: minio-db-secret
  namespace: default
  resourceVersion: "21318"
  selfLink: /api/v1/namespaces/default/secrets/minio-db-secret
  uid: 8851fd60-20c6-4aaf-98f7-30c377e5e9b2
type: Opaque
```
 - Добавляем строчку
```
        env:
        - name: MINIO_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: minio-db-secret
              key: MINIO_ACCESS_KEY
        - name: MINIO_SECRET_KEY
          valueFrom:
            secretKeyRef:
              name: minio-db-secret
              key: MINIO_SECRET_KEY
```
в блок containers нашего манифеста, запускаем и смотрим
```
kubectl apply -f minio-statefulset-secret.yaml 
statefulset.apps/minio created


kubectl get pods
NAME      READY   STATUS    RESTARTS   AGE
minio-0   1/1     Running   0          14s


kubectl get pvc
NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-minio-0   Bound    pvc-ef7b745c-ce49-4d04-8bc9-49741d3fa669   10Gi       RWO            standard       68m


kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
pvc-ef7b745c-ce49-4d04-8bc9-49741d3fa669   10Gi       RWO            Delete           Bound    default/data-minio-0   standard                69m


kubectl describe pvc
Name:          data-minio-0
Namespace:     default
StorageClass:  standard
Status:        Bound
Volume:        pvc-ef7b745c-ce49-4d04-8bc9-49741d3fa669
Labels:        app=minio
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
               volume.beta.kubernetes.io/storage-provisioner: kubernetes.io/host-path
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      10Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Mounted By:    minio-0
Events:        <none>
```

## Как запустить проект:
 - Запустить манифест через kubectl
 kubectl apply -f <file_name>.yaml


## Как проверить работоспособность:
 - kubectl get statefulsets
 - kubectl get pods
 - kubectl get pvc
 - kubectl get pv
 - kubectl describe pvc
 

## PR checklist:
 - [x] Выставлен label с номером домашнего задания
