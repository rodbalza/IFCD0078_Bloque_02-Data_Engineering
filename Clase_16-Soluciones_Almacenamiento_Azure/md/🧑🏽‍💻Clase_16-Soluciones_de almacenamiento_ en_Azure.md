# 🧑🏽‍💻 Clase 16 - Soluciones de almacenamiento en Azure

# 1. Soluciones de almacenamiento y bases de datos en Azure para DE

![image.png](image.png)

<aside>

Cuando trabajamos como Data Engineers, una de las primeras decisiones importantes no es  cómo hacer un dashboard, cómo entrenar un modelo de Machine Learning o cómo programar una transformación avanzada. Antes de todo eso, debemos responder una pregunta más básica: **¿Dónde van a vivir los datos?**

</aside>

![image.png](image%201.png)

> Al final de esta sesión se pretende entender en primera aproximación como desarrollar un pequeño pipeline como el que se ilustra:
> 

![image.png](image%202.png)

# 2. ¿Qué significa almacenar datos en Data Engineering?

<aside>

En el uso cotidiano, almacenar datos significa simplemente “guardar información”. Sin embargo, en Data Engineering el almacenamiento tiene una función mucho más importante.

</aside>

![image.png](image%203.png)

Por ejemplo, imaginemos una empresa de ventas online. Esta empresa puede tener:

![image.png](image%204.png)

<aside>

Todos estos datos no tienen la misma estructura ni el mismo uso. Por tanto, no deberían almacenarse necesariamente en el mismo lugar ni de la misma forma.

</aside>

---

# 3. Tipos de datos

Antes de elegir una tecnología, necesitamos entender qué tipo de datos tenemos.

## 3.1 Datos estructurados

<aside>

Son datos organizados en filas y columnas. Tienen un esquema claro.

</aside>

Ejemplo:

| cliente_id | nombre | ciudad | fecha_alta |
| --- | --- | --- | --- |
| 1 | Ana Pérez | Madrid | 2024-01-10 |
| 2 | Luis Gómez | Valencia | 2024-01-15 |

![image.png](image%205.png)

---

## 3.2 Datos semiestructurados

![image.png](56f251b5-23f4-4f67-8d63-db20f9f767dd.png)

![image.png](image%206.png)

![image.png](image%207.png)

---

## 3.3 Datos no estructurados

![image.png](91393597-1219-4d50-b49c-5f024b2291ac.png)

---

# 4. Servicios clave de almacenamiento en Azure

![image.png](e6d470f4-4285-4a4e-97b4-be8bfacd15ed.png)

# 5. Azure Storage Account

![image.png](0b60d011-8b85-4bd8-ad89-2ab3623e78c4.png)

---

# 6. Azure Blob Storage

![image.png](941a8ab8-255e-4ab6-82b8-06f4a67a9862.png)

## 6.1 ¿Cuándo usar Blob Storage?

![image.png](0eadc8a4-f40f-4eb7-b2f3-6da1f1d153e9.png)

## 6.2 Ejemplo

![image.png](66123141-cc2b-4e36-a32f-1a42b903bc49.png)

# 7. Azure Data Lake Storage Gen2

![image.png](6b5ea314-3144-40bf-9263-6299df178618.png)

## 7.1 Diferencia básica entre Blob Storage y ADLS Gen2

![image.png](image%208.png)

---

# 8. ¿Qué es un Data Lake?

![image.png](image%209.png)

## 8.1 Ejemplo

![image.png](37a8b2e7-5699-4190-8111-2a5ae44bfadb.png)

<aside>

#### Actividad 3.1 → Investiga lo siguiente:

Fabric tiene un servicio llamado **Onelake:
¿ Es un data lake ? ¿ Que features añade OneLake versus un Data Lake ?
¿ Está basado en ADLS Gen2 o no tienen nada que ver? 
Establece similitudes y diferencias.**

</aside>

---

# 9. Organización física de un Data Lake

## 9.1 Zona raw

<aside>

La zona `raw` contiene datos tal como llegan desde la fuente, o con muy poca modificación.

</aside>

![image.png](a85a7cf0-6791-4ed5-9dd5-70841fe9c2d0.png)

---

## 9.2 Zona bronze

<aside>

La zona `bronze` suele contener datos ya ingeridos y mínimamente estructurados.

</aside>

![image.png](image%2010.png)

