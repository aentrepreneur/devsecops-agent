# Development by Angel Esquivel (CyberSecurity) [DSOAgent] [Jul-2026]

# GUI_TEST_GUIDE.md — Guia de Prueba Manual de la GUI DSOAgent

> **Objetivo**: Probar cada ruta, boton y opcion de la interfaz grafica de forma sistematica.
> **Version**: v0.9.8-RC1 | Julio 2026
> **Plataforma**: Windows 11 (requiere display para GUI)
> **Requisitos**: Python 3.10+, venv activado, dependencias instaladas

---

## Comandos de Ejecucion

Ejecutar desde `C:\VSCode\DSOAgent_v1_2026\` en PowerShell:

```powershell
# 1. PRERREQUISITOS
python --version

# 2. ACTIVAR VENV
venv\Scripts\activate

# 3. SUITE COMPLETA
pytest tests\ -v --tb=short

# 4. VALIDACION BACKEND
python tests\validate_backend.py

# 5. VALIDACION GUI HEADLESS
python tests\validate_gui_headless.py

# 6. LANZAR APP
python main.py

# 7. COMPILAR .exe (opcional)
pyinstaller DSOAgent.spec --clean

# 8. VALIDAR .exe (post-build)
powershell -ExecutionPolicy Bypass -File scripts\validate_exe.ps1
```

### Archivos a copiar si se actualizan desde otro entorno

```
docs\TESTING.md
docs\ARCHITECTURE.md
docs\TECHNICAL.md
docs\CHANGELOG.md
docs\GUI_TEST_GUIDE.md
docs\GUÍA_RÁPIDA.md
docs\PLANNING.md
README.md
QUICKSTART.md
PROMPT.md
```

---

## 1. FLUJO DE ARRANQUE — Login y Perfiles

### 1.1 Primera ejecucion

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 1.1.1 | Ejecutar `python main.py` | Ventana de Login se abre centrada (500x600) | [ ] |
| 1.1.2 | Verificar titulo | "DSOAgent - Inicio de Sesion" | [ ] |
| 1.1.3 | Verificar deteccion de usuario | Muestra "Usuario: \<USERNAME\>" (lee `%USERNAME%` del SO) | [ ] |
| 1.1.4 | Sin perfiles creados | Mensaje "No se encontraron perfiles. Crea tu primer perfil:" + formulario visible | [ ] |

### 1.2 Crear perfil

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 1.2.1 | Dejar nombre vacio + intentar crear | Error "Ingresa un nombre de perfil" | [ ] |
| 1.2.2 | Poner nombre + password < 8 chars | Error "La contrasena debe tener al menos 8 caracteres" | [ ] |
| 1.2.3 | Poner password + confirmar diferente | Error "Las contrasenas no coinciden" | [ ] |
| 1.2.4 | Nombre valido + password 8+ chars + confirmar | Perfil creado, vault.key + recovery.key en `%APPDATA%\DSOAgent\` | [ ] |
| 1.2.5 | Verificar vault.key existe | `dir %APPDATA%\DSOAgent\` muestra vault.key y recovery.key | [ ] |

### 1.3 Login

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 1.3.1 | Seleccionar perfil + password incorrecta | Error "Clave-Maestra incorrecta" | [ ] |
| 1.3.2 | Seleccionar perfil + password correcta | Ventana principal se abre (1200x730), Login se cierra | [ ] |
| 1.3.3 | Verificar status bar | Muestra "Perfil: \<profile_id\>" | [ ] |

---

## 2. BARRA SUPERIOR — Siempre Visible

| Paso | Elemento | Accion | Resultado esperado | OK |
|------|----------|--------|--------------------|----|
| 2.1 | Combo proyecto | Seleccionar proyecto de la lista | Cambia contexto global, status bar: "Proyecto activo: \<nombre\>" | [ ] |
| 2.2 | Sin proyectos | Combo muestra "(sin proyectos)" + banner | Se oculta al crear primer proyecto | [ ] |
| 2.3 | Theme switch | Cambiar a "Claro" | Todas las vistas se recrean en tema claro | [ ] |
| 2.4 | Theme switch | Cambiar a "Oscuro" | Todas las vistas se recrean en tema oscuro | [ ] |
| 2.5 | Cerrar Sesion | Hacer clic | Vuelve al dialogo de Login, limpia sesion | [ ] |

---

## 3. SIDEBAR — 13 Navegaciones

### 3.1 Dashboard

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.1.1 | Navegar a Dashboard | Panel principal visible con tipos de analisis: SAST-Codigo, SAST-App, DAST-Web, DAST-App | [ ] |
| 3.1.2 | Ver metricas severidad | 5 badges: Critico (#d32f2f), Alto (#f57c00), Medio (#fbc02d), Bajo (#388e3c), Info (#1976d2) | [ ] |
| 3.1.3 | Ver metricas estado | 5 estados: Nuevo, Abierto, En Progreso, Resuelto, Cerrado | [ ] |
| 3.1.4 | Boton "Generar Reporte" | Genera reporte Excel del proyecto activo | [ ] |

### 3.2 Proyectos

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.2.1 | Navegar a Proyectos | Treeview con columnas: ID, Nombre, Proveedor, Ticket Proyecto, Creado | [ ] |
| 3.2.2 | Boton "Nuevo" | Formulario se limpia, modo "nuevo" activado | [ ] |
| 3.2.3 | Guardar sin nombre | Validacion: campo nombre es obligatorio | [ ] |
| 3.2.4 | Seleccionar proveedor | ComboBox con tipos del ProviderRegistry (7 tipos builtin) | [ ] |
| 3.2.5 | Ticket Project ID | Campo de texto libre para ID del proyecto en el proveedor | [ ] |
| 3.2.6 | Excel Path + "Examinar..." | Filedialog para seleccionar archivo .xlsx | [ ] |
| 3.2.7 | Evidence Dir + "Examinar..." | Filedialog para seleccionar carpeta | [ ] |
| 3.2.8 | Guardar proyecto nuevo | Crea proyecto en `%APPDATA%\DSOAgent\projects\`, aparece en treeview | [ ] |
| 3.2.9 | Seleccionar fila del tree | Carga datos en formulario, modo "editar" | [ ] |
| 3.2.10 | Actualizar (editar) | Guarda cambios, treeview se refresca | [ ] |
| 3.2.11 | Eliminar proyecto | Confirmacion "Eliminar este elemento?" → borra proyecto | [ ] |
| 3.2.12 | Abrir Excel | Abre el archivo .xlsx en app por defecto | [ ] |
| 3.2.13 | Abrir Evidencia | Abre la carpeta de evidencias en explorador de Windows | [ ] |
| 3.2.14 | F5 | Refresca la lista de proyectos | [ ] |
| 3.2.15 | Click en fila | Selecciona proyecto como activo en combo superior | [ ] |

### 3.3 Tickets

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.3.1 | Navegar a Tickets | Selector de proveedor + treeview de tickets | [ ] |
| 3.3.2 | Seleccionar proveedor | ComboBox con tipos category='tickets' (Azure DevOps, Jira, etc.) | [ ] |
| 3.3.3 | Conectar | Boton "Conectar" → status: Conectado/Desconectado | [ ] |
| 3.3.4 | Filtro estado | ComboBox: Todos/new/open/in_progress/resolved/closed | [ ] |
| 3.3.5 | Treeview tickets | Columnas: ID, Titulo, Estado, Prioridad, Asignado | [ ] |
| 3.3.6 | Crear ticket | Boton "Nuevo Ticket" → formulario con titulo, descripcion, prioridad, tipo | [ ] |
| 3.3.7 | Ver detalle | Seleccionar ticket → panel de descripcion completa | [ ] |
| 3.3.8 | Copiar ID | Ctrl+C copia ID del ticket seleccionado | [ ] |
| 3.3.9 | F5 | Refresca lista de tickets | [ ] |

### 3.4 Evidencias

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.4.1 | Navegar a Evidencias | Explorador de archivos en `projects/<id>/evidence/` | [ ] |
| 3.4.2 | Filtro categoria | ComboBox: (todas), screenshots, logs, documents, code, network, config, reports, other | [ ] |
| 3.4.3 | Cargar archivo | Boton "Cargar" → filedialog con filtros (Imagenes, Documentos, Logs, Codigo) | [ ] |
| 3.4.4 | Subir a SharePoint | Boton "Subir" → envia archivo via Graph API | [ ] |
| 3.4.5 | Abrir archivo | Seleccionar + Boton "Abrir" → abre con app del sistema | [ ] |
| 3.4.6 | Eliminar archivo | Confirmacion → borra archivo de disco | [ ] |
| 3.4.7 | Abrir carpeta | Boton "Abrir Carpeta" → abre explorador de Windows | [ ] |
| 3.4.8 | Estadisticas | Muestra tamano total, archivos por categoria | [ ] |
| 3.4.9 | Hash SHA-256 | Cada archivo tiene hash calculado | [ ] |
| 3.4.10 | F5 | Refresca lista | [ ] |

### 3.5 Reportes

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.5.1 | Navegar a Reportes | Treeview de reportes consolidados por proyecto | [ ] |
| 3.5.2 | Metricas | Totales por severidad y estado | [ ] |
| 3.5.3 | Exportar Excel | Boton "Exportar" → genera .xlsx con metricas | [ ] |
| 3.5.4 | F5 | Refresca datos | [ ] |

### 3.6 Reporte Multi-Perfil

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.6.1 | Navegar a Reporte Multi-Perfil | Dialogo se abre (700x600), titulo "Reporte Multi-Perfil" | [ ] |
| 3.6.2 | Lista perfiles | Muestra otros perfiles en el mismo VM con checkboxes | [ ] |
| 3.6.3 | Solicitar acceso | Ingresar Clave-Maestra de otro perfil | [ ] |
| 3.6.4 | Rango fechas | Campos fecha inicio/fin para filtrado temporal | [ ] |
| 3.6.5 | Filtro proyecto | ComboBox de proyectos | [ ] |
| 3.6.6 | Generar reporte | Consolida datos de perfiles seleccionados | [ ] |
| 3.6.7 | Validaciones cruzadas | Muestra duplicados y gaps de timeline | [ ] |

### 3.7 Exportar Perfil

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.7.1 | Navegar a Exportar Perfil | Dialogo 600x500, requiere perfil autenticado | [ ] |
| 3.7.2 | Sin perfil autenticado | Status bar: "No hay perfil autenticado" | [ ] |
| 3.7.3 | Checkbox "Exportacion completa" | Default: True, incluye todo el perfil | [ ] |
| 3.7.4 | Checkbox "Incluir evidencias" | Default: True | [ ] |
| 3.7.5 | Checkbox "Incluir proveedores" | Default: True | [ ] |
| 3.7.6 | Checkbox "Incluir configuracion" | Default: True | [ ] |
| 3.7.7 | Password proteccion | Campo password (min 8 chars), requerido | [ ] |
| 3.7.8 | Exportar | Genera archivo `.dsoprofile` cifrado | [ ] |

### 3.8 Importar Perfil

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.8.1 | Navegar a Importar Perfil | File dialog abre, filtro `*.dsoprofile` | [ ] |
| 3.8.2 | Seleccionar archivo | Input dialog para password del archivo exportado | [ ] |
| 3.8.3 | Importar | Carga datos del perfil, perfil disponible en login | [ ] |

### 3.9 Git Repos

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.9.1 | Navegar a Git Repos | Cabecera: proyecto activo + URL base + carpeta destino | [ ] |
| 3.9.2 | Treeview repos | Lista de repositorios pendientes (URL, branch, destino) | [ ] |
| 3.9.3 | Agregar repo | Boton "Agregar" → campos URL, branch, destino | [ ] |
| 3.9.4 | Clonar | Ejecuta `git clone`, consola muestra salida en tiempo real | [ ] |
| 3.9.5 | Pull | Ejecuta `git pull` en repos existentes | [ ] |
| 3.9.6 | Consola | Salida tipo terminal (fondo negro `#0d0d0d`, texto verde `#00ff41`, Courier New 10) | [ ] |
| 3.9.7 | Barra de progreso | Muestra avance de operacion | [ ] |
| 3.9.8 | Eliminar repo | Select + Delete → quita de la lista | [ ] |

