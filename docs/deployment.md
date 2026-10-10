# Guía de Despliegue y Troubleshooting — test-simple-stock-flow-infra

> **Puesta en Producción y Operación Continua**  
> **Prueba técnica · Ficha ADSO 3413974**

---

## 1. Ciclo de Vida del Despliegue

### Paso 1: Configurar Credenciales
```bash
cp .env.example .env
```
Asegurar que `APP_KEY` tenga una clave generada y que `MYSQL_PASSWORD` coincida entre la base de datos y la API.

### Paso 2: Construir y Levantar
```bash
docker compose up -d --build
```

### Paso 3: Migración Inicial
```bash
docker compose exec api php artisan migrate --force
docker compose exec api php artisan db:seed --force
```

---

## 2. Diagnóstico y Troubleshooting

| Síntoma | Causa Frecuente | Solución |
|---|---|---|
| Contenedor `api` reinicia en bucle | `APP_KEY` ausente o motor `db` no listo | Generar clave con `php artisan key:generate` y verificar logs con `docker compose logs api`. |
| Error `Access denied for user stockflow` | Desalineación de contraseñas en `.env` | Borrar el volumen persistente con `docker compose down -v` y volver a levantar para que MySQL tome la nueva clave. |
| Error 409 al registrar ventas | Conflicto de versión en concurrencia optimista | Comportamiento esperado por diseño cuando dos ventas compiten por la última unidad. |
| Error 401 en llamadas del Seeder | Token JWT expirado o credenciales incorrectas | Re-ejecutar login en el script de siembra. |
