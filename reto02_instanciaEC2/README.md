# Reto 2: Despliegue de Servidor Virtual (AWS EC2) y Configuración Linux via SSH

##  Objetivo
Desplegar una instancia virtual EC2 en AWS de forma segura, establecer conexión remota mediante llaves criptográficas (SSH) y realizar el aprovisionamiento inicial del sistema operativo Linux.

##  Guía Paso a Paso

### Paso 1: Lanzamiento de la Instancia EC2
Desplegué una instancia utilizando la AMI de **Ubuntu Server 24.04 LTS** (Capa gratuita) y un tipo de instancia **t2.micro**. Configuré un par de llaves (.pem) para el acceso.
![Lanzamiento EC2](./img/ec2-creation.png)

### Paso 2: Conexión Segura por SSH
Utilizando la terminal local y aplicando los permisos correctos a la llave privada (`chmod 400`), me conecté al servidor mediante su IP pública:
```bash
ssh -i "mi-llave.pem" ubuntu@://amazonaws.com
```
![Conexión SSH](./img/ssh-connection.png)

### Paso 3: Configuración y Actualización del Sistema
Una vez dentro del "dojo" Linux, ejecuté la actualización de los repositorios y del software del sistema para asegurar la estabilidad y parches de seguridad:
```bash
sudo apt update && sudo apt upgrade -y
```
![Actualización del Sistema](./img/system-update.png)

## 💡 Aprendizajes Clave
* Gestión de llaves SSH y permisos en entornos Unix.
* Ciclo de vida básico de una instancia de computación en la nube.
