# TECHNICAL.md - Documentación Técnica DSOAgent

## Arquitectura General del Sistema

DSOAgent implementa una arquitectura modular con separación clara entre la interfaz gráfica (CustomTkinter), la lógica de negocio y la gestión de datos.

### Componentes Principales

```
DSOAgent/
├── agent/                   # Núcleo del multi-agente
│   ├── orchestrator.py      # Orchestrator
│   ├── contracts.py         # Contratos para inyección de dependencias
│   └── ...
├── gui/                     # Interfaz CustomTkinter
│   ├── framework.py         # BaseLayout
│   ├── themes.py            # ThemeManager
│   ├── main_window.py       # MainWindow
│   ├── thread_pool.py       # ThreadPool
│   ├── log_panel.py         # LogPanel
│   ├── dialogs.py           # Dialogs
│   ├── progress_bar.py      # ProgressBar
│   ├── icon_manager.py      # IconManager
│   ├── settings_view.py     # Settings con tabs
│   ├── dashboard_view.py    # DashboardView (ReportManager)
│   ├── project_view.py      # ProjectView (ProjectManager)
│   ├── findings_view.py     # FindingsView (ExcelHandler)
│   ├── evidence_view.py     # EvidenceView (EvidenceManager)
│   ├── tickets_view.py      # TicketsView (TicketService + ProviderRegistry)
│   ├── scripts_view.py      # ScriptsView (ProviderRegistry + Sandbox)
│   ├── reports_view.py      # ReportsView (ReportManager)
│   ├── logs_view.py         # LogsView (visor de logs, integrado en SettingsView)
│   ├── help_view.py         # HelpView (manuales Markdown + legacy PDF)
│   ├── import_wizard.py     # ImportWizard (wizard 6 pasos, v0.9.7)
│   ├── i18n.py              # LanguageManager (ES/EN, persiste en AppSettings)
│   ├── git_view.py          # GitView (GitManager)
│   ├── scheduler_view.py    # SchedulerView (TaskScheduler, solo Windows)
│   └── scrollable_pane.py   # ScrollablePane (scroll dinamico)
├── security/                # Seguridad
│   └── vault.py             # SecurityVault + Recovery Key
├── data/                    # Datos
│   ├── provider_registry.py # ProviderRegistry (proveedores dinámicos)
│   ├── config_manager.py    # ConfigManager (AppSettings JSON)
│   ├── config_atomic.py     # AtomicConfig (shadowing + integridad)
│   ├── excel_handler.py     # ExcelHandler
│   ├── excel_importer.py    # ExcelImporter (importación con presets, v0.9.7)
│   ├── excel_presets.py     # ImportPreset (15 formatos predefinidos, v0.9.7)
│   ├── evidence_manager.py  # EvidenceManager (con Pillow)
│   ├── project_manager.py   # ProjectManager (CASE-YYYY-NNN)
│   └── report_manager.py    # ReportManager
├── integrations/            # Integraciones (instanciados via ProviderRegistry)
│   ├── ticket_service.py    # AbstractTicketAPI
│   ├── azure_provider.py    # AzureDevOpsProvider
│   ├── jira_provider.py     # JiraProvider
│   ├── git_manager.py       # GitManager (clone/pull)
│   ├── http_provider.py     # GenericHTTPProvider (REST configurable)
│   ├── sharepoint_provider.py   # SharePointProvider (Graph API OAuth2, v0.9.8)
│   └── connection_templates.py  # Plantillas de conexion predefinidas
├── sandbox/                 # Sandboxing
│   ├── runner.py            # SandboxRunner
│   ├── multiprocess_runner.py  # MultiprocessSandboxRunner (aislado)
│   ├── validator.py         # ScriptValidator
│   └── contract.py          # ScriptContract
├── utils/                   # Utilidades
│   ├── paths.py             # Sistema de paths universal
│   ├── rate_limiter.py      # Rate limiting + throttling
│   ├── retry.py             # Retry logic con tenacity
│   ├── timezone.py          # Manejo de zonas horarias
│   ├── tray.py              # System tray
│   ├── user_id.py           # UUID + machine ID
│   ├── task_scheduler.py    # TaskScheduler (Windows schtasks.exe)
│   ├── logging_system.py    # Logging centralizado
│   └── health.py            # Health check stubs (REST API v1.0)
├── templates/               # Plantillas Git
│   ├── template_clone.py    # Git clone
│   └── template_gitpull.py  # Git pull
└── main.py                  # Entry point (GUI exclusivamente)
```

