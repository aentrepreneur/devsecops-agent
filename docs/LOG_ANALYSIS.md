# LOG_ANALYSIS.md — Guia de Logs para Analisis Cruzado entre Ambientes
# =================================================================

# LOG_ANALYSIS.md — Analisis de Logs y Archivos de Salida

> **Proposito**: Guiar a otros agentes/LLMs/ambientes en el analisis de logs
> de DSOAgent para diagnostico, depuracion y mejora de procesos.
>
> **Version**: 0.9.8 | Julio 2026

---

## 1. Mapa Completo de Archivos de Salida

### 1.1 Logs de Aplicacion

| Archivo | Ubicacion | Formato | Contenido |
|---------|-----------|---------|-----------|
| `dsoagent.log` | `logs/dsoagent.log` | Texto plano | Log principal de la aplicacion |
| `dsoagent.log.{1,2,3}` | `logs/` | Texto plano | Backups de rotacion (max 10MB c/u) |
| `clone_YYYYMMDD_HHMMSS.log` | `logs/` | Texto plano | Resultado de git clone |
| `pull_YYYYMMDD_HHMMSS.log` | `logs/` | Texto plano | Resultado de git pull |
| `dsoagent_logs_YYYYMMDD_HHMMSS.log` | CWD (exportado) | Texto plano | Exportacion manual desde GUI |

### 1.2 Logs por Entorno

| Entorno | Ruta | Notas |
|---------|------|-------|
| **Dev (venv/source)** | `<raiz>/logs/dsoagent.log` | Rotacion 10MB, 3 backups |
| **Windows .exe** | `%APPDATA%\DSOAgent\logs\dsoagent.log` | Via `utils/paths.py` |
| **Linux .bin** | `~/.dsoagent/logs/dsoagent.log` | Fallback py2exe |
| **Docker** | `/app/logs/dsoagent.log` (volumen `agent-logs`) | `LOG_LEVEL=INFO` en Dockerfile |

### 1.3 Archivos de Configuracion

| Archivo | Ruta (dev) | Ruta (frozen) | Contenido |
|---------|------------|---------------|-----------|
| `dsoagent_config.json` | `config/` | `%APPDATA%/DSOAgent/config/` | AppSettings (JSON plaintext) |
| `vault.key` | `config/` | `%APPDATA%/DSOAgent/` | Clave Fernet (cifrada) |
| `recovery.key` | `config/` | `%APPDATA%/DSOAgent/` | Clave recuperacion |
| `providers/types.json` | `config/providers/` | Igual | Tipos de proveedor |
| `providers/connections.json` | `config/providers/` | Igual | Conexiones guardadas |

### 1.4 Datos de Proyecto y Perfil

| Archivo | Ruta (dev) | Contenido |
|---------|------------|-----------|
| `projects.json` | `projects/` | Indice de proyectos (plaintext) |
| `{project_id}.json` | `projects/` | Datos por proyecto (cifrados) |
| `index.json` | `profiles/` | Indice de perfiles |
| `{id}/metadata.json` | `profiles/{id}/` | Metadata publica del perfil |
| `{id}/data.enc` | `profiles/{id}/` | Datos cifrados del perfil |
| `scheduled_tasks.json` | `data/` | Estado del scheduler |

### 1.5 Archivos de Build

| Archivo | Ruta | Contenido |
|---------|------|-----------|
| `build_output.log` | raiz proyecto | Output de PyInstaller |
| `build_output.txt` | raiz proyecto | Output de PyInstaller (texto) |

---

## 2. Formato del Log Principal

### 2.1 Estructura de Linea

```
DD-MM-YY HH:MM:SS - nombre_modulo - LEVEL - [funcion] - mensaje
```

Ejemplo:
```
13-07-26 14:30:15 - agent.orchestrator - INFO - [run_analysis] - [Orchestrator] Iniciando analisis proyecto CASE-2026-001
13-07-26 14:30:16 - security.vault - WARNING - [decrypt] - [SecurityVault] Clave maestra cacheada
13-07-26 14:30:17 - integrations.azure_provider - ERROR - [create_work_item] - [AzureProvider] Error HTTP 429: rate limit
```

### 2.2 Niveles de Log

| Nivel | Uso | Ejemplo |
|-------|-----|---------|
| `DEBUG` | Detalle tecnico (solo archivo) | Valores de variables, paso a paso |
| `INFO` | Operaciones exitosas | `[Orchestrator] Analisis completado` |
| `WARNING` | Situaciones recuperables | `[AzureProvider] Rate limit, retry en 2s` |
| `ERROR` | Fallos que requieren atencion | `[JiraProvider] Auth fallida` |
| `CRITICAL` | Fallos criticos del sistema | `[SecurityVault] Clave maestra no encontrada` |

### 2.3 Prefijos de Modulo

