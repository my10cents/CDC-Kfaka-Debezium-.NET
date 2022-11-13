# CDC-Kfaka-Debezium-.NET
https://medium.com/jundevelopers/debezium-kafka-e-net-core-9cee3ca3e0db
https://medium.com/xp-inc/dica-r%C3%A1pida-kafka-cdc-sql-server-docker-dfaff87a3ca2


Configurar conector

```
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
        "database.password" : "p@ssw0rd",                                            
        "database.dbname" : "db",                                                    
        "database.history.kafka.bootstrap.servers" : "kafka:9092",                   
        "database.history.kafka.topic": "schema-changes.db"                           
    }                                                                                
}'                                                                                   
```


Excluir conector

```
curl -X DELETE http://localhost:8083/connectors/<nome-db-connector>
ex:
curl -X DELETE http://localhost:8083/connectors/db-connector
```