## Flujo Operativo Principal

### 1. Inicialización del Sistema

```
main.py → SecurityVault (cifrado) → ConfigManager → MainWindow
```

#### Flujo detallado de inicialización

```
main.py
  │
  ├─ SecurityVault(config_dir=None)
  │     │
  │     ├─ [NUEVO EQUIPO] vault.key no existe → _create_key() → guarda vault.key
  │     └─ [EQUIPO CONOCIDO] vault.key existe → _load_key() → carga Fernet
  │
  ├─ ConfigManager(config_path=None)
  │     │
  │     ├─ [PRIMER ARRANQUE] settings.json no existe → AppSettings() defaults → save()
  │     └─ [ARRANQUE NORMAL] settings.json existe → json.load() → AppSettings(**data)
  │
  └─ MainWindow(vault, config_manager, ...)
        │
        ├─ ProviderRegistry(registry_path=config/providers/)
        │     └─ [PRIMER ARRANQUE] sin conexiones → solo tipos builtin en memoria
        │
        └─ ProjectManager(base_path=config/projects/, use_vault=True)
              └─ [PRIMER ARRANQUE] projects.json no existe → lista vacía
```

#### Escenarios de fallo y recuperación

| Escenario | Comportamiento | Recuperación |
|---|---|---|
| `vault.key` no existe (primer arranque) | `_create_key()` genera nueva clave derivada del hardware | Automático |
| `vault.key` no existe (reinstalación / cambio de PC) | `_create_key()` genera nueva clave — no puede descifrar datos anteriores | Usar `recover_with_key(recovery_key)` antes de operar |
| `vault.key` corrupto | `_load_key()` lanza excepción → vault no inicializado | Eliminar `vault.key`, reiniciar (se crea nueva clave) |
| `settings.json` no existe | `ConfigManager.load()` retorna `AppSettings()` defaults, crea el archivo | Automático |
| `settings.json` corrupto | `except` en `json.load()` → retorna `AppSettings()` defaults | Automático (sin datos previos) |
| `projects.json` no existe | `ProjectManager.list_projects()` retorna `[]` | Automático |
| `connections.json` no existe | `ProviderRegistry` carga solo tipos builtin en memoria | Automático |

#### Rutas de configuración por entorno

| Entorno | `vault.key` + `recovery.key` | `settings.json` | `connections.json` |
|---|---|---|---|
| Desarrollo (venv) | `config_dir` pasado en `__init__` | `config/dsoagent_config.json` | `config/providers/connections.json` |
| Producción Windows | `%APPDATA%\DSOAgent\` | `config/dsoagent_config.json` | `config/providers/connections.json` |
| Producción Linux | `~/.dsoagent/` | `config/dsoagent_config.json` | `config/providers/connections.json` |
| Ejecutable .exe | `%APPDATA%\DSOAgent\` (separado del .exe) | relativo al .exe | relativo al .exe |
| Docker | `/app/data/` (volumen montado) | `/app/data/config.json` | `/app/data/providers.json` |

### 2. Flujo de Datos

```
┌──────────────────┐
│  Excel (hallazgos) │
│  SAST/DAST tools   │
└─────────┬────────┘
                 ↓ ExcelImporter / ExcelHandler
           Finding[] (normalizados)
                 ↓
     ┌─────────┼─────────┐
     ↓                 ↓
TicketService       EvidenceManager
     ↓                 ↓
┌──────┐ ┌─────┐ ┌─────┐ ┌─────────────┐
│Azure │ │Jira │ │HTTP │ │SharePoint    │
│DevOps│ │Cloud│ │Gen  │ │(Graph API)   │
└──────┘ └─────┘ └─────┘ └─────────────┘
     ↓                 ↓
