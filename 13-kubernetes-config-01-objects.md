# Домашнее задание к занятию "13.1 контейнеры, поды, deployment, statefulset, services, endpoints"
Настроив кластер, подготовьте приложение к запуску в нём. Приложение стандартное: бекенд, фронтенд, база данных. Его можно найти в папке 13-kubernetes-config.

## Задание 1: подготовить тестовый конфиг для запуска приложения
Для начала следует подготовить запуск приложения в stage окружении с простыми настройками. Требования:
* под содержит в себе 2 контейнера — фронтенд, бекенд;
* регулируется с помощью deployment фронтенд и бекенд;
* база данных — через statefulset.

Ответ:
Файлы с конфигами лежат в 13-01 My deployment configs/01 Task

```bash
$ kubectl apply -f statefulset.yml 
statefulset.apps/postgresql created
$ kubectl apply -f deployment.yaml
deployment.apps/apps created
$ kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
apps-5dcf598b4-wswf9   2/2     Running   0          2d4h
postgresql-0           1/1     Running   0          3m55s
```

## Задание 2: подготовить конфиг для production окружения
Следующим шагом будет запуск приложения в production окружении. Требования сложнее:
* каждый компонент (база, бекенд, фронтенд) запускаются в своем поде, регулируются отдельными deployment’ами;
* для связи используются service (у каждого компонента свой);
* в окружении фронта прописан адрес сервиса бекенда;
* в окружении бекенда прописан адрес сервиса базы данных.

Ответ:

Файлы с конфигами лежат в 13-01 My deployment configs/02 Task

```bash
$ kubectl apply -f database.yml 
statefulset.apps/db created
service/db created
$ kubectl apply -f deployment_backend.yml 
deployment.apps/backend created
service/backend created
$ kubectl apply -f deployment_frontend.yml 
deployment.apps/frontend created
service/frontend created
$ kubectl get po
NAME                        READY   STATUS    RESTARTS   AGE
backend-57bdc889df-jqwds    1/1     Running   0          3m33s
db-0                        1/1     Running   0          3m47s
frontend-857f8c6bc5-zpc6q   1/1     Running   0          3m28s
$ kubectl get statefulset
NAME         READY   AGE
db           1/1     4m23s
ubuntu@cp1:~$ kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
backend    1/1     1            1           4m25s
frontend   1/1     1            1           4m20s
$ kubectl get service
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
backend      ClusterIP   10.233.33.49   <none>        9000/TCP   4m32s
db           ClusterIP   10.233.27.59   <none>        5432/TCP   4m46s
frontend     ClusterIP   10.233.8.5     <none>        80/TCP     4m27s
kubernetes   ClusterIP   10.233.0.1     <none>        443/TCP    2d20h
```

## Задание 3 (*): добавить endpoint на внешний ресурс api
Приложению потребовалось внешнее api, и для его использования лучше добавить endpoint в кластер, направленный на это api. Требования:
* добавлен endpoint до внешнего api (например, геокодер).

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

В качестве решения прикрепите к ДЗ конфиг файлы для деплоя. Прикрепите скриншоты вывода команды kubectl со списком запущенных объектов каждого типа (pods, deployments, statefulset, service) или скриншот из самого Kubernetes, что сервисы подняты и работают.

---
