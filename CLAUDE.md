# Agente_test — Sistema multiagente Python+Tkinter

## Qué es

GUI de chat (Tkinter) que orquesta un agente LLM multi-modo con herramientas de sistema de archivos.
Backend activo: **Zhipu** (`glm-4.5-flash`). Soporta también: anthropic, ollama, claude-code.
Punto de entrada: `python agent.py`.

## Archivos clave

| Archivo | Rol |
|---------|-----|
| `agent.py` | Todo: herramientas, loops LLM por backend, GUI Tkinter |
| `agent_prompt_default.txt` | System prompt del modo Project Manager |
| `agent_prompt_fullstack.txt` | System prompt del modo Desarrollador Fullstack |
| `agent_prompt_*.txt` | Prompts de otros modos (ciberseguridad, devops, data_engineer, agente_creator) |
| `custom_modes.json` | Modos creados en runtime (auto-generado) |
| `custom_tools.json` | Herramientas creadas en runtime (auto-generado) |
| `knowledge_*.json` | Base de conocimiento persistente por modo (auto-generado) |
| `agent_context.json` | Contexto de sesión guardado en carpeta del proyecto usuario |
| `context_prefs.json` | Ruta preferida de guardado de contexto |
| `.env` | `ZAI_API_KEY` para Zhipu |

## Arquitectura

```
correr_agente()
  ├─ _predetectar_modo()       # cambia modo si el mensaje lo requiere
  ├─ _precargar_ruta()         # abre selector si no hay ruta, puebla cache
  └─ correr_agente_zhipu()     # loop agentico principal (MAX_ITER=12)
       ├─ inyecta _CACHE_ARCHIVOS como [PROYECTO EN CONTEXTO]
       ├─ inyecta historial truncado (MAX_HIST=12)
       ├─ loop: LLM → tool_calls → resultados → LLM...
       └─ auto-compacta si historial > 16 mensajes
```

### Modos disponibles (MODOS dict)

- `default` → Project Manager (enruta solicitudes)
- `fullstack` → Desarrollador Fullstack
- `ciberseguridad` → Experto en Ciberseguridad
- `data_engineer` → Ingeniero de Datos
- `devops` → Ingeniero DevOps
- `agente_creator` → KIKE – Creador de Agentes

### Herramientas nativas principales

`crear_archivo`, `editar_archivo`, `leer_archivo`, `listar_archivos`, `crear_carpeta`,
`buscar_en_proyecto`, `buscar_imagen_web`, `pedir_confirmacion`, `solicitar_ruta_proyecto`,
`cambiar_modo`, `crear_agente`, `crear_herramienta`, `agregar_conocimiento`, `buscar_conocimiento`,
`diagnosticar_impresion`, `ejecutar_comando`, `instalar_libreria`, `calcular`

## Decisiones de diseño relevantes

### Confirmación de archivos
- `pedir_confirmacion` la llama el LLM UNA vez por turno (cubre todos los archivos del lote).
- `crear_archivo` NO tiene confirmación interna propia (se eliminó la doble confirmación).
- Una aprobación cubre todos los `editar_archivo` / `crear_archivo` del mismo turno.

### Cache de archivos (dos capas)

**`_CACHE_INDEX`** — índice ligero (metadatos):
- Poblado por `_poblar_cache_proyecto()` al definir RUTA_PROYECTO. Solo lee nombre/líneas/tamaño.
- Inyectado como lista compacta (~100-300 chars) en el primer LLM call. No crece con el contenido.
- Permite al agente saber qué archivos existen sin pagar el costo de leer su contenido.

**`_CACHE_ARCHIVOS`** — contenido real (carga diferida):
- Vacío al inicio del turno. Solo crece cuando `leer_archivo` es llamado explícitamente por el LLM.
- También actualizado por `crear_archivo` y `editar_archivo`.
- Inyectado con contenido completo (≤1500 chars) o truncado (>1500 chars) en iteraciones posteriores.
- `editar_archivo` actualiza el cache automáticamente — NO releer el mismo archivo tras editar en el mismo turno.

### Loop Zhipu
- `MAX_ITER = 12` (reducido de 18)
- `MAX_HIST = 12` entradas de historial (mismo criterio que backend claude-code)
- Auto-compacta historial si >16 mensajes antes del turno
- Sin LLM extra al final: resumen post-loop es local (sin llamada a API)

### Enriquecimiento eliminado
- `_enriquecer_prompt_creacion` y `_generar_resumen_post_creacion` están definidas pero no se llaman.
- Se eliminaron del flujo GUI para evitar LLM calls extras (+30s cada uno).

## Bugs conocidos y fixes aplicados (sesión 2026-09-10)

### Bug: demora excesiva en ediciones (1729s para CSS+JS)
Causa raíz: 7 factores acumulados. Ver `bugs/demora_excesiva.txt` para tiempos reales.

Fixes aplicados:
1. **Doble confirmación** → eliminada `_solicitar_confirmacion` interna de `crear_archivo` (era redundante con `pedir_confirmacion`)
2. **LLM extra post-loop** → reemplazado por resumen local sin API call
3. **LLM extra pre-agente** → eliminado `_enriquecer_prompt_creacion` del flujo GUI
4. **MAX_ITER 18→12** → cap de iteraciones por turno
5. **MAX_HIST en Zhipu** → truncado igual que claude-code (12 entradas)
6. **Re-lectura post-edición** → instrucción obsoleta eliminada de `agent_prompt_fullstack.txt` y `_ZHIPU_ACTION_SUFFIX`
7. **Aprobación batch** → `approval_rule` y suffix reforzados: una confirmación cubre todo el lote

### Bug A: LLM imprime código en chat sin ejecutar herramientas (describe-but-no-act)
Causa raíz: el detector exigía `tool_log` no vacío — no se activaba en el primer call del turno.

Fix aplicado (detector ~línea 2449):
- Eliminado el requisito `tool_log` non-empty del elif.
- Detector ahora dispara incluso si `tool_log` está vacío (primer call).
- Keywords extendidos: incluye "editar_archivo", "crear_archivo", extensiones `.css/.js/.html/.py/.ts`, y frases de intención en español ("voy a editar", "editaré", "modificaré").

### Bug B: editar_archivo reporta éxito pero cambios no persisten
Causa raíz: el LLM generaba múltiples edits desde memoria del archivo original; si un edit anterior
cambió el texto, `texto_original` del siguiente edit ya no existe y el reemplazo no se aplica —
pero la herramienta solo retornaba "reemplazo aplicado" sin contexto verificable.

Fixes aplicados:
1. **Return value de editar_archivo** (~línea 1296): ahora incluye línea aproximada y preview del texto aplicado.
   Ejemplo: `"Archivo editado: script.js — línea ~42. Inicio del texto aplicado: 'function loadFeatured'"`
2. **Instrucción de verificación en `_ZHIPU_ACTION_SUFFIX`**: REGLA POST-EDICIÓN agrega que si el cambio
   es crítico, el LLM debe verificar con `buscar_en_proyecto`. Si no encuentra el texto → edit no pegó,
   usar `crear_archivo` con archivo completo.

## Patrones de trabajo con este proyecto

- Para editar prompts: modificar directamente `agent_prompt_*.txt`
- Para agregar herramientas runtime: usar `crear_herramienta` desde el chat o editar `custom_tools.json`
- Para agregar modos runtime: usar `crear_agente` desde el chat o editar `custom_modes.json`
- Para debuggear el loop: revisar `agent_debug.log` (se sobreescribe por sesión)
- `bugs/` contiene ejemplos reales de chats con tiempos para diagnóstico de regresiones
