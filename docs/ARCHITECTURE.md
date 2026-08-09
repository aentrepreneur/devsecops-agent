# ARCHITECTURE.md — Mapa de Navegación, Pseudocódigo y Guía de Modificaciones
# =================================================================

# ARCHITECTURE.md — DSOAgent v0.9.8

**Versión**: 0.9.8 (Pre-RC1)  
**Última actualización**: Junio 2026  
**Audiencias**: Agentes IA · Developer nuevo · Referencia rápida  
**Regla de mantenimiento**: Ver `PROMPT.md`

> Este documento responde: "¿dónde toco X?" y "¿cómo funciona Y en pseudocódigo?".
> Para reglas de desarrollo ver `PROMPT.md`. Para historial ver `CHANGELOG.md`.
> Para árbol de archivos ver `TECHNICAL.md`.

---

## Índice

1. [Resumen para IA/Agente](#1-resumen-para-iaagente)
2. [Flujo de Arranque](#2-flujo-de-arranque)
3. [Capa GUI — Navegación y Vistas](#3-capa-gui--navegación-y-vistas)
4. [Capa Data — Managers y Persistencia](#4-capa-data--managers-y-persistencia)
5. [Capa Security — Vault](#5-capa-security--vault)
6. [Capa Agent — Orchestrator](#6-capa-agent--orchestrator)
7. [Capa Integrations — Proveedores](#7-capa-integrations--proveedores)
8. [Capa Utils — Transversales](#8-capa-utils--transversales)
9. [Sandbox](#9-sandbox)
10. [Entregas — Cómo se Empaqueta](#10-entregas--cómo-se-empaqueta)
11. [Tests — Estructura y Dónde Tocar](#11-tests--estructura-y-dónde-tocar)
12. [Guía de Modificaciones Rápidas](#12-guía-de-modificaciones-rápidas)

---

## 1. Resumen para IA/Agente

### Capas del sistema (jerarquía de imports — no violar)

```
gui/           ← CustomTkinter UI. Conoce: agent/, data/, security/, utils/
               NO conoce: nada que importe gui/ de vuelta
agent/         ← Orchestrator. Conoce: data/, security/, utils/, integrations/
               NO conoce: gui/
data/          ← Managers y persistencia JSON. Conoce: security/, utils/
integrations/  ← Adaptadores externos (Azure, Jira, Git, SharePoint, HTTP)
               Conoce: utils/
sandbox/       ← Ejecución scripts aislada. Conoce: utils/
security/      ← SecurityVault (Fernet+PBKDF2). Conoce: utils/
utils/         ← Transversales. NO conoce ninguna otra capa del proyecto
```

### Dónde vive cada responsabilidad

| Responsabilidad | Archivo | Clase principal |
|-----------------|---------|-----------------|
| Arranque app | `main.py` | `main()` |
| Ventana principal + nav | `gui/main_window.py` | `MainWindow(BaseLayout)` |
| Base de ventanas CTk | `gui/framework.py` | `BaseLayout(ABC)` |
| Temas, colores, fuentes | `gui/themes.py` | `ThemeManager` + funciones sueltas |
| Strings internacionalizados | `gui/i18n.py` | `LanguageManager` + `_t()` |
| Login + selección de perfil | `gui/login_dialog.py` | `LoginDialog(CTkToplevel)` |
| Perfiles multi-usuario | `data/profile_manager.py` | `ProfileManager` |
| Proyectos CASE-YYYY-NNN | `data/project_manager.py` | `ProjectManager` |
| Cifrado + vault | `security/vault.py` | `SecurityVault` |
| Excel hallazgos | `data/excel_handler.py` | `ExcelHandler` + `Finding` |
| Import Excel 15 formatos | `data/excel_importer.py` | `ExcelImporter` |
| Presets de importación | `data/excel_presets.py` | `ImportPreset` (15 formatos) |
| Proveedores dinámicos | `data/provider_registry.py` | `ProviderRegistry` |
| Orchestrator flujo core | `agent/orchestrator.py` | `Orchestrator` |
| Evidencias + SHA-256 | `data/evidence_manager.py` | `EvidenceManager` |
| Reportes multi-proyecto | `data/report_manager.py` | `ReportManager` |
| Export/Import perfiles | `data/export_import_manager.py` | `ExportImportManager` |
| Configuración app | `data/config_manager.py` | `ConfigManager` + `AppSettings` |
| Config atómica + integridad | `data/config_atomic.py` | `AtomicConfig` |
| Azure DevOps | `integrations/azure_provider.py` | `AzureDevOpsProvider` |
| Jira Cloud | `integrations/jira_provider.py` | `JiraProvider` |
| SharePoint Graph API | `integrations/sharepoint_provider.py` | `SharePointProvider` |
| HTTP genérico REST | `integrations/http_provider.py` | `GenericHTTPProvider` |
| Git clone/pull | `integrations/git_manager.py` | `GitManager` |
| Contrato abstracto tickets | `integrations/ticket_service.py` | `AbstractTicketAPI(ABC)` |
| Scripts en sandbox | `sandbox/runner.py` | `SandboxRunner` |
| Validación scripts | `sandbox/validator.py` | `ScriptValidator` |
| Contrato ejecución | `sandbox/contract.py` | `ExecutionResult` |
| Rutas universales | `utils/paths.py` | funciones `get_*_dir()` |
| Logging centralizado | `utils/logging_system.py` | `get_logger()` + `setup_logging()` |
| System tray | `utils/tray.py` | `SystemTray` |
| Scheduler Windows | `utils/task_scheduler.py` | `TaskScheduler` |
| Rate limiting | `utils/rate_limiter.py` | `RateLimiter` |
| Retry lógica | `utils/retry.py` | decoradores tenacity |
| Timezone | `utils/timezone.py` | `now_utc()`, `now_local()` |
| UUID proyecto + hardware ID | `utils/user_id.py` | `generate_project_id()` |

---

## 2. Flujo de Arranque

### Pseudocódigo completo de inicialización

```
main.py :: main()
  setup_logging()                    # utils/logging_system.py — configura RotatingFileHandler
  import MainWindow from gui/main_window.py
  app = MainWindow()
    → super().__init__(title="DSOAgent", size=(1200, 730))    # BaseLayout.__init__
      self.window = None
      self._running = False
      self._tray = None
    → self.thread_pool = ThreadPool(max_workers=4)            # gui/thread_pool.py
    → self.current_view = "-NAV-DASHBOARD-"
    → self.active_project_id = None
    → self._setup_theme()
        ThemeManager.apply()          # ctk.set_appearance_mode("dark")
    → self._setup_callbacks()
        thread_pool.register_callback("on_complete", ...)
        thread_pool.register_callback("on_error", ...)
    → self._init_managers()          # gui/main_window.py:611
        ProviderRegistry()            # data/provider_registry.py
        ExcelHandler()                # data/excel_handler.py
        EvidenceManager(base_path)    # data/evidence_manager.py
        ReportManager()               # data/report_manager.py
        ProjectManager(projects_dir)  # data/project_manager.py
        ProfileManager()              # data/profile_manager.py
        ExportImportManager(profile_manager)
    → self._init_views()             # self.views = {}; self.view_frames = {}
  app.run()                          # gui/framework.py:BaseLayout.run()
    → self.create_window()
        ctk.CTk() → window
        window.protocol("WM_DELETE_WINDOW", self._on_close_window)
        → self.build_layout_content(window)    # gui/main_window.py:284
            sidebar  = self._build_sidebar(window)     # columna 0, ancho 222px
            main_area = self._build_main_area(window)  # columna 1, weight=1
            status_bar = self._build_status_bar(window) # fila 1, colspan=2
            self._update_theme_button()
            self._refresh_project_combo()
            self._refresh_current_view()       # carga DashboardView inicial
    → window.mainloop()              # event loop Tkinter

  PRIMER ARRANQUE (o perfiles detectados):
    LoginDialog(parent, profile_manager, on_login_success, on_create_profile)
      → CTkToplevel(parent)          # modal sobre MainWindow
      → geometry("500x600"), centrado en pantalla
      → self._detect_profiles()      # ProfileManager.list_profiles()
      SI no hay perfiles:
        _build_ui_create_profile()   # formulario nombre + password
      ELSE:
        _build_ui_select_profile()   # lista de perfiles existentes
      on_login_success(profile_id, master_password)
        → MainWindow._on_login_success()
          self.current_profile_id = profile_id
          self.current_profile_password = master_password
```

### Orden de inicialización de managers en MainWindow._init_managers()

```
1. ProviderRegistry()              ← sin dependencias (self-contained)
2. ExcelHandler()                  ← sin dependencias
3. EvidenceManager(evidence_dir)   ← necesita path
4. ReportManager()                 ← sin dependencias
5. ProjectManager(projects_dir)    ← necesita path
6. ProfileManager()                ← paths desde utils/paths.py
7. ExportImportManager(profile_manager)  ← necesita ProfileManager
```

---

## 3. Capa GUI — Navegación y Vistas

### Herencia de clases GUI

```
ABC
└── BaseLayout (gui/framework.py:21)
    ├── window: CTk
    ├── _running: bool
    ├── _tray: SystemTray
    ├── create_window() → CTk
    ├── build_layout_content(window)  ← abstracto (implementado por subclase)
    ├── run() → mainloop()
    ├── minimize_to_tray()
    ├── restore_from_tray()
    └── quit_app()
    └── MainWindow (gui/main_window.py:26)
        ├── NAV_ITEMS: list[tuple]    ← 13 items (constante de clase)
        ├── _NAV_I18N_KEYS: dict      ← clave i18n por nav_id
        ├── _NAV_ICONS: dict          ← nombre icono por nav_id
        ├── thread_pool: ThreadPool
        ├── current_view: str         ← nav_id activo
        ├── active_project_id: str
        ├── view_frames: dict         ← cache de vistas instanciadas
        ├── nav_buttons: dict         ← botones sidebar por nav_id
        ├── _build_sidebar()
        ├── _build_main_area()
        ├── _build_status_bar()
        ├── _handle_navigation(event) ← despacha nav o acción
        ├── _refresh_current_view()   ← carga vista según current_view
        └── _show_settings_view()     ← manejo especial Settings

CTkToplevel
└── LoginDialog (gui/login_dialog.py:25)
    ├── profile_manager: ProfileManager
    ├── on_login_success: Callable
    ├── on_create_profile: Callable
    ├── available_profiles: list
    ├── _detect_profiles()
    ├── _build_ui()
    └── _do_login() / _do_create_profile()
```

### Navegación: de click a vista

```
Usuario hace click en botón sidebar
  → btn.command = lambda: self._handle_navigation(nav_id)
  → MainWindow._handle_navigation(event)          # main_window.py:786
      SI event == "-NAV-EXPORT-":
          self.show_export_dialog()               # abre ExportDialog (acción)
      SI event == "-NAV-IMPORT-":
          self.show_import_dialog()               # abre ImportWizard (acción)
      SI event == "-NAV-MULTI-REPORT-":
          self.show_multi_profile_report_dialog() # abre MultiProfileReportDialog (acción)
      ELSE:
          self.current_view = event
          self._refresh_current_view()
  → MainWindow._refresh_current_view()            # main_window.py:749
      SI current_view == "-NAV-SETTINGS-":
          self._show_settings_view()              # manejo especial con cache
          return
      _nav_map = {
          "-NAV-DASHBOARD-": ("dashboard_view", "DashboardView"),
          "-NAV-PROJECTS-":  ("project_view",   "ProjectView"),
          "-NAV-TICKETS-":   ("tickets_view",   "TicketsView"),
          "-NAV-EVIDENCE-":  ("evidence_view",  "EvidenceView"),
          "-NAV-REPORTS-":   ("reports_view",   "ReportsView"),
          "-NAV-GIT-":       ("git_view",       "GitView"),
          "-NAV-SCHEDULER-": ("scheduler_view", "SchedulerView"),
          "-NAV-SCRIPTS-":   ("scripts_view",   "ScriptsView"),
          "-NAV-HELP-":      ("help_view",      "HelpView"),
      }
      self._show_view_by_class(nav_id, view_class_name)
  → MainWindow._show_view_by_class(nav_id, cls_name)  # main_window.py:655
      SI nav_id in view_frames Y widget.winfo_exists():
          view.pack(fill="both", expand=True)     # reusar vista cacheada
      ELSE:
          from gui.<module> import <ViewClass>
          view = ViewClass(main_content_frame, managers...)
          view.pack(fill="both", expand=True)
          view_frames[nav_id] = view              # cachear para reuso
```

### Sidebar — estructura y dónde tocar

```
_build_sidebar(window)                            # main_window.py:308
  CTkFrame(width=222, corner_radius=0)            # ancho fijo sidebar
    CTkLabel("DSOAgent", font=bold(20))           # título app
    CTkFrame(height=2, fg_color="gray70/gray30")  # separador
    ScrollablePane(sidebar)                       # área scrollable botones
      for nav_id, text in get_nav_items():
        CTkButton(
          text=f"  {text}",
          image=get_icon(_NAV_ICONS[nav_id], size=18),
          height=40,
          corner_radius=0,
          fg_color="transparent",               # sin fondo por defecto
          text_color=("gray10", "gray90"),      # light/dark
          hover_color=("gray70", "gray30"),     # light/dark
          anchor="w"
        )
    CTkButton("  Salir", ...)                     # botón exit fijo al fondo
```

**Para cambiar apariencia de botones sidebar** → `gui/main_window.py` líneas 337-350  
**Para cambiar ancho del sidebar** → `main_window.py:291` `minsize=222` + `main_window.py:318` `width=222`  
**Para agregar/quitar nav item** → ver sección 12 Guía de Modificaciones Rápidas

### Resaltado de navegación activa

```
_update_nav_highlight()                           # main_window.py:554
  active_color   = ("gray80", "gray25")           # light/dark
  inactive_color = "transparent"
  for nav_id, btn in nav_buttons.items():
      btn.configure(fg_color=active si nav_id == current_view)
```

**Para cambiar color de ítem activo** → `main_window.py:558` `active_color`

### Área principal — estructura

```
_build_main_area(window)                          # main_window.py:373
  CTkFrame(corner_radius=0)
    row 0: top_bar (fg_color="transparent")
      col 0: self.view_title  (CTkLabel, font=bold(18))   ← título de vista actual
      col 2: proj_bar
        CTkLabel("Proyecto:")
        self.cmb_active_project  (CTkComboBox, width=220) ← selector proyecto
        CTkButton("Proyectos")                            ← acceso rápido
    row 1: self.frm_no_project  (banner amarillo, oculto por defecto)
      fg_color=("#fff3cd", "#5a4500")  light/dark
      text_color=("#856404", "#ffd966") light/dark
    row 2: self._content_pane (ScrollablePane)
      → self.main_content_frame  ← aquí se cargan las vistas
```

**Para cambiar el banner de "sin proyecto"** → `main_window.py:424-432`  
**Para cambiar título de vista** → `_refresh_current_view()` → `_title_map` `main_window.py:751`

### Status bar — estructura

```
_build_status_bar(window)                         # main_window.py:442
  CTkFrame(height=30, corner_radius=0)
    LEFT:  self.theme_switch (CTkSwitch, text="Oscuro", width=46)
    LEFT:  separador visual (CTkFrame height=18, fg_color="gray70/gray40")
    LEFT:  CTkLabel(f"  Usuario: {username}")
    LEFT:  separador visual
    LEFT:  self.status_label  ← mensajes dinámicos de estado
    RIGHT: self.progress_bar (CTkProgressBar, width=200, oculto por defecto)
```

**Para cambiar altura status bar** → `main_window.py:451` `height=30`  
**Para cambiar fuente status bar** → `main_window.py:456` `ThemeManager.get_font(11)`

### Temas, colores y fuentes — dónde están

```
gui/themes.py
  ThemeManager (clase estática)
    DEFAULT_THEME = ThemeName.DARK
    get_font(size)      → ("Roboto", size)
    get_font_bold(size) → ("Roboto", size, "bold")
    get_layout_padding() → 10
    get_element_padding() → 5
    get_color_theme()   → "blue"   ← esquema de color CTk (blue/green/dark-blue)
    set_color_theme(theme)  ← cambiar esquema global

  get_treeview_colors() → dict        # colores ttk.Treeview adaptados al tema
    dark:  tree_bg="#2b2b2b"  tree_fg="#dcddde"  select_bg="#1f538d"
    light: tree_bg="#ffffff"  tree_fg="#1a1a1a"  select_bg="#3b7dd8"

  get_textbox_colors() → dict         # colores tk.Text (paneles de logs)
    dark:  bg="#1e1e1e"  fg="#d4d4d4"
    light: bg="#ffffff"  fg="#1a1a1a"

  create_sidebar_button(text, command, width=200) → CTkButton
    height=35, corner_radius=0
    fg_color="transparent"
    text_color=("gray10", "gray90")
    hover_color=("gray70", "gray30")
    anchor="w"

  create_status_bar(master, status_text, version) → CTkFrame
    height=30, corner_radius=0
```

**Para cambiar fuente global** → `gui/themes.py:84` `return ("Roboto", size)`  
**Para cambiar esquema de color CTk** → `gui/themes.py:61` `return "blue"` (opciones: blue, green, dark-blue)  
**Para cambiar colores dark de Treeview** → `gui/themes.py:133-142`  
**Para cambiar colores light de Treeview** → `gui/themes.py:143-151`

### Internacionalización (i18n)

```
gui/i18n.py
  STRINGS: dict[lang][key] → str     # diccionario estático ES/EN
    "es": { "nav.dashboard": "Dashboard", "nav.projects": "Proyectos", ... }
    "en": { "nav.dashboard": "Dashboard", "nav.projects": "Projects", ... }

  LanguageManager
    _current_language: str = "es"    # default español
    set_language(lang)
    get_language() → str
    t(key) → str                     # busca en STRINGS[lang][key]

  _t(key, fallback="") → str         # shortcut de LanguageManager.t()
```

**Para agregar un string nuevo** → `gui/i18n.py:23` — agregar en bloque "es" y "en"  
**Para agregar una clave de nav item** → `i18n.py` + `MainWindow._NAV_I18N_KEYS` + `MainWindow.NAV_ITEMS`  
**Para cambiar un texto visible al usuario** → buscar la clave en `STRINGS["es"]` en `i18n.py`

### System Tray

```
BaseLayout.minimize_to_tray()                     # framework.py:105
  window.withdraw()                  # oculta ventana
  self._tray = SystemTray(
      on_open=self._tray_safe_restore,
      on_settings=self._tray_safe_settings,
      on_help=self._tray_safe_help,
      on_quit=self._tray_safe_quit,
  )
  self._tray.start(title=self.title)

# Callbacks thread-safe (pystray thread → Tk thread via window.after)
_tray_safe_restore()  → window.after(0, self.restore_from_tray)
_tray_safe_quit()     → window.after(0, self.quit_app)

BaseLayout.restore_from_tray()                    # framework.py:156
  window.deiconify()
  window.state("normal")
  window.lift()
  self._tray.stop()
```

**Para cambiar el menú del tray** → `utils/tray.py` — `SystemTray.__init__` y `_build_menu()`

### Cache de vistas y cambio de tema

Las vistas se cachean en `self.view_frames[nav_id]` para evitar recreación en cada navegación.  
Al cambiar tema: `_destroy_cached_views()` destruye todo el cache, luego `_refresh_current_view()` recrea la vista activa.

---

## 4. Capa Data — Managers y Persistencia

### ProjectManager

```
data/project_manager.py
  @dataclass DirectoriesConfig   ← base, evidence, excel, repositories, logs
  @dataclass Project             ← id, name, client_name, description,
                                    excel_path, evidence_path, ticket_provider,
                                    ticket_project, directories, metadata,
                                    created, updated

  ProjectManager(base_path, use_vault=True)
    base_path/
      projects.json              ← índice (lista metadatos, sin cifrar)
      {project_id}.json          ← datos completos (cifrado SecurityVault)
    _lock: threading.Lock()

    create_project(name, client_name, ...) → Project
      id = generate_project_id()  # "CASE-2026-001" via utils/user_id.py
      guarda en disco (cifrado)
      actualiza projects.json (índice)

    list_projects() → list[Project]   ← lee projects.json (sin descifrar body)
    get_project(project_id) → Project  ← descifra {project_id}.json
    update_project(project) → bool
    delete_project(project_id) → bool
```

**ID de proyecto**: `CASE-YYYY-NNN` — generado por `utils/user_id.py:generate_project_id()`  
**Persistencia**: `%APPDATA%/DSOAgent/projects/` (Windows) o `~/.dsoagent/projects/` (Linux)  
**Cifrado**: con `SecurityVault` por defecto (`use_vault=True`)

### ProfileManager (multi-perfil)

```
data/profile_manager.py
  @dataclass ProfileMetadata     ← público (sin cifrar): profile_id, profile_name,
                                    windows_username, created_at, updated_at,
                                    project_count, evidence_count
  @dataclass ProfileData         ← privado (se cifra): todos los campos + password_hash

  ProfileManager(profiles_base_dir=None)
    profiles_dir = %APPDATA%/DSOAgent/profiles/  (por defecto)
    profiles_dir/
      {profile_id}/
        metadata.json            ← ProfileMetadata (sin cifrar)
        profile.enc              ← ProfileData (cifrado Fernet PBKDF2)
        vault/                   ← SecurityVault del perfil

    create_profile(name, master_password, windows_username) → ProfileMetadata
      genera salt único (os.urandom(16))
      deriva clave Fernet con PBKDF2HMAC(SHA256, 100000 iteraciones)
      cifra ProfileData con clave derivada
      timestamps con microsegundos para evitar colisiones

    authenticate(profile_id, master_password) → bool
      carga metadata.json
      deriva clave desde master_password + salt almacenado
      intenta descifrar profile.enc
      retorna True si descifra exitosamente

    list_profiles() → list[ProfileMetadata]
      escanea profiles_dir/*/metadata.json
      retorna sin descifrar bodies

    export_profile(profile_id, password, dest_path) → ExportResult
    import_profile(src_path, password) → ImportResult
    delete_profile(profile_id) → bool
```

**Salt**: único por perfil, almacenado en `profile.enc` header  
**PBKDF2**: SHA256, 100,000 iteraciones  
**Timestamps con microsegundos**: `datetime.now().strftime("%Y%m%d%H%M%S%f")`

### ExportImportManager

```
data/export_import_manager.py
  ExportImportManager(profile_manager)

  export_profile(profile_id, password, dest_path,
                 include_vault=True, include_evidence=False, include_projects=True)
    → ExportResult(success, size_bytes, error)
    genera archivo .dsoprofile (zip cifrado)

  import_profile(src_path, password)
    → ImportResult(success, new_profile_id, errors)
    desempaqueta .dsoprofile
    crea perfil nuevo en profiles_dir
```

### ExcelHandler + Finding dataclass

```
data/excel_handler.py
  @dataclass Finding
    id: str
    project: str
    title: str
    description: str
    severity: str       # "Alta" | "Media" | "Baja" | "Crítica" | "Informativa"
    status: str         # "Abierto" | "En progreso" | "Cerrado"
    file_path: str
    line_number: int
    created: datetime
    updated: datetime
    evidence: list[str] = None

  ExcelHandler(file_path=None)
    load(path) → bool          ← carga xlsx con pandas
    save() → bool              ← guarda con openpyxl
    create_template(path) → bool
    add_finding(finding: Finding) → bool
    list_findings() → list[Finding]
    update_finding(finding_id, fields) → bool
    delete_finding(finding_id) → bool
```

**Para agregar un campo a Finding** → `data/excel_handler.py` dataclass + columnas en `create_template()`

### ExcelImporter (15 presets)

```
data/excel_importer.py
  ExcelImporter()
    detect_format(file_path) → Optional[str]   ← detecta preset por columnas
    import_file(file_path, preset_id) → list[Finding]
    get_available_presets() → list[ImportPreset]

data/excel_presets.py
  @dataclass ImportPreset
    id: str           # "sonarqube", "burp_suite", "nessus", etc.
    name: str
    category: str     # SAST | DAST | Secret Scanning | SCA | Estándar
    column_mapping: dict   # {col_excel: campo_Finding}
    required_columns: list
    aliases: list     # backward-compat nombres legacy

  15 presets definidos: dsoagent_standard, sast_static_code, sast_static_app,
    dast_dynamic_web, dast_dynamic_app, burp_suite, nessus, owasp_zap,
    sonarqube, semgrep, checkmarx, veracode, github_codeql,
    github_secret_scanning, snyk
```

**Para agregar preset nuevo** → `data/excel_presets.py` + tabla en `PROMPT.md §Conectores Estado Integraciones`

### ProviderRegistry

```
data/provider_registry.py
  @dataclass ProviderFieldDef    ← campo de tipo: name, label, field_type, required, ...
    field_type: text|password|select|checkbox|file|number|url

  @dataclass ProviderTypeDef     ← tipo de proveedor
    id: str                      # "builtin_azure", "builtin_jira", etc.
    name: str
    category: tickets|scripts|source_control|storage|scanning|custom
    fields: list[ProviderFieldDef]
    builtin: bool
    supports_test: bool

  @dataclass ProviderConnection  ← instancia concreta con valores
    id: str                      # UUID corto 8 chars
    type_id: str                 # referencia a ProviderTypeDef.id
    name: str
    values: dict                 # {field_name: valor} — credenciales cifradas
    project_ids: list[str]       # proyectos asociados
    active: bool = True

  ProviderRegistry(registry_path=None)
    persiste en: get_config_dir()/providers/
      types.json       ← lista ProviderTypeDef
      connections.json ← lista ProviderConnection (valores cifrados)

    list_types() → list[ProviderTypeDef]
    list_connections() → list[ProviderConnection]
    list_connections_active(category=None) → list filtrada
    add_connection(conn) → ProviderConnection
    resolve_provider(conn) → AbstractTicketAPI instance
      # instancia el provider concreto según conn.type_id
      "builtin_azure" → AzureDevOpsProvider(conn.values)
      "builtin_jira"  → JiraProvider(conn.values)
      "builtin_http"  → GenericHTTPProvider(conn.values)
```

**7 tipos builtin**: `builtin_azure`, `builtin_jira`, `builtin_git`, `builtin_scripts`, `builtin_sharepoint`, `builtin_scan`, `builtin_http`

### ConfigManager

```
data/config_manager.py
  @dataclass AppSettings
    version: str = "0.9.8"
    theme: str = "dark"
    language: str = "es"
    log_level: str = "INFO"
    evidence_max_size_mb: int = 50
    auto_save: bool = True
    ... (más campos)

  ConfigManager(config_path=None)
    persiste en: get_config_dir()/settings.json
    load() → AppSettings     ← crea defaults si no existe
    save(settings) → bool
    get() → AppSettings
    update(**kwargs) → bool

data/config_atomic.py
  AtomicConfig                  ← escribe a temp file + rename atómico
    Previene corrupción en escritura incompleta (crash, power loss)
    Mantiene shadow copy para rollback
```

### EvidenceManager

```
data/evidence_manager.py
  EvidenceManager(base_path)
    base_path/{project_id}/
      evidence.json             ← índice con hashes y metadatos
      files/                    ← archivos de evidencia

    add_evidence(project_id, file_path, description) → EvidenceItem
      calcula hash SHA-256 del archivo
      copia archivo a base_path/{project_id}/files/
      registra en evidence.json

    list_evidence(project_id) → list[EvidenceItem]
    get_evidence(project_id, evidence_id) → EvidenceItem
    delete_evidence(project_id, evidence_id) → bool
    upload_to_sharepoint(evidence_id, sharepoint_conn) → bool
```

---

## 5. Capa Security — Vault

### Pseudocódigo de cifrado/descifrado

```
security/vault.py
  SecurityVault(vault_dir=None)
    vault_dir = ~/.dsoagent/ (Linux) o %APPDATA%/DSOAgent/ (Windows)
    vault_dir/
      vault.key       ← clave Fernet derivada del hardware (cifrada)
      recovery.key    ← clave de recuperación (generada una sola vez)

  INICIALIZACIÓN:
    SI vault.key existe:
        _key = derive_key_from_hardware()   ← usa machine_id + username
        descifra vault.key con _key         ← obtiene clave Fernet real
        self._fernet = Fernet(_key_deciphered)
    ELSE:
        _key = derive_key_from_hardware()
        genera nueva clave Fernet aleatoria
        cifra y guarda vault.key
        log "[Vault] Creando Nueva Boveda"

  derive_key_from_hardware() → bytes
    machine_id = utils/user_id.py:get_machine_id()   # UUID hardware
    username   = os.environ.get("USERNAME") o whoami
    salt       = SHA256(machine_id + username)[:16]
    kdf = PBKDF2HMAC(SHA256, 100000 iteraciones, salt, 32 bytes)
    return base64.urlsafe_b64encode(kdf.derive(password=machine_id.encode()))

  encrypt(data: Any) → bytes
    json.dumps(data).encode() → bytes
    self._fernet.encrypt(bytes) → token cifrado

  decrypt(token: bytes) → Any
    decrypted = self._fernet.decrypt(token) → bytes
    json.loads(decrypted.decode()) → Any

  generate_recovery_key() → str
    genera clave aleatoria UUID4
    la cifra con vault.key
    guarda en recovery.key
    retorna string "XXXX-XXXX-XXXX-XXXX"

  recover_with_key(recovery_key: str) → bool
    lee recovery.key
    descifra con recovery_key
    restaura vault.key
```

**Si se cambia de PC o hardware** → usar `recover_with_key()` con la clave guardada externamente  
**Para cifrar un dict** → `vault.encrypt({"key": "value"})` → bytes  
**Para descifrar** → `vault.decrypt(token)` → dict  
**Archivos generados**: `%APPDATA%/DSOAgent/vault.key` + `recovery.key`

---

## 6. Capa Agent — Orchestrator

### Pseudocódigo del Orchestrator

```
agent/orchestrator.py
  @dataclass ProcessingResult    ← finding_id, success, ticket_created,
                                    evidence_added, errors, warnings
  @dataclass SyncResult          ← total, processed, successful, failed,
                                    findings: list[ProcessingResult], duration

  Orchestrator(base_path, provider_registry=None)
    self.excel_handler    = ExcelHandler()
    self.evidence_manager = EvidenceManager(base_path/evidence)
    self.project_manager  = ProjectManager(base_path/projects)
    self.report_manager   = ReportManager()
    self.sandbox_runner   = SandboxRunner()
    self.provider_registry = provider_registry or ProviderRegistry()
    self._current_project = None
    self._active_provider = None

  @property ticket_service → AbstractTicketAPI | None
    SI _active_provider: retorna cacheado
    SI _current_project.ticket_provider:
        conn = _resolve_project_connection(ticket_provider)
        self._active_provider = provider_registry.resolve_provider(conn)
    retorna _active_provider

  set_active_project(project_id) → bool
    project = project_manager.get_project(project_id)
    self._current_project = project
    self._active_provider = None  ← reset para re-resolver
    retorna True

  sync_project(project_id) → SyncResult
    set_active_project(project_id)
    findings = excel_handler.list_findings()
    resultado = SyncResult(total=len(findings))
    para cada finding:
        result = _process_finding(finding)
        resultado.findings.append(result)
        SI result.success: resultado.successful += 1
        ELSE: resultado.failed += 1
    retorna resultado

  _process_finding(finding) → ProcessingResult
    SI ticket_service:
        ticket_id = ticket_service.create_ticket(...)
        result.ticket_created = (ticket_id is not None)
    evidencias = evidence_manager.list_evidence(project_id)
    result.evidence_added = len(evidencias)
    retorna result
```

### Contratos (agent/contracts.py)

```
agent/contracts.py
  TicketPriority (enum)          ← LOWEST, LOW, MEDIUM, HIGH, HIGHEST
  # Re-exporta desde integrations/ticket_service.py para desacoplar capas
```

**Orchestrator NO conoce gui/** — toda inyección de dependencias viene desde `MainWindow._init_managers()`  
**Para inyectar un ProviderRegistry custom** → `Orchestrator(base_path, provider_registry=mi_registry)`

---

## 7. Capa Integrations — Proveedores

### Herencia de providers de tickets

```
ABC
└── AbstractTicketAPI (integrations/ticket_service.py:65)
    ├── connect() → bool                  abstracto
    ├── disconnect() → bool               abstracto
    ├── create_ticket(TicketCreate) → str abstracto
    ├── update_ticket(id, TicketUpdate)   abstracto
    ├── get_ticket(id) → Ticket           abstracto
    ├── list_tickets(project, filters)    abstracto
    ├── close_ticket(id) → bool           abstracto
    └── add_attachment(id, path) → bool   abstracto
    ├── AzureDevOpsProvider (integrations/azure_provider.py:41)
    │   ├── API_VERSIONS = ["5.1", "6.0", "7.0"]
    │   ├── config: dict (organization, project, token, url_base, work_item_type)
    │   ├── _rate_limiter: AsyncRateLimiter
    │   ├── _retry: AsyncRetryHelper
    │   └── usa aiohttp + API REST Azure DevOps
    ├── JiraProvider (integrations/jira_provider.py)
    │   ├── config: dict (url, username, api_token, project_key)
    │   ├── _rate_limiter: AsyncRateLimiter
    │   └── usa aiohttp + Jira REST API v3
    └── GenericHTTPProvider (integrations/http_provider.py)
        ├── config: dict (base_url, endpoints, headers, auth_type)
        └── REST configurable vía endpoints JSON
```

### AzureDevOpsProvider — pseudocódigo

```
AzureDevOpsProvider(config)
  organization = config["organization"]
  project      = config["project"]
  token        = config["token"]          # almacenado cifrado en SecurityVault
  url_base     = config.get("url_base", "dev.azure.com")
  work_item_type = config.get("work_item_type", "Bug")
  _base_url = f"https://{url_base}/{organization}/{project}/_apis"

  connect() → bool
    intenta GET {_base_url}/projects?api-version=7.0
    SI 200: self._connected = True; retorna True
    SI RuntimeError: self._connected = True (tolerancia async); retorna True
    NUNCA raise → retorna False con log error

  create_ticket(ticket_create) → str | None
    SI no conectado: retorna None
    _rate_limiter.acquire()            # ventana deslizante 10 req/60s
    payload = _build_work_item_patch(ticket_create)
    POST {_base_url}/wit/workitems/${work_item_type}?api-version=7.0
    retorna str(work_item["id"]) o None

  _build_work_item_patch(tc) → list[dict]
    retorna lista de operaciones JSON Patch:
      [{"op": "add", "path": "/fields/System.Title", "value": tc.title}, ...]
```

**Configuración**: via GUI → Configuración → Proveedores → Azure DevOps  
**Token**: almacenado cifrado con `SecurityVault`, nunca en texto plano  
**Rate limit**: `AZURE_RATE_LIMIT` de `utils/rate_limiter.py` — 10 req/60s por defecto

### SharePointProvider — pseudocódigo

```
SharePointProvider(config)
  site_url      = config["site_url"]     # https://contoso.sharepoint.com/sites/Auditoria
  tenant_id     = config["tenant_id"]
  client_id     = config["client_id"]
  client_secret = config["client_secret"] # cifrado en SecurityVault
  library_name  = config.get("library_name", "Documents")
  folder_path   = config.get("folder_path", "")

  _GRAPH_BASE = "https://graph.microsoft.com/v1.0"
  _TOKEN_URL  = "https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token"

  get_token() → str               # OAuth2 client_credentials, renueva automático
    SI token_expiry > now: retorna token cacheado
    POST _TOKEN_URL con client_id + client_secret + scope
    self._token = response["access_token"]
    self._token_expiry = now + expires_in - 60s buffer

  upload_file(local_path, remote_name) → SharePointResult
    token = get_token()
    drive_id = _get_drive_id()
    PUT {_GRAPH_BASE}/drives/{drive_id}/items/root:/{folder}/{remote_name}:/content
    retorna SharePointResult(success, message, data)

  list_files(folder_path) → list[SharePointFile]
  download_file(item_id, dest_path) → SharePointResult
  create_folder(folder_name) → SharePointResult
```

**Requiere**: App Registration en Azure AD con permisos `Files.ReadWrite.All` en Graph API  
**Token**: renovación automática con buffer de 60s antes de expiración

### GitManager — pseudocódigo

```
integrations/git_manager.py
  @dataclass RepoEntry    ← name, branch, url
  @dataclass GitResult    ← success, repo, branch, output, error, duration_seconds

  GitManager()
    clone(url, branch, dest_path, progress_callback) → GitResult
      asyncio.create_subprocess_exec("git", "clone", "--branch", branch, url, dest_path)
      captura stdout/stderr sin shell=True   ← OBLIGATORIO (PROMPT.md Regla 1)
      retorna GitResult

    pull(repo_path, branch, progress_callback) → GitResult
      asyncio.create_subprocess_exec("git", "-C", repo_path, "pull", "origin", branch)
      retorna GitResult

    clone_multiple(repos, dest_base, progress_callback) → list[GitResult]
      ejecuta clones secuencialmente (no paralelo — evita saturar red)
```

**Sin shell=True** en ninguna operación — lista de argumentos siempre

### Proveedores builtin disponibles en ProviderRegistry

| ID builtin | Clase concreta | Archivo |
|------------|----------------|---------|
| `builtin_azure` | `AzureDevOpsProvider` | `integrations/azure_provider.py` |
| `builtin_jira` | `JiraProvider` | `integrations/jira_provider.py` |
| `builtin_sharepoint` | `SharePointProvider` | `integrations/sharepoint_provider.py` |
| `builtin_http` | `GenericHTTPProvider` | `integrations/http_provider.py` |
| `builtin_git` | manejado por `GitManager` | `integrations/git_manager.py` |
| `builtin_scripts` | manejado por `SandboxRunner` | `sandbox/runner.py` |
| `builtin_scan` | sin implementación concreta | futuro SAST/DAST |

**Para agregar proveedor nuevo** → ver sección 12 Guía de Modificaciones Rápidas

---

## 8. Capa Utils — Transversales

### paths.py — Rutas universales

```
utils/paths.py
  is_frozen() → bool
    getattr(sys, "frozen", False)     # True cuando es .exe PyInstaller

  get_project_root() → Path
    SI hasattr(sys, "_MEIPASS"): retorna Path(sys._MEIPASS)   # PyInstaller temp dir
    SI is_frozen():               retorna Path(sys.executable).parent
    ELSE:                         retorna Path(__file__).parent.parent  # raiz repo

  get_config_dir() → Path
    SI no frozen:   <project_root>/config/
    SI frozen Win:  %APPDATA%/DSOAgent/config/
    SI frozen Lin:  ~/.dsoagent/config/

  get_logs_dir() → Path
    SI no frozen:   <project_root>/logs/
    SI frozen Win:  %APPDATA%/DSOAgent/logs/
    SI frozen Lin:  ~/.dsoagent/logs/

  get_projects_dir() → Path    # <config_dir>/../projects/ o %APPDATA%/DSOAgent/projects/
  get_evidence_dir() → Path    # <config_dir>/../evidence/ o %APPDATA%/DSOAgent/evidence/
  get_data_dir() → Path        # <project_root>/data/  (código fuente del módulo data/)
  get_assets_dir() → Path      # <project_root>/assets/
  get_docs_dir() → Path        # <project_root>/docs/
```

**Regla**: Todo módulo debe usar `get_*_dir()` — nunca rutas hardcodeadas ni `os.getcwd()`

### logging_system.py — Logging centralizado

```
utils/logging_system.py
  MAX_BYTES    = 10 MB          # por archivo de log
  BACKUP_COUNT = 3              # máximo 3 rotaciones (4 archivos totales)

  setup_logging(log_level, max_bytes, backup_count)
    SI ya configurado (_setup_done): retorna sin hacer nada
    root_logger = logging.getLogger()
    handlers:
      RotatingFileHandler(get_logs_dir()/"dsoagent.log", maxBytes, backupCount)
      StreamHandler(sys.stdout)   # consola en desarrollo
    _FORMATTER = "%(asctime)s - %(name)s - %(levelname)s - [%(funcName)s] - %(message)s"

  get_logger(name) → logging.Logger
    SI name in _loggers: retorna cacheado
    logger = logging.getLogger(name)
    logger.propagate = True       # propaga al root logger
    _loggers[name] = logger
    retorna logger
```

**Formato de log**: `DD-MM-YY HH:MM:SS - modulo - LEVEL - [funcion] - [Modulo] Mensaje`  
**Archivo**: `%APPDATA%/DSOAgent/logs/dsoagent.log` (frozen) o `<root>/logs/dsoagent.log` (venv)  
**Para ver logs en tiempo real**: `Get-Content logs\dsoagent.log -Wait` (PowerShell) o `tail -f` (Linux)

### rate_limiter.py + retry.py

```
utils/rate_limiter.py
  RateLimitConfig(max_requests=10, time_window=60.0, retry_after=60.0)

  RateLimiter(config)             # ventana deslizante, sincrónico
    acquire() → bool              # True si puede proceder, False si en cooldown
    wait_if_needed()              # bloquea hasta que haya slot disponible

  AsyncRateLimiter(config)        # versión async para aiohttp
    acquire()  → await ...
    AZURE_RATE_LIMIT = RateLimitConfig(max_requests=10, time_window=60.0)
    JIRA_RATE_LIMIT  = RateLimitConfig(max_requests=10, time_window=60.0)

utils/retry.py
  AsyncRetryHelper(max_attempts=3, base_delay=1.0, max_delay=30.0, backoff=2.0)
    execute(coroutine) → result   # retry con exponential backoff
    # usa tenacity internamente
```

### task_scheduler.py — Scheduler Windows

```
utils/task_scheduler.py
  ScheduleFrequency: ONCE | DAILY | WEEKLY | MONTHLY

  @dataclass ScheduledTask
    task_name, path_script, frequency, start_time, days_of_week, enabled

  TaskScheduler()
    create_task(task) → TaskSchedulerResult
      asyncio.create_subprocess_exec("schtasks", "/Create", "/TN", task_name, ...)
      sin shell=True

    list_tasks() → list[ScheduledTask]
      asyncio.create_subprocess_exec("schtasks", "/Query", "/FO", "CSV", ...)

    delete_task(task_name) → TaskSchedulerResult
    enable_task(task_name) → TaskSchedulerResult
    disable_task(task_name) → TaskSchedulerResult
```

**Solo Windows**: verifica `platform.system() == "Windows"` antes de ejecutar  
**Linux**: SchedulerView muestra mensaje "No disponible en este sistema"

### tray.py — System Tray

```
utils/tray.py
  create_default_icon() → PIL.Image
    cuadro negro 64x64, borde y barras "#00d4aa" (color brand DSOAgent)

  SystemTray(on_open, on_settings, on_help, on_quit)
    start(title) → None
      self._icon = pystray.Icon(title, create_default_icon(), menu=_build_menu())
      threading.Thread(target=self._icon.run, daemon=True).start()

    stop() → None
      self._icon.stop()

    _build_menu() → pystray.Menu
      "Abrir DSOAgent"  → on_open
      "Configuración"   → on_settings
      "Ayuda"           → on_help
      "---" (separador)
      "Salir"           → on_quit
```

**Thread-safe**: callbacks son llamados desde thread pystray — deben usar `window.after(0, callback)` para tocar UI Tk

### user_id.py — IDs únicos

```
utils/user_id.py
  get_machine_id() → str         # UUID único de la máquina (registro Windows o /etc/machine-id)
  get_current_year() → int       # año actual
  generate_project_id() → str    # "CASE-2026-001", "CASE-2026-002", ...
    contador en config_dir/project_counter.json (thread-safe con lock)
```

---

## 9. Sandbox

### Arquitectura del sandbox

```
Flujo de ejecución de scripts:

ScriptsView (gui/scripts_view.py)
  usuario selecciona script .py y hace click "Ejecutar"
  → MainWindow.run_background("sandbox", SandboxRunner.execute_file, file_path, ...)
  → SandboxRunner.execute_file(file_path, project_id, finding_id, evidence_path)
      1. validator.validate(file_path) → ValidationResult
         SI no válido: retorna ExecutionResult(success=False, errors=[...])
      2. contract = create_contract(file_path, ExecutionContext(...))
      3. MultiprocessSandboxRunner.run(contract)
         → subprocess separado (aislamiento total de proceso)
         → timeout configurable (default 30s)
      4. retorna ExecutionResult(success, output, errors, warnings, logs)
```

### ScriptValidator — qué se valida

```
sandbox/validator.py
  ALLOWED_BUILTINS = { "print", "len", "range", "str", "int", "float", "bool",
                       "list", "dict", "set", "tuple", "type", "isinstance",
                       "hasattr", "getattr", "enumerate", "zip", "map", "filter",
                       "sorted", "reversed", "sum", "min", "max", "abs", "round",
                       "open", "json", "datetime", "pathlib" }

  ALLOWED_IMPORTS  = { "json", "datetime", "pathlib", "collections", "typing",
                       "re", "base64", "hashlib", "urllib", "xml" }

  BLOCKED_PATTERNS = [
      r"import\s+os",          r"import\s+subprocess",
      r"import\s+sys",         r"import\s+socket",
      r"import\s+requests",    r"__import__",
      r"eval\s*\(",            r"exec\s*\(",
      r"os\.system",           r"subprocess\.Popen",
      r"threading\.Thread",    r"asyncio\.create_subprocess_exec",
      ... (14 patrones totales)
  ]

  validate(file_path) → ValidationResult(valid, errors, warnings)
    1. leer código fuente
    2. parsear AST con ast.parse()
    3. revisar imports contra ALLOWED_IMPORTS
    4. buscar BLOCKED_PATTERNS con regex
    5. SI pasa todo: valid=True
```

### ScriptContract + ExecutionResult

```
sandbox/contract.py
  @dataclass ExecutionContext    ← project_id, finding_id, evidence_path, config, timestamp
  @dataclass ExecutionResult     ← success, output, errors, warnings, execution_time, logs

  ScriptContract
    REQUIRED_FUNCTIONS = ["execute"]
    OPTIONAL_FUNCTIONS = ["validate", "cleanup"]
    # El script debe tener def execute(context: ExecutionContext) → ExecutionResult

  create_contract(file_path, context) → ScriptContract
    carga el módulo Python del script
    verifica que tenga función "execute"
    retorna ScriptContract listo para ejecutar
```

### MultiprocessSandboxRunner

```
sandbox/multiprocess_runner.py
  MultiprocessSandboxRunner(config)
    run(contract) → ExecutionResult
      multiprocessing.Process(target=_run_in_process, args=(contract, queue))
      process.start()
      process.join(timeout=config.timeout)
      SI timeout: process.kill(); retorna ExecutionResult(success=False, errors=["Timeout"])
      retorna result desde queue
```

**Aislamiento**: el script se ejecuta en proceso separado — un crash del script no mata la app  
**Para agregar librería al whitelist** → `sandbox/validator.py:ALLOWED_IMPORTS` + `ALLOWED_BUILTINS`

---

## 10. Entregas — Cómo se Empaqueta

### Modo 1: venv (desarrollo / Linux sin EXE)

```
Estructura:
  DSOAgent_v1_2026/
    venv/                    ← entorno virtual (NO versionar, en .gitignore)
    requirements.txt         ← dependencias producción con GUI
    requirements.headless.txt ← dependencias sin GUI (para Docker headless)

Arranque:
  venv\Scripts\python.exe main.py    (Windows)
  venv/bin/python main.py            (Linux)

Datos en desarrollo:
  config/                   ← AppSettings, providers, vault
  logs/                     ← dsoagent.log
  projects/                 ← proyectos (cifrados)
  evidence/                 ← archivos de evidencia
```

**Regla**: SIEMPRE usar el Python del venv, nunca el global del sistema (PROMPT.md Regla 8)

### Modo 2: PyInstaller — EXE Windows

```
Build:
  venv\Scripts\python.exe -m PyInstaller DSOAgent.spec --clean

DSOAgent.spec
  Analysis(['main.py'], ...)
  datas:
    ('assets', 'assets')   ← iconos y recursos
    ('docs', 'docs')       ← manuales incluidos en el EXE
  hiddenimports: [         ← todos los módulos del proyecto + deps
    'agent', 'agent.orchestrator', 'agent.contracts',
    'data.*', 'gui.*', 'integrations.*',
    'sandbox.*', 'security.*', 'utils.*', 'templates.*',
    'customtkinter', 'pystray', 'PIL', 'pandas', 'openpyxl',
    'aiohttp', 'tenacity', 'cryptography', 'zoneinfo'
  ]
  EXE(
    name='DSOAgent',
    console=False,          ← sin consola (GUI pura)
    upx=True,               ← compresión UPX
    icon='assets/icons/icons8-gear-64.ico',
    version='file_version_info.txt'
  )

Resultado: dist/DSOAgent.exe  (~44 MB con UPX)

En runtime frozen:
  sys._MEIPASS   ← directorio temporal con assets extraidos
  sys.frozen=True
  get_config_dir() → %APPDATA%/DSOAgent/config/
  get_logs_dir()  → %APPDATA%/DSOAgent/logs/
```

**file_version_info.txt**: metadatos de versión Windows (ProductVersion, FileVersion, CompanyName)  
**Para agregar módulo al EXE**: agregar en `DSOAgent.spec:hiddenimports` + `DSOAgent_linux.spec:hiddenimports`

### Modo 3: Docker headless (servidor / CI)

```
Dockerfile (headless — sin GUI):
  FROM python:3.12-slim
  COPY requirements.headless.txt .
  RUN pip install -r requirements.headless.txt    ← sin customtkinter, sin pystray
  COPY . .                                         ← sin venv/ (.dockerignore lo excluye)
  ENV TIMEZONE=America/Mexico_City
  CMD ["python", "-c", "from agent.orchestrator import Orchestrator; ...loop..."]
  HEALTHCHECK: python -c "from agent.orchestrator import Orchestrator; print('ok')"

Dockerfile.gui (con GUI via Xvfb):
  FROM python:3.12-slim
  RUN apt-get install python3-tk tk-dev xvfb libxcb1 libx11-6 ...
  COPY requirements.txt .                          ← con customtkinter
  ENV DISPLAY=:99
  CMD ["bash", "-c", "Xvfb :99 -screen 0 1280x800x24 & sleep 2 && python main.py"]

docker-compose.yml:
  services:
    agent-core:               ← usa Dockerfile (headless)
      restart: unless-stopped
      volumes:
        - agent-config:/app/config
        - agent-logs:/app/logs
        - agent-projects:/app/projects
        - agent-evidence:/app/evidence
      healthcheck: cada 30s
```

**Diferencias headless vs GUI**:

| Aspecto | Dockerfile (headless) | Dockerfile.gui |
|---------|----------------------|----------------|
| GUI | No (solo Orchestrator) | Sí (CustomTkinter via Xvfb) |
| requirements | requirements.headless.txt | requirements.txt |
| Display | No | Xvfb :99 |
| Uso | Servidor, CI/CD | Testing GUI en Linux |

**Volúmenes persistentes**: `agent-config`, `agent-logs`, `agent-projects`, `agent-evidence`  
**Para producción**: fijar digest SHA256 en FROM (`FROM python:3.12-slim@sha256:...`)

### Modo 4: Linux binary (DSOAgent_linux.spec)

```
DSOAgent_linux.spec
  Analysis(['main.py'], ...)          ← idéntico a Windows
  datas: [('assets', 'assets'), ('docs', 'docs')]
  hiddenimports: [...igual que Windows...]
  EXE(
    name='DSOAgent',
    console=False,
    upx=True
  )

Build en Ubuntu 24.04:
  venv/bin/python -m PyInstaller DSOAgent_linux.spec --clean
  → dist/DSOAgent  (ELF binary, ~45 MB)
```

**Diferencia con Windows .spec**: no incluye `icon=` ni `version=file_version_info.txt` (Windows-only)

### Resumen de artefactos por modo

| Modo | Artefacto | Plataforma | GUI | Datos |
|------|-----------|------------|-----|-------|
| venv | `python main.py` | Win/Lin | Sí | `<root>/config/`, `logs/`, `projects/` |
| EXE Windows | `DSOAgent.exe` (~44 MB) | Windows | Sí | `%APPDATA%/DSOAgent/` |
| EXE Linux | `DSOAgent` (~45 MB) | Ubuntu | Sí | `~/.dsoagent/` |
| Docker headless | `agent-core` container | Linux | No | Volúmenes Docker |
| Docker GUI | container + Xvfb | Linux | Sí | `~/.dsoagent/` |

---

## 11. Tests — Estructura y Dónde Tocar

### Árbol de tests

```
tests/
  conftest.py                    ← fixtures compartidos (scope=session para CTk root)
  validate_backend.py            ← 12 checks de importación y sanidad de módulos
  validate_gui_headless.py       ← 13+ checks de instanciación GUI headless

  Suites consolidadas (optimización 2026-06-05):
  gui/
    test_views_comprehensive.py  ← tests unificados de todas las vistas GUI
  profile/
    test_profile_suite.py        ← tests sistema multi-perfil
  data/
    test_data_handlers.py        ← tests ExcelHandler, ExcelImporter, Finding
  integration/
    test_end_to_end.py           ← tests E2E (Orchestrator + managers)

  Tests individuales por módulo:
    test_vault.py                  test_provider_registry.py
    test_project_manager.py        test_evidence_manager.py
    test_excel_handler.py          test_excel_importer.py
    test_orchestrator.py           test_sandbox.py
    test_task_scheduler.py         test_git_manager.py
    test_i18n.py                   test_config_atomic.py
    test_sharepoint_provider.py    test_http_provider.py
    test_jira_provider.py          test_retry.py
    test_profile_system.py         test_security_startup.py
    test_gui_settings_view.py      test_gui_provider_management.py
    test_gui_tickets_view.py       test_gui_scripts_view.py
    ... (70+ archivos totales)

  Fixtures preconstruidos:
  fixtures/                      ← archivos .xlsx, .json de prueba
  data_dummy/                    ← datos dummy para tests de importación
  mock_api_server.py             ← servidor mock para tests HTTP/Azure/Jira
```

### conftest.py — fixtures clave

```
tests/conftest.py

  @pytest.fixture(scope="session")
  def tk_root_session() → ctk.CTk
    # Root CTk único compartido en toda la sesión — evita conflictos ttk.Style
    root = ctk.CTk()
    root.withdraw()                        # invisible
    root.geometry("1x1+-10000+-10000")     # fuera de pantalla
    root.attributes("-alpha", 0)           # transparente
    yield root
    root.destroy()

  def pump(widget, times=3)
    # Procesa eventos Tk sin mostrar ventana
    root = widget.winfo_toplevel()
    for _ in range(times): widget.update()
    root.withdraw()                        # re-ocultar tras update()

  @pytest.fixture
  def mock_project_manager() → MagicMock
    # ProjectManager completo con todos los métodos stubbeados

  @pytest.fixture
  def mock_provider_registry(tmp_path_factory) → ProviderRegistry
    # ProviderRegistry real con directorio temporal aislado

  @pytest.fixture
  def mock_evidence_manager(tmp_path_factory) → EvidenceManager
    # EvidenceManager real con directorio temporal aislado

  @pytest.fixture
  def root(tk_root_session)           # alias para compatibilidad
  @pytest.fixture
  def pump_fixture(tk_root_session)   # provee pump() como fixture
```

### Cómo ejecutar los tests

```powershell
# Suite completa (~1091 Linux / ~1104 Windows - 0 failed)
venv\Scripts\python.exe -m pytest tests/ -q --tb=short

# Solo GUI headless (validación estructural)
venv\Scripts\python.exe tests\validate_gui_headless.py

# Solo backend (importaciones y sanidad)
venv\Scripts\python.exe tests\validate_backend.py

# Módulo específico
venv\Scripts\python.exe -m pytest tests/test_vault.py -v

# Suites consolidadas
venv\Scripts\python.exe -m pytest tests/gui/ tests/data/ tests/profile/ -v

# Con cobertura
venv\Scripts\python.exe -m pytest tests/ --cov=. --cov-report=html
```

### Cómo agregar un test nuevo

```
1. Nombre: test_<nombre_del_modulo>.py  (PROMPT.md Regla 10)
   Ejemplo: gui/settings_view.py → tests/test_gui_settings_view.py

2. Si necesita CTk root:
   def test_algo(tk_root_session):
       view = MiVista(tk_root_session, ...)
       pump(view)
       assert ...

3. Si necesita managers:
   def test_algo(mock_project_manager, tmp_path):
       manager = RealManager(str(tmp_path))
       ...

4. Si crea archivos: usar tmp_path (pytest fixture) — nunca rutas fijas

5. Agregar al grupo correcto:
   - Tests de GUI:           tests/gui/test_views_comprehensive.py o nuevo archivo
   - Tests de data managers: tests/data/test_data_handlers.py o nuevo archivo
   - Tests E2E:              tests/integration/test_end_to_end.py
```

---

## 12. Guía de Modificaciones Rápidas

Esta sección responde directamente "¿dónde toco X?" — con archivo y contexto exacto.

---

### GUI — Apariencia general

| Qué cambiar | Dónde | Cómo |
|-------------|-------|------|
| Fuente global | `gui/themes.py:84` | `return ("Roboto", size)` — cambiar "Roboto" por otra fuente |
| Fuente bold | `gui/themes.py:97` | `return ("Roboto", size, "bold")` |
| Esquema color CTk | `gui/themes.py:61` | `return "blue"` → opciones: `blue`, `green`, `dark-blue` |
| Modo oscuro/claro default | `gui/themes.py:25` | `DEFAULT_THEME = ThemeName.DARK` |
| Padding layout | `gui/themes.py:107` | `return 10` — valor en px |
| Padding elementos | `gui/themes.py:117` | `return 5` — valor en px |

### GUI — Sidebar

| Qué cambiar | Dónde | Cómo |
|-------------|-------|------|
| Ancho sidebar | `gui/main_window.py:291` y `:318` | `minsize=222` y `width=222` — mismo valor ambos |
| Color botón nav inactivo | `gui/main_window.py:347` | `fg_color="transparent"` |
| Color botón nav al hover | `gui/main_window.py:348` | `hover_color=("gray70", "gray30")` — light/dark |
| Color botón nav activo | `gui/main_window.py:558` | `active_color = ("gray80", "gray25")` |
| Altura botones sidebar | `gui/main_window.py:344` | `height=40` |
| Color texto botones | `gui/main_window.py:346` | `text_color=("gray10", "gray90")` |
| Título "DSOAgent" sidebar | `gui/main_window.py:322` | `text="DSOAgent"` — o cambiar tamaño: `bold(20)` |

### GUI — Agregar / quitar item de navegación

```
1. gui/main_window.py:35  NAV_ITEMS
   Agregar: ("-NAV-MIMODULO-", "Mi Módulo")

2. gui/main_window.py:52  _NAV_I18N_KEYS
   Agregar: "-NAV-MIMODULO-": "nav.mimodulo"

3. gui/main_window.py:75  _NAV_ICONS
   Agregar: "-NAV-MIMODULO-": "miicono"    ← nombre en gui/icon_manager.py

4. gui/i18n.py:23  STRINGS["es"]
   Agregar: "nav.mimodulo": "Mi Módulo"
   En STRINGS["en"]: "nav.mimodulo": "My Module"

5. gui/main_window.py (en _refresh_current_view)  _nav_map
   Agregar: "-NAV-MIMODULO-": ("mi_view", "MiView")

6. Crear gui/mi_view.py con clase MiView(ctk.CTkFrame)

7. tests/gui/test_views_comprehensive.py
   Actualizar test_nav_items_count: expected = N+1
   Agregar "Mi Módulo" en test_nav_items_labels_default_language
```

### GUI — Status bar

| Qué cambiar | Dónde | Cómo |
|-------------|-------|------|
| Altura status bar | `gui/main_window.py:451` | `height=30` |
| Color "sin proyecto" (light) | `gui/main_window.py:425` | `fg_color=("#fff3cd", "#5a4500")` — primer valor = light |
| Color "sin proyecto" (dark) | `gui/main_window.py:425` | segundo valor en tuple |
| Texto "sin proyecto" | `gui/main_window.py:429` | cambiar `text=` |

### GUI — Colores de widgets Treeview y Text

| Qué cambiar | Dónde | Cómo |
|-------------|-------|------|
| Fondo tabla dark | `gui/themes.py:133` | `"tree_bg": "#2b2b2b"` |
| Texto tabla dark | `gui/themes.py:135` | `"tree_fg": "#dcddde"` |
| Selección tabla dark | `gui/themes.py:138` | `"select_bg": "#1f538d"` |
| Fondo tabla light | `gui/themes.py:143` | `"tree_bg": "#ffffff"` |
| Fondo panel logs dark | `gui/themes.py:164` | `"bg": "#1e1e1e"` |
| Texto panel logs dark | `gui/themes.py:165` | `"fg": "#d4d4d4"` |
| Fondo panel logs light | `gui/themes.py:171` | `"bg": "#ffffff"` |

### GUI — Strings visibles al usuario

```
Ubicación: gui/i18n.py:23  STRINGS["es"]
Buscar la clave y cambiar el valor:
  "nav.projects": "Proyectos"    ← cambiar "Proyectos" por el texto deseado
  "login.title":  "DSOAgent - Inicio de Sesión"
  "settings.tab.providers": "Proveedores"
  ...

Para agregar string nuevo:
  1. Agregar clave en STRINGS["es"] y STRINGS["en"]
  2. Usar en código: from gui.i18n import _t; label = _t("mi.clave")
```

### Data — Agregar campo a Finding

```
1. data/excel_handler.py  @dataclass Finding
   Agregar: mi_campo: str = ""

2. data/excel_handler.py  ExcelHandler.create_template()
   Agregar columna en la fila de headers del xlsx

3. data/excel_handler.py  ExcelHandler._row_to_finding()
   Mapear columna xlsx → campo Finding

4. data/excel_presets.py  presets relevantes
   Agregar "mi_campo" en column_mapping si aplica

5. tests/test_excel_handler.py
   Agregar al Finding de fixture el nuevo campo
```

### Data — Agregar preset de importación Excel

```
1. data/excel_presets.py
   Crear ImportPreset(
       id="mi_herramienta",
       name="Mi Herramienta",
       category="SAST",              # SAST | DAST | SCA | Secret Scanning | Estándar
       required_columns=["Columna1", "Columna2"],
       column_mapping={"Columna1": "title", "Columna2": "description", ...},
       aliases=["nombre_legacy"]      # opcional, para backward compat
   )

2. data/excel_presets.py  PRESETS list
   Agregar el nuevo ImportPreset

3. PROMPT.md  § Conectores Estado Integraciones
   Agregar fila en la tabla de presets

4. docs/ARCHITECTURE.md  §4 ExcelImporter
   Actualizar lista de presets
```

### Integrations — Agregar proveedor nuevo

```
1. integrations/mi_proveedor.py
   class MiProvider(AbstractTicketAPI):
       def connect() → bool: ...
       def create_ticket(tc) → str: ...
       ... (todos los métodos abstractos)

2. data/provider_registry.py  ProviderRegistry.resolve_provider()
   Agregar:
     elif conn.type_id == "builtin_mi":
         return MiProvider(conn.values)

3. data/provider_registry.py  _BUILTIN_TYPES list
   Definir ProviderTypeDef para el nuevo tipo con sus ProviderFieldDef

4. DSOAgent.spec + DSOAgent_linux.spec  hiddenimports
   Agregar: 'integrations.mi_proveedor'

5. PROMPT.md  § Conectores Estado Integraciones
   Agregar fila en la tabla

6. docs/ARCHITECTURE.md  §7 Proveedores builtin
   Actualizar tabla
```

### Seguridad — Cambiar número de iteraciones PBKDF2

```
1. security/vault.py  SecurityVault.derive_key_from_hardware()
   kdf = PBKDF2HMAC(SHA256, length=32, salt=salt, iterations=100000)
   Cambiar 100000 por el valor deseado (>= 100000 recomendado NIST)

2. data/profile_manager.py  ProfileManager
   Misma variable, aplicar el mismo cambio para consistencia
   NOTA: cambiar iterations invalida todos los perfiles existentes
```

### Versión — Sincronizar en todos los archivos

```
Al hacer bump de versión (ej: 0.9.8 → 0.9.9), actualizar TODOS:
  [ ] pyproject.toml:version
  [ ] PROMPT.md:Versión Actual
  [ ] docs/PLANNING.md
  [ ] docs/TECHNICAL.md:Versión
  [ ] docs/ARCHITECTURE.md:Versión (este documento)
  [ ] file_version_info.txt:filevers y prodvers
  [ ] gui/help_view.py: about dialog version string
  [ ] data/config_manager.py:AppSettings.version default
  [ ] gui/themes.py:create_status_bar(version="v0.9.8")
```

### Build — Agregar asset al EXE

```
1. Copiar archivo a assets/
2. DSOAgent.spec + DSOAgent_linux.spec  datas:
   Agregar: ('assets/mi_archivo.ext', 'assets')
3. En código, acceder via:
   from utils.paths import get_assets_dir
   path = get_assets_dir() / "mi_archivo.ext"
   # get_assets_dir() resuelve correctamente tanto en venv como en frozen
```

---

*Fin del documento. Última actualización: Junio 2026 — v0.9.8*

#End Development By Angel Esquivel (CyberSecurity) [DSOAgent 2026]
