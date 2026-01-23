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
| `AGENTS.md` | Conocimiento tecnico para agentes | Indice del repositorio, tecnicas de edicion de archivos en GitHub via web |
| `TO-DOs.md` | Lista de tareas pendientes | Actualmente vacio - usar para trackear tareas futuras |

### Instrucciones para LLMs

1. **Al iniciar una conversacion**: Leer `USER_CONTEXT.md` para obtener contexto personal del usuario
2. **Para tareas tecnicas**: Consultar las secciones relevantes de `AGENTS.md`
3. **Para agregar informacion nueva**: Actualizar `USER_CONTEXT.md` inmediatamente segun las instrucciones de enforcement en ese archivo
4. **Para tareas pendientes**: Usar `TO-DOs.md` para trackear items

---

## Edicion de archivos en GitHub via web

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
document.querySelector("[role=`textbox`]").textContent = content;
```

### Por que funciona

- El metodo `.textContent` setea el contenido sin pasar por el event handler del editor
- `String.fromCharCode(10)` evita problemas de sintaxis con template literals y caracteres especiales
- El bypass del input handling evita el auto-formateo problematico

### Notas importantes

- Siempre verificar el Preview antes de commitear
- Este metodo es util cuando el tipeo normal corrompe el formato
- Funciona para archivos markdown, codigo, y cualquier otro tipo de archivo en GitHub
