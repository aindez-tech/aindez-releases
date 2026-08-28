# aindez

Cliente de línea de comandos de la plataforma Aindez: consulta y opera tus
datos de CRM, reclutamiento (ATS) y ERP sin salir de la terminal.

## Instalación

**Windows (PowerShell):**

```powershell
irm https://github.com/aindez-tech/aindez-releases/releases/latest/download/install.ps1 | iex
```

**macOS / Linux:**

```sh
curl -fsSL https://github.com/aindez-tech/aindez-releases/releases/latest/download/install.sh | sh
```

Ambos verifican la integridad de la descarga. `aindez version` muestra qué
build tienes.

## Actualizar

El CLI avisa (como máximo una vez al día, por stderr) cuando hay una versión
nueva. Para actualizar:

```
aindez upgrade
```

Descarga la última versión, verifica su integridad y reemplaza el binario.
Para desactivar el aviso: `AINDEZ_NO_UPDATE_CHECK=1`.

## Empezar

```
aindez login               # abre el navegador para iniciar sesión
aindez whoami              # con qué cuenta, organización y rol estás
aindez org list            # tus organizaciones
aindez org switch <org_id> # cambia de organización sin re-loguear
aindez logout              # cierra la sesión (local y en el servidor)
```

La sesión queda guardada en el llavero del sistema; no hace falta loguearse
de nuevo entre comandos.

## Comandos

Todos los `list` aceptan `--page`, `--per-page` y `--search`, y cualquier
comando acepta `--timeout <segundos>` si una operación tarda demasiado.

**Datos de tu organización**

```
aindez contacts list | get <id> | lookup --email <email> | stats
aindez companies list | get <id> | contacts <id> | stats
aindez agreements list | get <id> | by-company <id> | stats
aindez candidates list | get <id> | workforce | stats
aindez job-offers list | get <id> | summaries <ids> | stats
aindez invoices list | get <id> | lines <id> | stats
aindez payments list | get <id> | stats
aindez compliance-policies list | get <id> | stats
aindez talent overview
aindez kanban boards | cards | history            # tableros de reclutamiento
aindez talent-documents requests list | review-queue
aindez analytics overview | tokens | users        # métricas de uso del chat
```

**Crear y modificar** (cada escritura muestra exactamente qué se va a
enviar y pide confirmación escrita; `--yes` la omite):

```
aindez contacts create <company_id> --first-name Ana --email ana@acme.com
aindez companies create "Acme" --industry software
aindez candidates update <id> --current-title "Data Engineer"
aindez kanban card-move <card_id> <columna_destino>
aindez invoices update <id> --client-name "Acme SA de CV"
```

Hay creates, updates y operaciones equivalentes en casi todos los
recursos — `aindez <recurso> --help` lista los disponibles.

**Schemas (tus tablas de datos)**

```
aindez schemas list
aindez schemas show <schema>              # tablas, columnas y cantidad de filas
aindez schemas preview <schema> <tabla>   # primeras filas (--limit)
aindez schemas download <schema> <tabla>  # exporta la tabla a .xlsx (--out)
aindez schemas export <schema>            # exporta el schema completo a .xlsx
aindez schemas chart <schema> <tabla> <columna_x>   # configura un chart
aindez schemas add <schema> <archivo...>  # sube .xlsx/.mdb (--mode append|replace)
aindez schemas delete <schema>            # destructivo — pide confirmación
aindez schemas share <schema> <user_id> <rol>       # comparte (viewer|editor|owner)
aindez schemas archive <schema> | unarchive <schema>
aindez schemas versions <schema> | version-restore <schema> <version_id>
aindez schemas sources                    # de dónde se puede importar
aindez schemas import <schema> <fuente> <áreas>
aindez schemas imports list               # estado de la cola de imports
```

**Integraciones**

```
aindez integrations list
aindez integrations connect <nombre>      # OAuth en el navegador, o credenciales guiadas
aindez integrations disconnect <nombre>   # afecta a toda la org — pide confirmación
aindez integrations sync                  # sincroniza todas las activas
aindez connectors health                  # salud de los conectores de datos
```

**Workflows (orquestador)**

```
aindez workflows list
aindez workflows get <id>
aindez workflow-runs list                 # corridas que esperan atención y recientes
aindez workflow-runs get <run_id>
aindez workflow-runs approvals <run_id>   # qué se escribiría afuera, ítem por ítem
aindez workflow-runs approve <run_id> <clave>...
aindez workflow-runs reject <run_id> <clave>... --note "razón"
```

Aprobar o rechazar escribe en sistemas externos, por eso siempre pide
confirmación escrita y no se puede omitir: es tu decisión.

**Mensajería, base de conocimiento y más**

```
aindez messaging history
aindez messaging send --template-id <id> --to contact:42   # pide confirmación
aindez knowledge list | search <consulta> | ingest-file <archivo>
aindez routines list
aindez talent-documents requests list
```

`aindez --help` y `aindez <comando> --help` documentan todo lo demás.

## Para scripts y agentes de IA

- `--json` en los comandos de lectura emite JSON parseable.
- Los códigos de salida distinguen el tipo de error (auth, permisos, red…).
- Cada escritura envía una clave de idempotencia: reintentar un comando que
  falló por red no duplica la operación en el servidor.
- Si usas el CLI con Claude Code u otro agente, el archivo `AGENTS.md` que
  viene junto al binario documenta el contrato completo.

## Problemas comunes

- **«Sesión expirada»** → `aindez login` de nuevo.
- **Errores intermitentes del servidor** → las lecturas reintentan solas; si
  persiste, ejecuta el comando con `--debug` y comparte el `request_id` con
  soporte.
- **Completado de shell** → `aindez completion bash|zsh|fish|powershell`.

## Desinstalar

1. `aindez logout` (revoca la sesión).
2. Borra el binario: `~/.local/bin/aindez` (macOS/Linux) o
   `%LOCALAPPDATA%\aindez\bin` (Windows).
3. Borra la configuración: `~/Library/Application Support/aindez` (macOS),
   `~/.config/aindez` (Linux), `%AppData%\aindez` (Windows).
