<p align="center">
  <img src="assets/icons/agentedso-16.png" alt="DSOAgent" width="80"/>
</p>

<h1 align="center">DSOAgent v0.9.8</h1>

<p align="center">
  <strong>Plataforma de automatizacion DevSecOps para gestion integral de vulnerabilidades, tickets y evidencias.</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-0.9.8-blue" alt="Version"/>
  <img src="https://img.shields.io/badge/python-3.10%2B-green" alt="Python"/>
  <img src="https://img.shields.io/badge/tests-1104%20Windows%20%7C%201091%20Linux-brightgreen" alt="Tests"/>
  <img src="https://img.shields.io/badge/platform-Windows%20%7C%20Ubuntu-lightgrey" alt="Platform"/>
  <img src="https://img.shields.io/badge/license-MIT-orange" alt="License"/>
</p>

<p align="center">
  <a href="#inicio-rapido">Inicio Rapido</a> ·
  <a href="#tutorial-paso-a-paso">Tutorial</a> ·
  <a href="#proveedores-y-conectores">Proveedores</a> ·
  <a href="#seguridad">Seguridad</a> ·
  <a href="docs/TECHNICAL.md">Docs Tecnica</a>
</p>

---

## Que es DSOAgent

DSOAgent es una **aplicacion de escritorio** que automatiza el ciclo de vida de hallazgos de ciberseguridad:

1. Importa resultados de escaneos SAST/DAST desde Excel
2. Crea tickets automaticamente en Azure DevOps, Jira, o cualquier API REST
3. Gestiona evidencias con trazabilidad criptografica (SHA256)
4. Genera reportes ejecutivos exportables
5. Ejecuta scripts custom en sandbox aislado

**No necesita base de datos** — toda la configuracion es JSON cifrado local.
**No necesita servidor** — es 100% desktop (Windows/Linux).

---

## Inicio Rapido

### Requisitos

- **Python 3.10** o superior
- **Windows 10/11** o **Ubuntu 24.04**
- Conexion a internet (solo para instalar dependencias)

### Instalacion en Windows

```bat
:: 1. Clonar el repositorio
git clone https://github.com/tu-usuario/DSOAgent.git
cd DSOAgent

:: 2. Crear entorno virtual
python -m venv venv
venv\Scripts\activate

:: 3. Actualizar pip (recomendado)
python.exe -m pip install --upgrade pip --trusted-host pypi.org --trusted-host files.pythonhosted.org

:: 4. Instalar dependencias
pip install -r requirements.txt

# Si hay error SSL (corporativo/proxy), usar:
pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org -r requirements.txt

:: 5. Ejecutar la aplicacion
python main.py
```

### Instalacion en Ubuntu 24.04

```bash
# 1. Clonar el repositorio
git clone https://github.com/tu-usuario/DSOAgent.git
cd DSOAgent

# 2. Ejecutar setup automatizado (instala deps del sistema + venv + pip)
chmod +x setup_linux.sh
./setup_linux.sh

# 3. Activar entorno e iniciar
source venv/bin/activate
python main.py
```

### Dependencias del sistema (Ubuntu)

El script `setup_linux.sh` las instala automaticamente, pero si prefieres hacerlo manual:

```bash
sudo apt-get install python3-tk libxcb1 libxkbcommon0 libx11-6 libxext6
sudo apt-get install python3-gi gir1.2-appindicator3-0.1  # para system tray
```

---

## Tutorial Paso a Paso

### 1. Primer inicio

Al ejecutar `python main.py` por primera vez:

- Se crea automaticamente una **clave maestra** para cifrado (archivo `vault.key`)
- Se genera el directorio de configuracion en `%APPDATA%/DSOAgent/config` (Windows) o `~/.dsoagent/config` (Linux)
- Se abre la GUI con tema oscuro por defecto

### 2. Crear tu primer proyecto

