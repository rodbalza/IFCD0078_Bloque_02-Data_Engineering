# 🧑🏽‍💻 Clase 21 - Ingesta EL desde Blob Storage hasta ADLS Gen2 via ADF

# **Ejercicio 3: Ingesta EL de archivos históricos desde Azure Blob Storage hacia ADLS Gen2**

## **1. Contexto**

![image.png](image.png)

## **2. Arquitectura**

![image.png](image%201.png)

## **3. Nombres recomendados**

![image.png](image%202.png)

# **Fase 1. Preparar los archivos**

![image.png](image%203.png)

# **Fase 2. Crear el grupo de recursos**

1. Accede a Azure Portal.
2. Busca **Resource groups** o **Grupos de recursos**.
3. Selecciona **Create**.
4. Configura:
    
    ```
    Resource group: rg-practica3
    Region: una región permitida por la suscripción
    ```
    
5. Selecciona **Review + create**.
6. Selecciona **Create**.
    
    ![image.png](image%204.png)
    

Usa la misma región para todos los recursos del laboratorio.

# **Fase 3. Crear Azure Blob Storage como origen**

## **Paso 1. Crear la cuenta**

1. Busca **Storage accounts**.
2. Selecciona **Create**.
3. Configura:
    
    ```
    Resource group: rg-practica3
    Storage account name: stblobpractica3<identificador>
    Region: la misma región
    Performance: Standard
    Redundancy: LRS
    Primary service: Azure Blob Storage
    ```
    
    ![image.png](image%205.png)
    
4. En **Advanced**, deja desactivado:
    
    ```
    Enable hierarchical namespace
    ```
    
    La cuenta funcionará como almacenamiento de objetos estándar, no como Data Lake jerárquico.
    
5. Para el laboratorio, permite acceso de red público desde todas las redes.
6. Selecciona **Review + create** y después **Create**.

![image.png](image%206.png)

## **Paso 2. Crear el contenedor**

1. Abre la cuenta de almacenamiento.
2. Ve a **Data storage → Containers**.
    
    ![image.png](image%207.png)
    
3. Selecciona **+ Container**.
4. Configura:
    
    ```
    Name: historico
    Public access level: Private
    ```
    
    ![image.png](image%208.png)
    
5. Selecciona **Create**.

## **Paso 3. Subir los archivos**

1. Abre el contenedor `historico`.
2. Crea o utiliza la carpeta virtual:
    
    ```
    ventas/
    ```
    
3. Sube:
    
    ```
    ventas_2020.csv
    ventas_2021.csv
    ventas_2022.csv
    ```
    

La estructura debe quedar:

```
historico/
└── ventas/
    ├── ventas_2020.csv
    ├── ventas_2021.csv
    └── ventas_2022.csv
```

![image.png](image%209.png)

---

# **Fase 4. Crear ADLS Gen2 como destino**

## **Paso 1. Crear la cuenta**

1. Busca **Storage accounts**.
2. Selecciona **Create**.
3. Configura:
    
    ```
    Resource group: rg-practica3
    Storage account name: stadlspractica3<identificador>
    Region: la misma región
    Performance: Standard
    Redundancy: LRS
    ```
    
    ![image.png](image%2010.png)
    
4. En **Advanced**, activa:
    
    ```
    Enable hierarchical namespace
    ```
    
    Esta opción convierte la cuenta en ADLS Gen2 y permite trabajar con una jerarquía real de directorios.
    
5. Para el laboratorio, permite acceso de red público desde todas las redes.
6. Selecciona **Review + create** y después **Create**.
    
    ![image.png](image%2011.png)
    

## **Paso 2. Crear el file system**

1. Abre la cuenta de ADLS Gen2.
2. Ve a **Data storage → Containers**.
3. Selecciona **+ Container**.
4. Crea:
    
    ```
    datalake
    ```
    
5. Dentro de `datalake`, crea la estructura inicial:
    
    ```
    raw/
    └── ventas/
        └── historico/
    ```
    
    ADF creará las subcarpetas de fecha y ejecución.
    

![image.png](image%2012.png)

# **Fase 5. Crear Azure Data Factory**

1. Busca **Data factories**.
2. Selecciona **Create**.
3. Configura:
    
    ```
    Resource group: rg-practica3
    Name: adf-practica3
    Region: la misma región
    Version: V2
    Git configuration: Configure later
    ```
    
4. Selecciona **Review + create**.
5. Selecciona **Create**.
    
    ![image.png](image%2013.png)
    
6. Cuando termine el despliegue, selecciona **Launch Studio**.
    
    ![image.png](image%2014.png)
    

# **Fase 6. Autorizar la identidad administrada de ADF**

> Un **Linked Service** representa la conexión. La **Managed Identity** es el mecanismo de autenticación usado por esa conexión.
> 

