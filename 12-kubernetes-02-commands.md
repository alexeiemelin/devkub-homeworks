# Домашнее задание к занятию "12.2 Команды для работы с Kubernetes"
Кластер — это сложная система, с которой крайне редко работает один человек. Квалифицированный devops умеет наладить работу всей команды, занимающейся каким-либо сервисом.
После знакомства с кластером вас попросили выдать доступ нескольким разработчикам. Помимо этого требуется служебный аккаунт для просмотра логов.

## Задание 1: Запуск пода из образа в деплойменте
Для начала следует разобраться с прямым запуском приложений из консоли. Такой подход поможет быстро развернуть инструменты отладки в кластере. Требуется запустить деплоймент на основе образа из hello world уже через deployment. Сразу стоит запустить 2 копии приложения (replicas=2). 

Требования:
 * пример из hello world запущен в качестве deployment
 * количество реплик в deployment установлено в 2
 * наличие deployment можно проверить командой kubectl get deployment
 * наличие подов можно проверить командой kubectl get pods

Ответ:

```bash
$ kubectl create deployment netology --image=k8s.gcr.io/echoserver:1.4 --replicas=2
deployment.apps/netology created
$ kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-node   1/1     1            1           29d
netology     2/2     2            2           14s
```

## Задание 2: Просмотр логов для разработки
Разработчикам крайне важно получать обратную связь от штатно работающего приложения и, еще важнее, об ошибках в его работе. 
Требуется создать пользователя и выдать ему доступ на чтение конфигурации и логов подов в app-namespace.

Требования: 
 * создан новый токен доступа для пользователя
 * пользователь прописан в локальный конфиг (~/.kube/config, блок users)
 * пользователь может просматривать логи подов и их конфигурацию (kubectl logs pod <pod_id>, kubectl describe pod <pod_id>)

Ответ:

```bash
$ kubectl config view | grep -A8 users
users:
- name: minikube
  user:
    client-certificate: /home/ubuntu/.minikube/profiles/minikube/client.crt
    client-key: /home/ubuntu/.minikube/profiles/minikube/client.key
- name: user1
  user:
    client-certificate: /home/ubuntu/cert/user1.crt
    client-key: /home/ubuntu/cert/user1.key
$ kubectl config get-contexts
CURRENT   NAME            CLUSTER    AUTHINFO   NAMESPACE
          minikube        minikube   minikube   default
*         user1-context   minikube   user1      
$ cat role.yaml 
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: netology
rules:
- apiGroups: [ "" ] # “” indicates the core API group
  resources: [ pods, pods/log ]
  verbs: [ get, list ]
$ cat role-binding.yaml 
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: netology_2
  namespace: default
subjects:
- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole #this must be Role or ClusterRole
  name: netology # must match the name of the Role
  apiGroup: rbac.authorization.k8s.io
$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
hello-node-697897c86-htdhb   1/1     Running   0          31d
netology-64c7499b96-92rrq    1/1     Running   0          2d18h
netology-64c7499b96-9cl84    1/1     Running   0          2d17h
netology-64c7499b96-ftw66    1/1     Running   0          2d18h
netology-64c7499b96-lv444    1/1     Running   0          2d17h
netology-64c7499b96-qjdhq    1/1     Running   0          2d17h
$ kubectl describe pod netology-64c7499b96-lv444
Name:             netology-64c7499b96-lv444
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Fri, 18 Nov 2022 15:19:28 +0000
Labels:           app=netology
                  pod-template-hash=64c7499b96
Annotations:      <none>
Status:           Running
IP:               172.17.0.10
IPs:
  IP:           172.17.0.10
Controlled By:  ReplicaSet/netology-64c7499b96
Containers:
  echoserver:
    Container ID:   docker://813f755fc592ad54c8bd7773847f6c97acba2e8f2317756ac66581b406cf1cb5
    Image:          k8s.gcr.io/echoserver:1.4
    Image ID:       docker-pullable://k8s.gcr.io/echoserver@sha256:5d99aa1120524c801bc8c1a7077e8f5ec122ba16b6dda1a5d3826057f67b9bcb
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 18 Nov 2022 15:19:31 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5gxlt (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-5gxlt:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:                      <none>
$ kubectl logs netology-64c7499b96-92rrq
```

## Задание 3: Изменение количества реплик 
Поработав с приложением, вы получили запрос на увеличение количества реплик приложения для нагрузки. Необходимо изменить запущенный deployment, увеличив количество реплик до 5. Посмотрите статус запущенных подов после увеличения реплик. 

Требования:
 * в deployment из задания 1 изменено количество реплик на 5
 * проверить что все поды перешли в статус running (kubectl get pods)

Ответ:

```bash
$ kubectl scale --replicas=5 deployment netology
deployment.apps/netology scaled
$ kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-node   1/1     1            1           29d
netology     5/5     5            5           16mx
$ kubectl get pods
NAME                         READY   STATUS    RESTARTS   AGE
hello-node-697897c86-htdhb   1/1     Running   0          29d
netology-64c7499b96-92rrq    1/1     Running   0          17m
netology-64c7499b96-9cl84    1/1     Running   0          109s
netology-64c7499b96-ftw66    1/1     Running   0          17m
netology-64c7499b96-lv444    1/1     Running   0          109s
netology-64c7499b96-qjdhq    1/1     Running   0          109s

```

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