Todos los logs usan prefijo `[Modulo]` para identificacion rapida:

| Prefijo | Modulo | Archivo |
|---------|--------|---------|
| `[Orchestrator]` | Nucleo principal | `agent/orchestrator.py` |
| `[SecurityVault]` | Cifrado | `security/vault.py` |
| `[ConfigManager]` | Configuracion | `data/config_manager.py` |
| `[ProfileManager]` | Perfiles | `data/profile_manager.py` |
| `[ProjectManager]` | Proyectos | `data/project_manager.py` |
| `[EvidenceManager]` | Evidencias | `data/evidence_manager.py` |
| `[ExcelHandler]` | Excel | `data/excel_handler.py` |
| `[ExcelImporter]` | Importacion | `data/excel_importer.py` |
| `[ReportManager]` | Reportes | `data/report_manager.py` |
| `[ExportImportManager]` | Export/Import | `data/export_import_manager.py` |
| `[MultiProfileReport]` | Reporte multi-perfil | `data/multi_profile_report_reader.py` |
| `[ProviderRegistry]` | Proveedores | `data/provider_registry.py` |
| `[AzureProvider]` | Azure DevOps | `integrations/azure_provider.py` |
| `[JiraProvider]` | Jira | `integrations/jira_provider.py` |
| `[SharePointProvider]` | SharePoint | `integrations/sharepoint_provider.py` |
| `[GitManager]` | Git | `integrations/git_manager.py` |
| `[HttpProvider]` | HTTP | `integrations/http_provider.py` |
| `[TicketService]` | Tickets | `integrations/ticket_service.py` |
| `[SandboxRunner]` | Sandbox | `sandbox/runner.py` |
| `[TaskScheduler]` | Scheduler | `utils/task_scheduler.py` |
| `[HealthCheck]` | Health | `utils/health.py` |

---

## 3. Patron de Analisis Cruzado

### 3.1 Flujo de Analisis

```
1. Recopilar logs de todos los ambientes
2. Normalizar formato (los timestamps pueden variar)
3. Buscar patrones de error por modulo
4. Correlacionar timestamps entre ambientes
5. Identificar diferencias de comportamiento
6. Generar reporte con hallazgos
```

### 3.2 Scripts de Recopilacion

**Windows (PowerShell):**
```powershell
# Copiar logs a directorio de analisis
$timestamp = Get-Date -Format "yyyyMMdd_HHmmss"
$dest = "analisis_$timestamp"
New-Item -ItemType Directory -Path $dest

# Logs de aplicacion
Copy-Item "$env:APPDATA\DSOAgent\logs\dsoagent.log" "$dest\"
Copy-Item "$env:APPDATA\DSOAgent\logs\dsoagent.log.*" "$dest\" -ErrorAction SilentlyContinue

# Config
Copy-Item "$env:APPDATA\DSOAgent\config\dsoagent_config.json" "$dest\"

# Projects
Copy-Item "$env:APPDATA\DSOAgent\projects\projects.json" "$dest\"

Write-Host "Logs recopilados en: $dest"
```

**Linux (Bash):**
```bash
#!/bin/bash
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
DEST="analisis_$TIMESTAMP"
mkdir -p "$DEST"

# Logs de aplicacion
cp ~/.dsoagent/logs/dsoagent.log "$DEST/" 2>/dev/null
cp ~/.dsoagent/logs/dsoagent.log.* "$DEST/" 2>/dev/null

# Config
cp ~/.dsoagent/config/dsoagent_config.json "$DEST/" 2>/dev/null

# Projects
cp ~/.dsoagent/projects/projects.json "$DEST/" 2>/dev/null

echo "Logs recopilados en: $DEST"
```

**Docker:**
```bash
# Extraer logs del volumen
docker compose cp agent-core:/app/logs/ ./analisis_docker/

# O montar volumen temporalmente
docker run --rm -v agent-logs:/logs -v $(pwd)/analisis:/analisis \
    alpine sh -c "cp /logs/dsoagent.log /analisis/"
```

### 3.3 Comandos de Analisis Rapido

```bash
# Errores del ultimo dia
grep -i "ERROR\|CRITICAL" logs/dsoagent.log | tail -50

# Errores por modulo
grep "\[AzureProvider\].*ERROR" logs/dsoagent.log
grep "\[JiraProvider\].*ERROR" logs/dsoagent.log
grep "\[SecurityVault\].*ERROR" logs/dsoagent.log

# Warnings de rate limiting
grep "rate limit\|429\|retry" logs/dsoagent.log

# Operaciones lentas (>5s)
grep "elapsed\|duration\|timeout" logs/dsoagent.log

# Timeline de una sesion especifica
grep "13-07-26" logs/dsoagent.log | head -100

# Conteo de errores por modulo
grep "ERROR" logs/dsoagent.log | sed 's/.*\[\(.*\)\].*/\1/' | sort | uniq -c | sort -rn
```

