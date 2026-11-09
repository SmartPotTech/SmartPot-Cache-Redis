# SmartPot-Cache-Redis (Legacy Stack A Reference)

> [!WARNING]
> **Deprecated (Legacy Stack A Reference Implementation)**  
> This repository contains the legacy Stack A (v1.0) caching layer implemented using Redis 8. It has been **deprecated and superseded in production** by the canonical modern Stack B cache component: [**`SmartPot-Cache`**](https://github.com/SmartPotTech/SmartPot-Cache) (Valkey 9.1.3-alpine). Preserved solely for historical reference and configuration archaeology.

## Project Status

[![Cache Image CI](https://github.com/SmartPotTech/SmartPot-Cache-Redis/actions/workflows/ci.yml/badge.svg)](https://github.com/SmartPotTech/SmartPot-Cache-Redis/actions/workflows/ci.yml)
[![Publish Docker Images](https://github.com/SmartPotTech/SmartPot-Cache-Redis/actions/workflows/packaging.yml/badge.svg)](https://github.com/SmartPotTech/SmartPot-Cache-Redis/actions/workflows/packaging.yml)

## Description

SmartPot-Cache-Redis provided in-memory caching, rate limiting, and session state storage for the legacy Stack A architecture using **Redis 8** with mandatory authentication and RDB snapshots.

## Seguridad

- Contraseña obligatoria de al menos 16 caracteres (`REDIS_PASSWORD`); sin ella el contenedor no arranca.
- La configuración se genera en `/tmp` al arrancar y nunca se imprime en los logs.
- `FLUSHALL`, `FLUSHDB`, `CONFIG`, `DEBUG` y `SHUTDOWN` están deshabilitados.
- Corre como el usuario `999`, con sistema de archivos de solo lectura y sin capacidades de Linux.
- En producción no publica puertos: solo la API lo alcanza por la red interna de Docker.

## Project Structure

```text
SmartPot-Cache/
├── .github/
│   ├── dependabot.yml
│   └── workflows/
│       ├── ci.yml              # Construye la imagen y corre la prueba de seguridad
│       ├── packaging.yml       # Publica la imagen en GHCR con SBOM y procedencia (y en Docker Hub con credenciales)
│       └── deploy.yml          # Pide el despliegue al workflow central de SmartPotTech/.github
├── tests/
│   └── smoke.sh                # Autenticación, comandos bloqueados y logs sin secretos
├── compose.yaml
├── Dockerfile                  # redis:8.8-alpine sin privilegios
├── entrypoint.sh               # Genera la configuración con la contraseña del entorno
└── .env.example
```

## Guía de Instalación

### Despliegue en un comando

La imagen es pública: `ghcr.io/smartpottech/smartpot-cache`. La plataforma completa, con este servicio incluido, se
levanta con un solo comando y sin descargar el código:

```bash
curl -fsSL https://raw.githubusercontent.com/SmartPotTech/.github/main/docker/demo/compose.yaml | docker compose -f - up -d --wait
```

Abre http://localhost:5173 e ingresa con `demo@smartpot.app` / `SmartPot2026`. Se detiene con
`docker compose -p smartpot-demo down` (con `-v` borra también los datos). Más detalle en la
[demo](https://github.com/SmartPotTech/.github/tree/main/docker/demo).

Solo este servicio, con la imagen publicada:

```bash
docker run -d --name smartpot-cache --read-only --tmpfs /tmp --cap-drop ALL \
  -p 127.0.0.1:6379:6379 -e REDIS_PASSWORD=<mínimo-16-caracteres> \
  ghcr.io/smartpottech/smartpot-cache:latest
```

### Desde el código con Docker Compose

```bash
cp .env.example .env    # define REDIS_PASSWORD
docker compose up -d
```

| Variable          | Por defecto | Descripción                                                |
|-------------------|-------------|------------------------------------------------------------|
| `REDIS_PASSWORD`  | —           | Obligatoria, mínimo 16 caracteres                          |
| `REDIS_MAXMEMORY` | `128mb`     | Memoria máxima; se descartan las llaves menos usadas (LRU) |
| `REDIS_DATABASES` | `4`         | Bases lógicas disponibles                                  |

### Prueba de humo

```bash
docker build -t smartpot-cache:ci .
sh tests/smoke.sh smartpot-cache:ci
```

## Imagen publicada

```bash
docker pull ghcr.io/smartpottech/smartpot-cache:latest
```

Cada cambio en `main` pasa por el CI, publica la imagen en GHCR (y en Docker Hub como réplica cuando el repositorio
tiene credenciales) y pide el despliegue al workflow central
de [SmartPotTech/.github](https://github.com/SmartPotTech/.github), que actualiza producción de a uno y verifica
`/health`.

## Documentación

Redis guarda solo datos efímeros: si se pierde, la plataforma sigue. Su documentación propia está en [
`docs/`](docs/SmartPot_Cache_Documentation.md) (también en [DOCX](docs/SmartPot_Cache_Documentation.docx)
y [PDF](docs/SmartPot_Cache_Documentation.pdf)), con sus diagramas en [`docs/diagrams`](docs/diagrams): el general de la
imagen y el de las llaves.
La [documentación técnica](https://github.com/SmartPotTech/.github/blob/main/docs/SmartPot_Technical_Documentation.md)
explica para qué usa la API cada llave y cómo actúa si Redis no responde. Los diagramas generales muestran la plataforma
completa en una sola imagen ampliable:

- [Arquitectura completa](https://github.com/SmartPotTech/.github/blob/main/docs/diagrams/SmartPot_Global_01_Architecture.svg):
  dónde vive Redis y qué llaves guarda
- [Linaje de los datos](https://github.com/SmartPotTech/.github/blob/main/docs/diagrams/SmartPot_Global_04_Data_Lineage.svg):
  en qué momento de cada flujo se consulta o se escribe una llave

## License

Este proyecto está bajo la licencia MIT. Consulta el archivo [LICENSE](LICENSE) para más detalles.
