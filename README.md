### Eventuate Local 서비스를 실행하도록 Docker 구성

[Eventuate Local](https://github.com/eventuate-local/eventuate-local) 은 Zookeeper, Kafka, MySQL 및 CDC 서비스와 같은 여러 서비스로 구성됩니다. 이러한 서비스를 실행하는 가장 쉬운 방법은 [Docker Compose를 사용하는 것](https://docs.docker.com/compose/) 입니다. 이 docker-compose.yml ( [github 버전](https://github.com/eventuate-local/eventuate-local/blob/master/docker-compose-eventuate-local.yml) ) 파일을 사용할 수 있습니다 .



### mac 환경설정

* vi ~/.zshrc

```javascript
if [ -z "$DOCKER_HOST_IP" ] ; then
    if [ -z "$DOCKER_HOST" ] ; then
      export DOCKER_HOST_IP=`hostname`
    else
      echo using ${DOCKER_HOST?}
      XX=${DOCKER_HOST%\:*}
      export DOCKER_HOST_IP=${XX#tcp\:\/\/}
    fi
fi

echo DOCKER_HOST_IP is $DOCKER_HOST_IP

export SPRING_DATASOURCE_URL=jdbc:mysql://${DOCKER_HOST_IP}/eventuate
export SPRING_DATASOURCE_USERNAME=mysqluser
export SPRING_DATASOURCE_PASSWORD=mysqlpw
export SPRING_DATASOURCE_DRIVER_CLASS_NAME=com.mysql.jdbc.Driver
export EVENTUATELOCAL_KAFKA_BOOTSTRAP_SERVERS=$DOCKER_HOST_IP:9092
export EVENTUATELOCAL_CDC_DB_USER_NAME=root
export EVENTUATELOCAL_CDC_DB_PASSWORD=rootpassword
export EVENTUATELOCAL_ZOOKEEPER_CONNECTION_STRING=$DOCKER_HOST_IP:2181

echo SPRING_DATASOURCE_URL=$SPRING_DATASOURCE_URL
```

### Eventuate Local을 사용하여 빌드 및 실행 

먼저 애플리케이션을 빌드합니다.

```java
./gradlew assemble
```

다음으로 Docker Compose를 사용하여 애플리케이션을 시작할 수 있습니다.

```java
./gradlew <database-mode>ComposeBuild
./gradlew <database-mode>ComposeUp
```

모드는 다음 3가지중에 선택합니다.

- `mysqlbinlog` -Binlog 기반 이벤트 게시와 함께 MySQL 사용
- `postgreswal` -Postgres WAL 기반 이벤트 게시와 함께 Postgres 사용
- `postgrespolling` -일반 JDBC 폴링 기반 이벤트 게시와 함께 Postgres 사용

**예시)  ./gradlew mysqlbinlogComposeBuild**

마지막으로 서비스에서 제공하는 Swagger UI를 사용하여 고객 및 주문을 생성하고 주문 내역을 볼 수 있습니다.

- `http://localhost:8081/swagger-ui.html` -고객 만들기
- `http://localhost:8083/swagger-ui.html` -주문 생성
- `http://localhost:8082/swagger-ui.html` -고객 및 주문보기