# Sistema Multi-Agente con GUI

Un **sistema multi-agente** con orquestador y expertos especializados, con interfaz gráfica en tkinter. El agente principal actúa como Project Manager y delega tareas a expertos por área. Los expertos y herramientas pueden crearse dinámicamente en tiempo de ejecución.

---

## Archivos del proyecto

| Archivo | Descripción |
|---|---|
| `agent.py` | Código principal — GUI, loop agéntico, gestión de modos y herramientas |
| `agent_prompt_default.txt` | Prompt del Project Manager (modo por defecto) |
| `agent_prompt_fullstack.txt` | Desarrollador Fullstack senior |
| `agent_prompt_ciberseguridad.txt` | Experto en ciberseguridad |
| `agent_prompt_data_engineer.txt` | Ingeniero de datos |
| `agent_prompt_devops.txt` | Ingeniero DevOps/SRE |
| `agent_prompt_agente_creator.txt` | Crea nuevos roles de experto |
| `custom_modes.json` | Modos creados dinámicamente (auto-generado) |
| `custom_tools.json` | Herramientas creadas por el agente (auto-generado) |
| `knowledge_<modo>.json` | Base de conocimiento por modo (auto-generado) |

---

## Instalación

### 1. Ambiente virtual

```bash
python3 -m venv env
source env/bin/activate   # macOS/Linux
# env\Scripts\activate   # Windows
```

### 2. Dependencias según el backend elegido

```bash
# Para Zhipu/GLM (recomendado, gratis con registro)
pip install zai-sdk certifi

# Para Anthropic API
pip install anthropic

# Para Ollama (modelo local)
pip install ollama
```

---

## Backends disponibles

Cambia la variable `BACKEND` en `agent.py` (línea ~77):

```python
BACKEND = "zhipu"   # "zhipu" | "anthropic" | "ollama" | "claude-code"
```

### Zhipu / GLM (recomendado para empezar)