## **Paso 1. Permiso de lectura en el origen**

1. Abre la cuenta `stblobpractica3`.
2. Ve a **Access control (IAM)**.
3. Selecciona **Add → Add role assignment**.
4. Elige:
    
    ```
    Storage Blob Data Reader
    ```
    
5. En miembros, selecciona:
    
    ```
    Managed identity
    ```
    
6. Busca el tipo **Data Factory** y selecciona `adf-practica3`.
    
    ![image.png](image%2015.png)
    
7. Confirma la asignación.

## **Paso 2. Permiso de escritura en el destino**

1. Abre `stadlspractica3`.
2. Ve a **Access control (IAM)**.
3. Añade el rol:
    
    ```
    Storage Blob Data Contributor
    ```
    
4. Asígnalo a la identidad administrada de la misma Data Factory.
5. Espera entre uno y cinco minutos para que se propaguen los permisos.

> Para este laboratorio, la asignación se realiza a nivel de cuenta. En producción conviene limitar el acceso al contenedor o directorio necesario y complementar con ACL de ADLS Gen2.
> 

# **Fase 7. Crear el Linked Service del Blob Storage de origen**

En Data Factory Studio:

1. Abre **Manage**.
2. Selecciona **Linked services**.
3. Selecciona **New**.
4. Busca:
    
    ```
    Azure Blob Storage
    ```
    
5. Selecciona **Continue**.
6. Configura:

```
Name: ls_blob_historico_practica3
Authentication type: System-assigned managed identity
Account selection method: From Azure subscription
Azure subscription: la suscripción del curso
Storage account name: stblobpractica3
```

1. Selecciona **Test connection**.
2. Debe aparecer **Connection successful**.
3. Selecciona **Create**.
    
    ![image.png](image%2016.png)
    

# **Fase 8. Crear el Linked Service de ADLS Gen2**

1. En **Manage → Linked services**, selecciona **New**.
2. Busca:
    
    ```
    Azure Data Lake Storage Gen2
    ```
    
3. Selecciona **Continue**.
4. Configura:
    
    ```
    Name: ls_adls_practica3
    Authentication type: System-assigned managed identity
    Account selection method: From Azure subscription
    Azure subscription: la suscripción del curso
    Storage account name: stadlspractica3
    ```
    
5. Selecciona **Test connection**.
6. Selecciona **Create**.

![image.png](image%2017.png)

# **Fase 9. Crear el dataset Binary de origen**

Usaremos el formato **Binary** para que ADF copie los bytes de los archivos sin analizarlos ni reescribirlos.

1. Abre **Author**.
2. En **Datasets**, selecciona **New dataset**.
3. Selecciona:
    
    ```
    Azure Blob Storage
    ```
    
    ![image.png](image%2018.png)
    
4. Elige:
    
    ```
    Binary
    ```
    
    ![image.png](image%2019.png)
    
5. Configura:
    
    ```
    Name: ds_blob_historico_ventas_bin
    Linked service: ls_blob_historico_practica3
    Container: historico
    Directory: ventas
    File: dejar vacío
    ```
    
    ![image.png](image%2020.png)
    
6. Selecciona **OK**.
7. Comprueba que puedes navegar hasta los archivos. Desde el Botón Browse.
    
    ![image.png](image%2021.png)
    
8. Ejecuta **Validate all y Publish All**

> No se importa esquema porque un dataset Binary no interpreta columnas.
> 

# **Fase 10. Crear el dataset Binary de destino**

1. En **Author → Datasets**, selecciona **New dataset**.
2. Selecciona:
    
    ```
    Azure Data Lake Storage Gen2
    ```
    
3. Elige:
    
    ```
    Binary
    ```
    
4. Configura:
    
    ```
    Name: ds_adls_raw_ventas_bin
    Linked service: ls_adls_practica3
    File system: datalake
    ```
    
5. En la pestaña **Parameters**, crea:
    
    ```
    Name: pDirectorio
    Type: String
    Default value: raw/ventas/historico
    ```
    
6. En **Connection**, configura el directorio mediante contenido dinámico:
    
    ```
    @dataset().pDirectorio
    ```
    
7. Deja el nombre de archivo vacío.
8. Ejecuta **Validate all y Publish All.**

> El nombre de cada archivo se conservará desde el origen.
> 

# **Fase 11. Crear el pipeline**

## **Paso 1. Crear el pipeline**

1. En **Author**, selecciona **Pipelines**.
2. Selecciona **New pipeline**.
3. Cambia el nombre a:
    
    ```
    pl_blob_historico_to_adls_raw
    ```
    

## **Paso 2. Añadir Copy data**

