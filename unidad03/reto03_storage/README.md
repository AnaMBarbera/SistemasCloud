---
---

[Mi Portfolio](../../README.md) -> [Unidad 3](../README.md) -> [Reto 3](./README.md)

# Reto 3 — Almacenamiento y recuperación de archivos

## Objetivo

En este reto se trabaja con servicios de almacenamiento de objetos en la nube.

En la parte correspondiente a **AWS**, se utiliza **Amazon S3** para:

- Crear un bucket de almacenamiento.
- Subir un archivo.
- Configurar los permisos de acceso.
- Generar una URL segura y temporal para recuperar el archivo.
- Comprobar la diferencia entre acceso privado, acceso mediante URL prefirmada y acceso público.

> **Nota de seguridad:** durante el desarrollo del reto se evitó almacenar información sensible. El acceso público se habilitó únicamente de forma temporal para realizar una prueba y posteriormente se volvió a bloquear.

---

# AWS — Amazon S3

## 1. Creación del bucket

Se creó un bucket de Amazon S3 denominado:

```text
ana-storage-reto3
```

El bucket se creó en la región:

```text
Europa (España) — eu-south-2
```

Se mantuvo activado el **bloqueo de acceso público** y se utilizó el cifrado predeterminado **SSE-S3**.

![Creación del bucket S3](./img/aws_S3_bucket.png)

### Configuración utilizada

| Configuración           | Valor                        |
| ----------------------- | ---------------------------- |
| Servicio                | Amazon S3                    |
| Bucket                  | `ana-storage-reto3`          |
| Región                  | Europa (España) `eu-south-2` |
| Propiedad de objetos    | Propietario del bucket       |
| ACL                     | Deshabilitadas               |
| Acceso público          | Bloqueado                    |
| Cifrado                 | SSE-S3                       |
| Clase de almacenamiento | S3 Standard                  |

---

## 2. Almacenamiento del archivo

Se creó y almacenó un archivo de prueba:

```text
reto3-aws.txt
```

El archivo contiene información de prueba relacionada con el ejercicio y tiene un tamaño de 70 bytes.

La consola de Amazon S3 confirmó que la carga se realizó correctamente.

![Carga del archivo en S3](./img/aws_S3_upload.png)

El archivo quedó disponible dentro del bucket:

```text
ana-storage-reto3/
└── reto3-aws.txt
```

---

## 3. Configuración de permisos

### Propiedad de objetos

El bucket utiliza la configuración:

> **Imposición de propietario del bucket (Bucket owner enforced)**

Con esta configuración, las ACL están deshabilitadas y la gestión del acceso se realiza mediante políticas de IAM y políticas del bucket.

[Configuración de permisos](./img/aws_S3_permissions.png)

### Bloqueo de acceso público

Se mantuvo activada la opción:

> **Bloquear todo el acceso público**

Esta configuración impide que las políticas o ACL puedan conceder accidentalmente acceso público al bucket u objetos.

[Bloqueo del acceso público](./img/aws_S3_public_access.png)

Esta configuración permite mantener los archivos privados y conceder acceso temporal cuando sea necesario mediante una URL prefirmada.

---

## 4. URL segura para recuperar el archivo

Para permitir el acceso temporal al archivo sin hacerlo público se utilizó una **URL prefirmada (presigned URL)**.

Desde AWS CloudShell se ejecutó:

```bash
aws s3 presign s3://ana-storage-reto3/reto3-aws.txt --expires-in 3600
```

El parámetro:

```text
--expires-in 3600
```

establece una validez de **3600 segundos (1 hora)** para la URL generada.

La URL prefirmada contiene una firma temporal que permite acceder al objeto sin modificar la configuración de acceso público del bucket.

### Comprobación

La URL generada se abrió desde el navegador y permitió recuperar correctamente el contenido de `reto3-aws.txt`.

![Recuperación del archivo mediante URL en el navegador](./img/aws_S3_presign_url.png)

También se comprobó la recuperación mediante `curl` desde CloudShell:

```bash
curl -o reto3-aws-descargado.txt "URL_PREFIRMADA"
```

El archivo descargado se comprobó posteriormente con:

```bash
cat reto3-aws-descargado.txt
```

obteniendo el contenido original del archivo.

[Recuperación del archivo mediante URL en la consola](./img/aws_S3_download.png)

> **Seguridad:** las URL prefirmadas contienen información temporal de autenticación. Por este motivo, no se incluye ninguna URL prefirmada completa en el repositorio.

---

# 5. Prueba de acceso público

Como parte del aprendizaje se realizó también una prueba para comprobar cómo funciona el acceso público en S3.

## 5.1 Desactivación temporal del bloqueo

