# Home Server Configuration

Este proyecto contiene una configuración lista para desplegar múltiples servicios en un servidor casero usando Docker Compose. El objetivo es centralizar y facilitar la administración de servicios útiles para el hogar, como almacenamiento, gestión de descargas, proxy, DNS, automatización, y más.

## Requisitos

- Docker
- Docker Compose
- Red Docker externa llamada `homelab` (créala con: `docker network create homelab`)

## Despliegue

1. Clona este repositorio en tu servidor.
2. Crea un archivo `.env` con las variables necesarias (usuarios, contraseñas, rutas, etc).
3. Ejecuta:
   ```sh
   docker-compose up -d
   ```

## Servicios incluidos

### 1. Portainer

- **Imagen:** `portainer/portainer-ce`
- **Descripción:** Interfaz web para administrar contenedores Docker de forma visual y sencilla.
- **Puertos:** 8000, 9443, 9001

### 2. Nginx Proxy Manager

- **Imagen:** `jc21/nginx-proxy-manager`
- **Descripción:** Proxy inverso con interfaz web para gestionar hosts, certificados SSL y redirecciones fácilmente.
- **Puertos:** 80, 81, 443

### 3. AdGuard Home

- **Imagen:** `adguard/adguardhome`
- **Descripción:** Servidor DNS y bloqueador de publicidad y rastreadores a nivel de red.
- **Puertos:** 53 (DNS), 85 (web), 3000, 853, 784, 8853, 5443

### 4. Samba

- **Imagen:** `dperson/samba`
- **Descripción:** Compartición de archivos en red (SMB/CIFS) para acceder a carpetas desde Windows, macOS o Linux.
- **Puertos:** 139, 445

### 5. Docker Registry

- **Imagen:** `registry`
- **Descripción:** Registro privado de imágenes Docker para almacenar y compartir imágenes localmente.
- **Puerto:** 5000

### 6. Docker Registry UI

- **Imagen:** `joxit/docker-registry-ui`
- **Descripción:** Interfaz web para visualizar y gestionar el registro privado de Docker.
- **Puerto:** 8088

### 7. MinIO

- **Imagen:** `minio/minio`
- **Descripción:** Almacenamiento de objetos compatible con S3, ideal para backups y archivos.
- **Puertos:** 9000 (API), 9002 (Web UI)

### 8. PostgreSQL

- **Imagen:** `postgres`
- **Descripción:** Base de datos relacional de código abierto.
- **Puerto:** 5432

### 9. n8n

- **Imagen:** `n8nio/n8n`
- **Descripción:** Plataforma de automatización de flujos de trabajo (workflows) tipo low-code.
- **Puerto:** 5678

### 10. Gitea

- **Imagen:** `gitea/gitea`
- **Descripción:** Servidor Git ligero y autohospedado, similar a GitHub.
- **Puertos:** 3002 (web), 2222 (SSH)

### 11. Plex

- **Imagen:** `jaymoulin/plex`
- **Descripción:** Servidor multimedia para organizar y transmitir películas, series y música en la red local.
- **Puertos:** 32400, 33400 (usando network_mode: host)

### 12. Transmission

- **Imagen:** `jaymoulin/transmission`
- **Descripción:** Cliente BitTorrent ligero y eficiente, con interfaz web.
- **Puertos:** 9091 (web), 51413 (p2p)

### 13. FlexGet

- **Imagen:** `wiserain/flexget`
- **Descripción:** Automatizador de descargas (torrents, RSS, etc) altamente configurable.
- **Puerto:** 5050

## Volúmenes y persistencia

Cada servicio utiliza volúmenes para persistir datos importantes. Revisa el `docker-compose.yml` para personalizar rutas y credenciales según tus necesidades.

## Personalización

- Modifica el archivo `.env` para tus usuarios, contraseñas y rutas locales.
- Puedes agregar/quitar servicios según tus necesidades.

## Créditos

- Inspirado en la comunidad de auto-hosting y homelab.

---

¡Contribuciones y sugerencias son bienvenidas!
