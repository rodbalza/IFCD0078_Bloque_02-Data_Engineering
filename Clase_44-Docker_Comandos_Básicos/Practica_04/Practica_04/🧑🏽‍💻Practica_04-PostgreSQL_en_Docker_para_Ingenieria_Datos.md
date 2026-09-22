# 🧑🏽‍💻Practica 04 - PostgreSQL en Docker para Ingeniería de Datos

## Escenario

Una empresa de ingeniería de datos recibe diariamente archivos CSV procedentes de sus sistemas de ventas.

El equipo necesita crear rápidamente una base de datos temporal para:

```
ventas.csv
     ↓
PostgreSQL en Docker
     ↓
Tabla staging_ventas
     ↓
Consultas de validación
```

En lugar de instalar PostgreSQL directamente en Ubuntu Server, vamos a ejecutarlo dentro de un contenedor Docker.

Todo se realizará desde:

```
Windows
   ↓
VS Code
   ↓ SSH
Ubuntu Server
   ↓
Docker Engine
   ↓
PostgreSQL Container
```

Este laboratorio reutiliza los comandos principales aprendidos en la Clase 1 de Docker.

---

# 1. Comprobar Docker

Desde la terminal remota de **VS Code conectada a Ubuntu Server**:

```bash
docker --version
```

Comprobamos los contenedores actuales:

```bash
docker ps
```

Y las imágenes disponibles:

```bash
docker images
```

---

# 2. Descargar PostgreSQL

Vamos a utilizar la imagen oficial:

```bash
docker pull postgres:16
```

Comprobamos que se ha descargado:

```bash
docker images
```

Deberíamos encontrar algo parecido a:

```
REPOSITORY   TAG
postgres     16
```

Aquí estamos utilizando dos comandos vistos en la clase:

```
docker pull
docker images
```

---

# 3. Crear el contenedor PostgreSQL

Ejecutamos:

```bash
docker run -d --name postgres-data -e POSTGRES_PASSWORD=curso123 -e POSTGRES_DB=empresa -p 5432:5432 postgres:16
```

Vamos a analizar el comando.

## `-d`

```
-d
```

Ejecuta PostgreSQL en segundo plano.

## `-name postgres-data`

```
--name postgres-data
```

Asigna el nombre:

```
postgres-data
```

al contenedor.

## `p 5432:5432`

```
-p 5432:5432
```

Relaciona:

```
Puerto 5432 Ubuntu → Puerto 5432 PostgreSQL
```

## `e`

La opción:

```
-e
```

permite definir variables de entorno.

En este caso:

```bash
-e POSTGRES_PASSWORD=curso123
```

define la contraseña del usuario administrador de PostgreSQL.

Y:

```bash
-e POSTGRES_DB=empresa
```

hace que PostgreSQL cree inicialmente una base de datos llamada:

```
empresa
```

---

# 4. Comprobar que el contenedor está funcionando

Ejecuta:

```bash
docker ps
```

Deberíamos observar algo similar a:

```
CONTAINER ID   IMAGE         PORTS                    NAMES
abc123...      postgres:16   0.0.0.0:5432->5432/tcp   postgres-data
```

Ahora tenemos:

```
Ubuntu Server
       |
       | 5432
       ↓
Docker
       |
       ↓
PostgreSQL
       |
       ↓
Base de datos empresa
```

---

# 5. Consultar los logs

PostgreSQL tarda unos segundos en inicializarse.

Podemos observar el proceso con:

```bash
docker logs postgres-data
```

Entre los mensajes deberíamos terminar encontrando algo parecido a:

```
database system is ready to accept connections
```

También podemos seguir los logs en tiempo real:

```bash
docker logs -f postgres-data
```

Para salir:

```
Ctrl + C
```

El contenedor continuará funcionando.

---

# 6. Entrar en PostgreSQL

Ahora utilizamos `docker exec`.

Ejecuta:

```bash
docker exec -it postgres-data psql -U postgres -d empresa
```

Estamos haciendo lo siguiente:

```
docker exec
       ↓
contenedor postgres-data
       ↓
ejecutar programa psql
       ↓
conectarse a BD empresa
```

El prompt debería cambiar a algo parecido a:

