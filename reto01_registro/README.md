# Reto 1: Configuración de Entornos Cloud y Gestión de Costes (AWS & GCP)

## Objetivo
Establecer las bases operativas en Amazon Web Services (AWS) y Google Cloud Platform (GCP), configurando las consolas de administración y aplicando buenas prácticas de control de costes para mantener el entorno dentro de la capa gratuita (Free Tier).

##  Proceso de Implementación

### 1. Alta y Configuración en AWS
Accedí a la consola de AWS y configuré el panel principal. Para asegurar la cuenta, validé los accesos básicos.
![Consola AWS](./img/aws-dashboard.png)
*(Nota: Información sensible como ID de cuenta y usuario root han sido pixelados por seguridad).*

### 2. Alta y Configuración en GCP
Creé la estructura de organización inicial en Google Cloud, familiarizándome con la navegación por proyectos.
![Consola GCP](./img/gcp-dashboard.png)

### 3. Estrategia de Control de Costes (Evitar Cargos)
Para evitar sorpresas de facturación, configuré una alerta de presupuesto (Billing Alarm) en AWS que me notificará por correo si los gastos proyectados superan los $0 USD.
![Alerta de Presupuesto AWS](./img/aws-billing.png)

## 🔒 Buenas Prácticas de Seguridad Aplicadas
* **Ofuscación de datos:** Se eliminaron IDs de cuenta, correos electrónicos y números de tarjetas de las capturas de pantalla.
* **Seguridad de Cuenta:** Configuración inicial para no utilizar el usuario Root para tareas diarias en el futuro.
