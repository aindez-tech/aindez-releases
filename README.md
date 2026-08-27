# aindez

Cliente de línea de comandos de la plataforma Aindez: consultá y operá tus
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
build tenés.

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

**Datos de tu organización**

```
aindez contacts list
aindez companies list
aindez agreements list
aindez candidates list
aindez job-offers list
aindez invoices list
aindez payments list
aindez compliance-policies list
```

**Schemas (tus tablas de datos)**

```
aindez schemas list
aindez schemas show <schema>              # tablas, columnas y cantidad de filas
aindez schemas preview <schema> <tabla>   # primeras filas (--limit)
aindez schemas download <schema> <tabla>  # exporta la tabla a .xlsx (--out)
aindez schemas add <schema> <archivo...>  # sube .xlsx/.mdb (--mode append|replace)
aindez schemas delete <schema>            # destructivo — pide confirmación
aindez schemas sources                    # de dónde se puede importar
aindez schemas import <schema> <fuente> <áreas>
```

**Integraciones**

```
aindez integrations list
aindez integrations connect <nombre>      # OAuth en el navegador, o credenciales guiadas
aindez integrations disconnect <nombre>   # afecta a toda la org — pide confirmación
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
confirmación tipeada y no tiene forma de saltearse: es tu decisión.

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
- Si usás el CLI con Claude Code u otro agente, el archivo `AGENTS.md` que
  viene junto al binario documenta el contrato completo.

## Problemas comunes

- **«Sesión expirada»** → `aindez login` de nuevo.
- **Errores intermitentes del servidor** → las lecturas reintentan solas; si
  persiste, corré el comando con `--debug` y compartí el `request_id` con
  soporte.
- **Completado de shell** → `aindez completion bash|zsh|fish|powershell`.

## Desinstalar

1. `aindez logout` (revoca la sesión).
2. Borrá el binario: `~/.local/bin/aindez` (macOS/Linux) o
   `%LOCALAPPDATA%\aindez\bin` (Windows).
3. Borrá la configuración: `~/Library/Application Support/aindez` (macOS),
   `~/.config/aindez` (Linux), `%AppData%\aindez` (Windows).