ReportManager   logs/ (auditoria)
```

### 3. Arquitectura de Capas

```
gui/             ← Interfaz CustomTkinter (NO conoce agent/)
     ↓ llama a
agent/           ← Núcleo del multi-agente (NO conoce gui/)
     ↓ usa
tools/           ← Herramientas especializadas (NO conocen agent/)
     ↓ usa
security/        ← Validación, cifrado, rate limiting
utils/           ← Utilidades transversales (timezone, paths, retry)
integrations/    ← Adaptadores externos (Azure, Jira, Git)
```

## Módulos Implementados

### SecurityVault

- **Cifrado**: Fernet (AES 128 + HMAC SHA256)
- **Derivación de claves**: PBKDF2 (100,000 iteraciones)
- **Ubicación**: Archivos JSON encriptados en ~/.dsoagent/

#### Sistema de Recuperación (CRÍTICO)

La bóveda genera una **clave de recuperación** que DEBE ser guardada
en un lugar seguro externo (NO en la misma PC).

**¿Cuándo se necesita?**
- Cambio de computadora
- Cambio de usuario Windows
- Formateo de PC
- Cambio de hardware (placa base)

**Archivos en ~/.dsoagent/**
| Archivo | Descripción | Backup externo |
|---------|-------------|----------------|
| `vault.key` | Clave derivada del hardware | NO |
| `recovery.key` | Clave de recuperación | **SÍ - OBLIGATORIO** |

**Flujo correcto del sistema de recuperacion (v2.0):**

```python
# 1. GENERAR - Al usar la app por primera vez (guarda vault.key cifrado en recovery.key)
from security.vault import SecurityVault
vault = SecurityVault()
recovery_key = vault.generate_recovery_key()
print(f"GUARDAR ESTA CLAVE EXTERNAMENTE: {recovery_key}")
# recovery.key en disco contiene vault.key cifrado con recovery_key

# 2. RECUPERAR - En nueva PC: copiar recovery.key, luego:
vault = SecurityVault()            # vault.key no existe aun
vault.recover_with_key(recovery_key)  # descifra vault.key y lo restaura en disco
```

**Que almacena recovery.key:**
```json
{
  "encrypted_vault_key": "<vault.key cifrado con recovery_key (Fernet)>",
  "version": "2.0"
}
```

**ADVERTENCIA**: Sin la clave de recuperacion, si se cambia de PC
o hardware, LOS DATOS ENCRIPTADOS SERAN IRRECUPERABLES.
Formato v1.0 (anterior) es obsoleto y lanzara `ValueError` indicando regenerar.

### Sistema de Paths (utils/paths.py)

Sistema universal que funciona en desarrollo y como .exe.

```python
from utils.paths import (
    get_project_root,    # Raíz del proyecto
    get_data_dir,        # Directorio de datos
    get_assets_dir,      # Directorio de assets
    get_docs_dir,        # Directorio de docs
    get_config_dir,      # Configuración (%APPDATA%/DSOAgent)
    get_logs_dir,        # Logs (%APPDATA%/DSOAgent/logs)
    is_frozen,           # Detecta si es .exe
)
```

### Rate Limiter (utils/rate_limiter.py)

Control de tasa para APIs externas.

```python
from utils.rate_limiter import (
    RateLimiter,         # Limitador síncrono
    AsyncRateLimiter,   # Limitador async
    Throttler,          # Backoff exponencial
    AZURE_RATE_LIMIT,   # Config Azure (30 req/min)
    JIRA_RATE_LIMIT,   # Config Jira (10 req/min)
)
```

### Retry Logic (utils/retry.py)

Reintentos automáticos con backoff exponencial.

```python
from utils.retry import (
    async_retry_with_backoff,
    sync_retry_with_backoff,
    AsyncRetryHelper,
)
```

### AtomicConfig

- **Perfiles**: Aislamiento por proyecto
- **Shadowing**: Herencia con overrides
- **Validación**: Tipos y rangos
- **Backup**: Versiones automáticas

### ExcelHandler

- **Lectura**: pandas read_excel
- **Escritura**: openpyxl con formato
- **Finding dataclass**: Normalización de datos
- **Operaciones**: CRUD completo

### ExcelImporter (data/excel_importer.py) - v0.9.7+

**Importador flexible de reportes Excel con mapeo dinámico:**

- **Formatos soportados**:
  - DSOAgent Standard (11 columnas inglés)
  - Legacy español (SAST, DAST-Web, DAST-App)
  - Burp Suite, Nessus, OWASP ZAP, Semgrep, SonarQube

- **Detección automática**: Por headers o nombres de hojas (`detect_preset_by_headers()`)
- **Mapeo dinámico**: Columnas arbitrarias → campos Finding
- **Normalización**: Severidades y estados de múltiples formatos → valores estándar
- **Multi-hoja**: Procesa múltiples hojas en un solo archivo
- **Preview**: Análisis de estructura antes de importar

```python
from data.excel_importer import ExcelImporter
from data.project_manager import ProjectManager