### 3.10 Tareas Programadas

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.10.1 | Navegar a Tareas | Cabecera: proyecto activo + boton "Abrir Programador de Windows" | [ ] |
| 3.10.2 | Abrir Programador | Ejecuta `taskschd.msc` | [ ] |
| 3.10.3 | Treeview tareas | Lista tareas DSOAgent: Nombre, Proxima ejecucion, Estado | [ ] |
| 3.10.4 | Tipo de tarea | ComboBox: "--Seleccione--", "Script personalizado (.ps1)", "Script del sistema (DSOAgent)" | [ ] |
| 3.10.5 | Nombre | Campo nombre de la tarea | [ ] |
| 3.10.6 | Script + "Examinar..." | Campo ruta de archivo .ps1/.py/.bat | [ ] |
| 3.10.7 | Iniciar en | Campo directorio de trabajo | [ ] |
| 3.10.8 | Argumentos | Campo argumentos del script | [ ] |
| 3.10.9 | Frecuencia | Diaria / Semanal / Mensual / Una vez | [ ] |
| 3.10.10 | Hora | Campo hora de ejecucion | [ ] |
| 3.10.11 | Fecha | Campo fecha (solo para "Una vez") | [ ] |
| 3.10.12 | Programar | Registra tarea en Windows Task Scheduler via `schtasks.exe`, `/TR` usa `cd /d "<dir>" && <script>` | [ ] |
| 3.10.13 | Limpiar | Elimina tareas seleccionadas de Task Scheduler | [ ] |
| 3.10.14 | F5 | Refresca lista de tareas | [ ] |

