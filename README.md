# Home Server Configuration

Este proyecto contiene una configuración lista para desplegar múltiples servicios en un servidor casero usando Docker Compose. El objetivo es centralizar y facilitar la administración de servicios útiles para el hogar, como almacenamiento, proxy, DNS, bases de datos, automatización, monitoreo y domótica.

## Requisitos

- Docker
- Docker Compose
- Red Docker externa llamada `homelab` (créala con: `docker network create homelab`)

## Estructura del proyecto

El archivo `docker-compose.yml` en la raíz no define servicios directamente: usa `include:` para cargar archivos de configuración organizados por categoría dentro de `services/`:

```
services/
├── applications/     # Gitea, Gitea Runner, n8n, Nginx Proxy Manager, AdGuard Home, Samba
├── databases/         # PostgreSQL, Redis
├── monitoring/        # Portainer, cAdvisor
└── home-assistant/    # Home Assistant
```

Todos los servicios se conectan a la red externa `homelab`, que debe existir antes de levantar el stack.

## Despliegue

1. Clona este repositorio en tu servidor.
2. Crea la red externa si no existe: `docker network create homelab`.
3. Crea un archivo `.env` a partir de `.env_example` con tus variables (usuarios, contraseñas, rutas, etc).
4. Ejecuta:
   ```sh
   docker compose up -d
   ```

Para reiniciar un solo servicio tras modificar su configuración:
```sh
./restart-container.sh <nombre-del-servicio>
```

Para sincronizar este repositorio con el servidor remoto vía rsync/SSH:
```sh
./sync.sh
```
> El destino remoto está fijo en `sync.sh` (`gumonet@192.16.0.20:/home/gumonet/docker-cluster/homelab`); ajústalo si tu servidor es otro.

## Servicios incluidos

### Aplicaciones

#### Gitea + Gitea Runner
- **Imágenes:** `gitea/gitea`, `gitea/act_runner`
- **Descripción:** Servidor Git ligero autohospedado con soporte de CI/CD (Gitea Actions) mediante un runner dedicado.
- **Puertos:** 3002 (web), 2222 (SSH)
- **Variables requeridas:** `GITEA_RUNNER_REGISTRATION_TOKEN` (se obtiene desde el panel de administración de Gitea)

#### n8n
- **Imagen:** `n8nio/n8n`
- **Descripción:** Plataforma de automatización de flujos de trabajo (workflows) tipo low-code. Usa PostgreSQL como base de datos.
- **Puerto:** 5678

#### Nginx Proxy Manager
- **Imagen:** `jc21/nginx-proxy-manager`
- **Descripción:** Proxy inverso con interfaz web para gestionar hosts, certificados SSL y redirecciones fácilmente.
- **Puertos:** 80, 81, 443

#### AdGuard Home
- **Imagen:** `adguard/adguardhome`
- **Descripción:** Servidor DNS y bloqueador de publicidad y rastreadores a nivel de red.
- **Puertos:** 53 (DNS), 85 (web), 3000, 853, 784, 8853, 5443
- **Nota:** en hosts Ubuntu es necesario desactivar `systemd-resolved` para poder usar el puerto 53:
  ```sh
  sudo systemctl stop systemd-resolved.service
  sudo systemctl disable systemd-resolved.service
  ```

#### Samba
- **Imagen:** `dperson/samba`
- **Descripción:** Compartición de archivos en red (SMB/CIFS) para acceder a carpetas desde Windows, macOS o Linux.
- **Puertos:** 139, 445

### Bases de datos

#### PostgreSQL
- **Imagen:** `postgres`
- **Descripción:** Base de datos relacional usada por n8n y otros servicios.
- **Puerto:** 5432
- **Variables requeridas:** `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`

#### Redis
- **Imagen:** `redis:7.4-alpine`
- **Descripción:** Almacén en memoria clave-valor, con persistencia (`appendonly`) habilitada.
- **Puerto:** 6379

### Monitoreo

#### Portainer
- **Imagen:** `portainer/portainer-ce`
- **Descripción:** Interfaz web para administrar contenedores Docker de forma visual y sencilla.
- **Puertos:** 8000, 9443, 9001

#### cAdvisor
- **Imagen:** `google/cadvisor`
- **Descripción:** Recolecta y expone métricas de uso de recursos (CPU, memoria, red) de los contenedores en ejecución.
- **Puerto:** 8080 (expuesto internamente en la red `homelab`)

### Domótica

#### Home Assistant
- **Imagen:** `ghcr.io/home-assistant/home-assistant:stable`
- **Descripción:** Plataforma de automatización del hogar. Corre en `network_mode: host` y modo privilegiado para el descubrimiento de dispositivos en la red local.
- **Variables requeridas:** `HA_CONFIG_PATH` (ruta al directorio de configuración), `TZ`

## Variables de entorno

Consulta `.env_example` para la lista completa de variables esperadas antes de crear tu `.env`:

- `MEDIA`, `STORAGE` — rutas usadas por Samba
- `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB`
- `GITEA_RUNNER_REGISTRATION_TOKEN`
- `HA_CONFIG_PATH`, `TZ`

## Volúmenes y persistencia

Cada servicio utiliza volúmenes con nombre para persistir sus datos (bases de datos, configuración de Gitea, AdGuard, Portainer, etc). Revisa el archivo `.yaml` correspondiente dentro de `services/` para personalizar rutas y credenciales según tus necesidades.

## Personalización

- Modifica el archivo `.env` para tus usuarios, contraseñas y rutas locales.
- Para agregar un nuevo servicio, crea un archivo `services/<categoría>/<nombre>.yaml` con su propia definición y agrégalo al bloque `include:` de `docker-compose.yml`.

## Créditos

- Inspirado en la comunidad de auto-hosting y homelab.

---

¡Contribuciones y sugerencias son bienvenidas!
