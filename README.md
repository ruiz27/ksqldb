
# 🚀 Ejemplo Básico con ksqlDB

Este proyecto es un ejercicio sencillo para demostrar cómo crear un **stream** en ksqlDB a partir de un topic de Kafka y consultarlo en tiempo real.

---

## 📋 Prerrequisitos

* [Docker](https://www.docker.com/get-started)
* [Docker Compose](https://docs.docker.com/compose/install/)

## Inicio Rápido

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

## Acceso web

Después de iniciar se puede acceder en:
```
http://localhost:9021/clusters
```

## Creacion topic

* **Vamos a crear un topic llamado `users` donde se publicarán los datos de nuestros usuarios.**

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

## Datos

* Data de ejemplo para insertar en el topic
``` json
Value:
{
  "id": 101,
  "user": "alice"
}

Key:
{
  "id": 101
}
```
## Consumidor 

* Crea un consumidor para asegurar que los mensajes lleguen al broker
```
kafka-console-consumer \
  --bootstrap-server localhost:9092 \
  --topic user \
  --from-beginning
```

## Stream

* Dentro de la consola de ksqlDB, ejecuta el siguiente comando SQL:

```
CREATE STREAM user_stream (
id INT,
user STRING
) WITH (
kafka_topic = 'user',
value_format = 'JSON',
partitions = 1
);
```


## Terminar el proceso
```
docker-compose down -v
```