### 3.11 Scripts

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.11.1 | Navegar a Scripts | Cabecera: proyecto activo | [ ] |
| 3.11.2 | Tipo script | ComboBox: "--Seleccione--", "PowerShell (.ps1)", "Python (.py)", "Batch (.bat/.cmd)" | [ ] |
| 3.11.3 | Nombre | Campo nombre del script | [ ] |
| 3.11.4 | Script + "Examinar..." | Campo ruta del archivo | [ ] |
| 3.11.5 | Config + "Examinar..." | Campo ruta config JSON | [ ] |
| 3.11.6 | Template + "Examinar..." | Campo ruta template | [ ] |
| 3.11.7 | Treeview scripts | Lista: Nombre, Tipo, Script, Configuracion | [ ] |
| 3.11.8 | Agregar | Agrega script a la lista | [ ] |
| 3.11.9 | Editar | Seleccionar fila → carga en formulario | [ ] |
| 3.11.10 | Eliminar | Confirmacion → quita de lista | [ ] |
| 3.11.11 | Ejecutar en Sandbox | Boton "Ejecutar en Sandbox" → SandboxRunner lanza proceso | [ ] |
| 3.11.12 | Consola Sandbox | Salida en dark theme, muestra resultado | [ ] |
| 3.11.13 | Sandbox Network on/off | Toggle de red | [ ] |
| 3.11.14 | Sandbox Filesystem on/off | Toggle de sistema de archivos | [ ] |
| 3.11.15 | Sandbox Timeout | Campo segundos | [ ] |
| 3.11.16 | Resultado | Success / Failed con codigo de retorno | [ ] |
| 3.11.17 | F5 | Refresca lista | [ ] |

