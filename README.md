# Домашнее задание по модулю 2

Написать ansible playbook для развертывания postgresql в patroni сетапе. Пример, который можно взять за основу.
Разворачиваем etcd, patroni, postgres и единственный инстанс Haproxy.
Написать helm chart для разворачивания api в выделенном неймспейсе. Docker image лежит в публичном registry, разворачивать стоит актуальную версию ghcr.io/ldest/sre-course/api
Из образа вытащить скрипт миграции для создания БД, настроить на работу с кластером api, проверить работоспособность

## Важно! Кластер k8s один на всех, ingress так же один, соответственно, и ip адрес внешний будет так же один.
## Разделять запросы будем через доменные имена, которые можно указать либо в заголовке Host, либо в файле hosts – как вам удобнее. Не забывайте про ingressClassName: nginx


# Развертывание

## 1) Настройка инфраструктуры
1) Установить Ansible для создания playbook и Helm для создания Helm chart.
2) Создать пустой проект Kubernetes и настройте kubectl для работы с этим проектом.
3) Убедитесь, что есть внешний IP-адрес и настроить Ingress controller с классом nginx.

## 2) Ansible playbook для развертывания PostgreSQL в Patroni сетапе

`deploy_postgresql.yml` - Ansible playbook. Через него устанавливаем необходимые пакеты, Patroni, настраиваем PostgreSQL, инициализируем Patroni и запускаем PostgreSQL и Patroni. Замените my_mts_hosts на целевой хост, на котором нужно развернуть PostgreSQL с Patroni.

## 3) Helm chart для API
Каталог api-chart - это каталогс с Helm chart с определенной под него структурой

В файле `values.yaml` указаны необходимые параметры для API, включая образ Docker и скрипт миграции.

В файле `templates/deployment.yaml` определены ресурсы для развертывания API, включая контейнер с образом Docker и скриптом миграции.

В файле `templates/service.yaml` определена службу Kubernetes для API.

## 4) Управление миграцией базы данных
В файле `api-chart/scripts/init_db.sql` - скрипт миграции базы данных

## 5) Разворачивание всего на кластере Kubernetes

### 1) Применяем Ansible playbook для развертывания PostgreSQL с Patroni на целевых хостах:
`ansible-playbook deploy_postgresql.yml`

### 2) Создайем ConfigMap для скрипта миграции:
`kubectl create configmap migration-script-configmap --from-file=api-chart/scripts/init_db.sql`

### 3) Установливаем Helm chart для API:
`helm install api-chart ./api-chart`

### 4) Настраиваем доменное имя в Ingress.
Для этого в файле `api-chart/api-ingress.yaml` описан Ingress ресурс для маршрутизации запросов на API.
Вместо my-api-domain.com подставить необходимое доменное имя.

Для применения Ingress ресурса вводим команду:
`kubectl apply -f api-ingress.yaml`