importer = ExcelImporter(ProjectManager())

# Preview
preview = importer.preview_file("hallazgos.xlsx")

# Importar con preset
result = importer.import_file(
    file_path="hallazgos.xlsx",
    preset_name="dsoagent_standard",  # o "legacy_sast", "burp_suite"
    project_id="CASE-2026-001",
    skip_existing=True
)
print(f"Importados: {result.imported}, Errores: {result.errors}")
```

### ExcelPresets (data/excel_presets.py)

**Mapeos predefinidos para formatos conocidos:**

| Preset | Hoja(s) | Columnas clave |
|--------|---------|-----------------|
| `dsoagent_standard` | - | ID, Project, Title, Severity, Status |
| `legacy_sast` | Análisis de Código Estático | Código, TipoVulnerabilidad, NivelRiesgo |
| `legacy_dast_web` | Análisis de Código Dinámico Web | Código, TipoVulnerabilidad, NivelRiesgo |
| `legacy_dast_app` | Análisis de Código Dinámico App | Código, TipoVulnerabilidad, NivelRiesgo |
| `burp_suite` | - | Issue, Severity, Host, Path |
| `nessus` | - | Plugin Name, Risk, Host |

- **Detección por headers**: `detect_preset_by_headers(headers)`
- **Listado por categoría**: `list_presets_by_category()`
- **Severidad/Estado maps**: Normalización por idioma (español → inglés)

### ImportWizard (gui/import_wizard.py)

**Wizard GUI de 6 pasos para importación:**

1. **Selección**: Archivo `.xlsx` (browse o drag-drop)
2. **Preview**: Vista previa de hojas y columnas detectadas
3. **Formato**: Confirmar preset detectado o seleccionar manual
4. **Mapeo**: Configurar columnas (si no hay preset)
5. **Validación**: Opciones de importación (proyecto, duplicados)
6. **Importar**: Progreso y resumen de resultados

**Acceso**: Botón "Importar Excel" en `ProjectView` (junto a "Ver Hallazgos")

### EvidenceManager

- **Estructura**: project/finding_id/type/file.ext (global) o project.evidence_path (per-project)
- **Per-project**: `EvidenceManager.for_project(project)` crea instancia vinculada a `project.evidence_path`
- **Constructor**: `__init__(base_path, project_path=None)` — `project_path` sobreescribe `base_path`
- **Tipos**: screenshot, log, code, network, config, document
- **Hash**: SHA256 para verificación
- **Manifesto**: Exportación de metadatos

### EvidenceView (gui/evidence_view.py)

- **Project-centric**: Explora `project.evidence_path`, no carpeta global
- **Treeview**: nombre, carpeta, tipo, tamaño, fecha modificación
- **Filtros**: por subcarpeta (categoría dinámica) + búsqueda por nombre
- **Acciones**: subir archivos, abrir carpeta, crear subcarpetas, eliminar, preview, **subir a SharePoint** (v0.9.8)
- **SharePoint**: botón "☁ Subir a SharePoint" — requiere `provider_registry` con conexión `builtin_sharepoint` activa; upload en thread daemon con `asyncio.run()`
- **Stats**: conteo de archivos y tamaño total en barra de estado
- **API**: `set_project(project)`, `refresh()`, `__init__(master, provider_registry=None)`

### ProjectManager

- **Modelo**: `Project` con ID tipo `CASE-YYYY-NNN`, `DirectoriesConfig`, timestamps ISO
- **Persistencia**: JSON index cifrado con SecurityVault (`use_vault=True/False`)
- **Operaciones**: Create, Read, Update, Delete
- **Multi-proyecto**: Aislamiento total, `threading.Lock` para IDs concurrentes
- **Backward compat**: Slug IDs legacy reconocidos

### ProviderRegistry (data/provider_registry.py)

Sistema dinamico de proveedores y conexiones. Reemplaza la configuracion hardcoded de Azure/Jira.

- **Modelos**:
  - `ProviderFieldDef`: campo con name, label, field_type (text/password/select/checkbox/file/number/url), required, default, options, placeholder, order
  - `ProviderTypeDef`: tipo con id, name, category, fields[], builtin, supports_test, supports_mappings
  - `ProviderConnection`: instancia con id, type_id, name, project_ids[], values{}, mappings{}, active
- **Tipos builtin** (6): Azure DevOps, Jira Cloud, Git Repos, Script Custom, SharePoint (activo v0.9.8), SAST/DAST
- **CRUD**: Tipos custom, campos por tipo, conexiones con valores
- **Asociacion**: Conexion-proyecto N:N via `project_ids[]`
- **Persistencia**: `config/providers/types.json` + `connections.json`
- **Import/Export**: `export_connections(conn_ids?)` → dict, `import_connections(data, overwrite?)`
- **Legacy**: `ProviderEntry` mantenido para backward compat

```python
from data.provider_registry import ProviderRegistry, ProviderConnection