1. Ir a la vista **Proyectos** (sidebar izquierdo)
2. Click en **Nuevo Proyecto**
3. Llenar: nombre del proyecto, cliente, descripcion
4. Se genera un ID automatico tipo `CASE-2026-001`
5. El proyecto queda seleccionado como activo

### 3. Configurar un proveedor de tickets

Ve a **Configuracion > Proveedores** y crea una nueva conexion:

#### Opcion A: Azure DevOps

| Campo | Valor ejemplo |
|-------|---------------|
| Tipo | Azure DevOps |
| Nombre | Mi Azure |
| Organization | tu-organizacion |
| Project | tu-proyecto |
| Token | tu-personal-access-token |

#### Opcion B: Jira Cloud

| Campo | Valor ejemplo |
|-------|---------------|
| Tipo | Jira Cloud |
| Nombre | Mi Jira |
| URL Base | tu-empresa.atlassian.net |
| Email | tu-email@empresa.com |
| Token | tu-api-token |
| Project Key | SEC |

#### Opcion C: Cualquier API REST (HTTP Generico)

Este es el conector mas flexible — funciona con **cualquier API** que maneje tickets/issues:

| Campo | Valor ejemplo |
|-------|---------------|
| Tipo | HTTP Generico |
| Nombre | ServiceNow Prod |
| URL Base | https://tu-instancia.service-now.com |
| Auth Type | basic |
| Usuario | admin |
| Token/Password | tu-password |
| Endpoint Test | /api/now/table/sys_user?sysparm_limit=1 |
| Endpoint Crear | /api/now/table/incident |
| Endpoint Obtener | /api/now/table/incident/{id} |
| Endpoint Actualizar | /api/now/table/incident/{id} |
| Endpoint Eliminar | /api/now/table/incident/{id} |
| Endpoint Listar | /api/now/table/incident |

4. Click **Probar Conexion** para validar
5. Click **Guardar**

### 4. Importar hallazgos

1. Ir a **Escaneos**
2. Click **Importar Excel**
3. Seleccionar archivo con hallazgos (formato: columnas con titulo, descripcion, severidad)
4. Los hallazgos se cargan en la tabla con busqueda rapida

### 5. Crear tickets desde hallazgos

1. Seleccionar uno o mas hallazgos en la tabla
2. Click **Crear Ticket**
3. Se usa el proveedor configurado para crear el ticket automaticamente
4. El ticket queda vinculado al hallazgo con su ID

### 6. Gestionar evidencias

1. Ir a **Evidencias**
2. Arrastrar o seleccionar imagenes/capturas
3. Se genera hash SHA256 automaticamente para trazabilidad
4. Las imagenes se convierten a formato estandar y se asocian al proyecto

### 7. Generar reportes

1. Ir a **Reportes**
2. Seleccionar tipo: Executive Summary o Full Report
3. Click **Exportar Excel**
4. Se genera un archivo `.xlsx` con todos los hallazgos, tickets y evidencias

---

## Proveedores y Conectores

DSOAgent usa un sistema de **proveedores dinamicos** — puedes conectar multiples APIs simultaneamente y activar/desactivar cada una:

### Proveedores Builtin (listos para usar)

| Proveedor | Tipo de Auth | Para que sirve |
|-----------|-------------|----------------|
| **Azure DevOps** | Personal Access Token | Work Items (Bug, Task, Feature) |
| **Jira Cloud** | Email + API Token | Issues (Bug, Story, Task) |
| **HTTP Generico** | Bearer / Basic / Header / None | Cualquier API REST |
| **Git** | Token + URL | Clone/Pull de repositorios |
| **Scripts** | Local | Ejecucion de scripts en sandbox |
| **SharePoint** | Token | Upload/download de archivos |
| **SAST/DAST** | Configurable | Integracion con escaner |

### HTTP Generico: el conector universal

El conector HTTP Generico te permite conectar con **cualquier API REST** sin escribir codigo. Soporta:

