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
| `AGENTS.md` | Conocimiento tecnico para agentes | Indice del repositorio, edicion de archivos en GitHub via API REST |
| `TO-DOs.md` | Lista de tareas pendientes | Actualmente vacio - usar para trackear tareas futuras |

### Instrucciones para LLMs

1. **Al iniciar una conversacion**: Leer `USER_CONTEXT.md` para obtener contexto personal del usuario
2. **Para tareas tecnicas**: Consultar las secciones relevantes de `AGENTS.md`
3. **Para agregar informacion nueva**: Actualizar `USER_CONTEXT.md` inmediatamente segun las instrucciones de enforcement en ese archivo
4. **Para tareas pendientes**: Usar `TO-DOs.md` para trackear items

---

## Edicion de archivos en GitHub via API REST

### Ventajas sobre el metodo web

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

### Ejemplo completo de flujo

```bash
# 1. Leer token
TOKEN=$(tail -1 C:\Users\maria\.claude\github_token.txt)

# 2. Obtener archivo actual y extraer SHA
SHA=$(curl -s --ssl-no-revoke \
  -H "Authorization: Bearer $TOKEN" \
  https://api.github.com/repos/mblua/user-context/contents/ARCHIVO.md \
  | jq -r '.sha')

# 3. Crear nuevo contenido en base64
CONTENT=$(echo "Nuevo contenido" | base64 -w 0)

# 4. Actualizar archivo
curl -s --ssl-no-revoke -X PUT \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  https://api.github.com/repos/mblua/user-context/contents/ARCHIVO.md \
  -d "{\"message\":\"Update\",\"content\":\"$CONTENT\",\"sha\":\"$SHA\"}"
```

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
