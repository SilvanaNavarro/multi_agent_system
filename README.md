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

## Requisitos previos

### Python 3.10 o superior

**macOS/Linux**

```bash
python3 --version
```

Si no está instalado, en macOS:
```bash
brew install python
```
En Ubuntu/Debian:
```bash
sudo apt install python3 python3-venv python3-pip
```

**Windows**

Descarga el instalador oficial desde [python.org/downloads](https://www.python.org/downloads/).

Durante la instalación:
- Marca **"Add Python to PATH"** (obligatorio)
- Marca **"Install pip"**
- Marca **"tcl/tk and IDLE"** (requerido para la GUI con tkinter)

Verifica la instalación abriendo **PowerShell** o **CMD**:
```powershell
python --version
pip --version
```

> En Windows el comando es `python` (no `python3`). En las instrucciones de este README se usa `python3` para macOS/Linux — reemplázalo por `python` si estás en Windows.

### tkinter (GUI)

Tkinter viene incluido con Python en Windows y macOS. En Linux puede requerir instalación manual:
```bash
sudo apt install python3-tk   # Ubuntu/Debian
sudo dnf install python3-tkinter   # Fedora
```

---

## Instalación

### 1. Clonar o descargar el repositorio

```bash
git clone <url-del-repo>
cd Agente_test
```

### 2. Ambiente virtual

**macOS / Linux**
```bash
python3 -m venv env
source env/bin/activate
```

**Windows — PowerShell**
```powershell
python -m venv env
env\Scripts\Activate.ps1
```

**Windows — CMD**
```cmd
python -m venv env
env\Scripts\activate.bat
```

> Si PowerShell rechaza el script con un error de permisos de ejecución, ejecuta primero:
> ```powershell
> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
> ```

El prompt cambia a `(env)` cuando el ambiente está activo. Para desactivar: `deactivate`.

### 3. Instalar dependencias

**Instalación completa (recomendado):**

```bash
pip install -r requirements.txt
```

**O por partes, según lo que vayas a usar:**

```bash
# Core (siempre requerido)
pip install python-dotenv

# Backend LLM — elige uno o varios
pip install zai-sdk certifi      # Zhipu/GLM  (gratuito, recomendado)
pip install anthropic             # Anthropic Claude
pip install ollama                # Modelos locales

# Lectura de documentos adjuntos (PDF, Word, Excel)
pip install pdfplumber python-docx openpyxl

# Pegar capturas de pantalla con Ctrl+V
pip install Pillow

# Drag-and-drop de archivos desde el explorador (opcional)
pip install tkinterdnd2
```

> **Nota:** Si falta alguna dependencia al usar el sistema, el agente detecta el error y ofrece instalarla automáticamente con confirmación tuya — no necesitas instalar todo desde el principio.

**Verificar instalación:**

```bash
python3 -c "import dotenv, anthropic, pdfplumber, docx, openpyxl, PIL; print('OK')"
```

En Windows:
```powershell
python -c "import dotenv, anthropic, pdfplumber, docx, openpyxl, PIL; print('OK')"
```

### 4. Configurar API key

Crea un archivo `.env` en la raíz del proyecto (junto a `agent.py`):

```
# Para Zhipu (backend recomendado)
ZAI_API_KEY=tu_clave_aqui

# Para Anthropic (opcional)
ANTHROPIC_API_KEY=tu_clave_aqui
```

> En Windows puedes crear el archivo con el Bloc de notas: Archivo → Guardar como → nombre: `.env`, tipo: "Todos los archivos". Asegúrate de que no quede como `.env.txt`.

El sistema carga `.env` automáticamente al iniciar.

---

## Obtener API keys

### Zhipu / Z.AI (gratuito con registro)

1. Entra a [z.ai](https://z.ai) → **Sign Up**
2. Verifica tu correo
3. En el dashboard ve a **API Keys** → **Create API Key**
4. Copia la clave y pégala en `.env`:
   ```
   ZAI_API_KEY=tu_clave
   ```
5. Plan gratuito incluye cuota suficiente para desarrollo

### Anthropic

1. Entra a [console.anthropic.com](https://console.anthropic.com) → **Sign Up**
2. Ve a **Settings → API Keys** → **Create Key**
3. Agrega créditos en **Billing** (mínimo $5 USD para comenzar)
4. Copia la clave y pégala en `.env`:
   ```
   ANTHROPIC_API_KEY=sk-ant-...
   ```

### Ollama y Claude Code

No requieren API key.
- **Ollama**: ver sección de instalación más abajo
- **Claude Code**: instala el CLI e inicia sesión con tu cuenta de claude.ai

---

## Backends disponibles

Cambia la variable `BACKEND` en `agent.py` (línea ~77):

```python
BACKEND = "zhipu"   # "zhipu" | "anthropic" | "ollama" | "claude-code"
```

### Zhipu / GLM (recomendado para empezar)

Gratis con registro en [bigmodel.cn](https://bigmodel.cn). Soporta tool calling nativo.

**macOS / Linux**
```bash
export ZAI_API_KEY="tu_clave_de_zhipu"
python3 agent.py
```

**Windows — PowerShell**
```powershell
$env:ZAI_API_KEY = "tu_clave_de_zhipu"
python agent.py
```

**Windows — CMD**
```cmd
set ZAI_API_KEY=tu_clave_de_zhipu
python agent.py
```

> O simplemente pon la clave en el archivo `.env` y no necesitas exportar nada.

Modelo usado: `glm-4.5-flash` (configurable en `MODELOS_POR_BACKEND`).

---

### Anthropic API

Requiere API key y créditos (desde $5 USD en console.anthropic.com).

**macOS / Linux**
```bash
export ANTHROPIC_API_KEY="sk-ant-..."
python3 agent.py
```

**Windows — PowerShell**
```powershell
$env:ANTHROPIC_API_KEY = "sk-ant-..."
python agent.py
```

**Windows — CMD**
```cmd
set ANTHROPIC_API_KEY=sk-ant-...
python agent.py
```

---

### Ollama (modelo local, sin costo)

**macOS**
```bash
brew install ollama
ollama serve
# En otra terminal:
ollama pull llama3.2
python3 agent.py
```

**Linux**
```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama serve
# En otra terminal:
ollama pull llama3.2
python3 agent.py
```

**Windows**

1. Descarga el instalador desde [ollama.com/download](https://ollama.com/download) → **Windows**
2. Ejecuta el `.exe` e instala normalmente
3. Ollama inicia automáticamente como servicio en segundo plano
4. Abre **PowerShell** o **CMD** y descarga un modelo:

```powershell
ollama pull llama3.2
python agent.py
```

> No necesitas correr `ollama serve` manualmente en Windows — el instalador lo configura como servicio de Windows.

Otros modelos compatibles: `mistral`, `phi3`, `gemma3`, `codellama`.

---

### Claude Code

Usa tu suscripción de claude.ai, sin créditos API separados. Requiere Claude Code CLI instalado y autenticado.

**macOS / Linux**
```bash
npm install -g @anthropic-ai/claude-code
claude --version
claude   # inicia sesión la primera vez
python3 agent.py
```

**Windows**

Requiere Node.js instalado ([nodejs.org](https://nodejs.org)). Luego en PowerShell:

```powershell
npm install -g @anthropic-ai/claude-code
claude --version
claude   # inicia sesión la primera vez
python agent.py
```

> En Windows, si `claude` no se reconoce después de instalar, reinicia PowerShell o ejecuta `refreshenv` si tienes Chocolatey instalado.

**Permisos para evitar interrupciones**

Para que Claude Code no interrumpa con prompts de permiso, edita el archivo de configuración:

macOS/Linux: `~/.claude/settings.json`  
Windows: `%USERPROFILE%\.claude\settings.json`

```json
{
  "permissions": {
    "allow": [
      "Write(C:/ruta/absoluta/a/Agente_test/**)",
      "Bash(mkdir *)"
    ]
  }
}
```

> En Windows usa barras normales `/` o barras dobles `\\` en la ruta, no barras simples `\`.

---

## Comparación de backends

| Backend | Costo | Tool use nativo | Streaming | Windows |
|---|---|---|---|---|
| `zhipu` | Gratis (con registro) | Sí | No | ✅ |
| `anthropic` | Créditos API | Sí | No | ✅ |
| `ollama` | Gratis (local) | Texto embebido | No | ✅ |
| `claude-code` | Suscripción claude.ai | Texto embebido | Sí | ✅ |

---

## Ejecutar

**macOS / Linux**
```bash
source env/bin/activate
python3 agent.py
```

**Windows — PowerShell**
```powershell
env\Scripts\Activate.ps1
python agent.py
```

**Windows — CMD**
```cmd
env\Scripts\activate.bat
python agent.py
```

Se abre una ventana de chat.

---

## Cómo usar el sistema

### Flujo básico

El sistema arranca en modo **Project Manager**. Habla en lenguaje natural — el PM decide si responde él o activa a un experto.

```
Tú:     "Crea un script Python que lea un CSV y genere un reporte HTML"
PM:     activa al Desarrollador Fullstack automáticamente
Full:   crea los archivos en tu proyecto
```

No necesitas decir "cambia al experto X". El sistema lo detecta solo.

### Comandos de sistema

```
/modo fullstack       → activa experto directamente sin pasar por el PM
/modo devops
/modo ciberseguridad
/modo data_engineer
/modo default         → vuelve al Project Manager
/compact              → compacta el historial cuando la conversación es larga
```

### Elegir carpeta de proyecto

Antes de crear archivos, el agente pide una carpeta destino mediante un diálogo gráfico. También puedes mencionarla en el mensaje:

```
"Crea la app en la carpeta development/mi_proyecto"
```

El agente la configura automáticamente si la ruta existe.

> En Windows las rutas con espacios funcionan normalmente desde el diálogo gráfico. Si las escribes en el chat, usa comillas: `"C:/Users/tu_usuario/Proyectos/mi app"`.

### Ejemplos de uso por área

**Desarrollo (Fullstack)**
```
"Crea una landing page con HTML, CSS y JS para una cafetería"
"Agrega un formulario de contacto con validación al index.html"
"Refactoriza el componente de login para usar async/await"
```

**Datos (Data Engineer)**
```
"Escribe una query SQL que agrupe ventas por mes y región"
"Crea un pipeline de ETL en Python para procesar logs de servidor"
"¿Qué índices debería agregar a esta tabla?"
```

**DevOps**
```
"Crea un Dockerfile para esta app Flask con Gunicorn"
"Genera un GitHub Actions workflow para CI/CD"
"Revisa este docker-compose y sugiere mejoras de seguridad"
```

**Seguridad**
```
"Audita este código Python en busca de vulnerabilidades"
"¿Cómo protejo esta API contra ataques de inyección SQL?"
"Genera un reporte de riesgos para esta arquitectura"
```

**Crear nuevos expertos**
```
"Necesito un experto en machine learning"
"Crea un agente especializado en diseño UX/UI"
"Quiero un experto en contratos legales"
```

El Creador de Agentes (KIKE) diseña el prompt, muestra un preview y pide confirmación antes de guardar.

### Conocimiento persistente

El agente puede recordar información entre sesiones:

```
"Recuerda que usamos PostgreSQL 15 con Django 4.2"
"Guarda que el equipo prefiere TypeScript sobre JavaScript"
"¿Qué sabes sobre la configuración de nuestra base de datos?"
```

El conocimiento se guarda por modo en `knowledge_<modo>.json` y se inyecta automáticamente en cada conversación.

### Crear herramientas en tiempo real

```
"Necesito una herramienta que consulte el precio del dólar en tiempo real"
"Crea una herramienta para convertir imágenes a base64"
```

El agente escribe el código Python, lo registra y lo usa de inmediato. Persiste en `custom_tools.json`.

### Tips

- **Sé específico con rutas**: "en la carpeta `src/components`" funciona mejor que "en algún lado"
- **Pide confirmación explícita** antes de operaciones destructivas: "antes de borrar, muéstrame qué vas a eliminar"
- **Usa `/compact`** si la conversación es muy larga y el modelo empieza a perder contexto
- **Cambia de backend** en el dropdown superior sin reiniciar la app
- **Windows**: si la ventana GUI no abre, verifica que tkinter esté instalado corriendo `python -m tkinter` — debe abrir una ventana de prueba pequeña

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

## Solución de problemas comunes en Windows

| Problema | Causa probable | Solución |
|---|---|---|
| `python3` no se reconoce | Windows usa `python` | Reemplaza `python3` por `python` en todos los comandos |
| Error de permisos al activar venv | Política de ejecución de PowerShell | `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` |
| La ventana GUI no abre | tkinter no instalado | Reinstala Python marcando "tcl/tk and IDLE" |
| `.env` cargado como `.env.txt` | Bloc de notas agrega extensión | Guarda con tipo "Todos los archivos" o usa VS Code |
| `claude` no reconocido tras instalación | PATH no actualizado | Reinicia PowerShell o abre CMD nuevo |
| Ollama connection refused | Servicio no iniciado | Abre el menú inicio → busca Ollama → inícialo |
| Ruta con espacios rompe el agente | Comillas faltantes | Usa rutas sin espacios o menciónalas entre comillas en el chat |

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
