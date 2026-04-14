# Hadoop + Hive para la materia de "Almacenamiento de Grandes Bases de Datos" de la Maestria en Ciencia de Datos de CUCUEA con Docker

Este repositorio contiene una version simplificada de Hadoop + Hive para uso academico en una sola computadora.

La idea es que el entorno sea:

- mas estable que una instalacion nativa
- mas facil de arrancar y detener
- persistente, para no perder datos si los contenedores se recrean

## Que incluye

- Hadoop 3.3.6
- Hive 3.1.3
- HiveServer2
- PostgreSQL como metastore de Hive

## Archivos principales

- [docker-compose.yml](docker-compose.yml)
- [.env.example](.env.example)
- [GUIA_LINUX.md](GUIA_LINUX.md)
- [GUIA_WINDOWS.md](GUIA_WINDOWS.md)

## Persistencia de datos

Este proyecto usa **named volumes** de Docker:

- `postgres-data`
- `hadoop-data`

Eso significa que si apagas o recreas los contenedores, los datos siguen existiendo mientras no borres los volumenes.

No uses este comando si quieres conservar tu informacion:

```bash
docker compose down -v
```

Ese comando borra tambien los volumenes.

## Puertos que usa el stack

- `5432` PostgreSQL
- `9820` HDFS IPC
- `9870` HDFS Web UI
- `8088` YARN UI
- `8042` NodeManager UI
- `9083` Hive Metastore
- `10000` HiveServer2
- `10002` HiveServer2 Web UI
- `19888` JobHistory UI

## URLs utiles

- HDFS: `http://localhost:9870`
- YARN: `http://localhost:8088`
- HiveServer2 UI: `http://localhost:10002`

Conexion JDBC:

```text
jdbc:hive2://localhost:10000/default
```

## Primeros pasos

1. Copia `.env.example` a `.env`
2. Cambia `POSTGRES_PASSWORD`
3. Levanta el stack

En Linux:

```bash
docker compose --env-file .env up -d
```

En Windows se recomienda usar **PowerShell** en lugar de `cmd.exe`, porque los comandos de copia de archivos y manejo de rutas funcionan mejor ahi.

En Windows, antes de ejecutar cualquier comando, abre **Docker Desktop** y dejalo corriendo. Si Docker Desktop no esta abierto, los comandos de terminal no van a funcionar.

En Windows PowerShell:

```powershell
docker compose --env-file .env up -d
```

## Estado esperado

Despues del arranque, deberias ver:

- `postgres` en estado `healthy`
- `hive` en estado `healthy`
- `hdfs-init` en estado `Exited (0)`

En el primer arranque puede tardar varios minutos. No intentes conectarte de inmediato. Primero revisa que los servicios ya terminaron de levantar.

Para revisarlo:

Linux:

```bash
docker compose ps
```

Windows:

```powershell
docker compose ps
```

## Recomendacion practica

No dejes este stack corriendo todo el tiempo. Lo mejor es:

1. encenderlo cuando lo vayas a usar
2. esperar a que `hive` este `healthy`
3. trabajar
4. detenerlo al terminar

Para detenerlo:

Linux:

```bash
docker compose stop
```

Windows:

```powershell
docker compose stop
```

En Windows, cuando ya termines de trabajar, ademas de detener el stack, puedes cerrar Docker Desktop.

## Guias completas

Para instrucciones paso a paso, usa la guia correspondiente a tu sistema:

- [GUIA_LINUX.md](GUIA_LINUX.md)
- [GUIA_WINDOWS.md](GUIA_WINDOWS.md)

En Windows, la guia esta escrita para **PowerShell**. Aunque Docker tambien puede usarse desde `cmd.exe`, no es la opcion recomendada para este proyecto.

## Nota importante

Esta version esta pensada para:

- tareas de clase
- practicas con HDFS
- tablas externas en Hive
- conexion desde Beeline o DBeaver

No esta pensada para produccion.