Gratis con registro en [bigmodel.cn](https://bigmodel.cn). Soporta tool calling nativo.

```bash
export ZAI_API_KEY="tu_clave_de_zhipu"
# BACKEND = "zhipu"
python3 agent.py
```

Modelo usado: `glm-4.5-flash` (configurable en `MODELOS_POR_BACKEND`).

### Anthropic API

Requiere API key y créditos (desde $5 USD en console.anthropic.com).

```bash
export ANTHROPIC_API_KEY="sk-ant-..."
# BACKEND = "anthropic"
python3 agent.py
```

### Ollama (modelo local, sin costo)

```bash
brew install ollama
ollama serve
ollama pull llama3.2   # en otra terminal
# BACKEND = "ollama"
python3 agent.py
```

Otros modelos compatibles: `mistral`, `phi3`, `gemma3`, `codellama`.

### Claude Code

Usa tu suscripción de claude.ai, sin créditos API separados. Requiere Claude Code CLI instalado y autenticado.

```bash
claude --version   # verificar instalación
# BACKEND = "claude-code"
python3 agent.py
```

Para que Claude Code no interrumpa con prompts de permiso, agrega tu carpeta en `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Write(/ruta/absoluta/a/Agente_test/**)",
      "Bash(mkdir *)"
    ]
  }
}
```

---

## Comparación de backends

| Backend | Costo | Tool use nativo | Streaming |
|---|---|---|---|
| `zhipu` | Gratis (con registro) | Sí | No |
| `anthropic` | Créditos API | Sí | No |
| `ollama` | Gratis (local) | Texto embebido | No |
| `claude-code` | Suscripción claude.ai | Texto embebido | Sí |

---

## Ejecutar

```bash
python3 agent.py
```

Se abre una ventana de chat.

---

## Interfaz

### Entrada de texto
- Soporta **múltiples líneas** — usa `Shift+Enter` para saltos de línea
- `Enter` envía el mensaje
- Barra de desplazamiento vertical si el mensaje es largo

### Botones

| Botón | Función |
|---|---|
| **Enviar** | Envía el mensaje al agente |
| **◼ Detener** | Interrumpe el turno actual (activo solo mientras el agente procesa) |
| **↺ Reiniciar** | Limpia el historial de conversación y reinicia el agente |

### Colores del chat

| Color | Significa |
|---|---|
| Azul | Tu mensaje |
| Verde | Respuesta del agente |
| Morado cursiva | Cambio de modo/experto |
| Gris cursiva | Mensajes del sistema / resultado de herramientas |
| Rojo | Error |

### Indicador de procesamiento

Mientras el agente trabaja, aparece bajo el área de chat:
```
Procesando ·
Procesando ··
Procesando ···
```

---

## Arquitectura multi-agente

```
Usuario
    ↓
Project Manager (modo default)
    ├── detecta qué expertise se necesita
    └── activa al experto con cambiar_modo()
            ↓
        Experto especializado ejecuta herramientas
            ↓
        Puede volver al PM con cambiar_modo("default")
```

Si ningún experto cubre la necesidad, el PM activa al **Creador de Agentes** para diseñar y registrar el nuevo rol en tiempo real.

### Expertos disponibles

| Modo | Rol | Activar manualmente |
|---|---|---|
| `default` | Project Manager | — (modo inicial) |
| `fullstack` | Desarrollador Fullstack senior | `/modo fullstack` |
| `ciberseguridad` | Experto en Ciberseguridad | `/modo ciberseguridad` |
| `data_engineer` | Ingeniero de Datos | `/modo data_engineer` |
| `devops` | Ingeniero DevOps | `/modo devops` |
| `agente_creator` | Creador de nuevos roles | automático |

### Cambio de modo manual

```
/modo fullstack
/modo devops
/modo default
```

El cambio es inmediato, sin pasar por el agente.

---

## Herramientas nativas

Estas herramientas están implementadas directamente en `agent.py` y disponibles para todos los backends:

| Herramienta | Descripción |
|---|---|
| `listar_archivos` | Lista archivos y carpetas del proyecto |
| `leer_archivo` | Lee el contenido de un archivo |
| `crear_archivo` | Crea o reemplaza un archivo completo |
| `editar_archivo` | Reemplaza un fragmento de texto en un archivo existente |
| `crear_carpeta` | Crea un directorio |
| `buscar_imagen_web` | Busca URL de imagen real en Wikipedia/Wikimedia |
| `calcular` | Evalúa expresiones matemáticas |
| `solicitar_ruta_proyecto` | Abre diálogo para que el usuario elija la carpeta del proyecto |
| `cambiar_modo` | Activa otro experto |
| `crear_agente` | Crea un nuevo rol de experto (requiere aprobación) |
| `crear_herramienta` | Crea una nueva herramienta Python en tiempo real |
| `agregar_conocimiento` | Guarda conocimiento permanente en la base del modo activo |
| `buscar_conocimiento` | Busca en la base de conocimiento del modo activo |
| `pedir_confirmacion` | Solicita confirmación del usuario antes de una acción |

---

## Herramientas dinámicas

El agente puede crear sus propias herramientas Python con `crear_herramienta`. Se guardan en `custom_tools.json` y se cargan automáticamente al iniciar.

```
"Crea una herramienta que consulte el clima"
"Necesito una herramienta que procese archivos CSV"
```

### Cómo funciona el loop agéntico

```
Tu mensaje
    ↓
El modelo recibe el mensaje + herramientas disponibles
    ↓
¿Necesita una herramienta?
    ├── NO → responde con texto → fin
    └── SÍ → llama la herramienta con parámetros
                    ↓
              el sistema ejecuta la función
                    ↓
              el resultado vuelve al modelo
                    ↓
              el modelo responde o llama otra herramienta
```

---

## Base de conocimiento por agente

Cada modo acumula conocimiento permanente en `knowledge_<modo>.json`. Se inyecta automáticamente en el system prompt al iniciar y al cambiar de modo.

```
"Recuerda que nuestro proyecto usa Django 4.2 con PostgreSQL"
"Guarda que el servidor de producción está en us-east-1"
"¿Qué sabes sobre autenticación en este proyecto?"
```

---

## Agregar herramientas manualmente

En `agent.py`, dos pasos:

**1. Agregar a la lista `TOOLS`:**
```python
{
    "name": "mi_herramienta",
    "description": "Descripción de qué hace",
    "input_schema": {
        "type": "object",
        "properties": {
            "parametro": {"type": "string", "description": "..."}
        },
        "required": ["parametro"]
    }
}
```

**2. Agregar el handler en `_despachar_herramienta`:**
```python
if nombre == "mi_herramienta":
    return str(mi_funcion(inputs["parametro"]))
```

---

## Crear nuevos expertos

Pídele al agente directamente:

```
"Necesito un experto en machine learning"
"Crea un agente especializado en UX/UI"
```

El creador de agentes construye el prompt, muestra un diálogo de confirmación con preview completo, y si se aprueba, el nuevo modo queda disponible de inmediato y persiste en `custom_modes.json`.