1. En **Activities**, busca **Copy data**.
2. Arrastra la actividad al lienzo.
3. Cambia su nombre a:
    
    ```
    cp_ventas_historicas_to_raw
    ```
    

## **Paso 3. Configurar Source**

1. Abre la pestaña **Source**.
2. Selecciona:
    
    ```
    Source dataset: ds_blob_historico_ventas_bin
    ```
    
3. En el tipo de ruta, selecciona:
    
    ```
    Wildcard file path
    ```
    
4. Configura:

```
Wildcard folder path: vacío, porque el dataset ya apunta a ventas/
Wildcard file name: ventas_*.csv
Recursive: desactivado
Delete files after completion: desactivado
```

![image.png](image%2022.png)

En algunas versiones de la interfaz puede solicitarse el directorio dentro de la configuración del wildcard. En ese caso, utiliza `ventas`.

El patrón `ventas_*.csv` seleccionará los tres archivos.

## **Paso 4. Configurar Sink**

1. Abre la pestaña **Sink**.
2. Selecciona:
    
    ```
    Sink dataset: ds_adls_raw_ventas_bin
    ```
    
3. Para el parámetro `pDirectorio`, selecciona **Add dynamic content**.
4. Introduce:
    
    ```
    @concat(
      'raw/ventas/historico/fecha_carga=',
      formatDateTime(utcNow(),'yyyy-MM-dd'),
      '/ejecucion=',
      formatDateTime(utcNow(),'yyyyMMdd_HHmmss')
    )
    ```
    
5. En **Copy behavior**, selecciona:
    
    ```
    Preserve hierarchy
    ```
    
6. Deja desactivado el staging (Pestaña Settings).
    
    El resultado esperado será:
    
    ```
    datalake/
    └── raw/
        └── ventas/
            └── historico/
                └── fecha_carga=AAAA-MM-DD/
                    └── ejecucion=AAAAMMDD_HHMMSS/
                        ├── ventas_2020.csv
                        ├── ventas_2021.csv
                        └── ventas_2022.csv
    ```
    

## **Paso 5. Mapping**

En una copia Binary no existe mapeo de columnas. La ausencia de Mapping confirma que no se están transformando los datos.

# **Fase 12. Validar y ejecutar en modo Debug**

1. Selecciona **Validate all**.
2. Corrige cualquier error.
3. Selecciona **Debug**.
4. Espera a que la actividad termine.

El estado esperado es:

```
Succeeded
```

En la salida de una copia Binary, revisa principalmente:

```
Files read: 3
Files written: 3
Data read 43627
Data written 43627
Duration
Throughput
```

```json
{
	"dataRead": 43627,
	"dataWritten": 43627,
	"filesRead": 3,
	"filesWritten": 3,
	"sourcePeakConnections": 3,
	"sinkPeakConnections": 3,
	"copyDuration": 12,
	"throughput": 14.542,
	"errors": [],
	"effectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (Germany West Central)",
	"usedDataIntegrationUnits": 4,
	"billingReference": {
		"activityType": "DataMovement",
		"billableDuration": [
			{
				"meterType": "AzureIR",
				"duration": 0.06666666666666667,
				"unit": "DIUHours"
			}
		],
		"totalBillableDuration": [
			{
				"meterType": "AzureIR",
				"duration": 0.06666666666666667,
				"unit": "DIUHours"
			}
		]
	},
	"usedParallelCopies": 3,
	"executionDetails": [
		{
			"source": {
				"type": "AzureBlobStorage",
				"region": "Germany West Central"
			},
			"sink": {
				"type": "AzureBlobFS",
				"region": "Germany West Central"
			},
			"status": "Succeeded",
			"start": "7/2/2026, 7:00:57 PM",
			"duration": 12,
			"usedDataIntegrationUnits": 4,
			"usedParallelCopies": 3,
			"profile": {
				"queue": {
					"status": "Completed",
					"duration": 7
				},
				"transfer": {
					"status": "Completed",
					"duration": 3,
					"details": {
						"listingSource": {
							"type": "AzureBlobStorage",
							"workingDuration": 0
						},
						"readingFromSource": {
							"type": "AzureBlobStorage",
							"workingDuration": 0
						},
						"writingToSink": {
							"type": "AzureBlobFS",
							"workingDuration": 0
						}
					}
				}
			},
			"detailedDurations": {
				"queuingDuration": 7,
				"transferDuration": 3
			}
		}
	],
	"dataConsistencyVerification": {
		"VerificationResult": "NotVerified"
	},
	"durationInQueue": {
		"integrationRuntimeQueue": 0
	}
}
```

Una copia Binary no suele mostrar `Rows read` y `Rows copied`, porque ADF no interpreta las filas.

# **Fase 13. Publicar**

Después de una ejecución correcta:

1. Selecciona **Publish all**.
2. Revisa los cambios pendientes.
3. Confirma con **Publish**.