### 3.12 Configuracion

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.12.1 | Navegar a Configuracion | Panel izquierdo: treeview proyectos. Panel derecho: 4 tabs | [ ] |
| 3.12.2 | **Tab Informacion** | ID del Proyecto, Nombre, Cliente, Creado, Ultima modificacion | [ ] |
| 3.12.3 | Guardar cambios (info) | Guarda info del proyecto | [ ] |
| 3.12.4 | **Tab Directorios** | Evidencias, Excel, Repositorios — cada uno con ruta + "Examinar..." | [ ] |
| 3.12.5 | Guardar directorios | Guarda rutas | [ ] |
| 3.12.6 | **Tab Proveedores** | Treeview de conexiones: Nombre, Tipo, Estado | [ ] |
| 3.12.7 | Nueva Conexion | Boton "Nueva" → formulario dinamico segun tipo de proveedor | [ ] |
| 3.12.8 | Formulario Azure | Organization, Project, Token, Work Item Type | [ ] |
| 3.12.9 | Formulario Jira | Domain, Email, API Token, Project Key | [ ] |
| 3.12.10 | Formulario SharePoint | Tenant ID, Client ID, Client Secret, Site URL | [ ] |
| 3.12.11 | Formulario Git | URL Base, Token, Default Branch | [ ] |
| 3.12.12 | Formulario HTTP | URL Base, Headers (JSON), Method | [ ] |
| 3.12.13 | Probar Conexion | Boton "Probar Conexion" → test real al proveedor → Conectado/Desconectado | [ ] |
| 3.12.14 | Guardar Conexion | Guarda en ProviderRegistry | [ ] |
| 3.12.15 | Editar Conexion | Seleccionar → carga en formulario | [ ] |
| 3.12.16 | Eliminar Conexion | Confirmacion → borra | [ ] |
| 3.12.17 | **Tab Logs** | Selector de nivel: DEBUG, INFO, WARNING, ERROR, CRITICAL | [ ] |
| 3.12.18 | Cambiar nivel de logs | Filtra logs en tiempo real | [ ] |