- **4 tipos de autenticacion**: Bearer Token, Basic Auth, Header custom, Sin auth
- **Endpoints configurables**: cada operacion (crear, leer, actualizar, eliminar, listar) apunta a su propia ruta
- **Field mappings**: mapea los campos de DSOAgent (title, description, status) a los nombres que use la API destino
- **Respuestas nested**: detecta automaticamente wrappers como `{"result": {...}}` (ServiceNow), `{"issue": {...}}` (Redmine), arrays directos (GitHub)
- **Campos extra**: agrega key-value adicionales sin limite

### Plantillas predefinidas

Para no empezar de cero, hay plantillas que precargan la configuracion de APIs conocidas:

| Plantilla | Auth | Endpoints preconfigurados |
|-----------|------|---------------------------|
| **ServiceNow** | Basic | `/api/now/table/incident` |
| **GitHub Issues** | Bearer | `/repos/{owner}/{repo}/issues` |
| **GitLab Issues** | Header (PRIVATE-TOKEN) | `/api/v4/projects/{id}/issues` |
| **Redmine** | Header (X-Redmine-API-Key) | `/issues.json` |
| **API REST Custom** | Bearer | `/api/tickets` (en blanco) |

### Campos extra dinamicos

Cada conexion puede tener **campos adicionales** mas alla de los definidos por el tipo:

```
Ejemplo: Conexion ServiceNow
├── Campos del tipo (definidos):
│   ├── base_url = https://instancia.service-now.com
│   ├── auth_type = basic
│   ├── auth_user = admin
│   └── endpoint_create = /api/now/table/incident
│
└── Campos extra (agregados por el usuario):
    ├── assignment_group = IT-Support
    ├── caller_id = USR001
    └── category = software
```

Los campos extra se persisten, se exportan/importan, y no rompen la logica del provider.

---

## Arquitectura del Proyecto

```
DSOAgent/
├── main.py                      # Entry point de la aplicacion
├── requirements.txt             # Dependencias (pip install -r ...)
├── pyproject.toml               # Metadata del proyecto
│
├── agent/                       # Motor del agente
│   ├── orchestrator.py          # Orquestador principal
│   └── contracts.py             # Interfaces e inyeccion de dependencias
│
├── gui/                         # Interfaz grafica (14 vistas + utilidades)
│   ├── main_window.py           # Ventana principal + sidebar + system tray
│   ├── dashboard_view.py        # Metricas en tiempo real
│   ├── findings_view.py         # Escaneos SAST/DAST
│   ├── tickets_view.py          # Crear/gestionar tickets
│   ├── evidence_view.py         # Evidencias con SHA256
│   ├── reports_view.py          # Reportes Excel
│   ├── settings_view.py         # Configuracion + Proveedores
│   ├── git_view.py              # Clone/Pull repositorios
│   ├── scheduler_view.py        # Tareas programadas (Windows)
│   ├── scripts_view.py          # Scripts custom + sandbox
│   ├── logs_view.py             # Visor de logs
│   ├── help_view.py             # Manuales + ayuda
│   ├── import_wizard.py         # Wizard importacion Excel (6 pasos)
│   └── i18n.py                  # Internacionalizacion ES/EN
│
├── data/                        # Capa de datos
│   ├── provider_registry.py     # Registro dinamico de proveedores
│   ├── config_manager.py        # Configuracion de la app
│   ├── config_atomic.py         # Escritura atomica segura
│   ├── project_manager.py       # Proyectos (CASE-YYYY-NNN)
│   ├── excel_handler.py         # Manejo de Excel con pandas
│   ├── excel_importer.py        # Importacion flexible (15 presets)
│   ├── excel_presets.py         # Presets de formatos Excel
│   ├── evidence_manager.py      # Motor de evidencias
│   └── report_manager.py        # Generacion de reportes
│
├── integrations/                # Conectores a servicios externos
│   ├── ticket_service.py        # AbstractTicketAPI (interfaz base)
│   ├── azure_provider.py        # Azure DevOps Work Items
│   ├── jira_provider.py         # Jira Cloud Issues
│   ├── http_provider.py         # HTTP Generico (cualquier REST API)
│   ├── connection_templates.py  # Plantillas predefinidas
│   └── git_manager.py           # Git clone/pull
│
├── sandbox/                     # Ejecucion segura de scripts
│   ├── runner.py                # SandboxRunner
│   ├── multiprocess_runner.py   # Aislamiento real de procesos
│   ├── validator.py             # Validacion pre-ejecucion
│   └── contract.py              # Contrato de scripts
│
├── security/                    # Seguridad y cifrado
│   └── vault.py                 # Fernet + PBKDF2 + Recovery Key
│
├── utils/                       # Utilidades compartidas
│   ├── paths.py                 # Resolucion de rutas (venv/exe/linux)
│   ├── logging_system.py        # Sistema de logs
│   ├── rate_limiter.py          # Rate limiting para APIs
│   ├── retry.py                 # Reintentos con backoff
│   ├── timezone.py              # Zona horaria
│   ├── tray.py                  # System tray (Windows/Linux)
│   ├── user_id.py               # UUID + machine ID
│   ├── task_scheduler.py        # Tareas programadas (schtasks)
│   └── health.py                # Health check del sistema
│
├── docs/                        # Documentacion
│   ├── PROMPT.md                # Reglas del proyecto (obligatorio leer)
│   ├── QUICKSTART.md            # Setup desde cero en VS Code
│   ├── CHANGELOG.md             # Historial de versiones
│   ├── GUÍA_RÁPIDA.md           # Guia simple para presentaciones
│   ├── PLANNING.md              # Cronograma, progreso, roadmap
│   ├── TECHNICAL.md             # Arquitectura detallada
│   └── manuales/                # Manuales detallados
│       ├── manual-desarrollo.md # Setup dev, arquitectura, tests
│       ├── manual-tecnico.md    # Instalacion, proveedores, seguridad
│       └── manual-usuario.md    # Flujo de trabajo, GUI, shortcuts
│
├── DSOAgent.spec                # PyInstaller Windows (.exe)
├── DSOAgent_linux.spec          # PyInstaller Linux
├── Dockerfile.gui               # Docker con GUI (X11)
├── docker-compose.yml           # Orquestacion Docker
└── setup_linux.sh               # Setup automatizado Ubuntu
```