> La publicación guarda el pipeline, datasets y Linked Services como versión desplegada.
> 

# **Fase 14. Verificar los archivos en ADLS Gen2**

1. Abre `stadlspractica3`.
2. Selecciona **Storage browser**.
3. Abre:
    
    ```
    datalake/
    raw/
    ventas/
    historico/
    fecha_carga=AAAA-MM-DD/
    ejecucion=AAAAMMDD_HHMMSS/
    ```
    
    ![image.png](image%2023.png)
    
4. Comprueba que existen:
    
    ```
    ventas_2020.csv
    ventas_2021.csv
    ventas_2022.csv
    ```
    
    ![image.png](image%2024.png)
    
5. Descarga o abre uno de los archivos.
6. Verifica:
    - la cabecera se conserva;
    - el archivo contiene 60 registros de datos;
    - las fechas corresponden al año del archivo;
    - no se han añadido ni eliminado columnas;
    - los nombres de archivo no han cambiado.

![image.png](image%2025.png)

# **Fase 15. Ejecutar como carga histórica**

Esta práctica representa una carga histórica puntual. La opción recomendada es:

1. Abrir el pipeline publicado.
2. Seleccionar **Add trigger**.
3. Seleccionar **Trigger now**.
    
    ![image.png](image%2026.png)
    
4. Confirmar la ejecución.
    
    ![image.png](image%2027.png)
    
    ![image.png](image%2028.png)
    

> No es necesario crear un trigger diario para tres archivos estáticos. Si en el futuro llegan nuevos archivos históricos, se puede añadir:
> 
> - un trigger programado;
> - un trigger basado en eventos de Blob Storage;
> - un proceso de archivo o cuarentena;
> - una tabla de control de archivos ya procesados.

# **Fase 16. Monitorización**

1. En Data Factory Studio, abre **Monitor**.
2. Entra en **Pipeline runs**.
3. Recuerda:
    - Debug: ejecuciones realizadas con Debug
    - Triggered: ejecuciones realizadas con Trigger now o un trigger
4. Localiza:
    
    ```
    pl_blob_historico_to_adls_raw
    ```
    
5. Abre la actividad:
    
    ```
    cp_ventas_historicas_to_raw
    ```
    
6. Revisa:
    
    ```
    Status
    Files read
    Files written
    Data read
    Data written
    Duration
    Throughput
    ```
    
    Pulsando en Details:
    
    ![image.png](image%2029.png)
    
7. El resultado esperado es (Pulsar en Output):
    
    ```json
    {
    	"dataRead": 43627,
    	"dataWritten": 43627,
    	"filesRead": 3,
    	"filesWritten": 3,
    	"sourcePeakConnections": 3,
    	"sinkPeakConnections": 3,
    	"copyDuration": 15,
    	"throughput": 10.907,
    	"errors": [],
    	"effectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (Germany West Central)",
    	"usedDataIntegrationUnits": 4,
    	"billingReference": {
    		"activityType": "DataMovement",
    		"billableDuration": [
    			{
    				"meterType": "AzureIR",
    				"duration": 0.06666666666666667,
    				"unit": "DIUHours"
    			}
    		],
    		"totalBillableDuration": [
    			{
    				"meterType": "AzureIR",
    				"duration": 0.06666666666666667,
    				"unit": "DIUHours"
    			}
    		]
    	},
    	"usedParallelCopies": 3,
    	"executionDetails": [
    		{
    			"source": {
    				"type": "AzureBlobStorage",
    				"region": "Germany West Central"
    			},
    			"sink": {
    				"type": "AzureBlobFS",
    				"region": "Germany West Central"
    			},
    			"status": "Succeeded",
    			"start": "7/2/2026, 7:09:25 PM",
    			"duration": 15,
    			"usedDataIntegrationUnits": 4,
    			"usedParallelCopies": 3,
    			"profile": {
    				"queue": {
    					"status": "Completed",
    					"duration": 9
    				},
    				"transfer": {
    					"status": "Completed",
    					"duration": 4,
    					"details": {
    						"listingSource": {
    							"type": "AzureBlobStorage",
    							"workingDuration": 0
    						},
    						"readingFromSource": {
    							"type": "AzureBlobStorage",
    							"workingDuration": 0
    						},
    						"writingToSink": {
    							"type": "AzureBlobFS",
    							"workingDuration": 0
    						}
    					}
    				}
    			},
    			"detailedDurations": {
    				"queuingDuration": 9,
    				"transferDuration": 4
    			}
    		}
    	],
    	"dataConsistencyVerification": {
    		"VerificationResult": "NotVerified"
    	},
    	"durationInQueue": {
    		"integrationRuntimeQueue": 0
    	}
    }
    ```