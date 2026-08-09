# Development by Angel Esquivel (CyberSecurity) [DSOAgent] [May-2026]®

# TESTING.md — Guia de Pruebas DSOAgent

> **Objetivo**: Ejecutar, interpretar y reportar resultados de tests de forma independiente.
> **Ultima actualizacion**: 14 Jul 2026

---

## 1. Requisitos

```bash
# Linux
sudo apt install python3-tk
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt

# Windows
python -m venv venv && venv\Scripts\activate
pip install -r requirements.txt
# Si falla SSL: pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org -r requirements.txt
```

---

## 2. Ejecucion Basica

### Suite completa

```bash
# Linux
venv/bin/python -m pytest tests/ -v --tb=short

# Windows
venv\Scripts\python -m pytest tests\ -v --tb=short
```

### Modo silencioso (solo resumen)

```bash
venv/bin/python -m pytest tests/ -q --tb=line
```

### Modo rapido (sin warnings)

```bash
venv/bin/python -m pytest tests/ -q --tb=line -W ignore::DeprecationWarning
```

---

## 3. Ejecucion por Modulo

```bash
# Un solo archivo
venv/bin/python -m pytest tests/test_vault.py -v

# Multiples archivos
venv/bin/python -m pytest tests/test_vault.py tests/test_excel_handler.py -v

# Por patron de nombre
venv/bin/python -m pytest tests/ -k "gui" -v          # todos los tests con "gui" en el nombre
venv/bin/python -m pytest tests/ -k "not windows" -v  # excluir Windows-only
venv/bin/python -m pytest tests/ -k "provider" -v     # todos los providers
```

### Grupos por categoria

```bash
# Backend puro (sin GUI)
venv/bin/python -m pytest tests/test_vault.py tests/test_project_manager.py tests/test_excel_handler.py tests/test_orchestrator.py tests/test_sandbox.py -v

# GUI (requiere display o headless)
venv/bin/python -m pytest tests/test_gui.py tests/test_gui_views.py tests/test_gui_settings_view.py -v

# Integracion
venv/bin/python -m pytest tests/integration/ tests/test_integration_pipeline.py -v

# Providers
venv/bin/python -m pytest tests/test_sharepoint_provider.py tests/test_jira_provider.py tests/test_http_provider.py -v
```

---

## 4. Validacion Independiente

Scripts standalone que no requieren pytest:

```bash
# Backend: importaciones, managers, logica de negocio (12 checks)
venv/bin/python tests/validate_backend.py

# GUI headless: temas, vistas, widgets (14 checks)
venv/bin/python tests/validate_gui_headless.py

# Sincronizacion PROMPT.md
venv/bin/python scripts/sync_prompt.py --check
```

### Salida esperada

| Script | Salida esperada |
|--------|----------------|
| `validate_backend.py` | `12/12 PASS` |
| `validate_gui_headless.py` | `14/14 PASS` |
| `sync_prompt.py --check` | `Synchronized` o `0 differences` |

---

## 5. Conteos de Tests por Plataforma

| Plataforma | Passed | Skipped | Failed | Notas |
|------------|--------|---------|--------|-------|
| **Linux** | ~1091 | 13 | 0 | Todos pasan |
| **Windows** | ~1104 | 0 | 0 | Todos pasan |

### Tests Windows-only (skip en Linux)

13 tests marcados `@pytest.mark.skipif(sys.platform != "win32")`:
- `TestTaskSchedulerWindows` (8 tests) — requiere `schtasks.exe`
- `TestSchedulerStartInFix` (3 tests) — Windows scheduler
- `TestUserIdExtended.test_get_system_info_display_name_on_windows` (1)
- 1 adicional en utilidades

### Aislamiento de mocks entre modulos

Tests que operan con aislamiento completo de mocks, sin contaminacion entre archivos:
- `test_gui_scripts_view_actions.py` — 3 tests con customtkinter mock aislado
- `test_gui_views.py::TestLogPanelEmulation` — 1 test + 2 helpers con customtkinter mock aislado
- `test_log_panel.py` — 4 tests con customtkinter mock aislado

**Arquitectura de aislamiento**: `tests/integration/test_end_to_end.py` implementa save/restore de `sys.modules['customtkinter']` con patrón try/finally para que cada modulo de tests reciba el modulo real sin residuos de mocks anteriores.

---

## 6. Guardar Resultados para Reportar

### Paso 1: Ejecutar y capturar salida

```bash
# Suite completa
venv/bin/python -m pytest tests/ -v --tb=short > test_results.txt 2>&1

# Backend
venv/bin/python tests/validate_backend.py > backend_validation.txt 2>&1

# GUI
venv/bin/python tests/validate_gui_headless.py > gui_validation.txt 2>&1

# Sync
venv/bin/python scripts/sync_prompt.py --check > sync_check.txt 2>&1
```

### Paso 2: Verificar que todo paso

```bash
# Debe mostrar: X passed, 0 failed
tail -3 test_results.txt

# Debe mostrar: 12/12 PASS
cat backend_validation.txt | grep -E "PASS|FAIL|Total"

# Debe mostrar: 14/14 PASS
cat gui_validation.txt | grep -E "PASS|FAIL|Total"
```

### Paso 3: Copiar resultados aqui

Pega el contenido de `test_results.txt` (o al menos las ultimas 10 lineas) en el chat para que pueda analizar los resultados.