Se desactivó temporalmente **Bloquear todo el acceso público**.

![Bloqueo público desactivado](./img/aws_S3_public_disabled.png)

Este cambio se realizó únicamente para comprobar el funcionamiento del acceso público y posteriormente se revirtió.

## 5.2 Política de bucket

Se añadió temporalmente una política que permitía la operación:

```text
s3:GetObject
```

sobre los objetos del bucket.

La política utilizada fue:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadForReto3",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::ana-storage-reto3/*"
    }
  ]
}
```

Esta política permite la lectura pública de los objetos, pero no concede permisos para:

- Subir archivos.
- Eliminar archivos.
- Modificar archivos.
- Listar el contenido del bucket.

[Política pública del bucket](./img/aws_S3_bucket_policy_public.png)

## 5.3 Comprobación mediante URL pública

Una vez aplicada la política, se pudo acceder al archivo mediante su URL pública, sin utilizar una URL prefirmada.

La prueba se realizó también desde una ventana privada del navegador para comprobar que el acceso no dependía de una sesión autenticada de AWS.

![Acceso mediante URL pública](./img/aws_S3_public_url.png)

El contenido del archivo se mostró correctamente.

---

## 5.4 Restauración de la configuración segura

Una vez finalizada la prueba:

1. Se eliminó la política de acceso público.
2. Se volvió a activar **Bloquear todo el acceso público**.
3. Se comprobó nuevamente que el acceso mediante la URL pública producía un error de acceso.

De esta forma, el bucket volvió a quedar configurado como recurso privado.

---

# 6. Comparación de los métodos de acceso

Durante el ejercicio se comprobaron tres escenarios:

| Configuración  | Tipo de URL    | Resultado          |
| -------------- | -------------- | ------------------ |
| Bucket privado | URL prefirmada | ✅ Acceso temporal |
| Bucket público | URL normal     | ✅ Acceso público  |
| Bucket privado | URL normal     | ❌ `AccessDenied`  |

La configuración final del bucket es la más segura de las tres:

```text
Bucket privado
    │
    ├── Acceso público bloqueado
    ├── ACL deshabilitadas
    ├── Cifrado SSE-S3
    │
    └── Acceso temporal
            │
            └── URL prefirmada
```

## Conclusión

Con este ejercicio se ha comprobado el funcionamiento de **Amazon S3** como servicio de almacenamiento de objetos.

Se ha creado un bucket en la región de España, almacenado un archivo y configurado su acceso manteniendo el bucket privado.

Además, se ha comprobado mediante una URL prefirmada que es posible proporcionar **acceso temporal a un objeto privado sin habilitar el acceso público**.

Finalmente, se realizó una prueba controlada de acceso público mediante una política de bucket y se restauró la configuración segura original.

Esto permite diferenciar entre:

- **Acceso privado:** requiere permisos.
- **URL prefirmada:** acceso temporal y controlado.
- **URL pública:** cualquier usuario que disponga de la URL puede acceder mientras el recurso sea público.

<br>

# Google Cloud Storage

## 1. Creación del bucket

Se creó el bucket:

```text
ana-storage-reto3-gcp
```

### Configuración utilizada:

| Configuración                 | Valor                   |
| ----------------------------- | ----------------------- |
| Ubicación europe-southwest1   | (Madrid)                |
| Tipo de ubicación             | Region                  |
| Clase de almacenamiento       | Standard                |
| Espacio de nombres jerárquico | Inhabilitado            |
| Rapid Cache                   | Inhabilitado            |
| Control de acceso             | Uniforme                |
| Prevención de acceso público  | Activada                |
| Encriptación                  | Administrada por Google |

---

![Creación del bucket](./img/gcp_storage_bucket.png)

La elección de la región de Madrid permite mantener los datos en una región de Google Cloud cercana al entorno de trabajo.

## 2. Protección de los datos

Se mantiene habilitada la política de eliminación no definitiva (Soft Delete) con la configuración predeterminada de Google Cloud.

Esta característica permite conservar temporalmente el bucket y los objetos después de su eliminación para poder recuperarlos durante el período establecido.

También se mantienen deshabilitados:

```text
Controles de versiones de objetos.
Política de retención del bucket.
Retención de objetos.
```

La encriptación utilizada es Administrada por Google, que proporciona cifrado de los datos almacenados sin necesidad de gestionar manualmente las claves.

[Configuración del bucket](./img/gcp_storage_bucket_config.png)

## 3. Subir un archivo

Se creó un archivo de prueba:

```text
reto3-gcp.txt
```

y se almacenó en el bucket:

```text
gs://ana-storage-reto3-gcp/reto3-gcp.txt
```

El archivo tiene un tamaño de 70 B y es de tipo text/plain.

![Carga del archivo](./img/gcp_storage_upload.png)

[Detalles del archivo](./img/gcp_storage_object.png)

## 4. Configuración de permisos

El bucket utiliza Control de acceso uniforme, por lo que los permisos se gestionan mediante IAM a nivel de bucket.

El bucket permanece configurado como:

```text
Acceso público: No público
```

No se concede acceso a allUsers.

[Configuración de permisos](./img/gcp_storage_permissions.png)

### Cuenta de servicio para generar URLs firmadas

Se creó la cuenta de servicio:

```text
reto3-storage-signer@bases-de-datos-con-firebase.iam.gserviceaccount.com
```

Esta cuenta dispone del rol:

```text
Storage Object Viewer
```

permitiéndole leer los objetos almacenados en el bucket.

Además, el usuario utilizado para realizar el ejercicio dispone sobre esta cuenta de servicio del rol:

```text
Creador de tokens de cuenta de servicio
```

Este permiso permite utilizar la cuenta de servicio para generar firmas mediante la suplantación de identidad.

[Cuenta de servicios](./img/gcp_storage_signed_url_permissions.png)

## 5. Generar una URL firmada

Para proporcionar acceso temporal al archivo sin hacerlo público se utilizó una Signed URL.

Desde Google Cloud Shell se comprobó primero la cuenta autenticada:

```bash
gcloud auth list
```

También se comprobó el proyecto activo:

```bash
gcloud config get-value project
```

La URL firmada se generó mediante:

```bash
gcloud storage sign-url gs://ana-storage-reto3-gcp/reto3-gcp.txt \
 --duration=1h \
 --region=europe-southwest1 \
 --impersonate-service-account=reto3-storage-signer@bases-de-datos-con-firebase.iam.gserviceaccount.com
