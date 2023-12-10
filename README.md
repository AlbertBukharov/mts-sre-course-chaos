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