---

## Modos de Despliegue

### 1. Desarrollo local (venv)

Lo que ya hiciste en "Inicio Rapido". Ideal para desarrollo y testing.

### 2. Ejecutable standalone (.exe / binario)

Genera un ejecutable que no necesita Python instalado:

```bash
# Windows
pyinstaller DSOAgent.spec --clean

# Ubuntu
pyinstaller DSOAgent_linux.spec --clean
```

El ejecutable queda en `dist/DSOAgent/`. La configuracion se guarda en:
- **Windows**: `%APPDATA%\DSOAgent\config\`
- **Linux**: `~/.dsoagent/config/`

### 3. Docker (solo desarrollo con GUI)

```bash
docker compose -f docker-compose.yml up gui
```

Requiere X11 forwarding o Xvfb para display virtual.

---

## Seguridad

| Aspecto | Implementacion |
|---------|----------------|
| **Cifrado en reposo** | Fernet (AES128-CBC + HMAC-SHA256) |
| **Derivacion de clave** | PBKDF2 con 480,000 iteraciones |
| **Recovery Key** | Clave de recuperacion para disaster recovery |
| **Credenciales** | Nunca en texto plano — siempre via SecurityVault |
| **Rate limiting** | Proteccion contra abuso de APIs externas |
| **Sandboxing** | Scripts ejecutados en procesos aislados |
| **Subprocesos** | Sin `shell=True` — solo `asyncio.create_subprocess_exec` |

---

## Configuracion avanzada

### Variables de entorno (opcionales)

| Variable | Descripcion | Default |
|----------|-------------|---------|
| `TIMEZONE` | Zona horaria | America/Mexico_City |

### Archivos de configuracion (auto-generados)

| Archivo | Contenido |
|---------|-----------|
| `config/settings.json` | Configuracion general de la app |
| `config/providers/types.json` | Tipos de proveedores registrados |
| `config/providers/connections.json` | Conexiones configuradas (cifradas) |
| `vault.key` | Clave maestra de cifrado |

### Importar/Exportar conexiones

Desde la GUI (Configuracion > Proveedores):
- **Exportar**: genera un JSON con todas las conexiones (tokens cifrados)
- **Importar**: carga conexiones desde un JSON exportado

Desde codigo (util para scripts de automatizacion):

```python
from data.provider_registry import ProviderRegistry

