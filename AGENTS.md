# Conocimiento para Agentes de IA

Este archivo contiene conocimiento acumulado sobre como realizar tareas especificas de manera efectiva.

---

## Indice del Repositorio

Este repositorio contiene contexto personal y conocimiento para agentes de IA que asisten a Mariano Blua.

### Archivos

| Archivo | Descripcion | Contenido Principal |
|---------|-------------|---------------------|
| `README.md` | Indice del repositorio | Lista de archivos y sus propositos |
| `USER_CONTEXT.md` | Contexto personal del usuario | Datos basicos (nombre, ubicacion, timezone), horario laboral (9:00-20:00), compromisos con hijo Facundo (custodia 50/50, horarios de busqueda), familia (pareja Josefina, mama Elsa Mujica) |
| `AGENTS.md` | Conocimiento tecnico para agentes | Indice del repositorio, metodos de edicion de archivos en GitHub (API REST y web) |
| `TO-DOs.md` | Lista de tareas pendientes | Actualmente vacio - usar para trackear tareas futuras |

### Instrucciones para LLMs

1. **Al iniciar una conversacion**: Leer `USER_CONTEXT.md` para obtener contexto personal del usuario
2. **Para tareas tecnicas**: Consultar las secciones relevantes de `AGENTS.md`
3. **Para agregar informacion nueva**: Actualizar `USER_CONTEXT.md` inmediatamente segun las instrucciones de enforcement en ese archivo
4. **Para tareas pendientes**: Usar `TO-DOs.md` para trackear items

---

## Edicion de archivos en GitHub

Existen dos metodos para editar archivos en este repositorio. Usar el apropiado segun el contexto:

| Agente | Metodo | Razon |
|--------|--------|-------|
| **Claude Code** (CLI) | API REST | Tiene acceso a curl y bash, mas rapido y confiable |
| **Claude for Chrome** (extension) | Web/JavaScript | Solo tiene acceso al navegador, no puede usar curl |

---

## Metodo 1: API REST (para Claude Code)

### Ventajas

- No requiere navegador ni automatizacion de UI
- Evita problemas de auto-formateo del editor web
- Mas rapido y confiable
- Funciona desde cualquier ambiente con curl

### Configuracion

El token de acceso personal (PAT) se encuentra en:
- **Archivo local**: `C:\Users\maria\.claude\github_token.txt`
- **Permisos**: Contents (Read and write), Metadata (Read-only)
- **Scope**: All repositories del usuario mblua

### Uso basico con curl

**Nota para Windows**: Agregar `--ssl-no-revoke` para evitar error SSL (exit code 35).

#### 1. Obtener contenido de un archivo

```bash
curl -s --ssl-no-revoke \
  -H "Authorization: Bearer <TOKEN>" \
  https://api.github.com/repos/mblua/user-context/contents/ARCHIVO.md
```

Retorna JSON con:
- `content`: contenido en base64
- `sha`: hash necesario para actualizar el archivo

#### 2. Decodificar contenido

```bash
echo "<CONTENT_BASE64>" | base64 -d
```

#### 3. Actualizar un archivo

```bash
curl -s --ssl-no-revoke -X PUT \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/mblua/user-context/contents/ARCHIVO.md \
  -d '{
    "message": "Descripcion del cambio",
    "content": "<NUEVO_CONTENIDO_BASE64>",
    "sha": "<SHA_DEL_ARCHIVO_ACTUAL>"
  }'
```

#### 4. Crear un archivo nuevo

Igual que actualizar, pero sin el campo `sha`.

### Endpoints utiles

| Operacion | Metodo | Endpoint |
|-----------|--------|----------|
| Listar archivos | GET | `/repos/{owner}/{repo}/contents/` |
| Leer archivo | GET | `/repos/{owner}/{repo}/contents/{path}` |
| Crear/Actualizar | PUT | `/repos/{owner}/{repo}/contents/{path}` |
| Eliminar archivo | DELETE | `/repos/{owner}/{repo}/contents/{path}` |

### Notas importantes

- El `sha` es obligatorio para actualizar (previene conflictos)
- El contenido siempre va en base64
- El mensaje de commit es requerido
- Para archivos grandes, usar la Git Data API en lugar de Contents API

---

## Metodo 2: Edicion Web con JavaScript (para Claude for Chrome)

### Problema

El editor web de GitHub tiene auto-formateo que puede corromper el contenido markdown cuando se usa el comando `type` (simulacion de tipeo). Sintomas:

- Agrega guiones (`-`) antes de los headings
- Crea listas anidadas incorrectamente
- Modifica la indentacion de manera impredecible

### Solucion

Usar JavaScript para setear el contenido directamente en el DOM, bypaseando el sistema de input del editor.

### Pasos

1. **Navegar al modo edicion** del archivo en GitHub

2. **Identificar el elemento del editor** - GitHub usa un elemento con `[role="textbox"]` (no CodeMirror ni textarea comun)

3. **Construir el contenido como string de JavaScript** usando `String.fromCharCode(10)` para los saltos de linea:

```javascript
var nl = String.fromCharCode(10);
var content = "# Titulo" + nl + nl + "Parrafo de texto." + nl;
```

4. **Setear el contenido directamente**:

```javascript
document.querySelector('[role="textbox"]').textContent = content;
```

### Por que funciona

- El metodo `.textContent` setea el contenido sin pasar por el event handler del editor
- `String.fromCharCode(10)` evita problemas de sintaxis con template literals y caracteres especiales
- El bypass del input handling evita el auto-formateo problematico

### Notas importantes

- Siempre verificar el Preview antes de commitear
- Este metodo es util cuando el tipeo normal corrompe el formato
- Funciona para archivos markdown, codigo, y cualquier otro tipo de archivo en GitHub
