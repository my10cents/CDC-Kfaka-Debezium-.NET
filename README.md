# CDC-Kfaka-Debezium-.NET
https://medium.com/jundevelopers/debezium-kafka-e-net-core-9cee3ca3e0db

https://medium.com/xp-inc/dica-r%C3%A1pida-kafka-cdc-sql-server-docker-dfaff87a3ca2

1) Install SQL Server - Docker
    ```PowerShell
    docker run -p 1433:1433 
               -e ACCEPT_EULA=Y 
               -e SA_PASSWORD=P@ssw0rd! 
               -e MSSQL_PID=Developer 
               -e MSSQL_AGENT_ENABLED=true 
               -d 'mcr.microsoft.com/mssql/server:2019-latest'
    ```

1) Criar infra com docker compose
	```PowerShell
	version: '3.1'

	services:

	  sqlserver:
	    container_name: "sqlserver.local"  
	    image: mcr.microsoft.com/mssql/server:2019-latest
	    ports:
	      - 1433:1433
	    environment: 
	      MSSQL_AGENT_ENABLED: "true"
	      MSSQL_PID: Standard
	      SA_PASSWORD: P@ssw0rd!
	      ACCEPT_EULA: "Y"
	      
	  zookeeper:
	    container_name: zookeeper.local
	    image: confluentinc/cp-zookeeper:latest
	    ports:
	      - "2181:2181"
	    environment:
	      ZOOKEEPER_CLIENT_PORT: 2181

	  kafka:
	    container_name: kafka.local
	    image: confluentinc/cp-kafka:latest
	    depends_on:
	      - zookeeper
	      #- mysql
	      - sqlserver
	    ports:
	      - "9093:9093"
	    environment:
	      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
	      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:9093
	      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
	      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
	      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
	      KAFKA_LOG_CLEANER_DELETE_RETENTION_MS: 5000
	      KAFKA_BROKER_ID: 1
	      KAFKA_MIN_INSYNC_REPLICAS: 1

	  connector:
	    container_name: debezium.local
	    image: debezium/connect:0.10
	    ports:
	      - "8083:8083"
	    environment:
	      GROUP_ID: 1
	      CONFIG_STORAGE_TOPIC: my_connect_configs
	      OFFSET_STORAGE_TOPIC: my_connect_offsets
	      BOOTSTRAP_SERVERS: kafka:9092
	    depends_on:
	      - zookeeper
	      #- mysql
	      - sqlserver
	      - kafka
	```

3) Criação do banco e tabelas

    ```SQL
    -- 1- Criação do database
    create database dbstore;
    GO

    use dbstore;
    GO

    create table products 
    (
	id int IDENTITY primary key,
	name varchar(50), 
	price int--, 
	--creation_time datetime default current_timestamp, 
	--modification_time datetime on update current_timestamp
    );

    insert into products(name, price) values("Red T-Shirt", 12);

    -- Habilitar o CDC
    EXEC sys.sp_cdc_enable_db
    GO


    -- Criar o CDC para a tabela Person
    EXEC sys.sp_cdc_enable_table
    @source_schema = N'dbo',
    @source_name   = N'products',
    @role_name     = N'Admin',
    @supports_net_changes = 1
    GO
    ```


Configurar conector

```PowerShell
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d \
'{ 
    "name": "db-store-connector", 
    "config": 
    { 
        "connector.class" : "io.debezium.connector.sqlserver.SqlServerConnector", 
        "tasks.max": "1", 
        "database.hostname": "sqlserver", 
        "database.port": "1433", 
        "database.user": "sa", 
        "database.password": "P@ssw0rd!",
        "database.server.name": "sqlserver", 
        "database.dbname" : "dbstore",
        "table.whitelist": "dbo.products", 
        "database.history.kafka.bootstrap.servers": "kafka:9092", 
        "database.history.kafka.topic": "dbhistory.dbstore",
    } 
}'                                                                                
```


Excluir conector

```
curl -X DELETE http://localhost:8083/connectors/<nome-db-connector>
ex:
curl -X DELETE http://localhost:8083/connectors/db-store-connector
```

Teste do container se kafka esta recebendo mensagem
```
 docker exec -it 96954b7fa94b sh /usr/bin/kafka-console-consumer --bootstrap-server kafka:9092 --topic sqlserver.dbo.products --from-beginning
```

Visualizar configuraçoes connector
```
http://localhost:8083/connectors/db-store-connector
```
Checar status do conector
```
http://localhost:8083/connectors/db-store-connector/status
```
