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
    CREATE DATABASE TesteEF

    Use TesteEF
    GO

    -- 2- Criação da tabela
    CREATE TABLE Person (
        Id INT IDENTITY(1,1) PRIMARY KEY,
        Name VARCHAR(100) NOT NULL,
        Address VARCHAR(200) NOT NULL,
        Phone VARCHAR(11)
    )

    -- Habilitar o CDC
    EXEC sys.sp_cdc_enable_db
    GO


    -- Criar o CDC para a tabela Person
    EXEC sys.sp_cdc_enable_table
    @source_schema = N'dbo',
    @source_name   = N'Person',
    @role_name     = N'Admin',
    @supports_net_changes = 1
    GO
    ```


Configurar conector

```PowerShell
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" http://localhost:8083/connectors/ -d \
'{
    "name": "db-connector",
    "config": {
        "connector.class" : "io.debezium.connector.sqlserver.SqlServerConnector",
        "tasks.max" : "1",
        "database.server.name" : "sqlserver",
        "database.hostname" : "sqlserver",
        "database.port" : "1433",
        "database.user" : "sa",
        "database.password" : "P@ssw0rd!",
        "database.dbname" : "db",
		"table.whitelist": "dbo.employee",
        "database.history.kafka.bootstrap.servers" : "kafka:9092",
        "database.history.kafka.topic": "sqlserver.dbo.employee"
    }
}'                                                                                  
```


Excluir conector

```
curl -X DELETE http://localhost:8083/connectors/<nome-db-connector>
ex:
curl -X DELETE http://localhost:8083/connectors/db-connector
```
