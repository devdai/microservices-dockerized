Создать три взаимодействующих между собой микросервиса МС1, МС2 и МС3. 
Микросервисы взаимодействую между собой следующим образом:
МС1 создает сообщение следующего формата:
```
{
	id: integer.
	“session_id”: integer,
        “MC1_timestamp”: datetime,
	“MC2_timestamp”: datetime,
	“MC3_timestamp”: datetime,
	“end_timestamp”: datetime
}
```

“session_id” - номер сеанса взаимодействия;

записывает в поле “MC1_timestamp” текущее время и отправляет сообщение в МС2 через WebSocket;

МС2 принимает сообщение от МС1, записывает в поле сообщения “МС2_timestamp” текущее время и отправляет сообщение в МС3 через топик брокера Kafka;

МС3 принимает сообщение от МС2, записывает в поле сообщения “МС3_timestamp” текущее время и отправляет сообщение в МС1 посредством отправки http запроса POST с телом, содержащим сообщение;

МС1 принимает сообщение от МС3, записывает в поле“end_timestamp” текущее время, записывает сообщение в базу данных;
повторить цикл взаимодействия в течение заданного интервала взаимодействия.

Длительность интервала взаимодействия задается в секундах параметром в конфигурационном файле.
Для целей трассировки микросервисов предусмотреть в микросервисах  отправку span-ов в систему Jaeger для каждого входящего сообщения.
В качестве БД использовать СУБД MariaDB. После остановки контейнеров с микросервисами и окружением база данных должна быть доступна для просмотра средствами СУБД.
Запуск микросервисов и окружения производить в docker-compose.
Старт взаимодействия осуществить отправкой запроса GET на /start/ без параметров в МС1.
Досрочную остановку взаимодействия осуществить отправкой запроса GET на /stop/ без параметров в МС1.
Начало взаимодействия микросервисов индицировать на консоль.
Завершение взаимодействия индицировать на консоль с выводом следующих параметров:
время взаимодействия;
количество сообщений, сгенерированных во время взаимодействия.
Добавить комментарии к коду и описание функций.



## docker-compose configuration.

### Runbook:

1) clone all 4 repositories in one directory
2) go to *mc1*, *mc2* and *mc3* and run `mvn clean package -DskipTests` in each project
3) go to **microservices-dockerized** (this) and run `docker-compose up --build`
4) Environment should be ready, check the logs for **mc1** to ensure
5) Access `http://localhost:8081/mc1/start` to start execution
6) Access `http://localhost:8081/mc1/stop` to stop execution

### Important!
I've set some conditions between services to start, but 
sometimes startup order can differ on different machines.
In case of such issue, this is the right order to start services:

``````
docker-compose up mariadb -d
docker-compose up jaeger -d
docker-compose up kafka -d
docker-compose up mc1 -d
//make sure mc1 is up and running before running next commands
docker-compose up mc3 -d
docker-compose up mc2 -d
``````

Yes, I know it
could be configured much better, using `/wait` or `dockerize` tools in Dockerfile .
Just didn't want to spend too much time on this yet. 