---

## 4. Correlacion entre Ambientes

### 4.1 Que Buscar

| Fenomeno | Windows | Linux | Docker | Accion |
|----------|---------|-------|--------|--------|
| Fallo import | Verificar `python3` vs `python` | Verificar path venv | Verificar `WORKDIR` | Comparar traceback |
| Archivo no encontrado | `%APPDATA%` path | `~/.dsoagent/` path | `/app/` path | Verificar `utils/paths.py` |
| Timeout de red | Firewall, proxy | iptables, proxy | Network bridge | Comparar tiempos |
| Fallo cifrado | `vault.key` corrupto | `vault.key` corrupto | Volumen corrupto | Regenerar vault |
| GUI crash | Display, Tcl/Tk | X11, display | Xvfb | Verificar DISPLAY |

### 4.2 Formato de Reporte Cruzado

```markdown
## Reporte de Analisis Cruzado — [Fecha]

### Ambientes Analizados
- Windows: [version], Python [version]
- Linux: [version], Python [version]
- Docker: imagen [tag]

### Hallazgos

#### [MODULO] — [ Severidad ]
- **Windows**: [comportamiento]
- **Linux**: [comportamiento]
- **Docker**: [comportamiento]
- **Diferencia**: [descripcion de la diferencia]
- **Recomendacion**: [accion sugerida]

### Errores Comunes
| Modulo | Windows | Linux | Docker |
|--------|---------|-------|--------|
| [mod] | [count] | [count] | [count] |

### Logs Adjuntos
- `dsoagent_windows.log`
- `dsoagent_linux.log`
- `dsoagent_docker.log`
```

---

## 5. Metricas Extraibles del Log

### 5.1 Contadores Automaticos

```bash
# Total de operaciones por tipo
grep -c "Iniciando" logs/dsoagent.log    # Operaciones totales
grep -c "completado\|exitoso" logs/dsoagent.log  # Exitosos
grep -c "ERROR" logs/dsoagent.log         # Errores
grep -c "WARNING" logs/dsoagent.log       # Warnings

# Tiempo promedio de operaciones (si el log tiene timestamps de inicio/fin)
grep "Iniciando\|completado" logs/dsoagent.log | tail -20

# Proveedor masUsed
grep "\[AzureProvider\]" logs/dsoagent.log | wc -l
grep "\[JiraProvider\]" logs/dsoagent.log | wc -l
grep "\[SharePointProvider\]" logs/dsoagent.log | wc -l
```

### 5.2 Deteccion de Anomalias

```bash
# Picos de errores en ventana de 5 minutos
awk '/ERROR/{ts=substr($1,1,5); count[ts]++} END{for(t in count) if(count[t]>10) print t, count[t]}' logs/dsoagent.log

# Modulos con mas warnings
grep "WARNING" logs/dsoagent.log | grep -oP '\[\K[^\]]+' | sort | uniq -c | sort -rn | head -10

# Errores unicos (deduplicados)
grep "ERROR" logs/dsoagent.log | sort -u | head -20
```

---

## 6. Integracion con Otros Agentes/LLMs

### 6.1 Para Agentes de QA

Recopilar logs de 3 ambientes, comparar:
1. Mismos tests pasan/fallan en todos?
2. Errores de entorno vs errores de codigo?
3. Performance inconsistente entre ambientes?

### 6.2 Para Agentes de DevOps

Analizar:
1. Docker: healthcheck pasando?
2. Volumenes montados correctamente?
3. Permisos de archivos en volumen?
4. Red entre servicios funcionando?

### 6.3 Para Agentes de Seguridad

Auditar:
1. Sin credenciales en logs (R3 de PROMPT.md)
2. Vault.key protegido (permisos, ubicacion)
3. Sin .env en repositorio
4. Archivos .enc accesibles solo por la app

### 6.4 Plantilla de Prompt para Otros Agentes

```
Analiza los logs de DSOAgent en esta sesion:

ARCHIVOS:
- logs/dsoagent.log (log principal)
- config/dsoagent_config.json (config actual)
- tests/validate_backend.py (script de validacion)

CONTEXTO:
- Proyecto: DSOAgent v0.9.8 (Pre-RC1)
- Stack: Python 3.10+ | CustomTkinter | asyncio
- Arquitectura: gui/ -> agent/ -> tools/ -> security/ -> utils/

TAREA:
1. Identificar errores o warnings inusuales
2. Correlacionar con cambios recientes en codigo
3. Proponer mejoras basadas en patrones observados
4. Usar formato: modulo: hallazgo: recomendacion

REGLAS:
- Seguir PROMPT.md para estilo de respuesta
- Sin output narrativo, solo datos y recomendaciones
```

---

*Development by Angel Esquivel (CyberSecurity) [DSOAgent 2026]*