reg = ProviderRegistry()
reg.export_connections("backup.json")
reg.import_connections("backup.json")
```

---

## Capacidades

### Disponible ahora (v0.9.8)

| Capacidad | Detalle |
|-----------|--------|
| **GUI completa** | 12 vistas (Dashboard, Proyectos, Tickets, Evidencias, Reportes, Git, Tareas, Scripts, Configuracion, Ayuda, Logs) |
| **Multi-proveedor de tickets** | Azure DevOps, Jira Cloud, HTTP Generico (cualquier REST API) |
| **HTTP Generico** | 4 tipos de auth, endpoints configurables, plantillas ServiceNow/GitHub/GitLab/Redmine |
| **Evidencias** | Trazabilidad SHA256, subida a SharePoint via Microsoft Graph API |
| **Importacion Excel** | 15 presets: Burp Suite, Nessus, SonarQube, Semgrep, CodeQL, Snyk, Checkmarx, Veracode y mas |
| **Reportes** | Executive Summary y Technical Report exportables a Excel |
| **Sandbox** | Scripts Python aislados (proceso separado), whitelist de modulos, timeout configurable |
| **Seguridad** | Fernet + PBKDF2, Recovery Key, sin `.env`, sin secrets en logs |
| **Despliegue** | venv (Windows/Linux), PyInstaller (.exe/binario), Docker |
| **i18n** | Interfaz en Espanol e Ingles con toggle en Configuracion |

### En camino (v1.0)

| Capacidad | Descripcion |
|-----------|------------|
| **Integracion SAST/DAST real** | Conexion con herramientas externas (Semgrep, SonarQube CLI, Trivy) |
| **REST API programatica** | Endpoints HTTP para automatizacion sin GUI |

---

## Documentacion adicional

| Archivo | Contenido |
|---------|-----------|
| [`docs/GUÍA_RÁPIDA.md`](docs/GUÍA_RÁPIDA.md) | Guía simple de uso para presentaciones |
| [`docs/PLANNING.md`](docs/PLANNING.md) | Cronograma, progreso histórico, planificación futura |
| [`docs/TECHNICAL.md`](docs/TECHNICAL.md) | Arquitectura detallada, flujos internos |
| [`docs/CHANGELOG.md`](docs/CHANGELOG.md) | Historial completo de versiones |
| [`PROMPT.md`](PROMPT.md) | Reglas del proyecto (obligatorio leer antes de contribuir) |
| [`QUICKSTART.md`](QUICKSTART.md) | Setup desde cero en VS Code, Windows, Linux, Docker |
| [`docs/manuales/manual-usuario.md`](docs/manuales/manual-usuario.md) | Flujo de trabajo, GUI, atajos de teclado |
| [`docs/manuales/manual-tecnico.md`](docs/manuales/manual-tecnico.md) | Instalacion, proveedores, seguridad, troubleshooting |
| [`docs/manuales/manual-desarrollo.md`](docs/manuales/manual-desarrollo.md) | Setup dev, arquitectura, tests, contribucion |

---

## Contribucion

Para contribuir al proyecto, consultar [`docs/TECHNICAL.md`](docs/TECHNICAL.md) y [`docs/manuales/manual-desarrollo.md`](docs/manuales/manual-desarrollo.md).

---

## Licencia

[MIT License](LICENSE)

---

<p align="center">
  <sub>Desarrollado por Angel Esquivel — DSOAgent 2026</sub>
</p>