reg = ProviderRegistry()
# Crear conexion
conn = reg.add_connection(ProviderConnection(
    type_id="builtin_azure", name="Azure Prod",
    values={"organization": "acme", "token": "pat-xxx"},
    project_ids=["CASE-2026-001"],
))
# Buscar por proyecto y categoria
conns = reg.list_connections(project_id="CASE-2026-001", category="tickets")
# Export / Import
data = reg.export_connections()
reg2.import_connections(data, overwrite=True)
```

### SharePointProvider (integrations/sharepoint_provider.py)

Proveedor de almacenamiento para Microsoft SharePoint via Microsoft Graph API.

- **Autenticación**: OAuth2 client credentials (Azure AD App Registration)
- **Configuración**: `site_url`, `tenant_id`, `client_id`, `client_secret`, `library_name`, `folder_path`
- **API pública**:
  - `connect() -> bool` — obtiene token OAuth2, resuelve `site_id` y `drive_id`
  - `test_connection() -> bool` — GET `/sites/{site_id}`
  - `upload_file(local_path, remote_name?, folder?) -> SharePointResult`
  - `download_file(remote_name, dest_path, folder?) -> SharePointResult`
  - `list_files(folder?) -> list[SharePointFile]`
  - `create_folder(folder_path) -> SharePointResult` — idempotente (409 = success)
  - `disconnect()` / `is_connected()`
- **Token**: Cache interno con renovación automática (60s de margen antes de expiración)
- **Registrado**: `ProviderRegistry._get_factories()["builtin_sharepoint"]` (v0.9.8)

### ReportManager (data/report_manager.py)

- **Métricas consolidadas**: Multi-proyecto
- **Reportes**: Executive Summary + Full Report
- **Dashboard**: Datos para visualización

### TicketService

- **AbstractTicketAPI**: Interfaz base
- **AzureDevOpsProvider**: Work items, proyectos (con rate limiting)
  - `area_path`: Ruta de area del Work Item (opcional, `System.AreaPath`)
  - `iteration_path`: Ruta de iteracion/sprint (opcional, `System.IterationPath`)
- **JiraProvider**: Issues, proyectos, epics (con rate limiting)

### GitManager (integrations/git_manager.py)

- **Operaciones**: `clone_repo()`, `clone_all()`, `pull_repo()`, `pull_all()`
- **Protocolo**: `asyncio.create_subprocess_exec` - sin shell=True (PROMPT.md R1)
- **Repos list**: `load_repos()` / `save_repos()` - formato CSV `nombre, rama`
- **Progreso**: callbacks `on_line(str)` y `on_progress(done, total)` por operacion
- **Timeout**: configurable (default 120s)
- **GitResult**: dataclass con success, repo, branch, output[], error, duration_seconds

### ScrollablePane (gui/scrollable_pane.py)

- **Proposito**: Contenedor con scrollbar vertical dinamica (aparece solo si el contenido no cabe).
- **Mecanismo**: `tk.Canvas` + `CTkScrollbar`. El metodo `_on_scroll_update` oculta el scrollbar
  cuando `first<=0` y `last>=1` (todo el contenido es visible).
- **Mousewheel**: Bind/unbind de `<MouseWheel>`, `<Button-4>`, `<Button-5>` al entrar/salir
  del area (compatible Windows/macOS/Linux).
- **Interior**: Propiedad `.interior` (CTkFrame transparente dentro del canvas) como padre para widgets.
- **API publica**: `scroll_to_top()`, `update_bg()`, `.interior`.
- **Uso en MainWindow**: Sidebar nav (botones) + content area (vistas). Referenciados como
  `self._nav_pane` y `self._content_pane`.

### GitView (gui/git_view.py)

- **Tab Clonar**: Treeview repos, consola dark (#0d0d0d/#00ff41), barra progreso, selector destino
- **Tab Git Pull**: Identico al clone, ejecuta pull sobre repos clonados locales
- **Repos list**: Agregar/Quitar repos manualmente, Cargar/Guardar `.txt`
- **set_project()**: Precarga `directories.repositories` y URL base desde ProviderRegistry (builtin_git o fallback Azure)
- **Threading**: `threading.Thread` + `asyncio.run()` para no bloquear GUI
- **ProviderRegistry**: Recibe `provider_registry` en constructor, inyectado desde main_window

### TicketsView (gui/tickets_view.py)

- **Precarga de credenciales**: `set_project()` busca conexion activa en ProviderRegistry por
  `project_id + type_id`. Usa `ProviderRegistry.resolve_provider()` para instanciar el concreto.
  El usuario solo hace clic en "Conectar" sin reingresar credenciales.
- **Boton Sincronizar**: Lee hallazgos Open/New del Excel del proyecto via `ExcelHandler`,
  crea tickets en el proveedor activo mapeando severidad→prioridad (Critical→Highest ... Info→Lowest).
  Ejecuta en hilo separado, muestra resumen al finalizar.

### SchedulerView / TaskScheduler

- **Backend**: `utils/task_scheduler.py` interactua con `schtasks.exe` (solo Windows)
- **Frecuencias soportadas**: once, daily, weekly, monthly
- **Operaciones**: create, delete, list, run
- **Result**: `TaskSchedulerResult` encapsula exito/error y salida del proceso
- **Async**: `asyncio.create_subprocess_exec` segun PROMPT.md regla 1
- **GUI**: `gui/scheduler_view.py` con `threading.Thread` + `asyncio.run()` para compatibilidad Tkinter
- **set_project()**: Muestra el proyecto activo en la cabecera al navegar a la vista

### SandboxRunner

- **Validacion**: AST + regex patterns
- **Bloqueos**: os, subprocess, sys, socket, eval, exec
- **Contrato**: ExecutionContext + ExecutionResult
- **Timeout**: Configurable

## Sistema de Logging

```python
# Log centralizado
from utils.logging_system import setup_logging, get_logger