```

La opción:

```bash
--duration=1h
```

establece una validez de una hora.

La opción:

```bash
--impersonate-service-account
```

permite utilizar la cuenta de servicio reto3-storage-signer para realizar la firma sin necesidad de crear ni descargar una clave privada.

Seguridad: la URL firmada contiene información de autenticación. La captura utilizada en el portfolio oculta los parámetros sensibles de la URL.

[Generación de URL](./img/gcp_storage_signed_url.png)

## 6. Recuperar el archivo mediante la URL firmada

La URL firmada se abrió desde un navegador sin necesidad de modificar la configuración de acceso público del bucket.

El contenido de:

```text
reto3-gcp.txt
```

se pudo visualizar correctamente.

Esto demuestra que es posible proporcionar acceso temporal a un objeto privado mediante una URL firmada.

![Visualización en el navegador](./img/gcp_storage_signed_url_access.png)

La configuración del bucket continúa siendo:

**Acceso público: No público**

Si se utiliza la url pública en este momento vemos que el acceso se deniega

![Visualización en el navegador](./img/gcp_storage_private_access.png)

Durante el ejercicio no se habilitó el acceso público al bucket.

Sin embargo, se documenta el procedimiento para conocer cómo se realizaría si fuera necesario.

Con el control de acceso uniforme, habría que:

Desactivar la prevención de acceso público del bucket.
Añadir el principal:

```text
allUsers
```

Concederle el rol:

```text
Storage Object Viewer
```

Utilizar la URL pública del objeto:

```text
https://storage.googleapis.com/ana-storage-reto3-gcp/reto3-gcp.txt
```

Con esta configuración cualquier usuario de Internet podría leer el objeto sin autenticarse.

## 8. Evidencias de aprendizaje

Las capturas utilizadas para documentar el ejercicio muestran:

Configuración del bucket.

Ubicación en Madrid.

Clase de almacenamiento Standard.

Protección de los datos.

Archivo almacenado.

Configuración de permisos.

Permisos de la cuenta de servicio.

Permiso para generar tokens.

Generación de la URL firmada.

Acceso al archivo mediante la URL firmada.

Las URLs firmadas y cualquier otro dato que pueda utilizarse como credencial o mecanismo de autenticación no se incluyen completas en el repositorio.

## Resultado

Se ha completado el almacenamiento y recuperación de archivos mediante Google Cloud Storage.

La configuración final mantiene el bucket privado y utiliza una URL firmada temporal para proporcionar acceso controlado al archivo.

## Navegación

🏠 [Índice del Portfolio](../../README.md)

<- [Unidad 3](../README.md)

<- [Reto 2: Despliegue y Configuración de Instancia EC2 en AWS](../reto02_instanciaEC2/README.md)

-> [Reto 4: Desplegar una máquina virtual en GCP y Microsoft Azure](../reto04_VM/README.md)
