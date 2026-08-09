# DSOAgent — Guía Rápida

> **Agente de escritorio para gestión de ciberseguridad**  
> Versión: v0.9.8-RC1 | Julio 2026

---

## ¿Qué es DSOAgent?

DSOAgent es una aplicación de escritorio (Windows/Ubuntu) que automatiza la gestión de hallazgos de seguridad, evidencias digitales y sincronización con sistemas de tickets (Azure DevOps, Jira).

**En términos simples**: Es un "organizador inteligente" para equipos de ciberseguridad que conecta el análisis de vulnerabilidades con los sistemas donde se reportan tickets.

---

## ¿Para qué sirve?

- 📊 **Importar hallazgos** desde 15 herramientas de seguridad (SonarQube, Checkmarx, Burp, Nessus, etc.)
- 📁 **Gestionar evidencias** digitales (capturas, logs, scripts)
- 🎫 **Sincronizar con tickets** en Azure DevOps o Jira
- ☁️ **Subir evidencias a SharePoint** (Microsoft 365)
- 🤖 **Ejecutar scripts** de automatización en sandbox
- 🔐 **Cifrar datos** de proyecto con clave de recuperación

---

## Instalación (3 pasos)

### Windows (ejecutable)

1. Descargar `DSOAgent.exe` (~45 MB)
2. **No requiere instalador** — doble clic para abrir
3. La primera vez creará automáticamente:
   - Vault de seguridad en `%APPDATA%\DSOAgent\`
   - Archivo de configuración local
   - **Guardar la clave de recuperación** que se muestra

### Ubuntu (código fuente)

```bash
git clone <repo>
cd DSOAgent
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
python main.py
```

### Docker (servidor)

```bash
docker-compose up -d
```

---

## Uso Básico

### 1. Crear un Proyecto

```
Menú: Proyectos → Nuevo
↓
Nombre: "Auditoría Banco XYZ"
Cliente: "Banco XYZ"
↓
Guardar
```

El proyecto recibe ID automático: `CASE-2026-001`

### 2. Conectar con Azure DevOps / Jira

```
Menú: Configuración → Proveedores → Nuevo
↓
Tipo: Azure DevOps (o Jira)
Nombre: "Conexión Principal"
URL: https://dev.azure.com/organizacion
Token: [tu Personal Access Token]
↓
Probar Conexión → Guardar
```

### 3. Importar Hallazgos

```
Menú: Proyecto Activo → Importar Excel
↓
Seleccionar archivo .xlsx del scanner
↓
Detecta automáticamente: SonarQube / Checkmarx / Burp / etc.
↓
Revisar hallazgos → Importar
```

### 4. Subir Evidencias

```
Menú: Evidencias → Agregar
↓
Arrastrar archivos: .png, .pdf, .log, .py
↓
Genera hash SHA-256 automáticamente
↓
[Opcional] Subir a SharePoint
```

### 5. Crear Tickets

```
Menú: Tickets → Sincronizar
↓
Seleccionar hallazgos a reportar
↓
Crea automáticamente work items en Azure/Jira
↓
Enlaza evidencias adjuntas
```

---

## Arquitectura Simple

```
┌─────────────────────────────────────┐
│           DSOAgent.exe              │
│  (Aplicación de escritorio Windows) │
└──────────────┬──────────────────────┘
               │
    ┌──────────┼──────────┐
    │          │          │
┌───▼───┐  ┌──▼───┐  ┌────▼────┐
│ Vault │  │ datos│  │ evidencia│
│cifrado│  │ JSON │  │  local   │
└───┬───┘  └──────┘  └────┬─────┘
    │                      │
    └──────────┬───────────┘
               │
    ┌──────────▼──────────┐
    │   Integraciones      │
    │  Azure | Jira | Git  │
    │  SharePoint | HTTP   │
    └──────────────────────┘
```

**Datos**: Almacenados localmente en `%APPDATA%\DSOAgent\` (Windows) o `~/.dsoagent/` (Linux).  
**Cifrado**: AES-128 con clave derivada del hardware + opcional contraseña maestra.

---

## Requisitos del Sistema

| Requisito | Mínimo | Recomendado |
|-----------|--------|-------------|
| Sistema | Windows 10/11 | Windows 11 |
| RAM | 4 GB | 8 GB |
| Disco | 100 MB | 500 MB (evidencias) |
| Red | Opcional | Internet (integraciones cloud) |

**Sin dependencias externas**: No requiere Python, SQL Server, ni instaladores complejos.

---

## Seguridad y Recuperación

### Clave de Recuperación

Al primer arranque, DSOAgent genera una **clave de recuperación** ( formato: `XXXX-XXXX-XXXX-XXXX` ).

**IMPORTANTE**: Guardar en lugar seguro (fuera de la PC). Se necesita si:
- Se cambia de computadora
- Se formatea el disco
- Se cambia el usuario Windows
- Se actualiza hardware (placa base)

### Sin Clave de Recuperación

Si no guardaste la clave y pierdes acceso:
- Los datos de proyecto **no se pueden recuperar**
- Debes crear nuevo vault y reproyectar desde cero

---

## Troubleshooting

| Problema | Solución |
|----------|----------|
| "No se detecta proveedor" al importar | Verificar formato Excel. Usar plantillas en `docs/manuales/` |
| Error conexión Azure/Jira | Revisar token no expirado. Probar en Configuración → Test |
| Ventana no aparece tras minimizar | Click en icono bandeja del sistema |
| Archivos de evidencia grandes | Configurar ruta evidencias en Configuración → Directorios |

---

## Soporte

- **Documentación técnica**: `docs/TECHNICAL.md`
- **Changelog**: `docs/CHANGELOG.md`
- **Email soporte**: [agregar email institucional]

---

> **Desarrollo**: Angel Esquivel — CyberSecurity DSOAgent 2026  
> **Licencia**: Interna — Equipo de Ciberseguridad

#End Development By Angel Esquivel (CyberSecurity) [DSOAgent 2026]
