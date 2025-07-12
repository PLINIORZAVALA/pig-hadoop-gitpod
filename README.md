# PIG + HADOOP + DOCKER en GITPOD (Ubuntu)

Este proyecto permite ejecutar scripts de Apache Pig sobre Hadoop completamente en la nube, utilizando contenedores Docker dentro de Gitpod.

---

## Requisitos previos

- Cuenta en [GitHub](https://github.com)
- Cuenta en [Gitpod](https://gitpod.io)
- Navegador en Ubuntu o cualquier SO compatible

---

## Estructura del proyecto

```

pig-hadoop-gitpod/
├── .gitpod.yml
├── docker-compose.yml
├── datasets/
│   └── ventas\_500.csv
├── pig/
│   └── scripts/
│       └── procesar\_ventas.pig

````

---

## Archivos clave

### `.gitpod.yml`

```yaml
image: gitpod/workspace-full

tasks:
  - init: docker-compose up -d && sleep 30
    command: echo "Entorno Hadoop + Pig levantado en la nube"

ports:
  - port: 50070
    onOpen: open-preview
    visibility: public
````

### `docker-compose.yml`

```yaml
version: '3'

services:
  namenode:
    image: suhothayan/hadoop-spark-pig-hive:2.9.2
    container_name: namenode
    ports:
      - "50070:50070"
    command: /etc/bootstrap.sh namenode

  datanode:
    image: suhothayan/hadoop-spark-pig-hive:2.9.2
    container_name: datanode
    depends_on:
      - namenode
    command: /etc/bootstrap.sh datanode

  pig:
    image: suhothayan/hadoop-spark-pig-hive:2.9.2
    container_name: pig
    depends_on:
      - namenode
      - datanode
    stdin_open: true
    tty: true
    volumes:
      - ./datasets:/datos
      - ./pig/scripts:/scripts
```

---

## PASOS PARA EJECUTAR EL PROYECTO

### 1. Abrir el proyecto en Gitpod

Accede al proyecto usando:

```
https://gitpod.io/#https://github.com/TU_USUARIO/pig-hadoop-gitpod
```

Reemplaza `TU_USUARIO` por tu nombre de usuario en GitHub.

---

### 2. Verifica que todos los contenedores estén activos

En la terminal de Gitpod:

```bash
docker ps
```

Debes ver: `namenode`, `datanode`, y `pig` activos.

---

### 3. Acceder al contenedor `pig`

```bash
docker exec -it pig bash
```

---

### 4. Subir archivo CSV al HDFS

Dentro del contenedor:

```bash
hdfs dfs -mkdir -p /datos
hdfs dfs -put -f /datos/ventas_500.csv /datos/
```

---

### 5. Ejecutar script de Pig

Asegúrate de que el archivo `procesar_ventas.pig` contenga lo siguiente:

```pig
ventas = LOAD '/datos/ventas_500.csv' USING PigStorage(',')  
    AS (id:int, nombre:chararray, monto:int, ciudad:chararray, categoria:chararray);

altas = FILTER ventas BY monto > 500;

por_ciudad = GROUP ventas BY ciudad;
total_ciudad = FOREACH por_ciudad GENERATE 
    group AS ciudad, 
    SUM(ventas.monto) AS total;

por_categoria = GROUP ventas BY categoria;
estadisticas = FOREACH por_categoria GENERATE 
    group AS categoria, 
    COUNT(ventas) AS cantidad, 
    AVG(ventas.monto) AS promedio;

ordenadas = ORDER altas BY monto DESC;

STORE total_ciudad INTO '/salida/por_ciudad' USING PigStorage(',');
STORE estadisticas INTO '/salida/por_categoria' USING PigStorage(',');
STORE ordenadas INTO '/salida/ventas_altas' USING PigStorage(',');
```

Luego ejecuta:

```bash
pig -x mapreduce /scripts/procesar_ventas.pig
```

---

### 6. Ver resultados en HDFS

```bash
hdfs dfs -cat /salida/por_ciudad/part-r-00000
hdfs dfs -cat /salida/por_categoria/part-r-00000
hdfs dfs -cat /salida/ventas_altas/part-r-00000
```

---

### 7. Ver interfaz web de Hadoop

En Gitpod, ejecuta:

```bash
gp url 50070
```

Esto abrirá la interfaz web del NameNode. Si ves el mensaje:

> Safe mode – awaiting reported blocks (0/31)

Puedes esperar unos segundos, o forzar la salida del modo seguro:

```bash
docker exec -it namenode hdfs dfsadmin -safemode leave
```

---

## ¿Qué logramos?

* Procesar un archivo CSV en Pig sobre Hadoop
* Usar contenedores Docker dentro de Gitpod
* Visualizar resultados en HDFS y en interfaz web
* Todo en la nube, sin instalar Hadoop localmente

---

## Créditos

Proyecto desarrollado por \[Plinior Zavala] — Basado en una guía para entornos educativos y de práctica en la nube con Pig + Hadoop.

```
