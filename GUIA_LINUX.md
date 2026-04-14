# Guia para Linux

Esta guia explica como instalar Docker, levantar el stack y usar comandos basicos de HDFS y Hive.

## 1. Instalar Docker en Linux

Estas instrucciones funcionan bien en Ubuntu.

### Instalar Docker

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2
```

### Iniciar Docker

```bash
sudo systemctl enable docker
sudo systemctl start docker
```

### Opcional: usar Docker sin `sudo`

```bash
sudo usermod -aG docker $USER
```

Despues de eso, cierra sesion y vuelve a entrar.

### Verificar instalacion

```bash
docker --version
docker compose version
```

## 2. Preparar el proyecto

Primero clona el repositorio:

```bash
git clone https://github.com/cfocoder/academic_hadoop_hive.git
```

Luego entra a la carpeta del proyecto:

```bash
cd academic_hadoop_hive
```

Copia el archivo de variables:

```bash
cp .env.example .env
```

Abre `.env` y cambia esta linea:

```env
POSTGRES_PASSWORD=CHANGE_ME
```

Por una contrasena real, por ejemplo:

```env
POSTGRES_PASSWORD=MiPasswordSegura123
```

## 3. Levantar el stack

```bash
docker compose --env-file .env up -d
```

## 4. Revisar si ya quedo listo

```bash
docker compose ps
```

Estado esperado:

- `postgres` healthy
- `hive` healthy
- `hdfs-init` Exited (0)

Tambien puedes abrir:

- `http://localhost:9870`
- `http://localhost:8088`
- `http://localhost:10002`

## 5. Detener el stack

```bash
docker compose stop
```

## 6. Volver a encenderlo

```bash
docker compose start
```

## 7. Entrar a Hive desde terminal

### Abrir Beeline

```bash
docker compose exec hive beeline -u jdbc:hive2://hive:10000/default -n anonymous
```

### Comandos basicos dentro de Hive

```sql
show databases;
use default;
show tables;
```

Ejemplo:

```sql
CREATE DATABASE IF NOT EXISTS maestria_db;
USE maestria_db;
SHOW TABLES;
```

Salir de Beeline:

```sql
!quit
```

## 8. Comandos basicos de HDFS

### Listar directorios

```bash
docker compose exec hive hdfs dfs -ls /
docker compose exec hive hdfs dfs -ls /user
```

### Crear directorios

```bash
docker compose exec hive hdfs dfs -mkdir -p /user/tu_usuario
docker compose exec hive hdfs dfs -mkdir -p /user/tu_usuario/input
docker compose exec hive hdfs dfs -mkdir -p /user/tu_usuario/parquet
```

### Borrar archivos

```bash
docker compose exec hive hdfs dfs -rm /user/tu_usuario/input/archivo.txt
```

### Borrar directorios

```bash
docker compose exec hive hdfs dfs -rm -r /user/tu_usuario/input
```

### Ver contenido de un archivo

```bash
docker compose exec hive hdfs dfs -cat /user/tu_usuario/input/archivo.txt
```

## 9. Copiar archivos de tu maquina al contenedor

Para subir archivos a HDFS, primero hay que copiarlos al contenedor, normalmente a la carpeta `/tmp`, y despues moverlos desde ahi hacia HDFS.

La razon es que el contenedor funciona como un entorno separado, parecido a una maquina virtual ligera. Hadoop dentro del contenedor no puede leer directamente archivos que estan en tu computadora host. Por eso el flujo correcto es:

1. tu maquina local -> `/tmp` dentro del contenedor
2. `/tmp` dentro del contenedor -> HDFS

Primero obten el ID del contenedor `hive`:

```bash
docker compose ps -q hive
```

Ejemplo de resultado:

```bash
3f8c2a1b7d9e
```

En los siguientes comandos, reemplaza el ID del contenedor por tu valor real.

### Copiar un archivo local a `/tmp` dentro del contenedor

```bash
docker cp /ruta/local/archivo.txt 3f8c2a1b7d9e:/tmp/archivo.txt
```

Ejemplo:

```bash
docker cp /home/usuario/Downloads/warandpeace.txt 3f8c2a1b7d9e:/tmp/warandpeace.txt
```

### Copiar una carpeta completa a `/tmp`

```bash
docker compose exec hive mkdir -p /tmp/input
docker cp /home/usuario/Downloads/mis_archivos/. 3f8c2a1b7d9e:/tmp/input/
```

### Verificar que el archivo si llego al contenedor

```bash
docker compose exec hive ls -lh /tmp
docker compose exec hive ls -lh /tmp/input
```

Si prefieres hacerlo en un solo comando sin copiar manualmente el ID, tambien puedes usar esta version:

```bash
docker cp /ruta/local/archivo.txt $(docker compose ps -q hive):/tmp/archivo.txt
```

Pero para empezar, suele ser mas claro obtener el ID primero y luego pegarlo de forma explicita.

## 10. Subir archivos desde `/tmp` del contenedor hacia HDFS

### Subir un archivo

```bash
docker compose exec hive hdfs dfs -put /tmp/archivo.txt /user/tu_usuario/input/
```

### Subir varios archivos

```bash
docker compose exec hive bash -lc 'hdfs dfs -put /tmp/input/* /user/tu_usuario/input/'
```

### Verificar en HDFS

```bash
docker compose exec hive hdfs dfs -ls /user/tu_usuario/input
```

## 11. Crear tablas externas en Hive

### Tabla externa sobre archivos Parquet

```bash
docker compose exec hive beeline -u jdbc:hive2://hive:10000/default -n anonymous
```

Dentro de Beeline:

```sql
CREATE DATABASE IF NOT EXISTS maestria_db;
USE maestria_db;

CREATE EXTERNAL TABLE IF NOT EXISTS ejemplo_parquet (
  id INT,
  nombre STRING
)
STORED AS PARQUET
LOCATION '/user/tu_usuario/parquet/ejemplo';
```

### Tabla externa sobre archivos CSV

```sql
CREATE DATABASE IF NOT EXISTS maestria_db;
USE maestria_db;

CREATE EXTERNAL TABLE IF NOT EXISTS ejemplo_csv (
  id STRING,
  nombre STRING
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.OpenCSVSerde'
WITH SERDEPROPERTIES (
  "separatorChar" = ",",
  "quoteChar" = "\"",
  "escapeChar" = "\\"
)
STORED AS TEXTFILE
LOCATION '/user/tu_usuario/csv/ejemplo'
TBLPROPERTIES ("skip.header.line.count"="1");
```

### Probar la tabla

```sql
SELECT * FROM ejemplo_parquet LIMIT 10;
SELECT COUNT(*) FROM ejemplo_parquet;
```

## 12. Ejemplos rapidos de consultas Hive

```sql
show databases;
use maestria_db;
show tables;
describe ejemplo_parquet;
select * from ejemplo_parquet limit 10;
select count(*) from ejemplo_parquet;
```

## 13. Borrar tablas en Hive

```sql
DROP TABLE IF EXISTS ejemplo_parquet;
```

Si quieres borrar la base completa:

```sql
DROP DATABASE IF EXISTS maestria_db CASCADE;
```

## 14. Si algo no funciona

### Revisar estado de contenedores

```bash
docker compose ps
```

### Ver logs de Hive

```bash
docker compose logs hive
```

### Ver logs de inicializacion HDFS

```bash
docker compose logs hdfs-init
```
