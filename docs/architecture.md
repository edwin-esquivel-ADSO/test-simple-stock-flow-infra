# Arquitectura de Infraestructura — test-simple-stock-flow-infra

> **Orquestación Multi-Contenedor y Persistencia**  
> **Prueba técnica · Ficha ADSO 3413974**  
> Rige el principio de aislamiento y soberanía del esquema (ADR-001 / ADR-010).

---

## 1. Alcance
Este repositorio define la topología de contenedores mediante `docker-compose.yml`, aislando la red de los servicios y gestionando el ciclo de vida de los datos persistentes.

---

## 2. Invariante de Motor Vacío
- El contenedor `db` utiliza la imagen oficial `mysql:8.4` sin scripts de inicialización en `/docker-entrypoint-initdb.d/`.
- Este repositorio **nunca ejecuta DDL ni migraciones directas**.
- El contenedor `api` tiene una directiva `depends_on: db: condition: service_healthy` para asegurar que el motor MySQL esté aceptando conexiones antes de que la aplicación intente ejecutar las migraciones mediante `artisan migrate`.

---

## 3. Topología de Red y Volúmenes
- **Red `stockflow-network`:** Tipo bridge privada. Solo los puertos definidos en `ports` son accesibles desde el host de desarrollo.
- **Volumen `db_data`:** Protege los datos ante comandos `docker compose down`. Para reiniciar la base de datos desde cero de forma explícita, se usa `docker compose down -v`.
