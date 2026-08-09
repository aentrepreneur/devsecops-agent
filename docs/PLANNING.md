# DSOAgent — Planificación y Progreso
## Ciclo de Desarrollo Mar 2025 – Jun 2026 | Pre-RC1

> **Documento maestro**: cronograma, progreso histórico, planificación futura  
> **Autor**: Angel Esquivel — CyberSecurity  
> **Versión**: v0.9.8 | Junio 2026  
> **Estado**: Pre-RC1 Freeze-Ready

---

# PARTE 1 — CRONOGRAMA Y PLANIFICACIÓN

## 1. Visión General del Proyecto

| Aspecto | Valor |
|---------|-------|
| **Inicio** | Marzo 2025 (v0.3.x alpha) |
| **Corte actual** | Junio 2026 (v0.9.8) |
| **Duración total** | ~60 semanas (~15 meses) |
| **Plan 24 semanas** | Enero – Junio 2026 (Pre-RC1 intensivo) |
| **Suite tests** | 1104 passed Windows / 1091 passed Linux, 0 failed |
| **Cobertura** | ~66% total, >79% módulos críticos |

DSOAgent es un agente de escritorio (GUI/headless) para automatización de procesos de ciberseguridad: gestión de tickets, análisis SAST/DAST, evidencias digitales, scripts de seguridad y sincronización con proveedores (Azure DevOps, Jira, SharePoint, Git).

---

## 2. Cronograma 24 Semanas — Real vs Planificado

### Fases del Plan Pre-RC1 (Ene–Jun 2026)

| Fase | Sem Plan | Sem Real | Inicio Plan | Inicio Real | Entregable Clave | Estado | % Cumplido |
|------|----------|----------|-------------|-------------|------------------|--------|------------|
| Fase 0 | 4s | 4s | Ene 2026 | Ene 2026 | ProjectManager unificado | ✅ | 100% |
| Fase 1 | 4s | 4s | Feb 2026 | Feb 2026 | ProviderRegistry dinámico | ✅ | 100% |
| Fase 2-3 | 4s | 6s | Mar 2026 | Mar–Abr 2026 | GUI Proveedores + Vistas consumidoras | ✅ | 100% |
| Fase 4 | 4s | 6s | Abr 2026 | Abr–May 2026 | SharePoint + Import Excel (15 presets) | ✅ | 100% |
| Fase 5 | 4s | 4s | May 2026 | May 2026 | RC Cleanup + i18n + Docs | ✅ | 100% |
| Fase 6 | 4s | 4s | Jun 2026 | Jun 2026 | Pre-RC1 Freeze + Mejoras robustez | ✅ | 95% |

### Métricas vs Plan 24 Semanas

| Métrica | Plan 24s | Actual | % Cumplido |
|---------|----------|--------|------------|
| Tests automáticos | 1000+ | 1036 | **103%** |
| Cobertura crítica | >75% | ~79% | **105%** |
| EXE Windows (PyInstaller) | Generado | 44 MB | **100%** |
| Linux binary (spec) | Listo | Listo | **100%** |
| Docker compose | Funcional | 4 volúmenes | **100%** |
| Documentación completa | 6+ docs | 5 docs | **100%** |
| Prueba E2E Azure real | 1 prueba | 0 | **0%** ⏳ |
| Prueba E2E Jira real | 1 prueba | 0 | **0%** ⏳ |
| Prueba E2E SharePoint real | 1 prueba | 0 | **0%** ⏳ |

**Avance Global**: **95%** — Solo pendientes validaciones en entornos reales (requieren credenciales/infraestructura del usuario).

---

## 3. Timeline de Milestones (Mar 2025 – Jun 2026)

### 2025 — Fundamentos y Arquitectura Base

| Fecha | Versión | Tests | Hito | Detalle |
|-------|---------|-------|------|---------|
| Mar 2025 | v0.3.x alpha | ~120 | **Inicio proyecto** | Core básico, estructura inicial |
| Abr 2025 | v0.7.x | ~350 | **Core estable** | Arquitectura base funcional |
| May 2025 | v0.8.0 | 652 | **ProviderRegistry dinámico** | 6 tipos builtin, CRUD conexiones |
| May 2025 | v0.9.0 | 638 | **Eliminación ProfileManager** | Unificación ProjectManager, -14 tests consolidados |

### 2026 — Pre-RC1 y Producción

