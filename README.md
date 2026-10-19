# n8n Service

Despliegue local de [n8n](https://n8n.io/) — plataforma de automatización de workflows y flujos de trabajo — usando **Docker Compose**.

n8n te permite conectar servicios, APIs y bases de datos mediante nodos visuales para automatizar tareas sin escribir código (o con poco código).

---

## Requisitos

- [Docker](https://docs.docker.com/get-docker/) (v20.10 o superior)
- [Docker Compose](https://docs.docker.com/compose/install/) (v2 o superior; viene incluido como `docker compose`)

Verifica la instalación:

```bash
docker --version
docker compose version
```

---

## Instalación y arranque

1. Clona o ubica el proyecto y entra en la carpeta:

   ```bash
   cd n8n_service
   ```

2. Levanta el servicio en segundo plano:

   ```bash
   docker compose up -d
   ```

3. Abre n8n en el navegador:

   ```
   http://localhost:5678
   ```

   La primera vez te pedirá crear la cuenta de propietario (email y contraseña).

---

## Comandos útiles

| Acción | Comando |
| --- | --- |
| Iniciar el servicio | `docker compose up -d` |
| Detener el servicio | `docker compose down` |
| Reiniciar el servicio | `docker compose restart` |
| Ver el estado de los contenedores | `docker compose ps` |
| Ver los logs en vivo | `docker compose logs -f n8n` |
| Actualizar a la última imagen | `docker compose pull && docker compose up -d` |

---

## Configuración

El archivo `docker-compose.yml` define el servicio:

```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n_container
    restart: unless-stopped
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
    environment:
      - N8N_SECURE_COOKIE=false # Desactiva cookies seguras solo para desarrollo en localhost
      - TZ=America/Bogota      # Ajusta tu zona horaria para los nodos de tiempo/cron
```

### Variables de entorno

| Variable | Valor | Descripción |
| --- | --- | --- |
| `N8N_SECURE_COOKIE` | `false` | Desactiva las cookies *secure* para poder trabajar sobre `http://localhost` en desarrollo. **No uses `false` en producción con HTTPS.** |
| `TZ` | `America/Bogota` | Zona horaria del contenedor. Afecta a los nodos de fecha/hora y a las ejecuciones programadas (cron). Cámbiala según tu región, por ejemplo `Europe/Madrid` o `UTC`. |

### Puerto

n8n escucha en el puerto `5678`. Si está ocupado, cambia el mapeo en `ports`, por ejemplo `"8080:5678"`, y accede por `http://localhost:8080`.

### Persistencia de datos

Los workflows, credenciales y ejecuciones se guardan en el volumen Docker `n8n_data`, montado en `/home/node/.n8n`. Esto significa que los datos **sobreviven** a reinicios y a la recreación del contenedor.

- El volumen es gestionado por Docker (`external: false`).
- Para hacer una copia de seguridad, exporta los workflows desde la interfaz o copia el volumen.
- ⚠️ Para eliminar también los datos usa `docker compose down -v`.

---

## Estructura del proyecto

```
n8n_service/
├── docker-compose.yml   # Definición del servicio n8n
├── README.md            # Esta documentación
└── .superset/
    └── config.json      # Configuración de arranque del workspace (Superset)
```

---

## Notas para producción

Esta configuración está pensada para **desarrollo local**. Antes de usarla en producción:

- Habilita HTTPS y vuelve a poner `N8N_SECURE_COOKIE=true`.
- Define `N8N_HOST`, `N8N_PROTOCOL` y `WEBHOOK_URL` con tu dominio real.
- Configura una base de datos externa (PostgreSQL) en lugar del SQLite por defecto para mayor robustez.
- Protege las credenciales con `N8N_ENCRYPTION_KEY` y usa secretos, no valores en texto plano.

Consulta la [documentación oficial de n8n](https://docs.n8n.io/) para todas las opciones de configuración.

---

## Recursos

- Documentación de n8n: <https://docs.n8n.io/>
- Repositorio de n8n: <https://github.com/n8n-io/n8n>
- Imagen Docker: <https://hub.docker.com/r/n8nio/n8n>