# Setup global
setup_logging()

# Get logger
logger = get_logger("module_name")
logger.info("Evento...")
```

### Niveles
- DEBUG: Detalle técnico
- INFO: Eventos normales
- WARNING: Situaciones inesperadas
- ERROR: Errores operativos
- CRITICAL: Errores sistémicos

### Ubicación de Logs
- Windows: `%APPDATA%/DSOAgent/logs/`
- Linux: `~/.dsoagent/logs/`

## Reglas de Seguridad (PROMPT.md)

| Regla | Estado | Descripción |
|-------|--------|-------------|
| shell=True prohibido | ✅ | asyncio.create_subprocess_exec en todo el proyecto |
| datetime.utcnow/now() naive prohibido | ✅ | now_utc() de utils/timezone.py |
| agent/ sin integrations/ | ✅ | Inyeccion de dependencias (contracts.py) |
| Rate Limiting | ✅ | Implementado en providers |
| Retry Logic | ✅ | tenacity con backoff |
| logging.getLogger(__name__) prohibido | ✅ | get_logger() centralizado (20 modulos) |

## Modos de Despliegue

DSOAgent soporta tres modos de despliegue independientes:

### Modo 1 - venv (Escritorio)

Instalacion completa con GUI. Soporta Windows 10/11 y Ubuntu 24.04.

**Windows:**
```bat
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
python main.py
```

**Ubuntu 24.04:**
```bash
chmod +x setup_linux.sh
./setup_linux.sh
source venv/bin/activate
python main.py
```

Dependencias del sistema Linux requeridas (instaladas automaticamente por setup_linux.sh):
```bash
sudo apt-get install python3-tk libxcb1 libxkbcommon0 libx11-6 libxext6
sudo apt-get install libjpeg-dev zlib1g-dev
sudo apt-get install python3-gi gir1.2-ayatana-appindicator3-0.1
```

### Modo 2 - PyInstaller (Ejecutable)

Genera un binario autocontenido sin necesidad de Python instalado.

**Windows (DSOAgent.exe):**
```bat
pyinstaller DSOAgent.spec --clean
dist\DSOAgent.exe
```

**Linux (DSOAgent binario):**
```bash
# Instalar dependencias del sistema primero (ver setup_linux.sh)
pyinstaller DSOAgent_linux.spec --clean
./dist/DSOAgent
```

Notas importantes del .spec:
- `'tkinter'` NO debe estar en `excludes` (CustomTkinter lo requiere)
- Usar `DSOAgent.spec` en Windows y `DSOAgent_linux.spec` en Linux
- El spec detecta automaticamente si existe `assets/icon.ico` o `assets/icon.png`

### Modo 3 - Docker

Ejecuta el `Orchestrator` directamente en contenedor Linux. Sin GUI interactiva.
`Dockerfile` corre `agent.orchestrator.Orchestrator` como proceso principal.
`Dockerfile.gui` agrega Xvfb para X11 forwarding (uso interno/desarrollo).

**Docker Compose (recomendado):**
```bash
# Iniciar (TIMEZONE configurable via variable de entorno, default: America/Mexico_City)
docker compose up -d