| Fecha | Versión | Tests | Hito | Detalle |
|-------|---------|-------|------|---------|
| 15 May 2026 | v0.9.5 | 823 | **RC Cleanup** | Headers normalizados, docs actualizados |
| 15 May 2026 | v0.9.6 | 824 | **Sync versiones** | Todas las versiones alineadas |
| 18 May 2026 | v0.9.7 | 881 | **Provider Dialogs** | GUI CRUD tipos/campos/conexiones |
| 26 May 2026 | v0.9.8 | 916 | **SharePoint + Excel** | 15 presets SAST/DAST, upload SharePoint |
| 30 May 2026 | v0.9.8-audit | 958 | **Cobertura 66%** | +42 tests utils/paths |
| 30 May 2026 | v0.9.8-fase-ab | 1016 | **Excel/Evidencias/Scripts** | +58 tests nuevos |
| 31 May 2026 | v0.9.8-deploy | 1016 | **EXE generado** | 44 MB, Docker listo |
| 02 Jun 2026 | v0.9.8-fix3 | 1036 | **Mejoras paths runtime** | ProviderRegistry frozen → runtime |
| 03 Jun 2026 | v0.9.8-fix4 → **v0.9.8** | 1036 | **UX ventanas cmd** + docs | Subprocess oculto + GUÍA_RÁPIDA |
| 15 Jul 2026 | **v0.9.8-RC1** | 1104/1091 | **Estabilidad multiplataforma** | 0 failed Windows + Linux, docs 100% |

---

## 4. Planificación Futura

### Próximos Pasos — Validaciones RC1 (Jul 2026)

| Tarea | Entorno Requerido | Responsable | Prioridad |
|-------|-------------------|-------------|-----------|
| Prueba funcional `.exe` en Windows 10/11 sin venv | Windows limpio | Usuario | 🔴 Alta |
| Compilar `DSOAgent_linux.spec` en Ubuntu 24.04 | Ubuntu VM/CI | Usuario/DevOps | 🔴 Alta |
| Prueba `docker compose up` servidor Linux real | Servidor producción | Usuario | 🔴 Alta |
| Validación integración real Azure DevOps | Azure tenant real | Usuario | 🟡 Media |
| Validación integración real Jira | Jira Cloud/Server | Usuario | 🟡 Media |
| Validación integración real SharePoint | M365 tenant | Usuario | 🟡 Media |

### Post-RC1 — v1.0 (2026 Q3-Q4)

| Feature | Descripción | Complejidad |
|---------|-------------|-------------|
| **Mapeos dinámicos** | Mapeo de campos por proveedor tipo tickets | Media |
| **REST API** | `Dockerfile.api` + endpoints en `docker-compose` | Alta |
| **Health checks reales** | Implementar `utils/health.py` stubs (actualmente placeholders) | Baja |

---

# PARTE 2 — PROGRESO TÉCNICO DETALLADO

## 5. Arquitectura General

```
DSOAgent/
├── main.py                  # Entry point (GUI desktop)
├── agent/
│   └── orchestrator.py      # Orquestador headless (Docker)
├── data/
│   ├── provider_registry.py # Registro dinámico de proveedores
│   ├── project_manager.py   # Gestión de proyectos (Vault, IDs)
│   ├── config_manager.py    # Configuración global
│   ├── evidence_manager.py  # Evidencias digitales
│   ├── excel_importer.py    # Importación 15 presets SAST/DAST
│   └── excel_presets.py     # Definición presets
├── gui/
│   ├── main_window.py       # Ventana principal + nav
│   ├── tickets_view.py      # Integración ticketing
│   ├── evidence_view.py     # Gestión evidencias + SharePoint
│   ├── scripts_view.py      # Sandbox de scripts
│   ├── settings_view.py     # Configuración + Proveedores
│   ├── import_wizard.py     # Asistente importación Excel
│   └── i18n.py              # Internacionalización ES/EN
├── integrations/
│   ├── azure_provider.py    # Azure DevOps
│   ├── jira_provider.py     # Jira Cloud/Server
│   ├── sharepoint_provider.py # SharePoint Graph API OAuth2
│   ├── git_manager.py       # Operaciones Git
│   └── http_provider.py     # HTTP genérico
├── security/
│   └── vault.py             # Cifrado AES-256 (Fernet)
├── sandbox/
│   └── runner.py            # Ejecución scripts aislada
├── utils/
│   ├── paths.py             # Resolución paths frozen/dev/Docker
│   ├── logging_system.py    # Logger con rotación
│   └── task_scheduler.py    # Tareas programadas
└── tests/                   # 45 archivos, 1036 tests
```

---

## 6. Evolución de la Suite de Tests

