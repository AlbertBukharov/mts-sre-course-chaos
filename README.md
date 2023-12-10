# mts-sre-course-chaos
Домашняя работа №4 Chaos Engineering
# Описание тестового стенда
https://private-user-images.githubusercontent.com/81142061/273467459-8045dbb1-d996-4c76-9d14-29952d917fdd.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDIyMjg4MTksIm5iZiI6MTcwMjIyODUxOSwicGF0aCI6Ii84MTE0MjA2MS8yNzM0Njc0NTktODA0NWRiYjEtZDk5Ni00Yzc2LTlkMTQtMjk5NTJkOTE3ZmRkLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFJV05KWUFYNENTVkVINTNBJTJGMjAyMzEyMTAlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjMxMjEwVDE3MTUxOVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTRhMDAxMDM3MWMxOTY3Y2M1OGVhOGNlYzU3N2UxMDY5ZDcyYzFlNzY2ZmM5YTkyOWFkYjU0ODcyZmEyYzg2Y2EmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.cu_7X4mSCp44llDUXkuxbhHTwLww5FvabAnTjn9o8g4
### Отключение узла: 
Планово остановить один из узлов кластера, чтобы проверить процедуру  
переключения ролей (failover). - Анализировать время, необходимое для восстановления и как  
система выбирает новый Master узел (и есть ли вообще там стратегия выбора?).  

1. Описание эксперимента:
  Остановим одну из нод с postgre
2. 
3. Ожидаемые результаты: Описание ожидаемого поведения системы в ответ
на условия эксперимента.
4. Реальные результаты: Что произошло на самом деле в ходе эксперимента.
Логи, метрики и выводы систем мониторинга и логирования.
5. Анализ результатов: Подробное сравнение ожидаемых и реальных
результатов. - Обсуждение возможных причин отклонений.
