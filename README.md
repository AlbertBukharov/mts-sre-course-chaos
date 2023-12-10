# mts-sre-course-chaos
Домашняя работа №4 Chaos Engineering
### Описание тестового стенда 
Оригинал схемы лежит в репозитории https://github.com/AlbertBukharov/mts-sre-course  
![image](https://github.com/AlbertBukharov/mts-sre-course-chaos/assets/81142061/e3d5fa8a-e7d8-430d-8bf7-90faf9029018)

### Отключение узла: 
Планово остановить один из узлов кластера, чтобы проверить процедуру  
переключения ролей (failover). - Анализировать время, необходимое для восстановления и как  
система выбирает новый Master узел (и есть ли вообще там стратегия выбора?).  

1. Описание эксперимента (отключение patroni-leader ноды):  
  Прогоним locust'ом нагрузку  
  Остановим patroni-leader-ноду (10.0.10.4)  
  Прогоним locust'ом нагрузку с выключенной нодой  
  Оценка результатов
  Включение ноды 10.0.10.4, проверка статусов
2. Ожидаемые результаты: влияния на сервис не ожидаем (в ходе нагрузочного тестирования узким местом системы в целом были определены поды в k8s-кластере), в графане должны увидеть смену лидера, должен прилететь алерт в Телеграм по недоступности ноды, после обратного включения ноды смены лидера не происходит
3. Реальные результаты: влияния на сервис нет, RPS не изменился (в ходе нагрузочного тестирования было определено, что система может обработать 30RPS с заявленным SLO:
![image](https://github.com/AlbertBukharov/mts-sre-course-chaos/assets/81142061/4182b523-dace-4f02-bf66-d02683577aca)
Пришло оповещение в ТГ:
![image](https://github.com/AlbertBukharov/mts-sre-course-chaos/assets/81142061/610448ec-0672-486d-8f43-1c949de69047)
После обратного включения ноды, переключение patroni-leader не происходит
![image](https://github.com/AlbertBukharov/mts-sre-course-chaos/assets/81142061/91118d8c-5146-4e1e-8ec2-6ec0503ff5aa)
4. Анализ результатов: Ожидаемые и реальные результаты в целом совпали.

### Имитация частичной потери сети: 
Использовать инструменты для имитации потери пакетов или разрыва TCP-соединений между узлами. Цель — проверить, насколько хорошо система  
справляется с временной недоступностью узлов и как быстро восстанавливается репликация.  
1. Описание эксперимента:  
С помощью утилиты tc сэмулируем потери на ноде 10.0.10.3 (нода является master для postgresql): sudo tc qdisc add dev ens160 root netem loss 70%  
После 5-тиминутного тестирования останавливаем генерацию потерь: sudo tc qdisc del dev ens160 root 
3. Ожидаемые результаты: достаточен ли уровень потерь в 70% для развала кластера - вопрос оставался открытым, предварительные тесты в 5% не давали особой деградации  
4. Реальные результаты:  
Фиксируем потерю в 70%  
```64 bytes from 10.0.10.3: icmp_seq=202 ttl=64 time=0.413 ms
^C
--- 10.0.10.3 ping statistics ---
202 packets transmitted, 50 received, 75.2475% packet loss, time 205816ms
rtt min/avg/max/mdev = 0.201/0.330/0.705/0.087 ms
```
При таком уровне потерь проблемная нода удалилась из кластера, потеряв связности с etcd-нодами:  
```
alberton@alb-patroni01:~$ sudo patronictl topology
2023-12-10 20:34:14,009 - WARNING - postgresql parameter max_prepared_transactions=0 failed validation, defaulting to 0
2023-12-10 20:34:15,754 - ERROR - Failed to get list of machines from http://10.0.10.6:2379/v3beta: MaxRetryError("HTTPConnectionPool(host='10.0.10.6', port=2379): Max retries exceeded with url: /version (Caused by ConnectTimeoutError(<urllib3.connection.HTTPConnection object at 0x7fda35725300>, 'Connection to 10.0.10.6 timed out. (connect timeout=1.6666666666666667)'))")
2023-12-10 20:34:17,422 - ERROR - Failed to get list of machines from http://10.0.10.7:2379/v3beta: MaxRetryError("HTTPConnectionPool(host='10.0.10.7', port=2379): Max retries exceeded with url: /version (Caused by ConnectTimeoutError(<urllib3.connection.HTTPConnection object at 0x7fda35725660>, 'Connection to 10.0.10.7 timed out. (connect timeout=1.6666666666666667)'))")
2023-12-10 20:34:19,090 - ERROR - Failed to get list of machines from http://10.0.10.5:2379/v3beta: MaxRetryError("HTTPConnectionPool(host='10.0.10.5', port=2379): Max retries exceeded with url: /version (Caused by ConnectTimeoutError(<urllib3.connection.HTTPConnection object at 0x7fda357258a0>, 'Connection to 10.0.10.5 timed out. (connect timeout=1.6666666666666667)'))")
2023-12-10 20:34:27,433 - ERROR - Failed to get list of machines from http://10.0.10.6:2379/v3beta: MaxRetryError('HTTPConnectionPool(host=\'10.0.10.6\', port=2379): Max retries exceeded with url: /version (Caused by ReadTimeoutError("HTTPConnectionPool(host=\'10.0.10.6\', port=2379): Read timed out. (read timeout=3.3326131973332926)"))')
2023-12-10 20:34:29,103 - ERROR - Failed to get list of machines from http://10.0.10.7:2379/v3beta: MaxRetryError("HTTPConnectionPool(host='10.0.10.7', port=2379): Max retries exceeded with url: /version (Caused by ConnectTimeoutError(<urllib3.connection.HTTPConnection object at 0x7fda35725de0>, 'Connection to 10.0.10.7 timed out. (connect timeout=1.6666666666666667)'))")
2023-12-10 20:34:30,772 - ERROR - Failed to get list of machines from http://10.0.10.5:2379/v3beta: MaxRetryError("HTTPConnectionPool(host='10.0.10.5', port=2379): Max retries exceeded with url: /version (Caused by ConnectTimeoutError(<urllib3.connection.HTTPConnection object at 0x7fda35725b40>, 'Connection to 10.0.10.5 timed out. (connect timeout=1.6666666666666667)'))")
```
на ноде alb-patroni02 видим как лидер перешел к ней, а затем alb-patroni01 вообще пропала из топологии:  
```alberton@alb-patroni02:~$ sudo patronictl topology
2023-12-10 20:34:00,030 - WARNING - postgresql parameter max_prepared_transactions=0 failed validation, defaulting to 0
+ Cluster: postgres-cluster --+---------+-----------+----+-----------+-----------------+
| Member          | Host      | Role    | State     | TL | Lag in MB | Pending restart |
+-----------------+-----------+---------+-----------+----+-----------+-----------------+
| alb-patroni01   | 10.0.10.3 | Leader  | running   |  8 |           | *               |
| + alb-patroni02 | 10.0.10.4 | Replica | streaming |  8 |         0 | *               |
+-----------------+-----------+---------+-----------+----+-----------+-----------------+
alberton@alb-patroni02:~$ sudo patronictl topology
2023-12-10 20:34:28,157 - WARNING - postgresql parameter max_prepared_transactions=0 failed validation, defaulting to 0
+ Cluster: postgres-cluster --+---------+---------+----+-----------+-----------------+
| Member          | Host      | Role    | State   | TL | Lag in MB | Pending restart |
+-----------------+-----------+---------+---------+----+-----------+-----------------+
| alb-patroni02   | 10.0.10.4 | Leader  | running |  9 |           | *               |
| + alb-patroni01 | 10.0.10.3 | Replica | running |  8 |         0 | *               |
+-----------------+-----------+---------+---------+----+-----------+-----------------+
alberton@alb-patroni02:~$ sudo patronictl topology
2023-12-10 20:35:03,957 - WARNING - postgresql parameter max_prepared_transactions=0 failed validation, defaulting to 0
+ Cluster: postgres-cluster +--------+---------+----+-----------+-----------------+
| Member        | Host      | Role   | State   | TL | Lag in MB | Pending restart |
+---------------+-----------+--------+---------+----+-----------+-----------------+
| alb-patroni02 | 10.0.10.4 | Leader | running |  9 |           | *               |
+---------------+-----------+--------+---------+----+-----------+-----------------+
```
В дополнение к ожидаемому результату также фиксировалось отсутствие данных в Графане по проблемной ноде 10.0.10.3   
4. Анализ результатов: В итоге тест с потерями на мастер-ноде свелся к тесту №1 с выключением ноды, эффект в целом такой же

### Высокая нагрузка на CPU или I/O: 
Запустить процессы, которые создают высокую нагрузку на CPU или дисковую подсистему одного из узлов кластера, чтобы проверить, как это влияет на производительность кластера в целом и на работу Patroni.  
1. Описание эксперимента: Сгенерировать нагрузку на CPU в 70%, сгенерировать запросы в систему с помощью locust-скрипта, оценить параметры системы
2. Ожидаемые результаты: загрузка в 70% CPU не должна вызвать проблем
3. Реальные результаты:
Генерируем нагрузку утилитой ChaosBlade (https://github.com/chaosblade-io/chaosblade)
root@alb-patroni02:/home/alberton/chaosblade-1.7.2# ./blade create cpu load --cpu-percent 70 --timeout 300
{"code":200,"success":true,"result":"96e8c67859c2a588"}
Система корректно работает при утилизации CPU в 70%, кратковременные всплески до 95+% (периодически фиксировались в выводе htop) не оказывают влияния![image](https://github.com/AlbertBukharov/mts-sre-course-chaos/assets/81142061/8f520e50-9928-4827-8593-48df723f87aa)
График утилизации с Node_exporter:
![image](https://github.com/AlbertBukharov/mts-sre-course-chaos/assets/81142061/1ce5a2b7-4753-4dad-9cf6-939690e58c23)  
4. Анализ результатов: Система ведет себя предсказуемо при высокой утилизации CPU

### Тестирование систем мониторинга и оповещения: 
С помощью chaos engineering можно также проверить, насколько эффективны системы мониторинга и оповещения. Например, можно искусственно вызвать отказ, который должен быть зарегистрирован системой мониторинга, и  
убедиться, что оповещения доставляются вовремя?  
В дополнение к алертам/оповещениям, предоставленным в тесте №1 (Отключение узла)), проверим также срабатывание алертов при отключении etcd-нод.
1. Описание эксперимента:  
  Выключим одну из etcd-нод  
2. Ожидаемые результаты: должны прийти алерты NodeDown и недостаточном количестве членов кластера etcd  
3. Реальные результаты:  
![image](https://github.com/AlbertBukharov/mts-sre-course-chaos/assets/81142061/2a2e2cbb-e94c-4e9c-bf14-93110edd1350)
4. Анализ результатов: Ожидаемые и реальные результаты в целом совпали.

## Задания дополнительные:
### 3. Сбои сервисов зависимостей: 
Изучить поведение кластера Patroni при сбоях в сопутствующих сервисах, например, etcd (которые используются для хранения состояния кластера), путем имитации его недоступности или некорректной работы.  
1. Описание эксперимента:  
   Отключаем etcd-ноды по очереди, смотрим за поведением patroni-кластера  
2. Ожидаемые результаты: недостаточное знание мат.части не позволяет определить ожидаемые результаты, будем подробно фиксировать реальные результаты, есть предположение, что на чтение базы останутся доступны.
3. Реальные результаты:
Выключение одной etcd-ноды не оказывает влияния на работу стабильной в остальном плане системы, оставим ее выключенной
Перезагрузка мастер-патрони-ноды вызывает смену лидера:  
![image](https://github.com/AlbertBukharov/mts-sre-course-chaos/assets/81142061/96aaadb4-6c43-48b8-9a43-281150cb7480)
Перезагруженная ВМ после рестарта становится репликой (ничего неожиданного):
![image](https://github.com/AlbertBukharov/mts-sre-course-chaos/assets/81142061/8b3a3391-dd11-4220-84f8-64b2148da3f9)
В etcd-кластере возможна смена лидера:
```
root@alb-etcd01:/home/alberton# etcdctl endpoint status -w table
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT    |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 127.0.0.1:2379 | e8cc3f7ff72fe07d |   3.5.9 |  1.2 MB |      true |      false |         6 |       3458 |               3458 |        |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
root@alb-etcd01:/home/alberton# etcdctl move-leader b586ded327f9460d
Leadership transferred from e8cc3f7ff72fe07d to b586ded327f9460d
root@alb-etcd01:/home/alberton# etcdctl endpoint status -w table    
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT    |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 127.0.0.1:2379 | e8cc3f7ff72fe07d |   3.5.9 |  1.2 MB |     false |      false |         7 |       3459 |               3459 |        |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
```
Выключим еще одну etcd-ноду (10.0.10.5 - НЕ-Лидер):
КЛАСТЕР etcd СЛОМАЛСЯ:
```
root@alb-etcd02:/home/alberton# etcdctl endpoint status -w table
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+-----------------------+
|    ENDPOINT    |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX |        ERRORS         |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+-----------------------+
| 127.0.0.1:2379 | b586ded327f9460d |   3.5.9 |  1.2 MB |     false |      false |         7 |       3465 |               3465 | etcdserver: no leader |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+-----------------------+
root@alb-etcd02:/home/alberton# etcdctl endpoint health
{"level":"warn","ts":"2023-12-10T21:37:59.769013Z","logger":"client","caller":"v3@v3.5.9/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"etcd-endpoints://0xc0002c6a80/127.0.0.1:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = context deadline exceeded"}
127.0.0.1:2379 is unhealthy: failed to commit proposal: context deadline exceeded
Error: unhealthy cluster
```
Это также вывело из строя patroni:  
```
patroni.dcs.etcd3.Etcd3Error: Etcd is not responding properly
```
Базы "остановились", клиентские запросы отбиваются с 503 ошибкой:  
![image](https://github.com/AlbertBukharov/mts-sre-course-chaos/assets/81142061/63e12996-178c-4f4f-8415-9ed3861a0d1e)
Также перезапускаются поды в кубернетес-кластеры (не проходят live-check'и):  
```
alberton@alberton-PC:~$ kubectl get pods
NAME                          READY   STATUS    RESTARTS       AGE
alb-srelol-85db97f6f9-6ltk7   1/1     Running   14 (84s ago)   14d
alb-srelol-85db97f6f9-fscrb   1/1     Running   12 (82s ago)   14d
alb-srelol-85db97f6f9-t9g2r   1/1     Running   13 (86s ago)   14d
```
4. Анализ результатов:  
Выключение 2-х etcd нод и следующая за ним потеря лидера вызывает полную деградацию сервиса, невозможно даже чтение из базы, необходимо провести дополнительное изучение данной темы.