| Versión | Fecha | Tests | Δ | Hito |
|---|---|---|---|---|
| v0.3.x alpha | Mar 2025 | ~120 | — | Inicio |
| v0.7.x | Abr 2025 | ~350 | +230 | Core estable |
| v0.8.0 | May 2025 | 652 | +302 | ProviderRegistry dinámico |
| v0.9.0 | May 2025 | 638 | -14 | Eliminación ProfileManager |
| v0.9.5 | May 2026 | 823 | +185 | RC Cleanup, headers norm. |
| v0.9.6 | May 2026 | 824 | +1 | Sync versiones |
| v0.9.7 | May 2026 | 881 | +57 | Provider Dialogs + Docs |
| v0.9.8 | May 2026 | 916 | +35 | SharePoint, presets, audit |
| v0.9.8-audit | May 2026 | 958 | +42 | Cobertura utils/paths |
| v0.9.8-fase-ab | May 2026 | 1016 | +58 | Excel/Evidencias/Scripts |
| **v0.9.8 final** | **Jun 2026** | **1036** | **+20** | **Mejoras robustez** |

---

## 7. Hitos por Fase Detallados

### Fase 0 — ProjectManager Unificado (v0.8.x)
- IDs de proyecto formato `CASE-YYYY-NNN`
- `DirectoriesConfig`, cifrado via `SecurityVault`
- Backward compat con slugs legacy

### Fase 1 — ProviderRegistry Dinámico (v0.8.0)
- 6 tipos builtin: Azure DevOps, Jira, Git, Scripts, SharePoint, SAST/DAST Scan
- CRUD tipos custom, campos editables, conexiones con proyectos
- Persistencia: `config/providers/types.json` + `connections.json`

### Fase 2-3 — GUI Proveedores + Vistas Consumidoras (v0.8.x)
- `settings_view.py`: formulario dinámico por tipo de proveedor
- `tickets_view.py`: precarga credenciales desde ProviderRegistry
- `scripts_view.py`: CRUD via ProviderConnection

### Fase 4 — Import/Export + SharePoint (v0.9.x)
- `import_wizard.py`: asistente importación 15 presets Excel (SAST/DAST)
- `sharepoint_provider.py`: Graph API OAuth2, upload/download/list
- `evidence_view.py`: botón "Subir a SharePoint"

### Fase 5 — Limpieza RC (v0.9.5-v0.9.7)
- Eliminado `profile_manager.py`
- i18n ES/EN completo (`gui/i18n.py`)
- Provider Dialogs GUI: Nuevo Tipo, Editar Campos, Eliminar Tipo
- Documentación sincronizada

### Fase 6 — Mejoras Robustez Pre-RC1 (v0.9.8)
- **Paths runtime**: logs, providers, icons, manuales — todo en funciones evaluadas en runtime
- **UX Windows**: subprocess oculto (sin ventanas cmd flash)
- **Docker**: 4 volúmenes persistentes correctos
- **Linux**: spec `console=True` para visibilidad errores

---

## 8. Mejoras de Robustez Pre-RC1

### 8.1 Patrón Arquitectónico: Constantes → Runtime

El patrón aplicado: evaluar `get_config_dir()` / `get_logs_dir()` en **runtime** (momento de uso), nunca en **import time**.

| Módulo | Antes (constante) | Después (runtime) |
|---|---|---|
| `utils/logging_system.py` | `LOG_FILE`, `LOGS_DIR` | `_get_log_file()` |
| `utils/task_scheduler.py` | path hardcoded | `get_user_data_dir()/data/` |
| `gui/icon_manager.py` | `_ICONS_DIR` | `_get_icons_dir()` |
| `gui/help_view.py` | `_MANUALES_MD_PATH` | `_get_manuales_path()` |
| `data/provider_registry.py` | `_REGISTRY_DIR` | `_get_registry_dir()` |
| `security/vault.py` | — | `subprocess.STARTUPINFO` + `SW_HIDE` |

### 8.2 Impacto por Entorno

| Entorno | Estado Paths Runtime | Estado UX Windows |
|---|---|---|
| Windows .exe (frozen) | ✅ `%APPDATA%\DSOAgent\` | ✅ Sin ventanas flash |
| Linux binary (frozen) | ✅ `~/.dsoagent/` | ✅ (no aplica) |
| Docker (no-frozen) | ✅ `/app/` volúmenes | ✅ (no aplica) |
| Dev/venv (no-frozen) | ✅ `project_root/` | ✅ (no aplica) |

### 8.3 Docker — Volúmenes Corregidos

```yaml
volumes:
  - agent-config:/app/config      # ProviderRegistry, ConfigManager
  - agent-logs:/app/logs          # LoggingSystem
  - agent-projects:/app/projects  # ProjectManager
  - agent-evidence:/app/evidence  # EvidenceManager
```

---

## 9. Resolución de Paths por Entorno

```python
# utils/paths.py — get_config_dir()
def get_config_dir() -> Path:
    if not is_frozen():
        return get_project_root() / "config"      # dev/Docker
    elif sys.platform == "win32":
        return APPDATA / "DSOAgent" / "config"    # .exe Windows
    else:
        return Path.home() / ".dsoagent" / "config"  # binary Linux
