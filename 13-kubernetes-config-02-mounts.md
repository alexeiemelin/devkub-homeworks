# Домашнее задание к занятию "13.2 разделы и монтирование"
Приложение запущено и работает, но время от времени появляется необходимость передавать между бекендами данные. А сам бекенд генерирует статику для фронта. Нужно оптимизировать это.
Для настройки NFS сервера можно воспользоваться следующей инструкцией (производить под пользователем на сервере, у которого есть доступ до kubectl):
* установить helm: curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
* добавить репозиторий чартов: helm repo add stable https://charts.helm.sh/stable && helm repo update
* установить nfs-server через helm: helm install nfs-server stable/nfs-server-provisioner

В конце установки будет выдан пример создания PVC для этого сервера.

## Задание 1: подключить для тестового конфига общую папку
В stage окружении часто возникает необходимость отдавать статику бекенда сразу фронтом. Проще всего сделать это через общую папку. Требования:
* в поде подключена общая папка между контейнерами (например, /static);
* после записи чего-либо в контейнере с беком файлы можно получить из контейнера с фронтом.

Ответ:

Файлы с конфигами лежат в 13-02 My deployment configs/01 Task

```bash
ubuntu@cp1:~$ kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
apps-6b4dd5d68b-hvfrz                 2/2     Running   0          6m58s
ubuntu@cp1:~$ kubectl exec apps-6b4dd5d68b-hvfrz -c backend -- ls -la /static
total 12
drwxrwxrwx 2 root root 4096 Jan  5 19:15 .
drwxr-xr-x 1 root root 4096 Jan  5 18:54 ..
-rw-r--r-- 1 root root    5 Jan  5 19:15 test.txt
ubuntu@cp1:~$ kubectl exec apps-6b4dd5d68b-hvfrz -c frontend -- ls -la /tmp/cache
total 12
drwxrwxrwx 2 root root 4096 Jan  5 19:15 .
drwxrwxrwt 1 root root 4096 Jan  5 18:54 ..
-rw-r--r-- 1 root root    5 Jan  5 19:15 test.txt
```

## Задание 2: подключить общую папку для прода
Поработав на stage, доработки нужно отправить на прод. В продуктиве у нас контейнеры крутятся в разных подах, поэтому потребуется PV и связь через PVC. Сам PV должен быть связан с NFS сервером. Требования:
* все бекенды подключаются к одному PV в режиме ReadWriteMany;
* фронтенды тоже подключаются к этому же PV с таким же режимом;
* файлы, созданные бекендом, должны быть доступны фронту.

Ответ:

Файлы с конфигами лежат в 13-02 My deployment configs/02 Task

Поднимаю поды и проверяю, что общая папка работает:

```bash
ubuntu@cp1:~$ kubectl apply -f deployment_backend.yml 
deployment.apps/backend created
service/backend unchanged
ubuntu@cp1:~$ kubectl apply -f deployment_frontend.yml 
deployment.apps/frontend created
service/frontend unchanged
ubuntu@cp1:~$ kubectl apply -f pvc.yml 
persistentvolumeclaim/pvc created
ubuntu@cp1:~$ kubectl apply -f pv.yml
persistentvolume/pv created

ubuntu@cp1:~$ kubectl get po,pv,pvc
NAME                                      READY   STATUS    RESTARTS   AGE
pod/backend-5dbc9f64fd-dm6pc              1/1     Running   0          103s
pod/db-0                                  1/1     Running   0          59m
pod/frontend-6bfb757f8-mvktq              1/1     Running   0          97s
pod/nfs-server-nfs-server-provisioner-0   1/1     Running   0          103m
pod/postgresql-0                          1/1     Running   0          99m

NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS   REASON   AGE
persistentvolume/pv   1Gi        RWX            Delete           Bound    default/pvc                           72s

NAME                        STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc   Bound    pv       1Gi        RWX                           79s

ubuntu@cp1:~$ kubectl exec backend-5dbc9f64fd-dm6pc -c backend -- sh -c "echo 'test' > /static/test.txt"

ubuntu@cp1:~$ kubectl exec backend-5dbc9f64fd-dm6pc -c backend -- sh -c 'ls -la /static'
total 12
drwxr-xr-x 2 root root 4096 Jan  5 19:46 .
drwxr-xr-x 1 root root 4096 Jan  5 19:42 ..
-rw-r--r-- 1 root root    5 Jan  5 19:46 test.txt

ubuntu@cp1:~$ kubectl exec pod/frontend-6bfb757f8-mvktq -c frontend -- sh -c "cat /tmp/cache/test.txt"
test
```

Удаляю поды и поднимаю заново, проверяю, что информация не потерялась.

```bash
ubuntu@cp1:~$ kubectl delete deployment frontend backend 
deployment.apps "frontend" deleted
deployment.apps "backend" deleted

ubuntu@cp1:~$ kubectl get po,pv,pvc
NAME                                      READY   STATUS    RESTARTS   AGE
pod/db-0                                  1/1     Running   0          70m
pod/nfs-server-nfs-server-provisioner-0   1/1     Running   0          114m
pod/postgresql-0                          1/1     Running   0          110m

NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS   REASON   AGE
persistentvolume/pv   1Gi        RWX            Delete           Bound    default/pvc                           12m

NAME                        STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc   Bound    pv       1Gi        RWX                           12m

ubuntu@cp1:~$ kubectl apply -f deployment_backend.yml 
deployment.apps/backend created
service/backend unchanged
ubuntu@cp1:~$ kubectl apply -f deployment_frontend.yml 
deployment.apps/frontend created
service/frontend unchanged

ubuntu@cp1:~$ kubectl get po,pv,pvc
NAME                                      READY   STATUS    RESTARTS   AGE
pod/backend-5dbc9f64fd-kdxbs              1/1     Running   0          25s
pod/db-0                                  1/1     Running   0          71m
pod/frontend-6bfb757f8-hrkkw              1/1     Running   0          18s
pod/nfs-server-nfs-server-provisioner-0   1/1     Running   0          115m
pod/postgresql-0                          1/1     Running   0          111m

NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM         STORAGECLASS   REASON   AGE
persistentvolume/pv   1Gi        RWX            Delete           Bound    default/pvc                           13m

NAME                        STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/pvc   Bound    pv       1Gi        RWX                           13m
ubuntu@cp1:~$ kubectl exec backend-5dbc9f64fd-kdxbs -c backend -- sh -c "cat /static/test.txt"
test
```

На второй ноде данные так же сохранены:

```bash
ubuntu@node2:~$ ls -la /data/pv
total 12
drwxr-xr-x 2 root root 4096 Jan  5 19:46 .
drwxr-xr-x 3 root root 4096 Jan  5 19:42 ..
-rw-r--r-- 1 root root    5 Jan  5 19:46 test.txt
```

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
