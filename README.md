
# Ejemplo Básico con ksqlDB

Este proyecto es un ejercicio sencillo para demostrar cómo crear un **stream** en ksqlDB a partir de un topic de Kafka y consultarlo en tiempo real.

---

## 📦 Prerrequisitos

* [Docker](https://www.docker.com/get-started)
* [Docker Compose](https://docs.docker.com/compose/install/)


## 🚀 Inicio Rápido

1. Creacion de directorio
```
 mkdir -p ./kraft-data
```
2. Permisos de escritura
```
sudo chown -R 1000:1000 ./kraft-data
```
3. Levantar servicio con docker compose 
```
docker compose up -d
```

## 🌐 Acceso a interfaz web

Después de iniciar se puede acceder en:
```
http://localhost:9021/clusters
```

## 🧵Creacion topic

* **Vamos a crear un topic llamado `user` donde se publicarán los datos de nuestros usuarios.**

``` 
kafka-topics --create \
  --bootstrap-server localhost:9092 \
  --topic user \
  --partitions 1 \
  --replication-factor 1 \
  --config retention.ms=604800000  # 7 días
```

* **(Opcional) Verificar la configuración del topic:**

```
kafka-configs --bootstrap-server localhost:9092 \
  --entity-type topics --entity-name user \
  --describe  
```

## 📥 Publicar Datos

* **Data de ejemplo para insertar en el topic**
``` json
Value:
{
  "id": 101,
  "user": "alice",
  "content": "hi"
}

Key:
{
  "id": 101
}
```
## 📤 Consumidor 

* **Crea un consumidor para asegurar que los mensajes lleguen al broker**
```
kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic user \
  --from-beginning
```

## 🔄 Crear un Stream en KsqlDB

* **Dentro de la consola de ksqlDB, ejecuta el siguiente comando SQL:**

```
CREATE STREAM rrss_stream (
  id INT,
  user VARCHAR,
  content VARCHAR
) 
WITH (
  KAFKA_TOPIC = 'rrss_topic',
  VALUE_FORMAT = 'JSON'
);
```

* **Verificación de datos del stream** 
 ``` sql
select * from rrss_stream EMIT CHANGES;
```

## 📊 Crear una tabla agregada 
* **Creacion de la tabla para agregar la información que pasa por el stream**
``` sql 
CREATE TABLE alice_counts_per_minute AS
SELECT
user,
COUNT(*) AS alice_count
FROM rrss_stream
WINDOW TUMBLING (SIZE 1 MINUTE)
WHERE user = 'alice'
GROUP BY user;
```

* **Consultar resultado**
``` sql
SELECT * FROM alice_counts_per_minute EMIT CHANGES;
```

## 🛑 Terminar el proceso
```
docker-compose down -v
```