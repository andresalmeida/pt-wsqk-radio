<div align="center">

# WSQK "The Squawk" Radio

</div>

<div align="center">

![Drupal](https://img.shields.io/badge/Drupal-10-0678be?style=for-the-badge&logo=drupal&logoColor=white)
![Vue.js](https://img.shields.io/badge/Vue.js-3-4FC08D?style=for-the-badge&logo=vue.js&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-Alpine-009639?style=for-the-badge&logo=nginx&logoColor=white)

**Drupal 10 Headless CMS + Vue.js Frontend**

*Prueba Técnica - Software Development Consultant (CMS Drupal) *

[Quick Start](#-quick-start) •
[Arquitectura](#-arquitectura) •
[Features](#-features) •
[API](#-api-endpoints)

</div>

---

## Descripción

**WSQK "The Squawk"** es una aplicación web para una estación de radio con estética retro de los 80s inspirada en *Stranger Things*. Implementa una arquitectura **headless** donde:

- **Backend**: Drupal 10 como CMS headless expone contenido vía JSON:API
- **Frontend**: Vue.js 3 consume la API y renderiza la interfaz
- **Infraestructura**: Docker Compose orquesta 4 contenedores

---

## Arquitectura

```mermaid
flowchart TB
    subgraph DOCKER["Docker Compose Network"]
        subgraph FRONTEND["📱 Frontend Container"]
            VUE["Vue.js 3 + Vite<br/>:3000"]
        end
        
        subgraph NGINX["Nginx Container"]
            NG["Nginx Alpine<br/>Reverse Proxy<br/>:80"]
        end
        
        subgraph DRUPAL["Drupal Container"]
            PHP["PHP-FPM 8.2<br/>Drupal 10 CMS"]
        end
        
        subgraph MYSQL["MySQL Container"]
            DB["MySQL 8.0<br/>wsqk_db"]
        end
    end
    
    BROWSER["Browser"]
    
    BROWSER -->|":80"| NG
    BROWSER -->|":3000"| VUE
    VUE -->|"JSON:API"| NG
    NG -->|"FastCGI"| PHP
    PHP -->|"TCP 3306"| DB
    
    style DOCKER fill:#1a1a2e,stroke:#e31c25,stroke-width:3px
    style FRONTEND fill:#4FC08D,stroke:#2c3e50,stroke-width:2px
    style NGINX fill:#009639,stroke:#2c3e50,stroke-width:2px
    style DRUPAL fill:#0678be,stroke:#2c3e50,stroke-width:2px
    style MYSQL fill:#4479A1,stroke:#2c3e50,stroke-width:2px
```

### Flujo de Datos

```mermaid
sequenceDiagram
    participant U as 🖥️ Usuario
    participant V as 📱 Vue.js
    participant N as 🌐 Nginx
    participant D as 💧 Drupal
    participant M as 🗄️ MySQL

    U->>V: Accede a localhost:3000
    V->>N: GET /jsonapi/node/noticia
    N->>D: FastCGI Request
    D->>M: SQL Query
    M-->>D: Data
    D-->>N: JSON Response
    N-->>V: JSON:API Data
    V-->>U: Render UI
```

---

## Quick Start

### Prerrequisitos

- [Docker](https://www.docker.com/get-started) & Docker Compose
- Git

### Instalación

```bash
# 1. Clonar el repositorio
git clone https://github.com/andresalmeida/pt-wsqk-radio.git
cd pt-wsqk-radio

# 2. Levantar los contenedores
docker-compose up -d

# 3. Esperar a que MySQL esté healthy (~30 segundos)
docker-compose ps

# 4. Instalar Drupal (primera vez)
docker exec -it wsqk_drupal bash
cd /var/www/html/web
drush site:install --db-url=mysql://wsqk_user:wsqk_password_2024@mysql/wsqk_db -y
exit
```

### Acceso

| Servicio | URL | Descripción |
|----------|-----|-------------|
| **Frontend** | http://localhost:3000 | Vue.js App |
| **Drupal Admin** | http://localhost/user/login | CMS Backend |
| **JSON:API** | http://localhost/jsonapi | API REST |

---

## Contenedores

| Container | Imagen | Puerto | Función |
|-----------|--------|--------|---------|
| `wsqk_nginx` | `nginx:alpine` | `:80` | Reverse proxy, sirve Drupal |
| `wsqk_drupal` | `php:8.2-fpm` | interno | PHP-FPM + Drupal 10 |
| `wsqk_mysql` | `mysql:8.0` | interno | Base de datos |
| `wsqk_frontend` | `node:20-alpine` | `:3000` | Vue.js dev server |

### Comandos Docker Útiles

```bash
# Ver logs
docker-compose logs -f

# Logs de un servicio específico
docker-compose logs -f drupal

# Entrar al contenedor de Drupal
docker exec -it wsqk_drupal bash

# Entrar a MySQL
docker exec -it wsqk_mysql mysql -u wsqk_user -pwsqk_password_2024 wsqk_db

# Reiniciar todo
docker-compose restart

# Parar y eliminar
docker-compose down

# Parar y eliminar incluyendo volúmenes
docker-compose down -v
```

---

## Features

### Backend (Drupal 10)

- **Headless CMS** - JSON:API habilitado
- **Content Types personalizados**:
  - **Noticia** - título, cuerpo, imagen, fecha
  - **Programa** - nombre, descripción, horario, imagen
  - **Conductor** - nombre, bio, foto
- **Tema personalizado** (`wsqk_theme`)
- **Roles y permisos**:
  - *Editor* - gestiona noticias
  - *Conductor* - gestiona programas y conductores

### Frontend (Vue.js 3)

- **Consumo de JSON:API**
- **Hero con video de YouTube** de fondo
- **Noticias** con cards e imágenes + modal detalle
- **Diseño responsive**
- **Estética Stranger Things / 80s**:
  - Tipografía Bebas Neue
  - Colores: rojo (#e31c25), amarillo (#ffd700), azul oscuro (#0a1628)
  - Efectos glow y neón

---

## API Endpoints

Base URL: `http://localhost/jsonapi`

| Endpoint | Método | Descripción |
|----------|--------|-------------|
| `/node/noticia` | GET | Lista de noticias |
| `/node/noticia?include=field_imagen` | GET | Noticias con imágenes |
| `/node/programa` | GET | Lista de programas |
| `/node/programa?include=field_imagen` | GET | Programas con imágenes |
| `/node/conductor` | GET | Lista de conductores |
| `/node/conductor?include=field_foto` | GET | Conductores con fotos |

### Ejemplo de Response

```json
{
  "data": [
    {
      "type": "node--noticia",
      "id": "uuid-here",
      "attributes": {
        "title": "Nueva programación de verano",
        "body": {
          "value": "<p>Contenido HTML...</p>"
        },
        "created": "2024-01-14T10:00:00+00:00"
      },
      "relationships": {
        "field_imagen": {
          "data": {
            "type": "file--file",
            "id": "file-uuid"
          }
        }
      }
    }
  ],
  "included": [
    {
      "type": "file--file",
      "id": "file-uuid",
      "attributes": {
        "uri": {
          "url": "/sites/default/files/2024-01/imagen.jpg"
        }
      }
    }
  ]
}
```

---

## Estructura del Proyecto

```
wsqk-radio/
├── docker-compose.yml      # Orquestación de contenedores
├── Dockerfile.drupal       # Build de PHP-FPM + extensiones
│
├── drupal/                 # Drupal 10 CMS
│   ├── composer.json
│   ├── vendor/
│   └── web/
│       ├── core/
│       ├── modules/
│       ├── sites/
│       └── themes/
│           └── custom/
│               └── wsqk_theme/    # Tema personalizado
│                   ├── css/style.css
│                   ├── templates/
│                   └── wsqk_theme.info.yml
│
├── frontend/               # Vue.js 3 + Vite
│   ├── package.json
│   ├── vite.config.js
│   ├── index.html
│   └── src/
│       ├── App.vue            # Componente principal + Hero
│       ├── main.js
│       ├── style.css          # Estilos globales
│       └── components/
│           ├── Noticias.vue   # Cards + Modal
│           ├── Programas.vue
│           └── Conductores.vue
│
├── nginx/
│   └── default.conf           # Configuración reverse proxy
│
└── docs/                   # Documentación adicional
```
---

## Desarrollo

### Frontend (Hot Reload)

```bash
cd frontend
npm install
npm run dev
```

### Drupal (Drush)

```bash
docker exec -it wsqk_drupal bash

# Cache clear
drush cr

# Exportar config
drush cex

# Importar config
drush cim

# Ver logs
drush watchdog:show
```

---

## Credenciales por Defecto

| Servicio | Usuario | Contraseña |
|----------|---------|------------|
| MySQL Root | `root` | `root_password_2024` |
| MySQL App | `wsqk_user` | `wsqk_password_2024` |
| Drupal Admin | `admin` | *(definido en instalación)* |

> ⚠️ **Nota**: Cambiar credenciales en producción

---

## 📄 Licencia

Este proyecto fue creado como prueba técnica para demostrar habilidades en:
- Drupal 10 (Headless CMS)
- Vue.js 3 (Frontend SPA)
- Docker (Infraestructura)
- API REST (JSON:API)

---