---

## 7. Troubleshooting Rapido

| Problema | Causa | Solucion |
|----------|-------|----------|
| `ModuleNotFoundError: customtkinter` | Dependencias faltantes | `pip install -r requirements.txt` |
| `ModuleNotFoundError: pandas` | pandas no instalado | `pip install "pandas>=2.2.0,<3.0.0"` |
| `No module named tkinter` | Python sin tkinter | `sudo apt install python3-tk` (Linux) |
| `FAILED test_*Windows*` | Tests Windows en Linux | Esperado — se saltan automaticamente |
| `SSL: CERTIFICATE_VERIFY_FAILED` | Proxy corporativo | `pip install --trusted-host pypi.org --trusted-host files.pythonhosted.org -r requirements.txt` |
| `error: could not create` | Permisos venv | `rm -rf venv && python -m venv venv` |
| Aislamiento de mocks entre modulos | Test end_to_end con save/restore en try/finally | Los mocks de customtkinter se restauran al final de cada modulo |

---

## 8. Cobertura (Opcional)

```bash
# Instalar pytest-cov
pip install pytest-cov

# Ejecutar con cobertura
venv/bin/python -m pytest tests/ --cov=. --cov-report=html --cov-report=term

# Abrir reporte HTML
# Linux: xdg-open htmlcov/index.html
# Windows: start htmlcov\index.html
```

Cobertura actual estimada: ~66% global.

---

## 9. Estructura de Tests

```
tests/
├── conftest.py                    # Fixtures: tk_root_session, pump(), mock_*
├── validate_backend.py            # Validacion standalone (12 checks)
├── validate_gui_headless.py       # Validacion standalone (14 checks)
├── test_vault.py                  # SecurityVault
├── test_project_manager.py        # ProjectManager
├── test_excel_handler.py          # ExcelHandler + Finding
├── test_excel_importer.py         # Importacion Excel
├── test_orchestrator.py           # Orchestrator
├── test_sandbox.py                # SandboxRunner
├── test_provider_registry.py      # ProviderRegistry
├── test_ticket_service.py         # AzureDevOps + Jira providers
├── test_sharepoint_provider.py    # SharePoint provider
├── test_http_provider.py          # HTTP generic provider
├── test_git_manager.py            # GitManager
├── test_task_scheduler.py         # TaskScheduler (Windows-only tests)
├── test_i18n.py                   # Internacionalizacion
├── test_config_atomic.py          # ConfigManager atomico
├── test_gui.py                    # Tests GUI basicos
├── test_gui_views.py              # Tests de vistas GUI
├── test_gui_settings_view.py      # Settings view
├── test_gui_provider_management.py # Provider management view
├── test_gui_tickets_view.py       # Tickets view
├── test_gui_scripts_view.py       # Scripts view
├── test_gui_scripts_view_actions.py # Scripts view acciones
├── test_gui_project_view.py       # Project view
├── test_gui_integration.py        # GUI integration
├── test_gui_profile_system.py     # Profile system GUI
├── test_gui_evidence_sharepoint.py # Evidence SharePoint GUI
├── test_gui_git_advanced.py       # Git advanced GUI
├── test_gui_provider_dialogs.py   # Provider dialogs
├── test_profile_system.py         # ProfileSystem logic
├── test_profile_edge_cases.py     # Profile edge cases
├── test_security_startup.py       # Security startup
├── test_report_manager.py         # ReportManager
├── test_log_panel.py              # LogPanel
├── test_regression_v090.py        # Regression tests v0.9.0
├── test_pump_regression.py        # Tk pump regression
├── test_treeview_utils.py         # Treeview utilities
├── test_utils.py                  # Utility functions
├── test_utils_extended.py         # Extended utilities
├── test_connection_templates.py   # Connection templates
├── test_deploy_paths.py           # Deploy paths
├── test_dialogs.py                # Dialogs
├── test_evidence_all_types.py     # All evidence types
├── test_evidence_manager.py       # EvidenceManager
├── test_import_all_formats.py     # Import all formats
├── test_main.py                   # Main entry point
├── test_provider_configurations.py # Provider configurations
├── test_jira_provider.py          # Jira provider
├── test_integration_evidence_pipeline.py # Evidence pipeline
├── test_integration_pipeline.py   # Integration pipeline
├── test_integration_vault_advanced.py # Vault advanced integration
├── test_orchestrator_e2e.py       # Orchestrator E2E
├── data/
│   └── test_data_handlers.py      # Data handlers
├── gui/
│   └── test_views_comprehensive.py # Comprehensive GUI views
├── integration/
│   └── test_end_to_end.py         # End-to-end integration
└── profile/
    └── test_profile_suite.py      # Profile test suite
```

---

## 10. Convenciones

- **Nombre**: `test_<modulo>.py` (PROMPT.md Regla 10)
- **Clases**: `class TestNombreModulo(unittest.TestCase)`
- **Fixtures**: `tk_root_session` (session-scoped), `pump()`, `mock_project_manager`
- **GUI tests**: usar `pump(widget)` para procesar eventos Tk sin mostrar ventana
- **Datos**: usar `tmp_path` (pytest fixture) — nunca rutas fijas
- **Mock**: `@patch("modulo.funcion")` en setUp/tearDown, no a nivel modulo
- **Plataforma**: `@pytest.mark.skipif(sys.platform != "win32")` para Windows-only
