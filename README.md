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

2) Criação do banco e tabelas

    ```SQL
    -- 1- Criação do database
    create database db;
    GO

    use db;
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
