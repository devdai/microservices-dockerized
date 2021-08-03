## NiceTask docker-compose configuration.

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