---

![image.png](image%2011.png)

## 9.3 Zona silver

![image.png](image%2012.png)

---

## 9.4 Zona gold

![image.png](image%2013.png)

# 10. Convenciones de organización

![image.png](image%2014.png)

![image.png](image%2015.png)

---

# 11. Formatos de archivo

<aside>

No basta con decidir dónde guardar los datos. También debemos decidir **en qué formato** guardarlos.

</aside>

---

## 12.1 CSV

![image.png](image%2016.png)

---

## 12.2 JSON

![image.png](image%2017.png)

---

## 12.3 Parquet

![image.png](image%2018.png)

---

## 12.4 Delta Lake

![image.png](image%2019.png)

---

# 13. Particionado físico

![image.png](image%2020.png)

---

# 14. Bases de datos relacionales en Azure

<aside>

Una base de datos relacional organiza datos en tablas relacionadas entre sí.

</aside>

> En este módulo usamos **Azure SQL Database** como servicio principal de base de datos relacional en Azure.
> 

## 14.1 ¿Qué es Azure SQL Database?

![image.png](image%2021.png)

![image.png](image%2022.png)

---

# 17. ¿Cuándo mover datos desde Azure SQL hacia un Data Lake?

![image.png](image%2023.png)

---

# 18. Bases de datos NoSQL en Azure

<aside>

No todas las situaciones encajan bien con una base relacional. A veces los datos son más flexibles, variables o documentales.

</aside>

> En Azure, una opción importante es **Azure Cosmos DB**.
> 

## 18.1 ¿Qué es una base de datos NoSQL?

![image.png](image%2024.png)

![image.png](image%2025.png)

---

## 18.2 Azure Cosmos DB

![image.png](image%2026.png)

---

---

# 19. Conceptos básicos de Cosmos DB

![image.png](image%2027.png)

---

![image.png](image%2028.png)

---

# 21. Cosmos DB frente a Azure SQL Database

![image.png](image%2029.png)

---

# 22. Azure Data Factory (ADF)

<aside>

Azure Data Factory es una herramienta para orquestar flujos de datos entre distintos sistemas y transformarlos para análisis.

</aside>

## 🔧 ¿Qué permite hacer?

![image.png](image%2030.png)

## 🏗️ Componentes principales

![image.png](image%2031.png)

## 🔄 ETL vs ELT en ADF

![image.png](image%2032.png)

<aside>

#### Actividad 3.2 → Equivalente de ADF en Fabric

Fabric tiene un componente que se llama **Data Factory**. Define y explica que es Data Factory   ¿ Es lo mismo que ADF? 
Establece similitudes y diferencias y redacta tus conclusiones.
¿ Podria desde Fabric conectarme a datos En Azure/AWS/GCP? De que formas se puede hacer esto y cual sería la utilidad?

</aside>

# ¿ Que cosas vamos a practicar en este modulo con ADF?

En las prácticas vamos a usar **Azure Data Factory** como mecanismo mínimo de copia:

```
Azure SQL Database → Azure Data Factory → ADLS Gen2
Cosmos DB → Azure Data Factory → ADLS Gen2
Blob Storage -> ADF -> ADLS Gen2
```

<aside>

Es importante entender el límite: en este módulo **no estudiaremos pipelines completos ni ETL avanzado** porque esos contenidos se reservan para módulos posteriores.

</aside>

En este módulo lo usaremos únicamente  para demostrar una idea básica:

> Los datos no solo se almacenan en una fuente; muchas veces deben copiarse hacia un lugar analítico donde puedan ser reutilizados.
> 
> 
> ADF nos permite visualizar el movimiento de datos desde una fuente hacia un destino.
> 

![image.png](image%2033.png)

![image.png](image%2034.png)

---

<aside>

#### Actividad 3.3 Servicios equivalentes en AWS y GCP

En una infografia estilo tabla comparativa coloca los equivalentes de AWS GCP y Fabric de estos servicios de Azure:

- Azure SQL Database
- Azure Data Factory
- Azure Data Lake Storage Gen 2

La tabla debe contener el logo/icono identificativo de cada servicio

Al final de la tabla comparativa escribe un resumen de cada servicio de AWS y GCP que has puesto en la tabla.

**Formato de entrega:** Word, pdf, powerpoint o markdown (o enlace del repo de Github) 

</aside>