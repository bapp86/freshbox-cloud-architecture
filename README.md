# FreshBox SpA — Arquitectura Cloud ☁️

[![AWS](https://img.shields.io/badge/AWS-Cloud%20Infrastructure-orange?logo=amazon-aws)](https://aws.amazon.com/)
[![Terraform](https://img.shields.io/badge/Terraform-Infrastructure%20as%20Code-7B42BC?logo=terraform)](https://www.terraform.io/)
[![Docker](https://img.shields.io/badge/Docker-Containers-2496ED?logo=docker)](https://www.docker.com/)
[![Node.js](https://img.shields.io/badge/Node.js-Backend-339933?logo=node.js)](https://nodejs.org/)
[![MariaDB](https://img.shields.io/badge/MariaDB-Database-003545?logo=mariadb)](https://mariadb.org/)

Repositorio oficial correspondiente a la **Evaluación Parcial N.º 1 (EP1)** de la asignatura **Arquitectura Cloud (ARY1102)**.

El proyecto implementa una arquitectura cloud para el e-commerce de **FreshBox SpA**, utilizando servicios de **Amazon Web Services (AWS)** bajo un modelo de **tres capas (Three-Tier Architecture)**.

La solución está orientada a entregar **seguridad, escalabilidad automática, tolerancia a fallos y una adecuada segmentación de red**, permitiendo soportar el crecimiento proyectado de la plataforma.

---

## 📋 Descripción del Proyecto

FreshBox SpA corresponde a una empresa dedicada a la venta online de productos orgánicos, que presenta un crecimiento sostenido del **40% trimestral** y requiere una plataforma capaz de soportar un catálogo online administrable.

La solución implementada considera:

* Infraestructura como Código mediante **Terraform**.
* Una **VPC** dedicada para la aplicación.
* Subredes públicas y privadas.
* Distribución de recursos en las Availability Zones:

  * `us-east-1a`
  * `us-east-1b`
* **Application Load Balancer (ALB)** como punto de entrada público.
* **Auto Scaling Group (ASG)** para la capa de aplicación.
* Instancias **EC2 `t4g.small` ARM64 / Graviton2**.
* Contenedores **Docker**.
* Repositorios privados en **Amazon ECR**.
* Base de datos **MariaDB** en una instancia EC2 privada.
* **AWS Systems Manager Session Manager** para administración de las instancias.
* **NAT Gateway** para permitir salida controlada desde las subredes privadas.

---

# 🏗️ Arquitectura de la Solución

La solución sigue una arquitectura clásica de **tres capas**, separando presentación, aplicación y datos.

```text
                                 INTERNET
                                     │
                                     ▼
                           ┌─────────────────────┐
                           │ Internet Gateway    │
                           └──────────┬──────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │ Application Load    │
                           │ Balancer (ALB)      │
                           └──────────┬──────────┘
                                      │
                    ┌─────────────────┴─────────────────┐
                    │                                   │
                    ▼                                   ▼
          ┌───────────────────┐               ┌───────────────────┐
          │   us-east-1a      │               │   us-east-1b      │
          │   EC2 App         │               │   EC2 App         │
          │                   │               │                   │
          │ Nginx / Frontend  │               │ Nginx / Frontend  │
          │ Node.js APIs      │               │ Node.js APIs      │
          │ Docker            │               │ Docker            │
          └─────────┬─────────┘               └─────────┬─────────┘
                    │                                   │
                    └─────────────────┬─────────────────┘
                                      │
                                      ▼
                           ┌─────────────────────┐
                           │  MariaDB / MySQL    │
                           │  EC2 Privada        │
                           │  10.0.2.6            │
                           └─────────────────────┘


        EC2 privadas
             │
             ▼
      ┌──────────────┐
      │ NAT Gateway  │
      └──────┬───────┘
             │
             ▼
          Internet
        / Amazon ECR
```

> La alta disponibilidad se implementa principalmente en la **capa de aplicación**, mediante un Auto Scaling Group con instancias distribuidas en distintas Availability Zones. La capa de datos corresponde a una instancia dedicada de MariaDB.

---

# 🌐 Diseño de Red

La infraestructura utiliza una **VPC con CIDR `10.0.0.0/22`**, distribuida en dos Availability Zones.

```text
VPC-FreshBox
CIDR: 10.0.0.0/22

├── us-east-1a
│   ├── Subred Pública
│   │   └── Application Load Balancer
│   │
│   └── Subred Privada
│       └── EC2 APP
│
└── us-east-1b
    ├── Subred Pública
    │   └── Application Load Balancer
    │
    └── Subred Privada
        └── EC2 APP

Subred Privada de Datos
└── EC2 MariaDB
    └── IP privada: 10.0.2.6
```

La separación de subredes permite restringir el acceso directo a los componentes internos y mantener la base de datos aislada de Internet.

---

# 🔐 Seguridad

La arquitectura aplica una segmentación de red basada en **Security Groups** y principio de menor privilegio.

### Security Group del ALB

Permite recibir tráfico HTTP desde Internet:

```text
Internet
   ↓
ALB
   ↓
HTTP :80
```

### Security Group de las aplicaciones

El acceso entrante está restringido para aceptar tráfico únicamente desde el Security Group del ALB.

```text
SG-ALB
   ↓
SG-APP
```

### Security Group de la base de datos

El acceso al puerto de base de datos está restringido exclusivamente a la capa de aplicación.

```text
SG-APP
   ↓
SG-DATA
   ↓
MariaDB :3306
```

De esta forma:

```text
Internet
   │
   ▼
ALB
   │
   ▼
Aplicación
   │
   ▼
MariaDB
```

No existe acceso directo desde Internet hacia la base de datos.

---

# ⚙️ Tecnologías Utilizadas

| Tecnología                    | Función                                   |
| ----------------------------- | ----------------------------------------- |
| **Amazon Web Services**       | Plataforma Cloud                          |
| **Amazon VPC**                | Segmentación de red                       |
| **Internet Gateway**          | Acceso de la capa pública a Internet      |
| **NAT Gateway**               | Salida controlada desde subredes privadas |
| **Application Load Balancer** | Balanceo del tráfico HTTP                 |
| **Amazon EC2**                | Cómputo de la aplicación y base de datos  |
| **Auto Scaling Group**        | Escalabilidad y tolerancia a fallos       |
| **Amazon ECR**                | Almacenamiento privado de imágenes Docker |
| **AWS Systems Manager**       | Administración mediante Session Manager   |
| **Terraform**                 | Infraestructura como Código               |
| **Docker**                    | Contenerización                           |
| **Nginx**                     | Frontend y proxy inverso                  |
| **Node.js**                   | Microservicios backend                    |
| **MariaDB**                   | Base de datos relacional                  |

---

# 🖥️ Capa de Aplicación

La capa de aplicación está ubicada en **subredes privadas** y es administrada mediante un **Auto Scaling Group**.

Las instancias utilizadas corresponden a:

```text
Tipo: t4g.small
Arquitectura: ARM64
Familia: Graviton2
Sistema operativo: Amazon Linux 2023
```

El Auto Scaling Group se configuró con:

```text
Capacidad mínima:     2 instancias
Capacidad deseada:    2 instancias
Capacidad máxima:     4 instancias
```

Las instancias se encuentran distribuidas entre:

```text
us-east-1a
us-east-1b
```

Esto permite mantener capacidad disponible ante la falla de una instancia o de una Availability Zone.

---

# 🐳 Contenedores Docker

Cada instancia de aplicación ejecuta cinco contenedores conectados mediante la red interna:

```text
freshbox-net
```

Los contenedores corresponden a:

1. `frontend`
2. `get-products`
3. `create-product`
4. `update-product`
5. `delete-product`

La arquitectura de contenedores puede representarse como:

```text
                 ┌──────────────────┐
                 │      Nginx       │
                 │    frontend      │
                 │      :80         │
                 └────────┬─────────┘
                          │
                ┌─────────┴─────────┐
                │                   │
                ▼                   ▼
       ┌────────────────┐   ┌──────────────────┐
       │ Microservicios │   │   Base de Datos  │
       │    Node.js     │──►│     MariaDB      │
       └────────────────┘   └──────────────────┘
```

---

# 📦 Microservicios

| Contenedor       | Descripción                               | Puerto | Endpoint            | Método HTTP |
| ---------------- | ----------------------------------------- | -----: | ------------------- | ----------- |
| `frontend`       | Interfaz web con Nginx, HTML y JavaScript |   `80` | `/`                 | `GET`       |
| `get-products`   | Consulta el catálogo                      | `3001` | `/api/products`     | `GET`       |
| `create-product` | Registra productos                        | `3002` | `/api/products`     | `POST`      |
| `update-product` | Actualiza productos                       | `3003` | `/api/products/:id` | `PUT`       |
| `delete-product` | Elimina productos                         | `3004` | `/api/products/:id` | `DELETE`    |

---

# 🗄️ Capa de Datos

La capa de datos corresponde a una instancia dedicada ubicada en una **subred privada aislada**.

```text
Servidor:
EC2-MySQL

Motor:
MariaDB

IP privada:
10.0.2.6

Puerto:
3306
```

La base de datos utilizada por la aplicación corresponde a:

```text
freshbox
```

El script `init.sql` se utiliza para:

* Crear la base de datos.
* Crear el usuario de aplicación.
* Crear las estructuras requeridas.
* Insertar los cinco productos iniciales del catálogo.

---

# 📋 Requisitos Previos

Para realizar el despliegue desde Ubuntu se requiere contar con:

* Git
* Curl
* Unzip
* Wget
* Terraform
* AWS CLI
* Docker

---

## 1. Actualizar Ubuntu

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Instalar herramientas básicas

```bash
sudo apt install git curl unzip wget gnupg software-properties-common -y
```

---

## 3. Instalar Terraform

```bash
wget -O- https://apt.releases.hashicorp.com/gpg \
  | gpg --dearmor \
  | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com \
$(lsb_release -cs) main" \
| sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update
sudo apt install terraform -y

terraform --version
```

---

## 4. Instalar AWS CLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
  -o "awscliv2.zip"

unzip awscliv2.zip

sudo ./aws/install

aws --version
```

---

## 5. Instalar Docker

```bash
sudo apt-get install docker.io -y

sudo systemctl start docker
sudo systemctl enable docker

sudo usermod -aG docker $USER
newgrp docker

docker --version
```

---

# 🚀 Despliegue

## Paso 1 — Clonar el repositorio

```bash
git clone https://github.com/bapp86/freshbox-cloud-architecture.git

cd freshbox-cloud-architecture
```

---

# Paso 2 — Aprovisionar infraestructura con Terraform

Inicializar Terraform:

```bash
terraform init
```

Revisar la infraestructura propuesta:

```bash
terraform plan
```

Aplicar la configuración:

```bash
terraform apply
```

Confirmar la operación cuando Terraform lo solicite:

```text
yes
```

La implementación provisiona los principales componentes de la arquitectura:

* VPC
* Subredes públicas y privadas
* Security Groups
* Internet Gateway
* NAT Gateway
* Application Load Balancer
* Auto Scaling Group
* Instancias EC2

---

# Paso 3 — Configurar la Base de Datos

Conectarse mediante **AWS Systems Manager Session Manager** a la instancia privada de base de datos.

La instancia utilizada en la implementación corresponde a:

```text
EC2-MySQL
IP privada: 10.0.2.6
```

Una vez establecida la conexión, ejecutar el script de inicialización:

```bash
mysql -h <MYSQL_PRIVATE_IP> -u alumno -p < init.sql
```

La credencial utilizada durante la implementación académica corresponde a:

```text
Usuario: alumno
Contraseña: alumno123
```

> **Importante:** Estas credenciales forman parte del entorno académico descrito en el proyecto. No deben utilizarse como credenciales de producción.

---

# Paso 4 — Amazon ECR

Las imágenes de los microservicios son almacenadas en repositorios privados de **Amazon Elastic Container Registry (ECR)**.

Autenticación:

```bash
aws ecr get-login-password --region us-east-1 \
| docker login \
--username AWS \
--password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```

Reemplazar:

```text
<AWS_ACCOUNT_ID>
```

por el ID correspondiente de la cuenta AWS.

Las imágenes utilizadas corresponden a cinco componentes:

```text
frontend
get-products
create-product
update-product
delete-product
```

Todas las imágenes fueron construidas para la arquitectura:

```text
ARM64
```

---

# Paso 5 — Despliegue de los contenedores

El acceso a las instancias de aplicación se realiza mediante:

```text
AWS Systems Manager — Session Manager
```

En cada instancia de aplicación se utiliza la red interna:

```bash
docker network create freshbox-net
```

Los contenedores deben conectarse a esta red para permitir la comunicación entre ellos.

Ejemplo del microservicio `get-products`:

```bash
docker run -d \
  --name get-products \
  --network freshbox-net \
  -p 3001:3001 \
  -e DB_HOST=<MYSQL_PRIVATE_IP> \
  -e DB_USER=alumno \
  -e DB_PASS=<DB_PASSWORD> \
  -e DB_NAME=freshbox \
  -e PORT=3001 \
  <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/freshbox-get-products:latest
```

El mismo procedimiento se aplica a los demás microservicios.

---

# 🔄 Flujo de Comunicación

El flujo esperado de una solicitud es:

```text
                  INTERNET
                     │
                     ▼
             Internet Gateway
                     │
                     ▼
            Application Load
               Balancer
                     │
                     ▼
             Nginx / Frontend
                     │
            ┌────────┴────────┐
            │                 │
            ▼                 ▼
      Node.js APIs       Archivos Web
            │
            ▼
        MariaDB
       10.0.2.6
```

Para el acceso de las instancias privadas a servicios externos:

```text
EC2 Privada
     │
     ▼
NAT Gateway
     │
     ▼
Internet / Amazon ECR
```

---

# 🔍 Validación End-to-End

Una vez implementada la infraestructura, se realizaron pruebas para comprobar el flujo completo de comunicación.

## 1. Acceso público

Se accedió al DNS proporcionado por el **Application Load Balancer**.

El frontend fue cargado correctamente, confirmando que:

```text
Internet
   ↓
ALB
   ↓
Instancia EC2 privada
   ↓
Nginx
```

funciona correctamente.

---

## 2. Validación de la API

La ruta:

```text
/api/products
```

fue consultada directamente a través del Load Balancer.

Resultado:

```text
HTTP 200 OK
```

La respuesta devolvió los **5 productos iniciales** almacenados en la base de datos.

Flujo validado:

```text
Cliente
   ↓
ALB
   ↓
Nginx
   ↓
Microservicio Node.js
   ↓
MariaDB
```

---

# ⚠️ Hallazgo Durante las Pruebas

Durante las pruebas funcionales se detectó un problema en el botón:

```text
"Cargar productos"
```

La interfaz devolvía:

```text
Connection Timed Out
```

El análisis permitió determinar que el archivo `app.js` intentaba conectarse directamente al puerto:

```text
3001
```

Esta conexión fue bloqueada correctamente por las reglas del **Security Group**, debido a que el microservicio backend no está expuesto directamente a Internet.

Sin embargo, la API respondió correctamente cuando fue consultada a través de la ruta:

```text
/api/products
```

obteniendo:

```text
HTTP 200
```

y los cinco productos de la base de datos.

Este comportamiento permitió comprobar que la infraestructura de red y el backend se encuentran operativos, mientras que la integración del frontend requiere ajustar la ruta de consumo de la API para utilizar el flujo definido por Nginx.

---

# ✅ Estado de la Implementación

| Componente | Estado |
|---|---|
| Terraform | ✅ Implementado |
| VPC | ✅ Implementada |
| Subredes públicas y privadas | ✅ Implementadas |
| Multi-AZ (`us-east-1a` / `us-east-1b`) | ✅ Implementado |
| Internet Gateway | ✅ Implementado |
| NAT Gateway | ✅ Implementado |
| Security Groups | ✅ Configurados |
| Application Load Balancer | ✅ Operativo |
| Auto Scaling Group | ✅ 2–4 instancias |
| EC2 `t4g.small` ARM64 | ✅ Operativas |
| Amazon ECR | ✅ Imágenes disponibles |
| Contenedores Docker | ✅ 5 contenedores |
| Red `freshbox-net` | ✅ Configurada |
| MariaDB | ✅ Operativa |
| Base de datos `freshbox` | ✅ Inicializada |
| Productos iniciales | ✅ 5 registros |
| Endpoint `/api/products` | ✅ HTTP 200 |
| DNS del ALB | ✅ Validado |
| Botón "Cargar productos" | ⚠️ Requiere ajuste de ruta API |

---

# 📈 Escalabilidad y Disponibilidad

La arquitectura implementa escalabilidad mediante **Auto Scaling Group**, configurado con:

```text
Mínimo:    2
Deseado:   2
Máximo:    4
```

Las instancias se distribuyen entre dos Availability Zones:

```text
us-east-1a
us-east-1b
```

El **Application Load Balancer** distribuye las solicitudes entre las instancias disponibles.

Esto permite mantener capacidad en la capa de aplicación ante fallos de instancias y absorber incrementos de demanda mediante el mecanismo de Auto Scaling.

> La capa de datos actual utiliza una única instancia MariaDB. Una evolución futura de la arquitectura contempla migrar esta capa a **Amazon RDS con Multi-AZ** para mejorar la disponibilidad de la base de datos.

---

# 💰 Optimización de Costos

La solución considera el uso de instancias:

```text
t4g.small
```

basadas en arquitectura **ARM64 / Graviton2**, buscando una adecuada relación entre costo y rendimiento.

El uso de **Auto Scaling** permite adaptar la capacidad de cómputo de la capa de aplicación según las necesidades del sistema.

---

# 🔮 Mejoras Futuras

Como parte de la evolución de la arquitectura, se identifican las siguientes mejoras:

### Amazon RDS

Migrar la instancia MariaDB actual hacia **Amazon RDS**, habilitando funcionalidades administradas como:

* Respaldos automatizados.
* Gestión de parches.
* Mayor disponibilidad.
* Configuración Multi-AZ.

### Amazon S3 + CloudFront

Separar los archivos estáticos del frontend y alojarlos en:

```text
Amazon S3
       ↓
Amazon CloudFront
```

Esto permitiría disminuir la carga sobre las instancias EC2 y distribuir el contenido estático mediante una CDN.

### CI/CD

Implementar un pipeline automatizado mediante servicios como:

```text
AWS CodePipeline
AWS CodeBuild
```

para automatizar la construcción y despliegue de nuevas versiones de los contenedores.

---

# 📁 Estructura General

```text
freshbox-cloud-architecture/
│
├── terraform/
│
├── frontend/
│
├── get-products/
│
├── create-product/
│
├── update-product/
│
├── delete-product/
│
├── init.sql
│
└── README.md
```

> La estructura mostrada corresponde a la organización conceptual del proyecto; los archivos concretos pueden variar según la implementación presente en el repositorio.

---

# 🎯 Estado de la Implementación

| Componente                  | Estado                         |
| --------------------------- | ------------------------------ |
| VPC                         | ✅ Implementada                 |
| Subredes públicas/privadas  | ✅ Implementadas                |
| Multi-AZ                    | ✅ Implementado                 |
| Internet Gateway            | ✅ Implementado                 |
| NAT Gateway                 | ✅ Implementado                 |
| Security Groups             | ✅ Implementados                |
| Application Load Balancer   | ✅ Implementado                 |
| Auto Scaling Group          | ✅ Implementado                 |
| EC2 `t4g.small`             | ✅ Implementadas                |
| Amazon ECR                  | ✅ Implementado                 |
| Docker                      | ✅ Implementado                 |
| Nginx                       | ✅ Implementado                 |
| Microservicios Node.js      | ✅ Implementados                |
| MariaDB                     | ✅ Implementado                 |
| Validación `/api/products`  | ✅ HTTP 200                     |
| Frontend `Cargar productos` | ⚠️ Requiere ajuste de ruta API |

---

# 👨‍💻 Autor

**Bryan Andrés Painemilla Panchillo**

**Asignatura:** Arquitectura Cloud — ARY1102
**Institución:** Duoc UC

---

## 📚 Proyecto Académico

**Evaluación Parcial N.º 1 — Arquitectura Cloud**

FreshBox SpA — Implementación de arquitectura cloud en AWS.

