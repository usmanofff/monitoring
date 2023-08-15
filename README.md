# monitoring

СПРИНТ 3 
```
Настройка сборки логов.

Представьте, что вы разработчик, и вам нужно оперативно получать информацию с ошибками работы приложения.

Выберите инструмент, с помощью которого такой функционал можно предоставить. Нужно собирать логи работы пода приложения. Хранить это всё можно либо в самом кластере Kubernetes, либо на srv-сервере.

2
Выбор метрик для мониторинга.

Так, теперь творческий этап. Допустим, наше приложение имеет для нас некоторую важность. Мы бы хотели знать, когда пользователь не может на него попасть — время отклика, сертификат, статус код и так далее. Выберите метрики и инструмент, с помощью которого будем отслеживать его состояние.

Также мы хотели бы знать, когда место на srv-сервере подходит к концу.

Важно! Весь мониторинг должен находиться на srv-сервере, чтобы в случае падения кластера мы все равно могли узнать об этом.
3
Настройка дашборда.

Ко всему прочему хотелось бы и наблюдать за метриками в разрезе времени. Для этого мы можем использовать Grafana и Zabbix — что больше понравилось.

4
Алертинг.

А теперь добавим уведомления в наш любимый мессенджер, точнее в ваш любимый мессенджер. Обязательно протестируйте отправку уведомлений. Попробуйте «убить» приложение самостоятельно, и засеките время от инцидента до получения уведомления. Если время адекватное, то можно считать, что вы справились с этим проектом!

Самое время положить все возможные конфигурации в Git-репозиторий, если вы этого ещё не сделали.
```

В качестве мониторинга лога выбран стек Grafana-Loki-Promtail:

![image](https://github.com/usmanofff/monitoring/assets/74288450/9ba12afd-8f88-4637-82fb-e46be8265d86)


Для начала создадим namespace  --loki:

```kubectl create ns loki```

Затем необходимо установить из стека grafana-loki c помощью helm репозитория. Добавляем репозиторий 

``` helm repo add grafana https://grafana.github.io/helm-charts ```

и из всего стека устанавливаем только promtail - агент для сборка и отправки логов в loki.

Grafana и loki будут развернуты на сервере SRV туда promtail будет слать логи нашего преложения. 

В качестве среды запуска Grafana-loki на сервере был выбран docker.

На сервере SRV скачиваем docker-compose.yml   ``` wget https://raw.githubusercontent.com/grafana/loki/v2.3.0/production/docker-compose.yaml -O docker-compose.yaml ```

и запускаем командой ``` docker-compose up -d ``` 

должны запустится сервисы Grafana доступная на 3000 порту и loki доступный на 3100 порту.

![image](https://github.com/usmanofff/monitoring/assets/74288450/5a6acd67-9e94-4a48-8c30-8c7f57262c8d)

далее в k8s разворачиваем promtail : команда helm -n loki upgrade --install -- value promtail-config.yaml promtail grafana/promtail

promtail-config.yaml необходимо указать url для подключения к loki 

config: 
  clients:
     - url: http://51.250.84.184/:3100/loki/api/v1/push

     
Переходим в интерфейс Grafana -- Data Source --- Add new Data Source --- выбираем loki core 

![image](https://github.com/usmanofff/monitoring/assets/74288450/0c364c31-25ac-4449-8280-3372c2346c73)

далее указываем url для подключения к loki 

![image](https://github.com/usmanofff/monitoring/assets/74288450/c0f2272b-5a20-4ae7-8dc2-ee66966f9f8a)

нажимаем save and test 

![image](https://github.com/usmanofff/monitoring/assets/74288450/e5346363-7bfe-47e0-9014-f7ecc31a4070)

далее можно настроить дашборд для логов ``` https://grafana.com/grafana/dashboards/ ```

![image](https://github.com/usmanofff/monitoring/assets/74288450/49937d56-e26e-4f88-861a-247a67fc3c09)

Устанавливаем дашборд. 

смотрим логи: 

![image](https://github.com/usmanofff/monitoring/assets/74288450/72d9af0f-806a-4119-8912-fa5c9eecedb2)

![image](https://github.com/usmanofff/monitoring/assets/74288450/caa738b3-060c-4b3a-98c3-e1d2f1d981d3)


Делее необходимо получать метри с k8s для этого буду использовать prometheus-stack

устанавливаем в кластер подготовленные магнифесты node-exporter и сервиса для открытие доступа. 

``kubectl create prometheus && kubectl -n prometheus apply -f /kubernetes-node-exporter ```

![image](https://github.com/usmanofff/monitoring/assets/74288450/ec216924-351f-4d4c-981f-e0ca79db37ee)


Затем устанавливаем prometheus-stack для docker-compose на сервере. 