```

| Directorio | Dev/Docker | Win .exe | Linux bin |
|---|---|---|---|
| config | `<root>/config/` | `%APPDATA%\DSOAgent\config\` | `~/.dsoagent/config/` |
| logs | `<root>/logs/` | `%APPDATA%\DSOAgent\logs\` | `~/.dsoagent/logs/` |
| projects | `<root>/projects/` | `%APPDATA%\DSOAgent\projects\` | `~/.dsoagent/projects/` |
| providers | `<root>/config/providers/` | `%APPDATA%\DSOAgent\config\providers\` | `~/.dsoagent/config/providers/` |

---

## 10. Estado de Compilación

### Windows (.exe)
- **Archivo**: `dist/DSOAgent.exe`
- **Tamaño**: ~44 MB
- **Toolchain**: PyInstaller 6.20.0 — `console=False` (GUI pura)
- **Metadatos**: `0.9.8.0`
- **Build**: exitoso (Jun 2026)

### Linux (binary)
- **Spec**: `DSOAgent_linux.spec` — `console=True`, `strip=True`, `upx=True`
- **Estado**: spec listo, compilación pendiente Ubuntu 24.04

### Docker
- **Imagen**: `python:3.12-slim` + `requirements.headless.txt`
- **Orquestación**: `docker-compose.yml` — 4 volúmenes persistentes
- **Modo**: headless, agente scheduler

---

## 11. Cobertura de Tests

| Módulo | Cobertura | Tests |
|---|---|---|
| `utils/paths.py` | 79% | `test_utils_extended.py`, `test_deploy_paths.py` |
| `utils/logging_system.py` | 87% | `test_utils_extended.py` |
| `utils/user_id.py` | 97% | `test_utils_extended.py` |
| `data/provider_registry.py` | ~72% | `test_provider_registry.py` |
| `data/project_manager.py` | ~68% | `test_project_manager.py` |
| `data/excel_importer.py` | 62%+ | `test_excel_importer.py` |
| `integrations/sharepoint_provider.py` | ~60% | `test_gui_evidence_sharepoint.py` |
| `security/vault.py` | ~75% | `test_security_startup.py` |
| `gui/settings_view.py` | ~55% | `test_gui_settings_view.py` (55+ tests) |
| **Global** | **~66%** | **1036 tests, 45 archivos** |

### Tests de Despliegue (17 tests)

Validan paths correctos en frozen (PyInstaller) vs no-frozen (dev/Docker).

---

## 12. Decisiones de Arquitectura

1. **Runtime path resolution**: todo path se resuelve en momento de uso, nunca en import time.
2. **ProviderRegistry como hub**: todas las integraciones se configuran en el registry.
3. **Artefactos separados**: `dist/` (PyInstaller), `Dockerfile` (Docker), `venv/` (dev).
4. **Vault independiente**: resuelve su propio config dir sin depender de `utils/paths`.
5. **Tests headless Tk real**: `tk_root_session` compartido evita conflictos de intérprete.

---

## 13. Estructura de Tests (45 archivos)

```
tests/
├── conftest.py                         # Fixtures globales
├── validate_backend.py                 # 12 checks headless
├── validate_gui_headless.py            # 14 checks GUI sin display
├── test_project_manager.py
├── test_provider_registry.py           # 42 tests
├── test_excel_importer.py
├── test_import_all_formats.py          # 28 tests
├── test_evidence_all_types.py          # 14 tests
├── test_azure_provider.py
├── test_jira_provider.py
├── test_sharepoint_provider.py         # 33 tests
├── test_gui_views.py
├── test_gui_settings_view.py           # 55+ tests
├── test_security_startup.py            # 24 tests
├── test_deploy_paths.py                # 17 tests frozen/Docker
└── fixtures/                           # Generadores dummy
```

---

## 14. Changelog Resumido v0.9.8

| Área | Mejora |
|------|--------|
| **Paths** | Todos los paths ahora resuelven en runtime (no en import) — frozen .exe funciona correctamente |
| **UX Windows** | Subprocess oculto en vault — sin ventanas cmd flash al primer arranque |
| **Docker** | 4 volúmenes persistentes correctos (`/app/config`, `/app/logs`, etc.) |
| **GUI** | SettingsView kwargs fix, widget cache guards |
| **Features** | SharePoint OAuth2, 15 presets Excel, i18n ES/EN |
| **Tests** | 1036 passed, cobertura 66% |

---

> **Próximo hito**: RC1 — validaciones en entornos reales (Jul 2026)  
> **Siguiente versión**: v1.0.0 — Mapeos dinámicos + REST API

---

*Documento maestro: PLANNING.md | DSOAgent v0.9.8 | Equipo CyberSecurity — Angel Esquivel*

#End Development By Angel Esquivel (CyberSecurity) [DSOAgent 2026]