### 3.13 Ayuda

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 3.13.1 | Navegar a Ayuda | Seccion de manuales + Acerca de | [ ] |
| 3.13.2 | Manual de Usuario | Boton → abre `docs/manuales/manual-usuario.md` | [ ] |
| 3.13.3 | Manual Tecnico | Boton → abre `docs/manuales/manual-tecnico.md` | [ ] |
| 3.13.4 | Manual de Desarrollo | Boton → abre `docs/manuales/manual-desarrollo.md` | [ ] |
| 3.13.5 | Acerca de | Version v0.9.8-RC1, descripcion, autor | [ ] |

---

## 4. BARRA DE ESTADO — Inferior

| Paso | Elemento | Resultado esperado | OK |
|------|----------|--------------------|----|
| 4.1 | Mensaje izquierda | Muestra contexto: "Proyecto activo: \<nombre\>" / "Perfil: \<id\>" / "Tarea \<id\> completada" | [ ] |
| 4.2 | Indicador derecho | "Version: v0.9.8-RC1" | [ ] |

---

## 5. SISTEMA DE RECUPERACION — Vault

### 5.1 Verificar vault

| Paso | Accion | Resultado esperado | OK |
|------|--------|--------------------|----|
| 5.1.1 | Verificar vault.key | Existe en `%APPDATA%\DSOAgent\vault.key` | [ ] |
| 5.1.2 | Verificar recovery.key | Existe en `%APPDATA%\DSOAgent\recovery.key` | [ ] |
| 5.1.3 | Verificar vault.key contenido | JSON con campos "salt" y "key" en base64 | [ ] |
| 5.1.4 | Verificar recovery.key contenido | JSON con campos "encrypted_vault_key" y "version": "2.0" | [ ] |

### 5.2 Flujo de recuperacion

| Paso | Escenario | Accion | Resultado esperado | OK |
|------|-----------|--------|--------------------|----|
| 5.2.1 | Cambio de PC | Copiar recovery.key a nueva PC | Clave disponible para restaurar | [ ] |
| 5.2.2 | Restaurar vault | Llamar `recover_with_key(clave_guardada)` | vault.key restaurado, vault funcional | [ ] |
| 5.2.3 | Verificar restauracion | Login con password original funciona | [ ] |

---

## 6. CHECKLIST RESUMEN

Marcar cada modulo al completar todas las pruebas:

```
[ ] Login/Perfil (Seccion 1)
[ ] Barra Superior (Seccion 2)
[ ] Dashboard (3.1)
[ ] Proyectos (3.2)
[ ] Tickets (3.3)
[ ] Evidencias (3.4)
[ ] Reportes (3.5)
[ ] Reporte Multi-Perfil (3.6)
[ ] Exportar Perfil (3.7)
[ ] Importar Perfil (3.8)
[ ] Git Repos (3.9)
[ ] Tareas Programadas (3.10)
[ ] Scripts (3.11)
[ ] Configuracion (3.12)
[ ] Ayuda (3.13)
[ ] Barra de Estado (Seccion 4)
[ ] Sistema de Recuperacion (Seccion 5)
```

---

## 7. REPORTAR RESULTADOS

Para reportar resultados, ejecutar y pegar la salida:

```powershell
# Suite automatizada
pytest tests\ -v --tb=short

# Validacion backend
python tests\validate_backend.py

# Validacion GUI
python tests\validate_gui_headless.py
```

Pegar las ultimas 3 lineas de cada ejecucion en el chat para validacion.
