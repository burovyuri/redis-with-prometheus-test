# Часть #1

Для DEV окружения нам необходимо развернуть кластер Redis из 3-х нод с 90 БД и Prometheus для его мониторинга. 
Prometheus должен ротировать данные раз в 7 дней.

Все действия производим на `minikube` (он должен быть запущен и получен его контекст).

## Запускаем кластер Redis

Запускаем deployments master и slave-нод:

```shell script
kubectl apply -f deployments.yaml
```

К нему стартуем services:

```shell script
kubectl apply -f services.yaml
```

Помним о том, что `minikube` не выдаст нам external_ip, а останется в Pending. 

Внешний адрес `minikube` на хостовой машине: `192.168.99.100`.

Коннект к Redis устанавливаем через порты указанные в запущенных сервисах кластера k8s: `kubectl get services`.

## Запускаем Prometheus

Для DEV-окружения будем использовать готовый `helm chart` Prometheus.

Разместим его в отдельном `namespace`.

```shell script
helm install stage-prometheus --set "server.retention=7d" prometheus-community/prometheus -n monitoring
```

> *Устанавливаем ротацию данных в 7 дней. 

По рекомендации создателей чарта вытаскиваем морду Prometheus наружу простым путём:

```shell script
export POD_NAME=$(kubectl get pods --namespace monitoring -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace monitoring port-forward $POD_NAME 9090
```

Идем на `http://127.0.0.1:9090/`:

* Убеждаемся в правильной настройке ротации данных: `http://127.0.0.1:9090/status` (Storage retention)
* Убеждаемся, что сбор метрик с Redis осуществляется: `http://127.0.0.1:9090/graph?g0.expr=redis_instance_info&g0.tab=1&g0.stacked=0&g0.range_input=1h&g1.expr=&g1.tab=1&g1.stacked=0&g1.range_input=1h`

# Часть 2

## Задача 1: Развернуть кластер Redis

* Удостовериться, что на локальной машине установлен minikube, и его контекст получен.
* Раскатать манифесты и запустить чарт, согласно мануалу из 1-й части.
* Удостовериться в работоспособности Redis & Prometheus.

## Задача 2: Сменить версию Prometheus

* Деинсталлим чарт Prometheus.
* Находим в документации чарта, как сменить версию `image` Prometheus.
* Наша цель, дангрейднуться на версию `2.20.1` (чтобы провести определенные тесты)
* Осуществить запуск чарта с даунгрейднутой версией
* Проверить, что версия Prometheus изменилась: `http://127.0.0.1:9090/status`