```
empresa=#
```

Ya estamos dentro de PostgreSQL.

---

# 7. Crear una tabla de Staging

Dentro de PostgreSQL:

```sql
CREATE TABLE staging_ventas (
    id INTEGER,
    fecha DATE,
    producto VARCHAR(100),
    cantidad INTEGER,
    precio NUMERIC(10,2)
);
```

Comprobamos la tabla:

```sql
SELECT * FROM staging_ventas;
```

Todavía estará vacía.

Salimos:

```
\q
```

---

# 8. Crear un pequeño dataset CSV

Ahora estamos nuevamente en Ubuntu Server.

Vamos a crear un archivo de datos:

```bash
echo "id,fecha,producto,cantidad,precio" > ventas.csv
```

Añadimos algunas ventas:

```bash
echo "1,2026-09-01,Portatil,2,1200.00" >> ventas.csv
```

```bash
echo "2,2026-09-01,Monitor,5,350.00" >> ventas.csv
```

```bash
echo "3,2026-09-02,Teclado,10,75.00" >> ventas.csv
```

```bash
echo "4,2026-09-02,Raton,15,35.00" >> ventas.csv
```

```bash
echo "5,2026-09-03,Portatil,1,1350.00" >> ventas.csv
```

Visualizamos:

```bash
cat ventas.csv
```

Resultado:

```
id,fecha,producto,cantidad,precio
1,2026-09-01,Portatil,2,1200.00
2,2026-09-01,Monitor,5,350.00
3,2026-09-02,Teclado,10,75.00
4,2026-09-02,Raton,15,35.00
5,2026-09-03,Portatil,1,1350.00
```

Aquí tenemos nuestro pequeño **dataset de origen**.

---

# 9. Copiar el CSV al contenedor

Utiliza:

```bash
docker cp ventas.csv postgres-data:/tmp/ventas.csv
```

El flujo es:

```
Ubuntu Server
ventas.csv
     |
     | docker cp
     ↓
Contenedor PostgreSQL
/tmp/ventas.csv
```

Podemos comprobar que llegó:

```bash
docker exec postgres-data ls /tmp
```

Debería aparecer:

```
ventas.csv
```

También podemos visualizarlo desde fuera del contenedor:

```bash
docker exec postgres-data cat /tmp/ventas.csv
```

---

# 10. Cargar el CSV en PostgreSQL

Entramos nuevamente:

```bash
docker exec -it postgres-data psql -U postgres -d empresa
```

Ejecutamos:

```sql
COPY staging_ventas
FROM '/tmp/ventas.csv'
DELIMITER ','
CSV HEADER;
```

PostgreSQL debería indicar:

```
COPY 5
```

Eso significa que ha cargado:

```
5 filas
```

---

# 11. Validar los datos

En Ingeniería de Datos no basta con cargar información.

Hay que comprobarla.

Ejecutamos:

```sql
SELECT * FROM staging_ventas;
```

## Contar registros

```sql
SELECT COUNT(*)
FROM staging_ventas;
```

Resultado esperado:

```
5
```

## Calcular ventas

```sql
SELECT
    producto,
    SUM(cantidad * precio) AS importe_ventas
FROM staging_ventas
GROUP BY producto
ORDER BY importe_ventas DESC;
```

Ahora ya estamos realizando una pequeña transformación analítica:

```
CSV
 ↓
Staging
 ↓
Validación
 ↓
Agregación
```

---

# 12. Salir de PostgreSQL

```
\q
```

---

# 13. Inspeccionar el contenedor

Utilizamos otro comando de la Clase 1:

```bash
docker inspect postgres-data
```

Busca visualmente información relacionada con:

```
IPAddress
Ports
State
Image
Name
```

---

# 14. Consultar el puerto

```bash
docker port postgres-data
```

Deberíamos obtener algo parecido a:

```
5432/tcp -> 0.0.0.0:5432
```

Es decir:

```
PostgreSQL
Container :5432
      ↑
      |
Ubuntu :5432
```

---

# 15. Consultar recursos utilizados

Ejecuta:

```bash
docker stats postgres-data
```

Podemos observar:

```
CPU %
MEM USAGE
MEM %
NET I/O
```

Para salir:

```
Ctrl + C
```

---

# 16. Detener PostgreSQL

Ejecuta:

```bash
docker stop postgres-data
```

Comprobamos:

```bash
docker ps
```

Ya no aparecerá.

Pero si ejecutamos:

```bash
docker ps -a
```

seguirá existiendo:

```
postgres-data
```

con estado similar a:

```
Exited
```

> **Detener un contenedor no significa eliminarlo.**
> 

---

# 17. Volver a iniciar PostgreSQL

Ejecuta:

```bash
docker start postgres-data
```

Comprobamos:

```bash
docker ps
```

PostgreSQL vuelve a estar funcionando.

---

# 18. Comprobar si los datos siguen allí

Ejecutamos:

```bash
docker exec -it postgres-data psql -U postgres -d empresa
```

Y después:

```sql
SELECT * FROM staging_ventas;
```

Los datos siguen presentes porque simplemente hemos detenido e iniciado **el mismo contenedor**.

Salimos:

```
\q
```

Esto refuerza la diferencia entre:

```
docker stop
      ↓
contenedor permanece

docker start
      ↓
volvemos a utilizarlo
```

---

# 19. Reiniciar PostgreSQL

Podemos hacerlo directamente:

```bash
docker restart postgres-data
```

Comprobamos:

```bash
docker ps
```

---

# 20. Eliminar el contenedor

Primero:

```bash
docker stop postgres-data
```

Después:

```bash
docker rm postgres-data
```

Comprobamos:

```bash
docker ps -a
```

`postgres-data` ya no existe.

Aquí aparece una lección importante para futuras clases:

> Los datos estaban almacenados dentro del contenedor. Al eliminar el contenedor, esos datos dejan de estar disponibles con él.
> 

Esto prepara el siguiente tema:

```
Docker Volumes
```

porque allí aprenderemos cómo conseguir:

```
Eliminar contenedor
       ↓

Datos sobreviven
       ↓

Crear otro contenedor
       ↓

Recuperar los mismos datos
```

---

# 21. La imagen PostgreSQL todavía existe

Aunque hayamos eliminado el contenedor:

```bash
docker images
```

seguiremos teniendo:

```
postgres:16
```

Esto refuerza nuevamente:

```
IMAGEN ≠ CONTENEDOR
```

La imagen es la plantilla.

El contenedor era una instancia creada a partir de ella.

---

# 22. Eliminar la imagen

Si queremos limpiar completamente:

```bash
docker rmi postgres:16
```

Comprobamos:

```bash
docker images
```

---

# 23. Comandos de la Clase 1 utilizados

Este laboratorio utiliza casi todos los comandos principales:

| Comando | Uso dentro del laboratorio |
| --- | --- |
| `docker --version` | Comprobar instalación |
| `docker pull` | Descargar PostgreSQL |
| `docker images` | Ver la imagen |
| `docker run` | Crear PostgreSQL |
| `docker ps` | Ver PostgreSQL activo |
| `docker ps -a` | Ver activo/detenido |
| `docker logs` | Revisar inicialización |
| `docker exec` | Ejecutar SQL dentro del contenedor |
| `docker cp` | Introducir el CSV |
| `docker inspect` | Examinar configuración |
| `docker port` | Consultar publicación 5432 |
| `docker stats` | Consultar recursos |
| `docker stop` | Detener PostgreSQL |
| `docker start` | Iniciarlo otra vez |
| `docker restart` | Reiniciarlo |
| `docker rm` | Eliminar el contenedor |
| `docker rmi` | Eliminar la imagen |

---

---

# 25. Flujo completo del laboratorio

```
ventas.csv
    │
    │ docker cp
    ▼
┌──────────────────────────┐
│ Docker Container         │
│                          │
│ PostgreSQL               │
│ ┌──────────────────────┐ │
│ │ staging_ventas       │ │
│ │                      │ │
│ │ id                   │ │
│ │ fecha                │ │
│ │ producto             │ │
│ │ cantidad             │ │
│ │ precio               │ │
│ └──────────────────────┘ │
└──────────────────────────┘
             │
             │ SQL
             ▼
      Validación
             │
             ▼
      Transformación
             │
             ▼
       Datos analíticos
```

---