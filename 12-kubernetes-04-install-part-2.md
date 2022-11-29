# Домашнее задание к занятию "12.4 Развертывание кластера на собственных серверах, лекция 2"
Новые проекты пошли стабильным потоком. Каждый проект требует себе несколько кластеров: под тесты и продуктив. Делать все руками — не вариант, поэтому стоит автоматизировать подготовку новых кластеров.

## Задание 1: Подготовить инвентарь kubespray
Новые тестовые кластеры требуют типичных простых настроек. Нужно подготовить инвентарь и проверить его работу. Требования к инвентарю:
* подготовка работы кластера из 5 нод: 1 мастер и 4 рабочие ноды;
* в качестве CRI — containerd;
* запуск etcd производить на мастере.

## Задание 2 (*): подготовить и проверить инвентарь для кластера в AWS
Часть новых проектов хотят запускать на мощностях AWS. Требования похожи:
* разворачивать 5 нод: 1 мастер и 4 рабочие ноды;
* работать должны на минимально допустимых EC2 — t3.small.

Ответ:
```
Поднял вм на yandex cloud. Установил kubespray на cp1.

$ yc compute instance list
+----------------------+-------+---------------+--------------+---------------+---------------+
|          ID          | NAME  |    ZONE ID    |    STATUS    |  EXTERNAL IP  |  INTERNAL IP  |
+----------------------+-------+---------------+--------------+---------------+---------------+
| fhm1uopceic2n65971uh | node4 | ru-central1-a | PROVISIONING | 51.250.10.157 | 192.168.10.7  |
| fhm5fufdu655ac2el45d | node2 | ru-central1-a | PROVISIONING | 51.250.8.245  | 192.168.10.35 |
| fhmk796m7g54ckvqi36b | cp1   | ru-central1-a | PROVISIONING | 62.84.118.155 | 192.168.10.23 |
| fhmlq08h86btekdbjhgd | node1 | ru-central1-a | PROVISIONING | 62.84.116.26  | 192.168.10.20 |
| fhmr25fldv8sap2pivjc | node3 | ru-central1-a | PROVISIONING | 62.84.115.6   | 192.168.10.34 |
+----------------------+-------+---------------+--------------+---------------+---------------+

Изменил для последующей установки список хостов в файле inventory/mycluster/hosts.yaml

all:
  hosts:
    cp1:
      ansible_host: 192.168.10.23
      ip: 192.168.10.23
      access_ip: 192.168.10.23
      ansible_user: ubuntu
    node1:
      ansible_host: 192.168.10.20
      ip: 193.168.10.20
      access_ip: 192.168.10.20
      ansible_user: ubuntu
    node2:
      ansible_host: 192.168.10.35
      ip: 192.168.10.35
      access_ip: 192.168.10.35
      ansible_user: ubuntu
    node3:
      ansible_host: 192.168.10.34
      ip: 192.168.10.34
      access_ip: 192.168.10.34
      ansible_user: ubuntu
    node4:
      ansible_host: 192.168.10.7
      ip: 192.168.10.7
      access_ip: 192.168.10.7
      ansible_user: ubuntu
  children:
    kube_control_plane:
      hosts:
        cp1:
    kube_node:
      hosts:
        node1:
        node2:
        node3:
        node4:
    etcd:
      hosts:
        cp1:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}

Раскрутил playbook, добавил конфиг в домашнюю папку пользователя для управления кластером 
и проверил ноды:

ansible-playbook -i inventory/mycluster/hosts.yaml cluster.yml -b -v

$mkdir -p $HOME/.kube
ubuntu@cp1:~/kubespray$sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
ubuntu@cp1:~/kubespray$sudo chown $(id -u):$(id -g) $HOME/.kube/config

$ kubectl get nodes
NAME    STATUS   ROLES           AGE   VERSION
cp1     Ready    control-plane   19m   v1.25.4
node1   Ready    <none>          17m   v1.25.4
node2   Ready    <none>          17m   v1.25.4
node3   Ready    <none>          17m   v1.25.4
node4   Ready    <none>          17m   v1.25.4

```
---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