# Ver logs
docker compose logs -f agent-core

# Detener
docker compose down
```

**Docker manual:**
```bash
# Build imagen principal
docker build -t dsoagent:latest .

# Ejecutar
docker run -d --name dsoagent-core \
  -v dsoagent-data:/root/.dsoagent \
  dsoagent:latest

# Build imagen GUI (X11 forwarding)
docker build -f Dockerfile.gui -t dsoagent-gui:latest .
```

**Arquitectura de contenedores (docker-compose.yml):**
```
agent-core   <- Orchestrator (Dockerfile, activo)
api          <- REST API (pendiente v1.0, comentado en compose)
```

**Nota sobre vulnerabilidades de imagen Docker:**  
El lint de Docker reporta 3 advertencias por imagen no pinned con digest SHA256.  
Para produccion, fijar con: `FROM python:3.12-slim@sha256:<digest>`

## Empaquetado

```bash
# Instalar solo agente core
pip install -e .

# Instalar con GUI
pip install -e ".[gui]"

# Build package
make build

# PyInstaller Windows
make package

# PyInstaller Linux
make package-linux

# Docker principal
make docker-build

# Docker GUI (X11)
make docker-gui
```

## Configuracion

No se requiere archivo `.env` ni variables de entorno. Toda la configuracion
se realiza desde la GUI (Configuracion > Proveedores). Las credenciales se
almacenan cifradas via `SecurityVault` (Fernet + PBKDF2, auto-generado).

Unica variable de entorno opcional (Docker): `TIMEZONE` (default: America/Mexico_City).

---

**Version**: 0.9.8  
**Ultima actualizacion**: 2026-07-15  
**License**: MIT
