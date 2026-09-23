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

**Homebrew (macOS/Linux) o Scoop (Windows):**

```sh
brew tap aindez-tech/tap https://github.com/aindez-tech/aindez-releases
brew install --cask aindez
```

```powershell
scoop bucket add aindez https://github.com/aindez-tech/aindez-releases
scoop install aindez
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

Todos los `list` aceptan `--page`, `--per-page` y `--search`; cualquier
comando acepta `--timeout <segundos>`. Para dar formato a las lecturas:
`--output table|json|csv|yaml`, `--fields nombre,email` (elige columnas) y
`--jq '.data[].email'` (transforma el resultado, sin instalar jq). En las
escrituras, `--dry-run` muestra exactamente qué se enviaría sin enviarlo,
y el resumen de confirmación marca con ⚠ cuando estás en producción.

**Preguntas sobre tus datos**

Pregunta en español, como se lo preguntarías a una persona. No hace falta
saber en qué sistema ni en qué tabla está el dato:

```
$ aindez bi "¿cuántos candidatos contratamos el último trimestre, por fuente?"
Contrataron 37 candidatos entre abril y junio: LinkedIn 18, referidos 11 y Computrabajo 8.
Datos al 21-09-2026 · candidato (teamtailor), postulacion (teamtailor)
```

Si los datos para responder no están conectados, la respuesta lo dice y
nombra lo que falta («No hay datos de nómina conectados»).

- `--explain` agrega de dónde salió la respuesta: las tablas, cómo se
  unieron, el SQL y las filas.
- `aindez bi` sin pregunta abre una conversación. Las preguntas siguientes
  («¿y el trimestre anterior?») toman en cuenta las anteriores. Escribe
  `salir` o presiona Ctrl-D para terminar.
- Para descargar todas las filas en .xlsx:
  `aindez schemas export-result <clave>` (la clave aparece con `--explain`).

```
$ aindez bi
Pregunta sobre los datos de la empresa. Escribe «salir» o presiona Ctrl-D para terminar.
› ¿cuántas vacantes abiertas hay por área?
Hay 12 vacantes abiertas: Operaciones 5, Ventas 4 y Tecnología 3.
› ¿y cuántas se cerraron este año?
…
› salir
```

**Datos de tu organización**

```
aindez contacts list | get <id> | lookup --email <email> | stats
aindez companies list | get <id> | contacts <id> | stats
aindez agreements list | get <id> | by-company <id> | stats
aindez candidates list | get <id> | workforce | stats
aindez job-offers list | get <id> | candidates <id> | summaries <ids> | stats
aindez invoices list | get <id> | lines <id> | stats
aindez payments list | get <id> | stats
aindez compliance-policies list | get <id> | stats
aindez talent overview
aindez kanban boards | cards | history            # tableros de reclutamiento
aindez talent-documents requests list | review-queue
aindez analytics overview | tokens | users        # métricas de uso del chat
```

`job-offers candidates <id>` muestra los candidatos de una vacante, su
estado y en qué sistemas está cada uno (Teamtailor, SAP SuccessFactors…).

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
recursos — `aindez <recurso> --help` lista los disponibles. Para cargas
grandes, `--from-file datos.json` (o `-` para leer de stdin) toma los
argumentos desde un archivo en vez de escribirlos en la línea de comandos:

```
aindez candidates bulk-create --from-file candidatos.json
cat empresas.json | aindez companies bulk-create --from-file -
```

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
aindez schemas imports get <job> --watch  # espera a que el import termine
```

**Preguntas sobre tus datos (Query Studio)**

```
aindez query "cuántos candidatos hay por ciudad" --schema candidatos
aindez query "vacantes con más postulaciones" --schema teamtailor \
  --table teamtailor.jobs --table teamtailor.job_applications --limit 10
```

Muestra el SQL que se usó, las tablas, las primeras filas (`--limit`, hasta
50) y el total. Espera hasta que la consulta termine (`--timeout` no la
corta; Ctrl-C sí). Cuando la consulta une tablas gracias a la ontología, lo
dice: «Usa la ontología: job_applications ↔ candidates
(job_applications.candidate-id = candidates.id)».

**Ontología (qué es cada tabla y qué significa cada columna)**

```
aindez ontology status                    # los números del catálogo
aindez ontology entities | tables | relations
aindez ontology table <origen> <tabla>    # una tabla: descripción y columnas
aindez ontology resolve --source-kind connector --source-id teamtailor:default \
  --table candidates persona.email        # qué columna tiene cada concepto
aindez ontology build --wait              # recalcula y muestra el avance hasta el final
aindez ontology job                       # la construcción en curso o la última
aindez ontology sensitive <columna_id> --on   # marca una columna como dato sensible
```

`build` y `sensitive` piden confirmación escrita (`--yes` la omite).
`build --wait` termina cuando la construcción termina; si se acaba el tiempo
de espera (`--timeout 20m`), la construcción sigue y `aindez ontology job
--wait` la retoma.

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

- `--json` en los comandos de lectura emite JSON parseable. En `aindez bi`,
  una línea JSON por pregunta, también en la conversación por pipe
  (`printf '¿…?\n¿y el mes pasado?\n' | aindez bi --json`), sin los valores
  de columnas personales.
- Los códigos de salida distinguen el tipo de error (auth, permisos, red…).
- Cada escritura envía una clave de idempotencia: reintentar un comando que
  falló por red no duplica la operación en el servidor.
- Si usas el CLI con Claude Code u otro agente, el archivo `AGENTS.md` que
  viene junto al binario documenta el contrato completo.

## Configuración y aliases

```
aindez config set per-page 50      # default persistente para los list
aindez config set output json      # formato por defecto de las lecturas
aindez alias set cq "contacts list --search"
aindez cq ana                      # = aindez contacts list --search ana
```

Si algo no funciona, `aindez doctor` revisa versión, environment, sesión y
conectividad con la API en un solo paso.

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
