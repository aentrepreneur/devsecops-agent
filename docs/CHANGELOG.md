# CHANGELOG

Todas las versiones desde 0.3.0-alpha.0 hasta 1.0.0

Este archivo sigue el formato [Keep a Changelog](https://keepachangelog.com/es-ES/1.0.0/).

## [0.9.8] - 2026-06-03

### Migracion AGENTS.md → PROMPT.md

#### Mejora — Sistema i18n ahora soporta fallback personalizado

- **Capacidad**: Funcion `_t(key, fallback)` permite traduccion con valor por defecto cuando una clave no existe en el diccionario de idioma
- **Alcance**: 72 llamadas en `login_dialog.py`, `export_dialog.py`, `multi_profile_report_dialog.py`
- **Archivo**: `gui/i18n.py` — `_t()` y `LanguageManager.t()`

#### Mejora — Iconos de upload y download agregados

- **Capacidad**: Navegacion visual completa con iconos `upload.png` y `download.png` en barra de herramientas
- **Archivo**: `assets/icons/upload.png`, `assets/icons/download.png`

- `docs/AGENTS.md` eliminado (1100 lineas, ~300 duplicadas con global)
- `PROMPT.md` creado en raiz del proyecto — reglas especificas DSOAgent, renumeradas 1-18
- Reglas duplicadas con `~/.config/opencode/AGENTS.md` eliminadas
- Referencias a `PENDINGS.md`, `AUDIT_FUNCTIONAL.md`, `DUMMIES.md` (fantasmas) eliminadas
- Numeracion corregida: sin duplicados ni huecos
- 40 referencias cruzadas actualizadas en ARCHITECTURE.md, TECHNICAL.md, README.md, manuales
- Nuevo: Regla 14 (backup pre-modificacion archivos criticos)
- Nuevo: Protocolo de Sesion (INICIO → CLASIFICACION → PUERTAS → CIERRE)
- Tabla de conectores actualizada con SharePoint (Graph API OAuth2)
- Nomenclatura actualizada con 6 modulos faltantes (ExportImportManager, MultiProfileReportReader, LoginDialog, ExportDialog, ProfileManager, MultiProfileReportDialog)

### Nuevos Archivos

- `PROMPT.md` (raiz): Reglas del proyecto autocontenido, 26 reglas numeradas, Protocolo de Sesion, Build Watchdog, Formato de Salida, README Estandar
- `QUICKSTART.md` (raiz): Setup desde cero en VS Code, Windows, Linux, Docker
- `docs/LOG_ANALYSIS.md`: Guia completa de logs para analisis cruzado entre ambientes
- `scripts/sync_prompt.py`: Sincronizador de reglas globales (opencode AGENTS.md) -> PROMPT.md
- `docker-compose.yml`: Servicio GUI comentado (Dockerfile.gui + Xvfb)

### Estandares Agregados a PROMPT.md (desde global)

- Type hints obligatorios, 100 chars/linea, 50 lineas/funcion
- Convenciones de nomenclatura (PascalCase, snake_case, UPPER_CASE)
- pathlib.Path obligatorio para BASE_DIR
- datetime.now(timezone.utc) — sin naive datetimes
- asyncio.timeout para operaciones I/O
- Error return pattern estructurado
- No produccion en tests/
- No documentar bugs como mejoras
- Pre-commit checklist
- Output formatting (raw CLI, directo, estructurado)
- README estandar (shields.io, hero, badges)
- Build Watchdog adaptativo (baseline Z-Score, heartbeat, resource check)

### Mejoras de Robustez Pre-RC1

#### Mejora — Paths runtime para despliegue PyInstaller (Windows .exe, Linux binary)

- **Patrón aplicado**: convertir constantes de módulo → funciones evaluadas en runtime
- **Módulos afectados**:
  - `utils/logging_system.py`: `LOG_FILE`/`LOGS_DIR` → `_get_log_file()` — logs persisten en `%APPDATA%\DSOAgent\logs\`
  - `utils/task_scheduler.py`: path hardcoded → `get_user_data_dir()/data/` — tareas persisten correctamente
  - `gui/icon_manager.py`: `_ICONS_DIR` → `_get_icons_dir()` — iconos cargan en .exe
  - `gui/help_view.py`: `_MANUALES_MD_PATH` → `_get_manuales_path()` — manuales accesibles
  - `data/provider_registry.py`: `_REGISTRY_DIR` → `_get_registry_dir()` — **conexiones Azure/Jira/SharePoint/Git persisten en .exe**
- **Impacto**: Todos los paths de datos de usuario ahora resuelven correctamente en frozen executables, no en directorio temporal `_MEIPASS`.

#### Mejora — UX primer arranque Windows (sin ventanas cmd flash)

- **Problema**: Al ejecutar `DSOAgent.exe` sin perfil guardado, aparecían ventanas de consola (`cmd.exe`) que se abrían y cerraban rápidamente — comportamiento que parecía sospechoso/malware.
- **Causa**: `security/vault.py` ejecutaba `wmic baseboard get uuid` y `whoami /groups` via `subprocess.run()` sin ocultar ventana.
- **Solución**: Agregar `startupinfo` con `STARTF_USESHOWWINDOW` + `SW_HIDE` para subprocess.
- **Archivo**: `security/vault.py` método `_get_hardware_identifier()`
- **Impacto**: UX profesional en primer arranque. Sin ventanas sospechosas.

#### Mejora — Docker volúmenes persistentes

- **Cambio**: `VOLUME` actualizado de `/root/.dsoagent` (path frozen incorrecto) a `/app/config`, `/app/logs`, `/app/projects`, `/app/evidence`
- **Archivos**: `Dockerfile`, `docker-compose.yml`
- **Impacto**: Datos persisten correctamente en contenedores Docker.

#### Mejora — Linux binary visibilidad errores

- **Cambio**: `DSOAgent_linux.spec`: `console=False` → `console=True`
- **Impacto**: Errores de arranque visibles en terminal Linux (el binario se lanza desde terminal, no GUI doble-clic como Windows).

#### Mejora — GUI estabilidad

- `gui/settings_view.py`: declarar `on_language_change: Optional[Callable] = None` en `__init__` — evita kwargs explosion en CTkFrame
- `gui/main_window.py`: guard `winfo_exists()` en `_show_settings_view` y `_show_view_by_class` — evita "bad window path name" con widgets destruidos

#### Tests de regresión agregados

- `tests/test_deploy_paths.py`: +17 tests para paths frozen/Docker
- `tests/test_regression_v090.py`: tests para ProviderRegistry str/path

#### Suite

- **1036 passed, 0 failed**

#### Documentación

- **NUEVO**: `docs/GUÍA_RÁPIDA.md` — documentación unificada simple para presentaciones ejecutivas
- **NUEVO**: `docs/PLANNING.md` — documento maestro con cronograma 24 semanas, progreso histórico, planificación futura
- **Consolidación**: DUMMIES.md + PENDINGS.md + PROGRESO.md + RESUMEN_EJECUTIVO_RC1.md + AUDIT_FUNCTIONAL.md → eliminados, contenido fusionado en PLANNING.md y GUÍA_RÁPIDA.md

#### Compilación

- `dist/DSOAgent.exe`: **46 MB** recompilado con PyInstaller 6.20.0 — build exitoso (2026-06-02)
- Suite: **1034 passed, 0 failed** (+18 tests de cobertura de despliegue)

#### Tests de regresion agregados (`tests/test_deploy_paths.py` — NUEVO, +14 tests)

- `test_logs_dir_frozen_win32/linux` — `get_logs_dir()` apunta a `APPDATA/DSOAgent/logs` / `~/.dsoagent/logs` en frozen
- `test_config_dir_frozen_win32`, `test_user_data_dir_frozen_win32/linux` — idem para config y user_data
- `test_get_log_file_frozen_win32_points_to_appdata` — `_get_log_file()` NO apunta a `_MEIPASS` en frozen
- `test_get_log_file_unfrozen_under_project_root` — en dev/Docker apunta bajo `project_root`
- `test_task_scheduler_tasks_file_unfrozen/frozen_win32` — `_tasks_file` cambia de `project_root/data/` a `APPDATA/DSOAgent/data/` en frozen
- `test_icons_dir_unfrozen_contains_assets`, `test_icons_dir_is_function_not_constant` — `_get_icons_dir` es callable (no constante)
- `test_manuales_path_unfrozen_contains_docs`, `test_manuales_path_is_function_not_constant` — idem para `_get_manuales_path`
- `test_logs_dir_docker_unfrozen_under_project_root` — valida path que Docker monta como volumen

`tests/test_gui_settings_view.py::TestSettingsViewKwargs` — +2 tests:
- `test_on_language_change_kwarg_accepted` — instanciar con `on_language_change=` no lanza excepcion
- `test_on_language_change_stored_as_attribute` — el callback queda en `view.on_language_change`

`tests/test_gui_views.py::TestViewCacheGuard` — +2 tests:
- `test_show_view_recovers_from_destroyed_widget` — widget destruido se elimina del cache
- `test_show_view_reuses_alive_widget` — widget vivo pasa el guard `winfo_exists()`

---

## [0.9.8-RC1] - 2026-07-15

### Estabilidad multiplataforma consolidada

#### Suite — 0 incidencias en ambas plataformas

- **Windows**: 1104 passed, 0 failed
- **Linux**: 1091 passed, 0 failed, 13 skipped (tests Windows-only)

#### Mejora — Aislamiento total de tests GUI sin Tcl

- **Capacidad**: `test_main.py` ahora intercepta `from gui.main_window import MainWindow` via `patch.dict("sys.modules")` — el modulo entero se reemplaza sin activar el interprete Tcl ni crear widgets reales
- **Impacto**: 3 tests de `main.py` pasan en Windows y Linux sin dependencias de display
- **Archivos**: `tests/test_main.py`, `tests/conftest.py`, `tests/integration/test_end_to_end.py`

#### Mejora — Tests multiplataforma sin contaminacion de mocks

- **Capacidad**: `tests/test_gui_profile_system.py` aísla mock de pandas con setUp/tearDown — otros modulos no heredan el mock
- **Capacidad**: i18n y GUI integration usan aserciones dinamicas que se adaptan al numero real de items de navegacion
- **Capacidad**: Tests Windows-only (`schtasks`, display name) se skippean automaticamente en Linux via `@pytest.mark.skipif`
- **Archivos**: `tests/test_gui_profile_system.py`, `tests/test_i18n.py`, `tests/test_gui_integration.py`, `tests/test_task_scheduler.py`, `tests/test_regression_v090.py`

#### Mejora — Task Scheduler soporta directorio de trabajo

- **Capacidad**: `utils/task_scheduler.py` inyecta `cd /d "<directorio>"` en el comando `/TR` de schtasks — las tareas programadas ejecutan scripts en el directorio especificado por `start_in`
- **Archivo**: `utils/task_scheduler.py`

#### Documentacion sincronizada con estado real

- **Capacidad**: Todos los conteos de tests, badges y referencias reflejan el estado actual (1091 Linux / 1104 Windows)
- **Archivos actualizados**: `QUICKSTART.md`, `PROMPT.md`, `docs/TESTING.md`, `docs/ARCHITECTURE.md`, `README.md`, `docs/CHANGELOG.md`, `docs/TECHNICAL.md`

---

## [0.9.8-deploy] - 2026-05-31

### Compilación y Despliegue (v0.9.8)

#### Fase 1 — PyInstaller Windows (.exe)

- `file_version_info.txt`: versión `0.9.7` → `0.9.8` en `filevers`, `prodvers`, `FileVersion`, `ProductVersion`
- `data/config_manager.py`: `AppSettings.version` → `"0.9.8"`
- `gui/themes.py`: `create_status_bar()` default `version` → `"v0.9.8"`
- `tests/validate_backend.py`: banner log → `DSOAgent v0.9.8`
- `DSOAgent.spec`: icon corregido de `.png` → `icons8-gear-64.ico` (Windows requiere `.ico`)
- `dist/DSOAgent.exe`: **44 MB** generado con PyInstaller 6.20.0 — build exitoso

#### Fase 2 — Docker

- `.dockerignore`: ya existía y correcto — `venv/`, `tests/`, `logs/`, `dist/` excluidos
- `requirements.headless.txt`: `tzdata>=2024.1` agregado — necesario para `zoneinfo` en contenedores Linux
- `.env.template`: **NUEVO** — documenta `TIMEZONE` y `LOG_LEVEL` para `docker compose`

#### Fase 3 — Linux venv

- `requirements.txt`: comentario pystray corregido (`gir1.2-appindicator3-0.1` → `gir1.2-ayatana-appindicator3-0.1` — nombre correcto Ubuntu 22.04+)
- `setup_linux.sh`: footer sincronizado al formato estándar; aviso `DISPLAY` agregado al finalizar (detecta servidor sin display)
- `DSOAgent_linux.spec`: header/footer sincronizados al formato estándar; `('docs', 'docs')` agregado a `datas` (consistencia con spec Windows)
- `docs/DUMMIES.md`: sección **C5 — Producción Linux (venv directo)** agregada; tabla de verificación ampliada a 5 plataformas (C1-C5)

---

## [0.9.8-fase-ab] - 2026-05-30

### Tests — Fase A: Importación y Evidencias (v0.9.8)

#### Fixtures dummy completos para los 15 presets Excel

- `tests/fixtures/generate_dummy.py` — 12 generadores nuevos añadidos (sast_app, dast_web, dast_app, burp_suite, nessus, owasp_zap, openvas, qualys, checkmarx, veracode, sonarqube, pentest_custom). Total: 15 generadores + 15 archivos `.xlsx` en `tests/fixtures/`.
- `_write_simple_xlsx()`: fórmula `startrow = max(0, start_row - 2)` para alinear con `header=start_row-2` de `excel_importer.py` — permite que los presets SAST/DAST legacy lean headers desde la fila correcta.
- `tests/fixtures/generate_evidence_dummies.py` — NUEVO: 5 generadores de evidencia (PNG binario, PDF mínimo, script Python, JSON config, log multi-línea).

#### test_import_all_formats.py — NUEVO (28 tests)

- `TestImportAllFormats`: 15 tests — un test por preset, verifica `errors=0`, `imported>=1`, `preset_used` correcto.
- `TestSeverityMaps`: 5 tests — normalización de severidad para cada preset con valores variados.
- `TestImportErrorCases`: 5 tests — archivo corrupto, columnas faltantes, hoja incorrecta, sin preset detectable.
- `TestImportLegacyAliases`: 3 tests — alias `legacy_sast_code`, `legacy_sast_app`, `legacy_dast_web` backward-compat.

#### test_evidence_all_types.py — NUEVO (14 tests)

- `TestEvidenceAddAllTypes`: 5 tests — agrega PNG, PDF, Python, JSON, log via `EvidenceManager`.
- `TestEvidenceHashIntegrity`: 3 tests — hash SHA-256 consistente, cambia con contenido diferente.
- `TestEvidenceDuplicates`: 3 tests — no duplica por hash, permite mismo nombre diferente contenido.
- `TestEvidenceIsolation`: 3 tests — proyectos distintos tienen evidencias aisladas.

### Tests — Fase B: GUI Funcionales (v0.9.8)

#### test_gui_scripts_view_actions.py — NUEVO (25 tests)

- `TestScriptsViewInstantiation`: 7 tests — widget, tree, entries, combobox, sandbox controls, console, buttons.
- `TestScriptsViewFormActions`: 7 tests — clear_form, editing state, save sin proyecto, save sin nombre, save con datos válidos, combobox, project_ids asociado.
- `TestScriptsViewSandboxActions`: 7 tests — run sin selección, btn_run estado, consola disabled/clear/append, set_result ok/fail, ejecución con mock runner.
- `TestScriptsViewProjectIntegrationGUI`: 4 tests — set_project label, set_project None, carga scripts en tree.

#### TestFindingsViewActions en test_gui_views.py (+4 tests)

- `test_severity_combo_filters_tree` — severity_combo dispara `_apply_filter` y llama `list_findings`.
- `test_search_var_filters_by_title` — `_search_var` filtra por título (SQL vs XSS, 2→1 item).
- `test_status_combo_triggers_apply_filter` — status_combo dispara filtro.
- `test_load_data_calls_excel_handler` — `load_data()` delega al handler.

#### Consolidación test_scripts_view.py

- `tests/test_scripts_view.py` **eliminado** — todos sus tests estaban duplicados en `test_gui_scripts_view_actions.py`. El test único `test_save_and_load_script` (verificación `project_ids`) fue absorbido como `test_save_associates_project_id` en `_actions.py`.

### Limpieza (v0.9.8)

- `pyproject.toml`: versión `0.9.7` → `0.9.8`; `pytest-cov>=4.1.0` agregado a `[dev]`
- `requirements.txt`: `pytest-cov>=4.1.0` agregado en sección dev
- `gui/main_window.py`: docstring `_toggle_language` limpiado (eliminado tag `[DEPRECATED]`, redactado claro)
- `utils/health.py`: docstrings de stubs `check_database`/`check_llm` actualizados para indicar que son placeholders para v1.0 REST API
- `tests/validate_gui_headless.py`: carácter Unicode `→` reemplazado por `->` en log de test 13 — fix `charmap` encoding en Windows stdout

#### Suite final

- `pytest tests/ -q`: **1016 passed, 0 failed** (neto: +70 nuevos, -12 consolidados de test_scripts_view.py)
- `validate_backend.py`: **12/12 PASS**
- `validate_gui_headless.py`: **14/14 PASS**

---

## [0.9.8-audit] - 2026-05-30

### Auditoría Funcional + Cobertura + GUI Headless (v0.9.8)

#### Cobertura — pytest-cov instalado y configurado

- `.coveragerc` creado: excluye `venv/`, `tests/`, `build/`, código de entrada.
- Cobertura total baseline: **66%** (9023 stmts). Módulos core todos medidos.
- Módulos corregidos esta sesión:

| Módulo | Antes | Después |
|--------|-------|---------|
| `utils/paths.py` | 37% | 79% (+42%) |
| `utils/user_id.py` | 56% | 97% (+41%) |
| `utils/logging_system.py` | 67% | 87% (+20%) |
| `data/excel_importer.py` | 53% | 62%+ |

#### Tests nuevos

- `tests/test_utils_extended.py` — **NUEVO** (35 tests, 3 clases):
  - `TestPathsExtended`: 12 tests — `get_data_dir`, `get_assets_dir`, `get_docs_dir`, `get_scripts_dir`, `get_user_data_dir`, `get_projects_dir`, `get_evidence_dir`, `get_default_browse_dir`, `resolve_path`, simulación frozen Win32/Linux.
  - `TestUserIdExtended`: 12 tests — `get_machine_id`, `get_user_uuid` determinismo, `get_system_info` claves+display_name Win, `generate_project_id` formato/padding, `parse_project_id` válido/inválido.
  - `TestLoggingSystemExtended`: 11 tests — `get_log_path`, `read_logs` filtros módulo/nivel/líneas, `get_log_summary` estructura, `clear_logs`, `setup_logging` idempotente, cache loggers.
- `tests/test_excel_importer.py` — `TestImportFileEdgeCases` **+6 tests**:
  - `import_file` con ruta inexistente, sin preset detectable, preset inválido.
  - `detect_format` con archivo desconocido y con archivo `dsoagent_standard`.
  - `ImportResult` estado inicial.

#### validate_gui_headless.py — 9 → 14 tests

| Test | Descripción |
|------|-------------|
| Test 10 | Regresión: `_refresh_current_view` definido exactamente 1 vez |
| Test 11 | `_on_tray_view_logs` apunta a `-NAV-HELP-` (sin huérfano `-NAV-LOGS-`) |
| Test 12 | Labels de navegación en español (7/7 claves verificadas) |
| Test 13 | `nav_map` de `_refresh_current_view` cubre todos los `NAV_ITEMS` |
| Test 14 | Las 10 clases de vista son exportadas y llamables |

#### Suite final

- `pytest tests/ -q`: **958 passed, 0 failed, 2 warnings** (warnings: Tkinter thread pre-existente)
- `validate_backend.py`: **12/12 PASS**
- `validate_gui_headless.py`: **14/14 PASS**

---

## [0.9.8-fix] - 2026-05-29

### Estabilidad — Restauración y consolidación de la suite de tests

- Suite restaurada a 916 passed con aislamiento completo entre modulos de tests.
- `gui/main_window.py`: navegación i18n con 13 items traducidos via `_NAV_I18N_KEYS`.
- `gui/help_view.py`: versión expuesta como atributo de instancia para acceso directo.
- Fixtures de tests consolidados: todos usan `tk_root_session` compartido, eliminando interferencia entre módulos.
- Ventana principal dimensionada a 1200×730.

### Suite tras consolidación

- `pytest tests/ -q`: **916 passed, 1 skipped, 0 failed, 0 errors**
- `validate_gui_headless.py`: **9/9 PASS**
- `validate_backend.py`: **12/12 PASS**

---

## [0.9.8] - 2026-05-26

### Feature — SharePoint Integration Completa (Graph API OAuth2)

- `integrations/sharepoint_provider.py`: nuevo `SharePointProvider` — autenticación OAuth2 client credentials flow (Azure AD App Registration), operaciones completas via Microsoft Graph API usando `aiohttp` (sin dependencias nuevas):
  - `connect()` / `disconnect()` / `is_connected()` — gestión de sesión con token cache
  - `test_connection()` — GET al site de SharePoint
  - `upload_file(local_path, remote_name?, folder?)` — PUT `/drives/{id}/root:/{path}:/content`
  - `download_file(remote_name, dest_path, folder?)` — descarga binaria con escritura en disco
  - `list_files(folder?)` — GET children con filtro por `file` mimeType
  - `create_folder(folder_path)` — POST children, idempotente (409 = ya existe → success)
  - Token refresh automático (cache con margen de 60s antes del vencimiento)
  - Sin credenciales en logs (AGENTS.md R2/R3)
- `data/provider_registry.py`: factory `_sharepoint(conn)` agregada a `_get_factories()` — `builtin_sharepoint` ahora resuelve a instancia real en vez de `None`.

### Feature — GUI EvidenceView Subida a SharePoint

- `gui/evidence_view.py`: botón **"☁ Subir a SharePoint"** en barra de acciones:
  - `__init__` acepta `provider_registry=None` (kwarg retrocompatible)
  - `_upload_to_sharepoint()`: detecta conexiones `builtin_sharepoint` activas del proyecto, muestra selector si hay múltiples, sube archivos seleccionados en thread daemon con `asyncio.run()`, actualiza `lbl_stats` via `self.after()`
- `gui/main_window.py`: `EvidenceView` se instancia con `provider_registry=self.provider_registry`.

### Tests

- `tests/test_sharepoint_provider.py`: 22 tests (4 clases) — headless, sin SharePoint real, `AsyncMock`:
  - `TestSharePointConfig` (4): config válida, campos faltantes, defaults, folder vacío
  - `TestSharePointProviderConnect` (5): connect() exitoso, token error 401, token expirado, token válido, test_connection()
  - `TestSharePointProviderOperations` (10): upload success/not_found/HTTP403, download success/not_found, list_files items/vacío/404, create_folder nuevo/existente
  - `TestSharePointProviderRegistry` (3): resolve_provider() retorna instancia, campos incompletos → None, factory key presente
- `tests/test_gui_evidence_sharepoint.py`: 11 tests (3 clases) — headless Tk:
  - `TestEvidenceViewSharePointInit` (3): acepta registry kwarg, default None, botón presente
  - `TestEvidenceViewSharePointUpload` (7): sin selección, sin proyecto, sin registry, sin conexión, con mock provider, múltiples archivos, stats bar
  - `TestMainWindowInjectsRegistry` (1): instanciación con registry

### Fixtures / Dummies

- `tests/fixtures/generate_sharepoint_dummy.py`: generador de datos simulando Microsoft Graph API:
  - `generate_dummy_token_response()` → dict OAuth2
  - `generate_dummy_site_response()` → dict GET /sites/{host}:{path}
  - `generate_dummy_drive_response()` → dict drives list
  - `generate_dummy_files_list(n)` → dict GET /drive/items/{id}/children
  - `generate_dummy_upload_response(name, size)` → dict PUT /content
  - `generate_dummy_evidence_files(output_dir, n)` → list[Path] archivos reales
- `tests/data_dummy/evidence_sample.txt`, `evidence_log.log`, `evidence_screenshot.png` — archivos de evidencia reutilizables en tests.

### Fix — validate_gui_headless.py

- `tests/validate_gui_headless.py`: Test 5 (Items de navegación) usaba `MainWindow.NAV_ITEMS` (acceso de clase a una instancia `property` → `property` object sin `len()`). Corregido a `MainWindow.get_nav_items()` (classmethod). Resultado: **9/9 PASS** (era 8/9).

### Cleanup Pre-RC1 — Docs y Presets

- `data/excel_presets.py`: presets renombrados de `legacy_*` a nombres descriptivos funcionales:
  - `legacy_sast_code` → `sast_static_code` ("SAST - Análisis Estático de Código")
  - `legacy_sast_app` → `sast_static_app` ("SAST - Análisis Estático de Aplicaciones")
  - `legacy_dast_web` → `dast_dynamic_web` ("DAST - Análisis Dinámico Web")
  - `legacy_dast_app` → `dast_dynamic_app` ("DAST - Análisis Dinámico de Aplicaciones")
  - Aliases `_LEGACY_ALIASES` para backward compatibility — `get_preset("legacy_sast_code")` sigue funcionando
  - `list_presets_by_category()` actualizado: categorías "SAST" y "DAST" consolidan presets propios + herramientas externas; categoría "Legacy" eliminada
  - `detect_preset_by_headers()`: actualizado a retornar `sast_static_code` en detección automática
- `data/excel_importer.py`: `_import_legacy_multisheet()` actualizado a nuevos nombres de preset
- `tests/test_excel_importer.py`: 43 tests (era 42) — actualizado a nuevos nombres + test nuevo `test_preset_legacy_alias_backward_compat`
- `docs/MIGRATION_GUIDE.md`: **eliminado** (migración desde reviews/ completada; no aplica a nuevos usuarios)
- `docs/TROUBLESHOOTING_IMPORT.md`: **eliminado** — contenido absorbido en `docs/manuales/manual-tecnico.md` §8.5 (Importación Excel — Errores Comunes)
- `docs/AGENTS.md`: versión 0.9.7 → 0.9.8, tabla de docs actualizada, tabla presets actualizada (15 formatos), sección **Protocolo Pre-RC1** agregada (pasos 1-9 bloqueantes, fallos pre-existentes documentados, regla de promoción)
- `docs/PENDINGS.md`: criterios RC1 reestructurados en 3 pasos ordenados (tests auto → E2E real → compilación), validación SharePoint agregada como bloqueante
- `docs/DUMMIES.md`: **Sección D** nueva — Guía Técnica de Tests (D1 arquitectura suite, D2 pyimage explicado, D3 AsyncMock patrón, D4 fixtures/aislamiento, D5 tiempos y comandos, D6 interpretación de fallos, D7 guía agregar nuevo test)
- `logs/dsoagent.log`: truncado para baseline limpio Pre-RC1
- `docs/manuales/*.md`: headers actualizados a formato canónico AGENTS.md, versión 0.9.8, footers agregados
- `gui/help_view.py`: sección manuales legacy PDF (reviews/) eliminada; solo quedan manuales Markdown (docs/)
- `gui/scripts_view.py`, `integrations/connection_templates.py`: headers normalizados a formato canónico, referencias legacy removidas de docstrings
- `README.md`: versión 0.9.8, badge tests 916

### Suite

882 + 33 + 1 (alias compat) = **916 passed**, 0 failed nuevos.

### Version

Doc actualizada: 0.9.7 → 0.9.8

---

## [0.9.7] - 2026-05-18

### Seguridad — Tests y Validación

- `tests/test_integration_vault_advanced.py`: `test_recover_with_key_success` + `test_recover_restores_full_vault` — flujo recovery completo con simulación de pérdida de `vault.key`.
- `tests/test_project_manager.py`: clase `TestProjectManagerWithVault` (4 tests) — verifica que archivos `.json` están cifrados en disco, índice JSON plano, integridad tras recarga.
- `tests/test_security_startup.py`: nuevo archivo con 6 clases de escenario (24 tests) — zero-config, multi-proyecto cifrado, multi-proveedor, campos extra, recovery round-trip, configuración compleja.

### Validación Scripts

- `tests/validate_backend.py`: Tests 10-12 añadidos — cifrado PM, vault recovery round-trip, ConfigManager zero-config.
- `tests/validate_gui_headless.py`: Test 9 añadido — datos persistidos reflejados en managers tras reload.

### Documentación

- `docs/DUMMIES.md`: sección A0 (arranque zero-config), A11 (escenarios completos), A7 ampliada con recovery round-trip, nota sobre dos directorios de config, tablas de escenarios y mapa de tests.
- `docs/TECHNICAL.md`: flujo detallado de inicialización, tabla de escenarios de fallo/recuperación, rutas de config por entorno (venv/exe/Docker).
- `security/vault.py`: docstring `__init__` explica rutas dev/exe/Docker y cuándo usar `config_dir` en tests.
- `data/config_manager.py`: docstring `ConfigManager` diferencia JSON plano vs `security.vault.ConfigManager` cifrado.

### GUI — Tests Dialogs ProviderRegistry

- `tests/test_gui_provider_dialogs.py`: nuevo archivo con 4 clases (22 tests) — cobertura completa de los dialogs del tab Proveedores:
  - `TestProviderTypeDialogLogic`: 7 tests headless — CRUD tipos y campos vía `ProviderRegistry` directo.
  - `TestNewTypeDialog`: 6 tests Tk real — apertura `CTkToplevel`, creación de tipo, validación nombre vacío y duplicado, refresco de combo.
  - `TestEditFieldsDialog`: 6 tests Tk real — CRUD campos vía UI, apertura `CTkToplevel`, refresco de formulario dinámico.
  - `TestDeleteTypeDialog`: 3 tests Tk real — builtin bloqueado, tipo custom eliminado, aviso de conexiones huérfanas.

### Reglas de Desarrollo

- `docs/AGENTS.md`: Regla 18 — "Documentación Refleja Estado Real": toda función completada debe marcarse en `PENDINGS.md` y `CHANGELOG.md` en la misma iteración, sin dejar entradas stale.
- `docs/PENDINGS.md`: dialogs GUI ProviderRegistry movidos de "pendiente v1.0" a completados. Pendientes v1.0 reales: mapeos dinámicos y REST API.

### Seguridad — Fix Sandbox Multiprocess (2026-05-19)

- `sandbox/multiprocess_runner.py`: eliminados `open` e `input` de `safe_globals["__builtins__"]` — bypass de aislamiento que permitía lectura/escritura de archivos desde scripts en multiprocess sandbox.
- `tests/test_sandbox.py`: +2 tests de regresión (`test_open_not_available_in_sandbox`, `test_input_not_available_in_sandbox`).

### Suite

883 passed, 0 failed.

### Version

Sincronizada en toda la aplicación: 0.9.7

---

## [0.9.6-rc1] - 2026-05-15

### Internacionalización (i18n)

- Nuevo módulo `gui/i18n.py`: `LanguageManager` con diccionarios ES/EN (60+ claves).
- `gui/main_window.py`: navegación dinámica via `_NAV_KEYS` + `get_nav_items()`.
- `gui/settings_view.py`: `CTkOptionMenu` para cambio de idioma en tab Información.
- Idioma persiste en `AppSettings.language` via `ConfigManager`.
- `tests/test_i18n.py`: 21 tests (LanguageManager, strings, nav items).
- Ortografía española corregida en todos los strings de UI (tildes y casing).

### Documentación Sandbox

- `docs/manuales/manual-usuario.md`: sección 8 "Scripts y Sandbox (Experimental)" — flujo GUI, ejemplos, restricciones, FAQ.
- `docs/manuales/manual-tecnico.md`: sección 9 "Sandbox — Arquitectura Técnica" — diagrama de capas, BLOCKED_PATTERNS, ciclo de vida ExecutionResult.
- `docs/manuales/manual-desarrollo.md`: sección 8 "Agregar y Testear Scripts" — ScriptContract API, ExecutionContext, 4 ejemplos pytest, convenciones.

### Guía de pruebas de campo

- `docs/DUMMIES.md`: documento nuevo con Parte A (mock/local, 10 secciones por módulo) y Parte B (E2E real con placeholders para Azure, Jira, Git).

### Cleanup y calidad

- `reviews/` eliminado completamente (legacy).
- `evidencias/` → `evidence/` normalizado en backend, config, .gitignore, .dockerignore.
- `tests/test_bi_reporter.py` renombrado a `tests/test_report_manager.py` (nomenclatura correcta).
- `data/__pycache__/bi_reporter.cpython-310.pyc` eliminado (stale).
- Docstrings de módulos backend migrados de español a inglés.
- Tests thin/duplicados consolidados.
- Headers normalizados a formato canónico en 37 archivos.
- Suite: **829 passed, 0 failed**.

### PyInstaller fix (2026-05-18)

- `DSOAgent.spec` + `DSOAgent_linux.spec`: agregados `data.excel_importer`, `data.excel_presets`, `gui.import_wizard` a hiddenimports (bug bloqueante — el .exe crasheaba al importar Excel).
- `README.md`: versión actualizada v0.9.5 → v0.9.6, badge 752 → 829 tests, tabla docs completa.
- `docs/MIGRATION_GUIDE.md`: referencia rota corregida, test count actualizado.
- `docs/manuales/manual-usuario.md`: test count 824 → 829.
- `docs/DUMMIES.md`: índice completo de 40 archivos de test agrupados por módulo.
- `docs/AGENTS.md`: `TROUBLESHOOTING_IMPORT.md` añadido a tabla de documentación.

### Refactor Orchestrator — ProviderRegistry (2026-05-18)

- `agent/orchestrator.py`: argumento `ticket_provider` reemplazado por `provider_registry` (inyectable). Eliminados `_internal_ticket_service`, `_provider_factories` e `importlib`. Añadido `_active_provider` como referencia al proveedor activo.
- `ticket_service` property resuelve ahora via `ProviderRegistry.resolve_provider()` — sin proxy ni rama dual.
- `_process_finding` unificado: siempre usa `TicketCreate` dataclass; la rama `if ticket_provider / else internal` eliminada.
- `connect_ticket_provider`: crea o reutiliza `ProviderConnection` en el registry y resuelve instancia concreta.
- `disconnect_all`: desconecta `_active_provider` y limpia la referencia.
- `register_provider_factory()` eliminado (el registry es la fuente de verdad).
- Tests actualizados: `test_orchestrator.py` (+4 tests nuevos), `test_orchestrator_e2e.py`, `test_integration_pipeline.py`.
- Suite: **831 passed, 0 failed**.

### AGENTS.md

- Reglas 15–17 añadidas: ortografía española, nomenclatura test=módulo, pytest antes de cada iteración.
- `gui/i18n.py` / `LanguageManager` añadidos al árbol de nomenclatura.
- Checklist de auditoría actualizada (831 passed, Mayo 2026).
- `DUMMIES.md` añadido a la tabla de referencias de documentación.

---

## [0.9.6] - 2026-05-12

### Excel Importer - Importación flexible de hallazgos

**Nuevo módulo `data/excel_importer.py`**:
- Importador flexible de reportes Excel con mapeo dinámico de columnas.
- Soporte para múltiples formatos: DSOAgent Standard, Legacy español multi-hoja,
  Burp Suite, Nessus, OWASP ZAP, Semgrep, SonarQube.
- Detección automática de formato por headers o nombres de hojas.
- Normalización de severidades (Crítico/Alto/Medio/Bajo → Critical/High/Medium/Low).
- Normalización de estados (Nuevo/Abierto/En Progreso → New/Open/In Progress).
- Preview de archivos con análisis de estructura antes de importar.
- Procesamiento de evidencias vinculadas a hallazgos.

**Nuevo módulo `data/excel_presets.py`**:
- Presets de importación con mapeos de columnas predefinidos.
- **12 presets implementados**:
  - DSOAgent Standard
  - Legacy español: SAST Código, SAST App, DAST Web, DAST App
  - DAST: Burp Suite, Nessus, OWASP ZAP
  - SAST: SonarQube, Semgrep, Checkmarx, Veracode
  - GitHub: CodeQL, Secret Scanning
  - SCA: Snyk
- Mapeos de severidad y estado para cada formato.
- Detección automática por headers para todos los presets.
- Funciones para listar presets por categoría (SAST, DAST, SCA, etc.).

**Core - Persistencia implementada**:
- `_finding_exists()`: Verificación de duplicados contra Excel existente.
- `_save_finding()`: Guardado persistente en Excel del proyecto.
- `_process_evidence()`: Copia de evidencias a carpeta del proyecto.
- Integración con EvidenceManager para organización automática.

**Tests**:
- `tests/test_excel_importer.py`: 42 tests unitarios e integración.
- Cobertura: detección de formatos, mapeo de columnas, normalización,
  importación con preset, importación legacy multi-hoja, preview, persistencia.

**Documentación**:
- `docs/AGENTS.md`: Reglas de trazabilidad actualizadas con formato de headers.
- **3 manuales nuevos en Markdown** (`docs/manuales/`):
  - `manual-usuario.md`: Guía completa para usuarios finales.
  - `manual-tecnico.md`: Guía de instalación, configuración y troubleshooting.
  - `manual-desarrollo.md`: Guía para contribuidores y desarrolladores.
- **Help View actualizado**: Sección de manuales Markdown + legacy PDF.
- Headers de desarrollo actualizados en archivos Python.

### Version

Sincronizada en toda la aplicación: 0.9.6

---

## [0.9.5] - 2026-05-10

### Pre-Producción - Última versión antes de RC1.0.1

**Preparación para Release Candidate 1.0.1**:
- Roadmap documentado: v0.9.5 → RC1.0.1 → 1.0.0
- Todos los features planeados para v1.0 completados.

### GUI - Mejoras UX

**Barra de progreso** (`gui/main_window.py`):
- `CTkProgressBar` oculta por defecto.
- Solo visible durante operaciones en segundo plano.
- Elimina "inicio azul" permanente en la barra de estado.

### Documentación

**Actualizaciones**:
- `docs/TECHNICAL.md`: Estructura actualizada (LogsView integrado en Settings).
- `docs/AGENTS.md`: Roadmap v0.9.5 → RC1.0.1 → 1.0.0 documentado.

### Seguridad

**Dockerfile**:
- Imagen base con digest SHA256 fijado (`python:3.12-slim@sha256:2b0079...`).
- Previene supply chain attacks por tag mutables.

### Version

Sincronizada en toda la aplicación: 0.9.5

---

## [0.9.1] - 2026-05-05

### Rediseño - Evidencias centradas en proyecto

**EvidenceView rediseñada** (`gui/evidence_view.py`):
- Vista ahora centrada en `project.evidence_path` en vez de carpeta global.
- Explorador de archivos con Treeview (nombre, carpeta, tipo, tamaño, fecha).
- Filtro por subcarpeta (categoría) con detección dinámica de subdirectorios.
- Búsqueda por nombre de archivo en tiempo real.
- Acciones: subir archivos, abrir carpeta en explorador, crear subcarpetas, eliminar.
- Barra de estadísticas: conteo de archivos y tamaño total.
- `set_project(project)` crea el directorio de evidencias si no existe.
- `refresh()` como alias público de `_load_files()`.

**EvidenceManager per-project** (`data/evidence_manager.py`):
- Nuevo parámetro `project_path` en `__init__` que sobreescribe `base_path`.
- Nuevo class method `EvidenceManager.for_project(project)`: crea instancia vinculada
  a `project.evidence_path` o genera ruta por defecto `projects/<id>/evidence/`.

**Bug fix**: `_load_files` tenía código muerto en branch `else` (variable `root_files`
asignada pero nunca usada, `scan_dirs = []` redundante). Limpiado.

### Eliminado - Sección "Escaneos" del sidebar

- `-NAV-SCANS-` eliminado de `NAV_ITEMS`, `_NAV_ICONS`, `title_map` y routing en `main_window.py`.
- `findings_view` removido del `view_map` y su instanciación especial.
- Sidebar ahora tiene **11 items** (antes 12).

### Integrado - Hallazgos accesibles desde ProjectView

**ProjectView** (`gui/project_view.py`):
- Acepta `excel_handler` como parámetro del constructor.
- Nuevo botón "Ver Hallazgos" en la barra de acciones del proyecto.
- `_open_findings()`: abre `FindingsView` en ventana `CTkToplevel` modal
  para el proyecto seleccionado, cargando su Excel de hallazgos.

**MainWindow** (`gui/main_window.py`):
- Pasa `excel_handler` a `ProjectView` durante instanciación.

### Tests

- **666 passed** (era 652 en v0.9.0, +14 tests nuevos):
  - `test_gui_views.py`: +11 tests `TestEvidenceViewRedesigned` (instanciación, set_project,
    carga archivos, filtro subcarpeta, búsqueda, columnas treeview, refresh, stats).
  - `test_gui_views.py`: +3 tests `TestEvidenceManagerForProject` (for_project con/sin
    evidence_path, project_path override).
  - `test_gui_integration.py`: NAV_ITEMS count actualizado 12→11, labels sin "Escaneos".

## [0.9.0] - 2026-05-04

### Corregido - Bugs detectados en revision general

- **TclError theme toggle**: `SchedulerView` entry focus crash al cambiar tema.
  Fix: `window.focus_set()` antes de destruir vistas cacheadas (`main_window.py`).
- **Azure/Jira session NoneType**: `connect()` dejaba `_session=None` sin event loop.
  Fix: lazy init via `_ensure_session()` en `_get/_post/_patch/_put/_delete`.
- **Scheduler `/RI` misuse**: `start_in` (directorio) se pasaba como intervalo de minutos.
  Fix: wrap con `cd /d <dir> && script` en `task_scheduler.py`.
- **ProviderRegistry str path**: `__init__` no aceptaba `str` como `registry_path`.
  Fix: coerce a `Path` si es `str`.
- **TicketsView connect sin provider**: error silencioso al conectar sin conexion configurada.
  Fix: auto-preload + mensaje claro si no hay conexion en `tickets_view.py`.
- **Scheduler multi-fire**: boton "Programar Tarea" se deshabilitaba durante ejecucion.
- **13 tests de regresion** agregados en `test_regression_v090.py`.

### Eliminado - ProfileManager y referencias hardcoded Azure/Jira

**ProfileManager eliminado** (`data/profile_manager.py`):
- Archivo completamente eliminado del proyecto.
- Hiddenimport removido de `DSOAgent.spec` y `DSOAgent_linux.spec`.

**settings_view.py refactorizado**:
- Tabs Azure/Jira/Mapeos estaticos eliminados. Solo 3 tabs: Info, Directorios, Proveedores.
- Metodos legacy eliminados: `_test_azure_connection`, `_test_jira_connection`, `_on_save_mappings`,
  `_save_provider_conn`, `_load_mappings_from_conn`, `_find_conn_for_project`.
- `_conn_test` dinamizado via `_CONN_TEST_FACTORIES` (factory pattern extensible).
- `_on_save_project` simplificado: solo `ProjectManager.update_project`.

**tickets_view.py dinamizado**:
- `_PROVIDER_LABELS` reemplazado por `_build_provider_labels()` (lee de ProviderRegistry).
- `_try_preload_credentials` reescrito con `_PRELOAD_FACTORIES` (factory pattern).
- Hints y mensajes de error sin referencias hardcoded a Azure/Jira.

**project_view.py dinamizado**:
- ComboBox proveedor generado desde `ProviderRegistry` (category='tickets').
- Acepta parametro `provider_registry`, inyectado desde `main_window.py`.

**dialogs.py generalizado**:
- Secciones "Tickets Azure DevOps" y "Tickets Jira" fusionadas en "Proveedores de Tickets".
- Referencias a configuracion actualizadas a "Proveedores" tab.

**Tests**: 652 -> 638 passing (3 tests ProfileManager eliminados, tests adaptados):
- `test_gui_settings_view.py`: `proj_mgr` reemplaza `profile_mgr`, tests tabs Azure/Jira removidos.
- `test_gui_integration.py`: `TestSettingsViewTabs` actualizado a 3 tabs.
- `test_integration_vault_advanced.py`: `TestProfileManagerEncryptionCycle` eliminado.

## [0.8.0] - 2026-05-03

### Agregado - Sistema de Proveedores Dinamicos

**Fase 0: ProjectManager unificado** (`data/project_manager.py`):
- Modelo `Project` con ID tipo CASE-YYYY-NNN, `DirectoriesConfig`, timestamps ISO.
- `threading.Lock` para generacion segura de IDs concurrentes.
- Backward compat con slug IDs legacy.

**Fase 1: ProviderRegistry dinamico** (`data/provider_registry.py`):
- Modelos: `ProviderFieldDef`, `ProviderTypeDef`, `ProviderConnection`.
- 6 tipos builtin: Azure DevOps, Jira Cloud, Git Repos, Script Custom, SharePoint, SAST/DAST.
- CRUD completo de tipos custom, campos, y conexiones.
- Asociacion conexion-proyecto N:N con `list_connections(project_id, category)`.
- `duplicate_connection()`, `toggle_connection_active()`.
- Import/Export: `export_connections()` y `import_connections()` con soporte overwrite.
- Persistencia en `config/providers/` (types.json + connections.json).
- Backward compat: `ProviderEntry` legacy mantenido.

**Fase 2: GUI Settings tab Proveedores** (`gui/settings_view.py`):
- Nuevo tab "Proveedores" con Treeview de conexiones y formulario dinamico.
- Combo selector de tipos con campos generados desde `ProviderFieldDef`.
- CRUD conexiones desde GUI: crear, editar, duplicar, eliminar.
- Boton "Probar Conexion" para Azure y Jira.

**Fase 3: Vistas consumidoras adaptadas**:
- `gui/tickets_view.py`: `_try_preload_credentials` lee de `ProviderRegistry` (con fallback).
- `gui/scripts_view.py`: CRUD migrado a `ProviderConnection` tipo `builtin_scripts`.
- `gui/main_window.py`: inyecta `ProviderRegistry` a TicketsView y ScriptsView.

**Tests**: 572 -> 652 passing (+80 tests nuevos):
- +42 tests `test_provider_registry.py` (CRUD tipos/conexiones/campos + import/export)
- +18 tests `test_gui_provider_management.py` (sistema dinamico)
- +14 tests `test_gui_settings_view.py` (tab Proveedores)
- +5 tests `test_gui_tickets_view.py` (precarga ProviderRegistry)
- +3 tests `test_gui_integration.py` (tab_providers)

## [0.7.1] - 2026-04-27

### Corregido - Paths, File Dialogs, Project Toggle, PyInstaller

**FIX-01: Paths condicionales dev vs exe** (`utils/paths.py`):
- `get_logs_dir()` y `get_config_dir()` ahora usan rutas locales del proyecto en dev (venv)
  y `%APPDATA%/DSOAgent/` (Win) o `~/.dsoagent/` (Linux) solo cuando esta empaquetado como .exe.
- Nuevas funciones: `get_user_data_dir()`, `get_projects_dir()`, `get_evidence_dir()`.
- `_init_managers()` en `main_window.py` actualizado para usar `get_projects_dir()` y `get_evidence_dir()`.

**FIX-02: File dialogs al directorio del usuario** (`utils/paths.py` + 8 vistas GUI):
- Nueva funcion `get_default_browse_dir()`: retorna Documentos > Descargas > Home del usuario.
- 15 dialogos `filedialog` en 8 vistas actualizados con `initialdir` al directorio del usuario.
- Vistas actualizadas: `project_view`, `findings_view`, `evidence_view`, `scripts_view`,
  `scheduler_view`, `reports_view`, `git_view`, `dashboard_view`.

**FIX-03: Toggle Activar/Desactivar proyecto** (`gui/project_view.py`):
- `_activate_project()` ahora es toggle: si el proyecto ya esta activo, lo desactiva.
- Boton cambia texto entre "Activar" y "Desactivar" segun estado.
- `_on_row_select()` sincroniza texto del boton al seleccionar fila.
- Nuevo metodo `set_project()` para recibir proyecto activo desde MainWindow.

**FIX-04: Icono Scripts vacio en sidebar** (`gui/icon_manager.py`):
- Agregado `"scripts": "documents-stack.png"` a `ICON_MAP` (mismo icono que "projects").

**FIX-05: hiddenimports faltantes en .spec** (`DSOAgent.spec`, `DSOAgent_linux.spec`):
- Agregados: `gui.scripts_view`, `data.provider_registry`, `integrations.sharepoint_provider`,
  `integrations.scan_manager`.

**FIX-06: Test colgado** (`tests/test_gui_project_view.py`):
- `test_delete_without_selection_is_noop` parchea `messagebox.showwarning` para evitar
  dialogo modal bloqueante.

**Tests**: 568 -> 572 passing (33 archivos):
- 4 tests nuevos en `test_gui_project_view.py`: toggle activar/desactivar, set_project sync.

---

## [0.7.0] - 2026-04-25

### Pre-RC1 — Nuevas Features GUI + Scan Manager + Multi-Proveedor

**Feature 1: Scripts Custom View** (`gui/scripts_view.py`):
- Nueva vista completa con formulario CRUD (tipo, nombre, ruta, config, plantilla)
- Treeview de scripts configurados por proyecto (`projects/<id>/scripts.json`)
- Seccion sandbox: ejecucion aislada con toggles network/filesystem y timeout
- Consola de salida dark para resultados de ejecucion
- Referencia legacy: `reviews/views/scripts_config_ui.py` (Ui_ConfiguracionScripts)
- Registrado como `-NAV-SCRIPTS-` en MainWindow (12 nav items total)

**Feature 2: SAST/DAST Scan Manager** (`integrations/scan_manager.py`):
- `ScanManager` con registro de herramientas extensible (`AbstractScanTool`)
- 4 tipos de escaneo: `SAST_CODE`, `SAST_APP`, `DAST_WEB`, `DAST_APP`
- Ejecucion async con validacion de config y historial de resultados
- Dataclasses: `ScanConfig`, `ScanResult`, `ScanStatus`, `ScanType`

**Feature 3: Multi-Proveedor** (`data/provider_registry.py` + `gui/settings_view.py`):
- `ProviderRegistry`: CRUD persistente de N conexiones (Azure, Jira, custom)
- Tab "Proveedores" en SettingsView con Treeview y formulario
- Activar/desactivar conexiones, persistencia JSON automatica
- `ProviderEntry` dataclass con auto-ID de 8 caracteres

**Feature 4: PDF filename fix** (`gui/help_view.py`):
- Corregido "Manual de instalacion ADSO.pdf" a "Manual de instalacion ADSO.pdf" (acento)
- Verificado que los 3 manuales PDF abren correctamente desde la GUI

**Feature 5: SharePoint connector** (`integrations/sharepoint_provider.py`):
- `SharePointProvider` skeleton con `SharePointConfig` y `SharePointResult`
- Metodos stub: `test_connection`, `upload_file`, `download_file`, `list_files`, `create_folder`
- Listo para integracion real con Microsoft Graph API en RC1

**Tests**: 413 -> 568 passing (33 archivos):
- `test_scripts_view.py` (13 tests): import, instanciacion, CRUD, sandbox UI
- `test_scan_manager.py` (15 tests): ScanManager, ScanTool, historial, dataclasses
- `test_provider_registry.py` (13 tests): CRUD, persistencia, toggle, reload
- `test_sharepoint_provider.py` (12 tests): config, conexion, upload/download, list
- `test_gui_integration.py`: actualizado NAV_ITEMS count 11 -> 12 y labels

**Documentacion**:
- `README.md`: reescrito profesionalmente para v0.7.0 (badges, features, vistas, roadmap)
- `CHANGELOG.md`: entrada v0.7.0 completa
- `PENDINGS.md`: features 1-3-5 marcadas como completadas, conteos actualizados
- `DOUBTS.md`: dudas 3-4-5 resueltas, version y conteos actualizados
- `RESUMEN.md`: actualizado a v0.7.0 con nuevos modulos y conteos
- `GUI_ERRORS.md`: regenerado con conteos actuales
- `GUI_TEST_COVERAGE_ANALYSIS.md`: actualizado cobertura post-implementacion

---

## [0.6.0] - 2026-04-19

### Feature Complete — Camino a RC1

**Bugs corregidos en pruebas de campo**:
- `utils/task_scheduler.py`: `schtasks /Query` retorna CP1252 en Windows en espanol; `stdout.decode()` falla con byte `0xa2`. Corregido a `decode("cp1252", errors="replace")`.
- `gui/scheduler_view.py`: `TclError: invalid command name` al cerrar app mientras callback `after()` pendiente. Corregido con `winfo_exists()` antes de `self.after()`.
- `gui/help_view.py`, `gui/themes.py`, `data/config_manager.py`: version strings actualizados de `0.5.0-beta`/`0.1.0`/`0.2.0-beta.0` a `0.6.0`.
- `data/config_manager.py`: caracteres chinos en log message eliminados (Regla 10).

**Tests**: 313 → 413 passing (25 archivos):
- `test_task_scheduler.py`: test de regresion para encoding CP1252 (`test_list_tasks_cp1252_encoding`).
- `tests/test_gui_views.py`: nuevo archivo, 26 tests de emulacion GUI real con `ctk.CTk().withdraw()`. Cubre HelpView, LogsView, FindingsView, DashboardView, LogPanel, ThemeManager, ScrollablePane.
- `tests/test_gui_tickets_view.py`: 14 tests de emulacion profunda TicketsView.
- `tests/test_gui_settings_view.py`: 17 tests de emulacion SettingsView.
- `tests/test_gui_project_view.py`: 14 tests de emulacion ProjectView.
- `tests/test_gui_integration.py`: 29 tests de integracion backend-GUI.

**Deployment readiness**:
- `.dockerignore` creado: excluye `venv/`, `reviews/`, `tests/`, `build/`, `dist/`, `.env`.
- `setup_linux.sh`: paquete pystray corregido a `gir1.2-ayatana-appindicator3-0.1` (Ubuntu 24.04).
- `setup_linux.sh`: agregados `libjpeg-dev` y `zlib1g-dev` para Pillow.

**Documentacion**:
- `docs/AGENTS.md`: unica fuente (eliminado `AGENTS.md` raiz); reglas 11-14 consolidadas.
- `docs/TECHNICAL.md`: seccion Docker corregida (eliminadas referencias a `--headless` y `DSOAGENT_HEADLESS` que no existen en `main.py`); paquete `gir1.2-ayatana-appindicator3-0.1` corregido.
- `docs/RESUMEN.md`, `docs/PENDINGS.md`, `docs/DOUBTS.md`: version y conteos actualizados.
- `README.md`: version, tests, descripcion Docker y estado actualizados.

**Auditoria documentacion (2026-04-20)**:
- Conteos de tests actualizados en TODOS los docs: 339→413, 22→25 archivos.
- Emojis eliminados de `docs/DOUBTS.md` (Regla 10 AGENTS.md).
- `GUI_ERRORS.md` identificado como stale (bugs ya corregidos en codigo).
- Archivos faltantes detectados: `DSOAgent.spec`, `DSOAgent_linux.spec`, `requirements.headless.txt`.
- Linea duplicada corregida en `PENDINGS.md` (setup_linux.sh --headless → setup_linux.sh).
- `datetime.now()` sin timezone aun presente en 12+ archivos de produccion (pendiente calidad).

---

## [0.5.2-beta] - 2026-04-17

### Reorganizacion del Proyecto

**Eliminacion modo headless de main.py**:
- `main.py` simplificado: solo lanza GUI. `_run_headless()` y la logica `--headless` eliminados.
- La app es exclusivamente de escritorio; no se ejecuta como servidor.

**Infraestructura Docker**:
- `Dockerfile`, `Dockerfile.gui`, `docker-compose.yml`, `requirements.headless.txt` permanecen en la raiz del proyecto.
- `docker-compose.yml`: volumen corregido a `/root/.dsoagent` (consistente con paths.py).

**Limpieza de dependencias**:
- `pyproject.toml`: version `0.5.1-beta`, eliminados `pydantic`, `pyyaml`, `gitpython`;
  entry point `dsoagent-headless` eliminado; `requires-python` corregido a `>=3.10`.

**AGENTS.md actualizado y copiado a docs/**:
- Estructura de directorios corregida a archivos reales del proyecto.
- Version, conteo de tests (312), stack y dependencias actualizados.
- Copia disponible en `docs/AGENTS.md`.

**Otros ajustes**:
- `setup_linux.sh`: eliminada opcion `--headless`.
- `.env.example`: eliminadas variables `DSOAGENT_HEADLESS` y `DSOAGENT_DATA_PATH`.
- `.gitignore`: agregados `logs/`, `evidencias/`, `projects/`, `migrations/`, `venv/`.
- `README.md`: version, tests, tabla de despliegue actualizados.

---

## [0.5.1-beta] - 2026-04-17

### Corregido - Revision exhaustiva de cimientos, procesos y modulos

**CRITICO-01: Rutas de usuario inconsistentes** (`utils/paths.py`):
- `get_config_dir()` ahora retorna `%APPDATA%/DSOAgent/config` (Windows) y `~/.dsoagent/config` (Linux).
  Antes apuntaba a `%APPDATA%/config` sin subdirectorio DSOAgent.
- `get_logs_dir()` ahora retorna `%APPDATA%/DSOAgent/logs` (Windows) y `~/.dsoagent/logs` (Linux).
  Antes apuntaba a `%APPDATA%/logs`.
- Ambas rutas ahora coinciden con `SecurityVault._get_default_config_dir()`.

**CRITICO-02: Vista de Logs siempre vacia** (`utils/logging_system.py`):
- `LOGS_DIR` ahora usa `get_logs_dir()` en lugar de `project_root/logs/`.
- `logs_view.py` ya usa `get_logs_dir()`, por lo que ambos modulos ahora
  leen y escriben en el mismo directorio. El bug hacia invisible todos los logs en UI.

**CRITICO-03: API mismatch en Orchestrator** (`agent/orchestrator.py`):
- `_process_finding()` llamaba `ticket_service.create_ticket(provider=..., title=..., ...)`
  con kwargs pero `TicketService.create_ticket()` espera `ticket: TicketCreate`.
- Corregido: rama headless construye `TicketCreate` y llama al servicio interno correcto;
  rama con `ticket_provider` inyectado usa el contrato `TicketProvider` directamente.

**REGLA3-01: datetime.now() sin zona horaria** (`data/profile_manager.py`):
- Sustituidos 4 usos de `datetime.now()` por `now_utc()` de `utils/timezone.py`.

**REGLA3-02: datetime.now() sin zona horaria** (`data/project_manager.py`):
- Sustituidos 3 usos de `datetime.now()` por `now_utc()` en `create_project()` y `update_project()`.

**REGLA-LOGGER: logging.getLogger() en lugar de get_logger()** (4 modulos):
- `gui/framework.py`, `security/vault.py`, `data/config_atomic.py`, `data/project_manager.py`
  ahora usan `get_logger()` centralizado del sistema de logging.

**CONSISTENCIA-01: Ruta vault Linux** (`security/vault.py`):
- `_get_default_config_dir()` en Linux usaba `~/.config/DSOAgent` (inconsistente con paths.py).
  Corregido a `~/.dsoagent` para coincidir con `get_config_dir()`.

**MEJORA-01: ID de proyecto mas robusto** (`data/project_manager.py`):
- `create_project()` ahora usa `re.sub(r"[^a-z0-9]+", "-", ...)` para normalizar
  caracteres especiales en el slug del ID, en lugar de solo reemplazar espacios.

**MEJORA-02: create_status_bar() corregida** (`gui/themes.py`):
- Funcion de utilidad `create_status_bar()` requeria un widget padre (`master`)
  que faltaba; agregado como primer parametro obligatorio.

**REGLA-LOGGER segunda pasada** (12 modulos adicionales):
- `utils/health.py`, `utils/task_scheduler.py`, `sandbox/contract.py`,
  `sandbox/validator.py`, `sandbox/runner.py`, `sandbox/multiprocess_runner.py`,
  `data/evidence_manager.py`, `data/bi_reporter.py`, `integrations/azure_provider.py`,
  `integrations/jira_provider.py`, `gui/dialogs.py`, `gui/thread_pool.py`
  migrados de `logging.getLogger(__name__)` a `get_logger()` centralizado.

**REGLA-LOGGER extras**:
- `integrations/git_manager.py`: eliminado `import logging` huerfano (ya usaba `get_logger`).
- `gui/main_window.py`: eliminado `logging.basicConfig` redundante en `main()`.
  `setup_logging()` ya esta activo al importar `utils.logging_system`.
- `utils/retry.py`: movido `import logging` al inicio del archivo (estaba en linea 141).

**Tests** (`tests/test_integration_pipeline.py`):
- Mocks `_create_ticket` y `_failing_create` actualizados para aceptar el
  argumento posicional `TicketCreate` alineados con la API real.
- 303/303 tests pasando.

---

## [0.5.0-beta] - 2026-04-16

### Corregido - Tema dark/light: widgets nativos Tk/ttk adaptativos

- **THEME-01: Helpers adaptativos** (`gui/themes.py`):
  - `get_treeview_colors()` retorna 8 colores (frame_bg, tree_bg, tree_fg, field_bg,
    heading_bg, heading_fg, select_bg, select_fg) segun `ctk.get_appearance_mode()`.
  - `get_textbox_colors()` retorna bg, fg, insert_bg, select_bg para widgets `tk.Text`.
  - Ambas se evaluan en tiempo de construccion del widget, no al importar.

- **THEME-02: Vistas actualizadas** (8 archivos):
  - `findings_view.py`, `project_view.py`, `scheduler_view.py`, `settings_view.py`,
    `logs_view.py`, `tickets_view.py`, `reports_view.py`, `git_view.py`:
    todos los `tk.Frame(bg="#2b2b2b")` y `ttk.Style` con colores hex fijos
    reemplazados por `get_treeview_colors()`.
  - Cada Treeview recibe su `ttk.Style` propio (Findings/Projects/Sched/Settings/
    Logs/Tickets/Report/Git) con colores adaptativos.

- **THEME-03: LogPanel** (`gui/log_panel.py`):
  - `scrolledtext.ScrolledText` pasaba `bg="#1E1E1E"`, `fg="#D4D4D4"` hardcodeados.
    Reemplazado por `get_textbox_colors()`. La consola terminal de `git_view.py`
    mantiene su fondo negro intencional (`_CONSOLE_BG = "#0d0d0d"`).

- **THEME-04: Toggle tema** (`gui/main_window.py`):
  - `_toggle_theme()` ahora llama `view_frames.clear()` + `_refresh_current_view()`
    tras el cambio, forzando la recreacion de la vista activa con los nuevos colores.
  - `_nav_pane.update_bg()` y `_content_pane.update_bg()` actualizan el canvas
    bg del `ScrollablePane` al cambiar de tema.

### Anadido - Layout y scroll dinamicos

- **LAYOUT-01: ScrollablePane** (`gui/scrollable_pane.py`):
  - Nuevo componente `ScrollablePane(CTkFrame)` con canvas + `CTkScrollbar`.
  - Scrollbar vertical **dinamica**: aparece solo cuando el contenido supera
    la altura visible; se oculta automaticamente cuando todo cabe.
  - Soporte de scroll con rueda de mouse (Windows/macOS/Linux) al pasar el cursor
    sobre el area de contenido.
  - Metodo `scroll_to_top()` para resetear posicion al navegar entre vistas.
  - Metodo `update_bg()` para actualizar el canvas al cambiar de tema.
  - Propiedad `.interior` como contenedor padre para los widgets hijos.

- **LAYOUT-02: Sidebar scrollable** (`gui/main_window.py`):
  - Los 11 botones de navegacion se colocan dentro de un `ScrollablePane`.
  - El boton "Salir" queda fijo fuera del scroll (pinned al fondo del sidebar).
  - `sidebar.pack_propagate(False)` + `width=222` impiden que el sidebar se colapse.
  - `window.grid_columnconfigure(0, minsize=222)` garantiza ancho minimo del sidebar.

- **LAYOUT-03: Content area scrollable** (`gui/main_window.py`):
  - `main_content_frame` es ahora el `.interior` de un `ScrollablePane`.
  - Al navegar a cualquier vista se llama `scroll_to_top()` automaticamente.
  - `_show_settings_view()` tambien resetea el scroll al cargar configuracion.

- **LAYOUT-04: Ventana minima** (`gui/framework.py`):
  - `window.minsize(960, 580)` previene que la ventana se colapse al redimensionar.

### Anadido - UX y navegacion

- **UX-01: Highlight de navegacion activa** (`gui/main_window.py`):
  - `_update_nav_highlight()`: resalta el boton del sidebar correspondiente a la
    vista activa con `fg_color=("gray80", "gray25")`. El resto permanece transparente.
  - Se llama automaticamente desde `_refresh_current_view()`.

- **UX-02: Teclado Enter en formularios** (4 archivos):
  - `project_view.py`: Nombre → Descripcion → TicketProj → guardar con Enter.
    Escape cancela y vuelve al modo Nuevo.
  - `settings_view.py`: Tab Info (Nombre → Cliente → guardar), Tab Azure
    (Org → Project → Token → guardar), Tab Jira (Domain → Email → Token → Key → guardar).
  - `git_view.py`: entry_name → entry_branch con Enter; entry_branch → agregar repo con Enter.
  - `scheduler_view.py`: Nombre → Script con Enter; Hora → crear tarea con Enter.
    Escape limpia el formulario de tarea.

- **UX-03: Busqueda rapida en Findings** (`gui/findings_view.py`):
  - Campo de texto "Buscar:" en la barra de filtros. Filtra el Treeview en tiempo
    real por titulo del hallazgo mientras el usuario escribe (StringVar + trace_add).
  - Escape limpia el campo de busqueda. El label "Total:" cambia a "Mostrando: N"
    cuando hay filtro de texto activo.

### Total tests

- **303/303 tests passing** (sin cambios — nuevos componentes no requieren tests unitarios directos por ser wrappers de Tkinter)

---

## [0.4.0-beta] - 2026-04-16

### Tests - Cobertura de modulos criticos (0% -> alta cobertura)

- **TEST-01: `tests/test_git_manager.py`** (44 tests - GitManager al 77%):
  - `TestRepoEntry`: to_line, url, branch defaults.
  - `TestGitResult`: success/failure, default output.
  - `TestGitManagerInit`: base_url normalizada, timeout default/custom.
  - `TestGitManagerReposFile`: load valido/invalido, ignora comentarios, columna unica, save, roundtrip, lista vacia.
  - `TestGitManagerClone`: success/failure, on_line callback, url personalizada, TimeoutError, clone_all progress, clone_all count. Subproceso completamente mockeado.
  - `TestGitManagerPull`: directorio faltante, success/failure, pull_all count y progress. Subproceso mockeado.

- **TEST-02: `tests/test_task_scheduler.py`** (19 tests - TaskScheduler al 85%):
  - `TestScheduleFrequency`: valores y conteo del enum.
  - `TestScheduledTask`: campos requeridos y defaults opcionales.
  - `TestTaskSchedulerResult`: success/failure, data, default empty dict.
  - `TestTaskSchedulerPlatform`: Windows/Linux/macOS detection.
  - `TestTaskSchedulerNonWindows`: todas las operaciones rechazadas en no-Windows.
  - `TestTaskSchedulerWindows`: create/delete/list/run con subprocess mockeado, timeout, prefijo DSOAgent_.

- **TEST-03: `tests/test_main.py`** (11 tests - main.py al 89%):
  - `TestHeadlessDetection`: --headless argv, DSOAGENT_HEADLESS env, default GUI.
  - `TestRunHeadless`: inicializa Orchestrator, llama disconnect_all, loguea inicio.
  - `TestRunGui`: ImportError -> sys.exit(1), excepcion generica -> sys.exit(1).
  - `TestMainFunction`: main() llama _run_headless/gui segun flag/env.

### Metricas de cobertura alcanzadas

| Modulo | Antes | Despues |
|--------|-------|--------|
| `integrations/git_manager.py` | 0% | **77%** |
| `utils/task_scheduler.py` | 0% | **85%** |
| `main.py` | 0% | **89%** |
| Total proyecto | 53% | **59%** |

### Total tests

- **303/303 tests passing** (era 240 en v0.3.4)
- 21 archivos de tests
- Cobertura de logica de negocio critica: vault 79%, sandbox 86%, git_manager 77%, task_scheduler 85%, main 89%

---

## [0.3.4-beta.0] - 2026-04-16

### Anadido

- **GIT-01: GitManager** (`integrations/git_manager.py`):
  - Nuevo modulo para operaciones Git (clone, pull) usando `asyncio.create_subprocess_exec` (sin shell=True).
  - `clone_repo()` / `clone_all()`: Clona repos desde Azure DevOps u otro remote con control de progreso.
  - `pull_repo()` / `pull_all()`: Ejecuta `git pull origin <branch>` en repos locales existentes.
  - `load_repos()` / `save_repos()`: Lee y guarda el archivo `repos_pendientes.txt` (formato: `nombre, rama`).
  - `GitResult` dataclass encapsula exito, salida, duracion y error por operacion.
  - Timeout configurable (default 120s) por operacion.

- **GIT-02: GitView** (`gui/git_view.py`):
  - Nueva vista Git accesible desde el sidebar como "Git Repos".
  - Tab "Clonar Repositorios": Treeview editable de repos, consola dark terminal (fondo negro/texto verde), barra de progreso, selector de carpeta destino.
  - Tab "Git Pull": Identico al clone pero ejecuta pull en repos locales ya clonados.
  - Botones Agregar/Quitar repo, Cargar/Guardar `.txt` de repos.
  - Ejecucion en hilo separado (no bloquea GUI), callbacks de linea y progreso.
  - `set_project()`: Precarga `directories.repositories` y URL base Azure del proyecto activo.

- **GIT-03: Registro en MainWindow** (`gui/main_window.py`):
  - `-NAV-GIT-` agregado a `NAV_ITEMS` (entre Reportes y Tareas).
  - `git_view` agregado a `view_map` y `_refresh_current_view()`.
  - Titulo de vista: "Git - Repositorios".

- **CFG-01: area_path + iteration_path en AzureProjectConfig** (`data/profile_manager.py`):
  - Nuevos campos `area_path: str = ""` e `iteration_path: str = ""` en el dataclass.
  - Ambos se persisten en el perfil cifrado via `SecurityVault`.

- **CFG-02: Campos Azure en SettingsView** (`gui/settings_view.py`):
  - Dos nuevas entradas en el tab Azure: "Area Path" e "Iteration Path".
  - Con placeholder descriptivo y carga/guardado completo.

- **CFG-03: Uso en create_ticket()** (`integrations/azure_provider.py`):
  - `__init__` lee `area_path` e `iteration_path` desde config.
  - `create_ticket()` incluye `System.AreaPath` y `System.IterationPath` en el Work Item si estan configurados.

### Tests

- **240/240 tests passing** (sin regresiones).

---

## [0.3.3-beta.0] - 2026-04-16

### Corregido

- **SEC-01: Recovery Key rota en SecurityVault** (`security/vault.py`):
  - `generate_recovery_key()` ahora cifra el contenido de `vault.key` con la recovery key (Fernet)
    y almacena el vault cifrado en `recovery.key`. Antes generaba una clave independiente sin ningun vinculo al vault.
  - `recover_with_key()` ahora descifra el `vault.key` desde `recovery.key`, lo restaura en disco
    y recarga la boveda. Antes intentaba usar la recovery key como clave Fernet directa, lo que
    nunca podia descifrar datos cifrados con la clave original del vault.
  - Soporte de deteccion de formato obsoleto (v1.0): lanza `ValueError` con mensaje de accion requerida.

### Añadido

- **GUI-01: Boton Sincronizar Excel→Tickets en TicketsView** (`gui/tickets_view.py`):
  - Nuevo boton "Sincronizar" en la barra de herramientas de la vista de tickets.
  - Lee hallazgos Open/New del Excel del proyecto activo via `ExcelHandler`.
  - Crea tickets en el proveedor activo (Azure/Jira) por cada hallazgo, mapeando severidad a prioridad.
  - Ejecucion en hilo separado (no bloquea GUI). Muestra resumen de exitos/errores al completar.

- **GUI-02: Precarga automatica de credenciales en TicketsView** (`gui/tickets_view.py`):
  - `set_project()` busca en `ProfileManager` un perfil con el mismo nombre del proyecto activo.
  - Si encuentra coincidencia y el proveedor esta habilitado, registra automaticamente el
    `AzureDevOpsProvider` o `JiraProvider` con las credenciales del perfil.
  - El usuario solo necesita hacer clic en "Conectar" sin reingresar credenciales manualmente.
  - `TicketsView.__init__` acepta nuevo parametro `profile_manager` (opcional).
  - `MainWindow._show_view_by_class` pasa `profile_manager` al crear `TicketsView`.

- **GUI-03: set_project() en SchedulerView** (`gui/scheduler_view.py`):
  - Nuevo metodo `set_project(project)` que actualiza `lbl_active_project` en la cabecera.
  - El scheduler ahora recibe y muestra correctamente el proyecto activo al navegar a la vista.

### Tests

- **71 tests de integracion nuevos** (3 archivos):
  - `test_integration_pipeline.py`: 27 tests — Orchestrator pipeline Excel→Tickets, mapeo severidad→prioridad, metricas BI consolidadas, concurrencia de proyectos.
  - `test_integration_vault_advanced.py`: 24 tests — SecurityVault persistencia, concurrencia, tamper detection, recovery key (incluyendo validacion del nuevo flujo correcto).
  - `test_integration_evidence_pipeline.py`: 20 tests — EvidenceManager hash integrity, aislamiento por proyecto/finding, RateLimiter thread-safe, Throttler backoff.
- **240/240 tests passing** (era 169/169 en v0.3.2)

---

## [0.3.2-beta.0] - 2026-04-16

### Corregido

- **C-01: self.url AttributeError en JiraProvider** (`integrations/jira_provider.py`):
  - `_post`, `_get`, `_put` y `delete_ticket` usaban `self.url` (no existe); reemplazado por `self._base_url`
  - Eliminada doble ruta `/rest/api/3/` en construccion de URLs
- **C-02: TicketStatus.REOPENED inexistente** (`integrations/jira_provider.py`):
  - `_map_status_from_jira` referenciaba `TicketStatus.REOPENED` que no esta en `contracts.py`
  - Mapeado a `TicketStatus.OPEN` como fallback correcto
- **C-03: Metodos _get/_patch duplicados en AzureProvider** (`integrations/azure_provider.py`):
  - Segunda definicion de `_get` y `_patch` (sin rate limiting) sobreescribia las versiones con rate limiting
  - Eliminados duplicados; conservadas las versiones con `AsyncRateLimiter`
- **C-04: Emojis en NAV_ITEMS y botones** (`gui/main_window.py`):
  - Reemplazados todos los emojis por texto Latin conforme a AGENTS.md R10
  - NAV_ITEMS simplificado a 2 campos (nav_id, text) eliminando campo icon
- **C-05: Dashboard mostraba textbox en lugar de DashboardView** (`gui/main_window.py`):
  - `-NAV-DASHBOARD-` ahora carga `DashboardView` con metricas BI
  - `_get_dashboard_content()` eliminado (era solo texto sin datos)
- **C-06: main_content CTkTextbox destruido al navegar** (`gui/main_window.py`):
  - `_show_view_by_class` destruia todos los children de `main_content_frame` incluido el textbox
  - Reemplazado por metodo `_show_placeholder(title, message)` que crea frames temporales
  - `_update_content` delegado a `_show_placeholder`
- **C-07: issue_type no persistido en JiraProjectConfig** (`data/profile_manager.py`, `gui/settings_view.py`):
  - Campo `issue_type: str = "Bug"` agregado a `JiraProjectConfig`
  - `settings_view.py`: `combo_jira_type` ahora carga y guarda `issue_type` del proyecto
- **C-08: _clear_form fallaba en entries disabled** (`gui/settings_view.py`):
  - Entries con `state="disabled"` temporalmente habilitadas antes de limpiar y restauradas al estado original

### Deployment (multi-modo)

- **main.py**: Flag `--headless` y variable `DSOAGENT_HEADLESS=1`; separa `_run_gui()` y `_run_headless()` con manejo de señales SIGTERM/SIGINT
- **Dockerfile**: Reescrito para modo headless; elimina `libpq-dev` (PostgreSQL no usado); usa `requirements.headless.txt`; agrega HEALTHCHECK
- **Dockerfile.gui**: Nuevo; modo GUI con Xvfb para X11 forwarding (desarrollo/testing en Linux)
- **docker-compose.yml**: Elimina servicio PostgreSQL innecesario; arquitectura microservicios `agent-core` + `api` (comentado, pendiente v1.0); volumenes correctos
- **DSOAgent.spec**: Bug critico corregido — `'tkinter'` estaba en `excludes` (rompia toda la GUI); hiddenimports completos (tkinter, cryptography.hazmat, pydantic_core, PIL, pystray._win32, openpyxl.styles, aiohttp.connector, etc.)
- **DSOAgent_linux.spec**: Nuevo; especifico Ubuntu 24.04 (pystray._xorg, strip=True, excluye backends Windows/Darwin)
- **requirements.headless.txt**: Nuevo; solo dependencias del agente core sin GUI (para Docker)
- **requirements.txt**: Reorganizado en secciones (agente/GUI/dev/packaging); anotaciones sistema Linux para apt-get
- **pyproject.toml**: Version `0.3.2-beta.0`; `[project.dependencies]` correcto para `pip install -e .`; extras `[gui]` y `[dev]`; `[tool.setuptools.packages.find]` para excluir venv/reviews
- **.env.example**: Limpiado; elimina variables no usadas (PostgreSQL, LLM, Telegram, DATABASE_URL); agrega `DSOAGENT_HEADLESS`, `ENCRYPTION_KEY`, vars Azure/Jira comentadas
- **setup_linux.sh**: Nuevo; instala Python 3.12, dependencias sistema (GUI o headless), crea venv, instala requirements, crea .env
- **Makefile**: Nuevos targets: `install-gui`, `package-linux`, `docker-build`, `docker-gui`, `docker-up`, `docker-down`, `setup-linux`, `run-headless`

### Tests

- **169/169 tests passing** confirmados despues de todos los cambios de esta version

---

## [0.3.1-beta.0] - 2026-04-15

### Corregido

- **B-01/B-02: CTkTreeview y CTkScrollbar** (`gui/findings_view.py`, `gui/evidence_view.py`):
  - Reemplazados con `tkinter.ttk.Treeview` y `tkinter.ttk.Scrollbar` que si existen en la libreria
- **B-03: CTkMessagebox** (`gui/framework.py`):
  - Reemplazado con `tkinter.messagebox` estandar
- **B-04: EvidenceManager sin base_path** (`gui/main_window.py`):
  - Inicializacion corregida con argumento `base_path` requerido
- **B-05: ProjectManager no inicializado** (`gui/main_window.py`):
  - Instancia creada en `_init_managers()` y registrada en `view_map`
- **B-06: view_map key incorrecto** (`gui/main_window.py`):
  - Clave `"dashboard_view"` corregida a `"dashboard_view"` con navegacion correcta por ID
- **B-07: asyncio.create_task()** (`gui/scheduler_view.py`):
  - Reemplazado con `threading.Thread` + `asyncio.run()` compatible con Tkinter mainloop
- **B-08: ticket_service.register()** (`agent/orchestrator.py`):
  - Corregido a `register_provider()` segun API real de `TicketService`
- **B-09: import logging faltante** (`gui/main_window.py`):
  - Agregado import al inicio del modulo
- **B-10: _delete_project() vacio** (`gui/project_view.py`):
  - Implementado con confirmacion de usuario y llamada a `project_manager.delete_project()`
- **B-11: Enums fuera de lugar** (`agent/contracts.py`):
  - `TicketPriority` y `TicketStatus` movidos antes del footer del archivo
- **B-12: Constantes duplicadas** (`data/evidence_manager.py`):
  - Eliminadas las 4 definiciones duplicadas de `MAX_IMAGE_WIDTH`, `MAX_IMAGE_HEIGHT`, `MAX_IMAGE_SIZE_MB`, `SUPPORTED_IMAGE_FORMATS`

### Tests

- Corregido `test_excel_handler_with_path`: comparacion de `Path` en lugar de string (compatibilidad Windows/Unix)
- Corregido `test_log_panel_append`: fixture `module`-scoped para evitar multiples instancias `CTk()` en el mismo proceso
- **169/169 tests passing** en venv con Python 3.12

---

## [0.3.0-alpha.0] - 2026-04-14

### Añadido

- **Vistas GUI conectadas** (`gui/main_window.py`):
  - Dashboard (BIReporter)
  - Settings (ProfileManager)
  - Projects (ProfileManager)
  - Findings/Scans (ExcelHandler)
  - Evidence (EvidenceManager)
- **Theme toggle button**: Botón para cambiar entre dark/light
- **Dialog modal**: Nuevo proyecto aparece sobre ventana principal
- **Scrollbar en mappings**: CTkScrollableFrame para tab de mapeos
- **Logs en directorio del proyecto**: `logs/` en lugar de `~/.dsoagent/logs`
- **Sistema de navigation**: Navegación funcional en sidebar

### Corregido

- **Geometry manager**: pack vs grid en settings_view.py
- **Mapeos tab**: CTkScrollableFrame en lugar de CTkCanvas (no existe)
- **ProfileManager import**: Duplicado eliminado
- **Dashboard view**: Argumento corregido (bi_reporter)
- **ProjectManager import**: Corregido en views

### Notas de versión

- Alpha: Funcionalidades principales en desarrollo
- Testing confirmado en Windows (Python 3.10.6)
- 169/169 tests passing
- Logs en `DSOAgent/logs/` (directorio del proyecto)

---

## [0.2.1-beta.0] - 2026-03-31

### Añadido

- **Mapeos de campos configurables** (`data/profile_manager.py`): 
  - `severity_mapping` y `status_mapping` para Azure y Jira
  - GUI de configuración en Settings > Mapeos (`gui/settings_view.py`)
- **Conversión de imágenes con Pillow** (`data/evidence_manager.py`):
  - Conversión automática JPG/BMP → PNG
  - Validación de tamaño (1920x1080 máx, 10MB máx)
  - Constantes: `MAX_IMAGE_WIDTH`, `MAX_IMAGE_HEIGHT`, `MAX_IMAGE_SIZE_MB`
  - Funciones: `validate_image_size()`, `_convert_and_validate_image()`
- **MultiprocessSandboxRunner** (`sandbox/multiprocess_runner.py`):
  - Ejecución de scripts en procesos separados
  - Verdadero aislamiento de seguridad
  - Timeout configurable
  - Pool de procesos con `max_workers`

### Tests

- Tests completos para `MultiprocessSandboxRunner` (11 tests)
- Tests para validación de imágenes en `test_evidence_manager.py` (6 tests)

---

## [0.2.0-beta.0] - 2026-03-30

### Añadido

- **Agent Core (Orchestrator)**: Cerebro principal que coordina todos los módulos
- **Sistema de Paths universal** (`utils/paths.py`): Funciona en desarrollo y como .exe
- **Rate Limiting** (`utils/rate_limiter.py`): Control de solicitudes para Azure/Jira
- **Retry Logic** (`utils/retry.py`): Reintentos con backoff exponencial (tenacity)
- **Contratos para inyección de dependencias** (`agent/contracts.py`): Desacoplamiento agent/integrations
- **Recovery Key System** (`security/vault.py`): Clave de recuperación para disaster recovery

### Mejorado

- **Seguridad AGENTS.md**: shell=True eliminado de subprocess
- **datetime.utcnow() reemplazado por** datetime.now(timezone.utc) en logger.py
- **Rate Limiting** integrado en AzureDevOpsProvider y JiraProvider
- **Inyección de dependencias**: Agent ya no importa directamente de integrations/

### Documentación

- Actualizada la versión en pyproject.toml a 0.2.0-beta.0
- Actualizada la documentación técnica docs/TECHNICAL.md
- Actualizada la guía de desarrollo AGENTS.md
- Actualizado README.md con comandos correctos
- Eliminada carpeta files/ (no tenía uso)

---

## [0.1.0-alpha.0] - 2025-03-27

### Añadido

- **CustomTkinter GUI**: Interfaz moderna multiplataforma (MIT License)
- **SecurityVault**: Cifrado Fernet + PBKDF2 para credenciales
- **Configuración Atómica**: Perfiles de proyecto aislados
- **ThreadPool**: Ejecución de tareas en background
- **LogPanel**: Logging centralizado con QueueHandler
- **Dialogs System**: WarningDialog, HelpBrowser, ExceptionHandler
- **TicketService**: Abstracción para proveedores de tickets
- **AzureDevOpsProvider**: Integración completa con Azure DevOps
- **JiraProvider**: Integración con Jira Cloud
- **ExcelHandler**: Manejo de archivos Excel con pandas/openpyxl
- **Finding dataclass**: Estructura normalizada para hallazgos
- **EvidenceManager**: Gestión de evidencias por proyecto
- **ProjectManager**: Multi-proyectos con persistencia JSON
- **BIReporter**: Reportes Business Intelligence consolidados
- **Sandboxing V1**: Entorno seguro para scripts

---

### Notas de versión

- Versiones 0.x.x → alpha/beta (desarrollo)
- Versiones 1.x.x → release (producción)

#End Development By Angel Esquivel (CyberSecurity) [DSOAgent 2026]
