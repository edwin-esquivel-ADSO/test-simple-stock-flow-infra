# Arquitectura de Infraestructura — test-simple-stock-flow-infra

> **Orquestación de Contenedores y Red**
> **Prueba técnica · Ficha ADSO 3413974**

---

## 1. Alcance
Este repositorio contiene la definición de contenedores Docker (`docker-compose.yml`), volúmenes y redes para orquestar la solución.

## 2. Invariante de Persistencia (ADR-001 / ADR-010)
- El contenedor `db` ejecuta MySQL 8.4 iniciando con el motor **completamente vacío**.
- Este repositorio **NO contiene scripts DDL, ni esquemas iniciales, ni migraciones**.
- El propietario del esquema relacional es exclusivamente `test-simple-stock-flow-api`.
