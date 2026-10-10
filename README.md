# test-simple-stock-flow-infra

> **Prueba técnica · Ficha ADSO 3413974**
> Horario: de **9:00 a. m. a 3:00 p. m.** (15:00)

Este repositorio es la **infraestructura** de *Simple Stock Flow*: contenedores Docker, red, volúmenes y motor de base de datos vacío. **Empieza vacío a propósito**: se construye en el fork de cada aprendiz.

## Instrucciones

Cada aprendiz debe **crear el fork** de los seis repositorios del proyecto y **resolver el proyecto
con el spec planteado**.

1. Hacer fork, a su cuenta de GitHub, de cada repositorio de la tabla del final.
2. Leer el spec en [`test-simple-stock-flow-docs`](https://github.com/code-sena/test-simple-stock-flow-docs).
   Se entrega en dos versiones: `spec-python/` y `spec-.net/`.
3. Desarrollar en los forks.

## El reto se desarrolla con React y PHP (Laravel)

El spec está escrito para Python y para .NET, pero el reto **no** se hace en esos lenguajes:

| Capa | Tecnología del reto |
|---|---|
| Frontend | React |
| Backend | PHP con Laravel |

Lo que el spec define sobre el negocio —historias, criterios de aceptación, reglas, contrato de la
API, modelo de datos— se respeta. Lo que define sobre la tecnología se traduce a React y Laravel.

## La prueba no consiste en escribir el código

El propósito principal es ver la **capacidad de desempeño con SDD** (*Spec-Driven Development*,
desarrollo guiado por especificación): cómo se lee, se interpreta y se aplica una especificación
para llevarla a un stack distinto. El código es el medio, no el fin.

## Los seis repositorios

| Repositorio | Qué va ahí |
|---|---|
| [`test-simple-stock-flow-docs`](https://github.com/code-sena/test-simple-stock-flow-docs) | El spec: `spec-python/` y `spec-.net/` |
| [`test-simple-stock-flow-api`](https://github.com/code-sena/test-simple-stock-flow-api) | Backend en PHP (Laravel) |
| [`test-simple-stock-flow-app`](https://github.com/code-sena/test-simple-stock-flow-app) | Frontend en React |
| [`test-simple-stock-flow-page`](https://github.com/code-sena/test-simple-stock-flow-page) | Sitio público estático de presentación |
| [`test-simple-stock-flow-infra`](https://github.com/code-sena/test-simple-stock-flow-infra) | Contenedores, red, volúmenes y motor de base de datos vacío |
| [`test-simple-stock-flow-tool`](https://github.com/code-sena/test-simple-stock-flow-tool) | Utilidades: sembrador de datos de demostración |

---

# Documentación Técnica de Infraestructura — Nivel Senior

## 1. Alcance y Principios Fundamentales (ADR-010)

Este repositorio centraliza la orquestación de contenedores mediante Docker Compose para desplegar el ecosistema completo de *Simple Stock Flow*.

### Invariantes Innegociables:
1. **Motor de Base de Datos Vacío:** El contenedor de base de datos (`db` con MySQL 8.4) inicia **completamente vacío**. Este repositorio **NO contiene scripts DDL, ni esquemas iniciales, ni migraciones SQL**.
2. **Soberanía del Esquema:** El dueño absoluto del esquema relacional es la API (`test-simple-stock-flow-api`), la cual aplica sus migraciones al arrancar.
3. **Aislamiento de Red:** Todos los contenedores se comunican a través de una red puente interna dedicada (`stockflow-network`).

---

## 2. Topología de Red y Servicios Orquestados

```
                             RED: stockflow-network (bridge)
                             
  Host Port :8080                                           Host Port :8000
        │                                                         │
        ▼                                                         ▼
┌─────────────────────────┐                               ┌─────────────────────────┐
│  stockflow-app          │  Peticiones de proxy /api     │  stockflow-api          │
│  (Nginx / React SPA)    ├──────────────────────────────►│  (PHP-FPM / Laravel)    │
└─────────────────────────┘                               └───────────┬─────────────┘
                                                                      │
                                                Puerto interno :3306  │ (healthcheck)
                                                                      ▼
                                                          ┌─────────────────────────┐
                                                          │  stockflow-db           │
                                                          │  (MySQL 8.4 LTS)        │
                                                          │  Volumen: db_data       │
                                                          └─────────────────────────┘
```

| Servicio | Imagen / Contexto | Puertos (Host:Contenedor) | Propósito | Dependencias |
|---|---|---|---|---|
| `db` | `mysql:8.4` | `3306:3306` | Motor relacional MySQL 8.4 con healthcheck (`mysqladmin ping`). | Ninguna |
| `api` | `../test-simple-stock-flow-api` | `8000:8000` | Backend Laravel con almacenamiento persistente en `api_storage`. | `db` (healthy) |
| `app` | `../test-simple-stock-flow-app` | `8080:80` | Frontend React empaquetado y servido en Nginx de alto rendimiento. | `api` |

---

## 3. Volúmenes Persistentes

1. **`db_data`:** Volumen Docker con nombre para la persistencia del directorio `/var/lib/mysql`. Garantiza que la información de usuarios, catálogo y transacciones sobreviva al reinicio de los contenedores.
2. **`api_storage`:** Volumen Docker montado en `/var/www/storage/app/public` para el almacenamiento de archivos multimedia y fotos de productos.

---

## 4. Variables de Entorno (`.env`)

Copiar `.env.example` a `.env` en la raíz de este repositorio:

| Variable | Valor por Defecto | Descripción |
|---|---|---|
| `MYSQL_ROOT_PASSWORD` | `rootsecret` | Contraseña administrativa de MySQL. |
| `MYSQL_DATABASE` | `stockflow` | Nombre de la base de datos de la solución. |
| `MYSQL_USER` | `stockflow` | Usuario de conexión para el servicio de backend. |
| `MYSQL_PASSWORD` | `secret` | Contraseña del usuario de base de datos. |
| `MYSQL_PORT` | `3306` | Puerto de exposición en el host (configurable). |
| `APP_KEY` | `base64:...` | Clave de cifrado de la aplicación Laravel. |
| `JWT_SIGNING_KEY` | `clave_secreta_jwt_minimo_32_chars` | Firma HMAC para los tokens de sesión. |
| `ADMIN_USERNAME` | `admin` | Usuario administrador inicial. |
| `ADMIN_PASSWORD` | `Admin123456*` | Contraseña del administrador inicial. |

---

## 5. Guía de Despliegue Rápido

```bash
# 1. Configurar variables
cp .env.example .env

# 2. Levantar toda la infraestructura en modo desacoplado
docker compose up -d --build

# 3. Esperar que el motor de base de datos esté sano
docker compose ps

# 4. Ejecutar las migraciones y semillado desde la API
docker compose exec api php artisan migrate --force
docker compose exec api php artisan db:seed --force

# 5. Ejecutar la verificación automatizada
bash verify.sh
```

### URLs del Sistema Desplegado:
- **Frontend SPA (Portal Web):** `http://localhost:8080`
- **Backend API Directa:** `http://localhost:8000`
- **Healthcheck Probe:** `http://localhost:8000/health`
- **Motor MySQL:** `localhost:3306` (Usuario: `stockflow`, Base: `stockflow`)
