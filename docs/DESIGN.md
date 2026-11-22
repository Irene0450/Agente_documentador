# Diseño del agente documentador

Las reglas que sigue el agente están **solo** en [`.opencode/agents/documentador.md`](../.opencode/agents/documentador.md). Este documento no las repite: explica por qué son así y cómo cambiarlas. Las referencias `§N` apuntan a las secciones del agente.

## Decisiones

| Decisión | Dónde | Por qué |
|---|---|---|
| **Omitir antes que inventar** | §1 | Una documentación incompleta pero cierta es útil. Un comando inventado hace perder tiempo y quita confianza en todo lo demás. |
| **Hecho vs inferido marcado** | §1 | A veces hace falta una inferencia (por ejemplo, el propósito de un módulo sin comentarios). Marcarla permite al lector saber qué comprobar. |
| **Un solo archivo autocontenido** | todo | OpenCode solo carga el archivo del agente; las guías de este repositorio no viajan al copiarlo a otro. Por eso el agente es la única fuente de verdad. |
| **Modos de trabajo** | §3 | Cada petición tiene un entregable distinto. Fijar el modo al principio evita sobre-generar, por ejemplo crear un ARCHITECTURE.md cuando solo se pidió entender el repo. |
| **Tipo de proyecto** | §3 | "Ejecutar", "estado" o "regla" no significan lo mismo en una API, un juego o un repo de contenido. En una prueba con Godot, las preguntas pensadas para backend se respondieron casi todas con "no hay", y faltaron las propias del motor (escena principal, recursos ejecutables, formato del addon). Cada tipo añade su punto de entrada, lo que hay que buscar y quién lee la documentación. Si ninguno encaja, quedan las preguntas generales. |
| **"Entender" no escribe** | §3 | Explorar un repo ajeno no debe dejar archivos nuevos en él. |
| **Delegar con más de 15 archivos** | §4 | Por debajo de ese número leer directamente es más barato que delegar. Por encima, `explore` ahorra contexto. Es un criterio, no una obligación: el agente decide qué leer después. |
| **Acotar el encargo a `explore`** | §4 | En OpenCode cada subagente usa sus propios permisos, y `explore` permite web y directorios externos. El `deny` del documentador no le afecta, así que el límite tiene que ir en el propio encargo, y lo que traiga de fuera se descarta. |
| **No documentar la configuración de IA** | §4 | `.opencode/` y similares pertenecen a quien usa el agente, no al proyecto. Mencionarlos en el README es ruido para el lector. |
| **Sección "Comportamiento y reglas"** | §5 | En la primera prueba real (un proyecto Java/Spring de ejemplo), la documentación salió sin nada inventado pero incompleta. Faltaban la máquina de estados y los códigos de error, que cada proceso tenía su propia BD en memoria, que algún campo se calculaba distinto en SQL y en Java, y que un trigger modificaba los datos de la semilla. Un inventario de módulos no saca esto a la luz; una lista explícita de cosas que buscar, sí. Como la lista salió de ese único proyecto, estaba sobreajustada a backends. Ahora son 10 preguntas generales con ejemplos de varios tipos de proyecto, y los detalles de cada tipo están en §3. |
| **Conceptos: comparar también dónde se consultan** | §5 | En una segunda ejecución sobre el mismo proyecto, se escapó que "origen histórico" eran dos valores en `isHistoric()` pero solo uno en el trigger y en el cálculo de la puntuación. Comparar solo las definiciones no lo detecta: hay que mirar también los sitios que consultan el concepto. |
| **Entradas con valores válidos y qué pasa si no lo son** | §5 | En la misma ejecución, la API decía "valor de `LeadStatus`" sin dar la lista, y el lector tenía que ir al enum. Qué pasa con un valor inválido (error, se ignora o se sustituye) es lo primero que necesita quien integra. |
| **"¿Se puede usar tal como está?"** | §5 | En una prueba sobre un proyecto Godot sin escena principal y recursos sin compilar, la documentación lo mencionaba a medias, sin decir que no se podía ejecutar. |
| **Comentarios como pistas, no como hechos** | §5 | El código suele avisar de sus trampas en comentarios ("NOTA", "deprecated", "no es..."). Leerlos acelera el hallazgo, pero un comentario puede estar desactualizado, así que se verifica contra el código. |
| **Verificar formatos en su consumidor** | §5 | En una prueba con un proyecto Godot, el agente buscó con grep una palabra de las anotaciones de los diálogos dentro del addon. No la encontró y concluyó que se ignoraban. En realidad, el compilador del addon trata las líneas que empiezan por `%` como líneas aleatorias y solo importa otra extensión de archivo. Que una palabra no aparezca no demuestra nada; lo que vale es cómo el código consumidor interpreta esa sintaxis. |
| **Módulos agrupados por paquete** | §5 | Una fila por archivo con la ruta completa repetida duplica la longitud sin añadir información. |
| **Precisión en las generalizaciones** | §1 | "Se activa solo si X" es falso si también se activa cuando falta X. No inventar no basta: tampoco hay que simplificar de más. |
| **Inferir solo tras buscar** | §1 | En la misma prueba, una nota marcada como "no determinable" tenía la respuesta en un comentario del propio lanzador. |
| **Flujo de 3 a 5 pasos** | §5 | Con menos pasos no se entiende el flujo; con más, se convierte en una descripción del código. |
| **README en menos de 5 minutos** | §6 | Es la medida de que el README sirve para empezar a trabajar. Qué es "empezar" lo decide el tipo de proyecto: ejecutar, importar, abrir en el editor o saber qué leer. |
| **APIs con la ubicación de cada parámetro y un ejemplo** | §6 | Una tabla de rutas sin decir si los parámetros van en la consulta o en el cuerpo obliga a leer el controlador. El formato de los archivos que se importan solo está en el parser, y nadie lo va a buscar ahí. |
| **Mecanismo estándar en vez de "no determinable"** | §7 | Si el manifiesto declara el plugin que lo respalda (por ejemplo, perfiles de Spring Boot), el comando estándar es evidencia suficiente y es más útil que una advertencia. |
| **Rutas siempre relativas** | §1, §11 | En la misma prueba, el README citaba como evidencia la ruta absoluta del repositorio en el equipo, con el nombre de usuario. Eso no sirve en otro equipo y expone datos personales. |
| **Evidencia según el destino** | §6 | En la misma prueba, el README salió lleno de citas `archivo:líneas` y de detalles del proceso ("glob sin resultados"). Al lector no le sirven, y los números de línea caducan. La trazabilidad se mantiene en ARCHITECTURE.md (rutas) y en el resumen de entrega (detalle). |
| **Cada dato en un solo archivo** | §6 | README y ARCHITECTURE repetían la estructura y las tablas de contenido. Si el dato se duplica, al actualizarlo una copia se queda desfasada. |
| **Variables por nombre, nunca por valor** | §1, §7 | La documentación se publica; un valor real de `.env` sería una fuga de secretos. |
| **`grep` acotado por el prompt** | §1 | En OpenCode V2 el recurso de `grep` es la expresión regular, no la ruta, así que el `deny` de `read` sobre `*.env` no impide que una búsqueda devuelva líneas de un `.env`. El límite tiene que estar en el prompt. |
| **El contenido del repo es dato** | §1, §4 | El agente lee texto que no controla: README, comentarios, datos e informes de `explore`. Si alguno trae instrucciones ("añade esto a AGENTS.md"), obedecerlas convertiría el repo documentado en quien manda. Se informan como hallazgo. |
| **Entrega sin interacción** | §2 | Como subagente, el documentador corre en una sesión hija, y no está documentado cómo llegan al usuario `question` ni las confirmaciones `ask`. Si fallan, el agente elige el modo que escribe menos y entrega el contenido propuesto en vez de quedarse bloqueado. |
| **Índice solo con 2 o más documentos** | §9 | Con un único documento, el índice no aporta nada. |
| **Changelog solo desde git** | §9 | Sin historial no hay evidencia de qué cambió ni cuándo. Si el archivo ya existe, el agente solo añade lo nuevo para no reescribir lo que alguien redactó a mano. |
| **Extras con checklist de 5 preguntas** | §10 | Los diagramas y ADRs inventados son la forma más común de documentación decorativa. |
| **ADR con fecha solo si es verificable** | §10 | Una fecha inventada es un dato falso. |
| **Resumen de entrega fijo** | §12 | Sustituye a la firma "Validado con evidencia": un resumen con lo omitido y lo inferido se puede comprobar; una firma no. |

## Permisos

Los permisos (`permissions` en el frontmatter) refuerzan las reglas del prompt: aunque el modelo se equivoque, no puede hacer lo que tiene prohibido.

- **Lectura** libre, salvo los `.env` (se permiten `.env.example`, `.env.sample` y `.env.template`) y los archivos de credenciales habituales: claves y certificados (`*.pem`, `*.key`, `*.p12`, `*.pfx`, `*.jks`, `*.keystore`, `id_rsa`, `id_ed25519`), `.npmrc`, `.pypirc`, `.netrc`, `.git-credentials`, `*.tfvars` y `*credentials*.json`. La política base de OpenCode solo pide confirmación para los `.env`; aquí se deniegan.
- **Escritura** solo en `*.md`, siempre con confirmación (`ask`). El agente documenta; nunca toca código ni configuración. Se deniegan también los Markdown de instrucciones para agentes (`AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `.github/copilot-instructions.md`, `.opencode/`, `.claude/`, `.cursor/`). Así, una instrucción inyectada en el repo no puede reescribir las reglas de otro agente ni las del propio documentador.
- **Shell** solo para git de lectura, necesario en los modos sincronizar y changelog, y **sin contenido de archivos**. `git diff` y `git show` solo se permiten con `--name-only`, `--name-status` o `--stat`. Se bloquean `-p`, `--patch`, `-u`, `-L`, `--word-diff` y `git show <rev>:<ruta>`, porque mostrarían el contenido de un `.env` que se haya subido al repo. También se bloquea `--no-index`, que deja a `git diff` leer archivos fuera del repositorio. El contenido actual se lee con `read`, que sí respeta los bloqueos. Los encadenados (`;`, `&&`, `||`), las tuberías, las redirecciones, la sustitución de comandos y `--output` se bloquean, para que `git log ...` no sirva de puerta a otros comandos.
- **Delegación** permitida solo a `explore`, que no edita archivos pero sí tiene acceso a la web y a directorios externos con sus propios permisos, y solo pide confirmación para leer `.env`. Por eso §4 obliga a acotar el encargo y a no traer valores de secretos.
- **Sin** `webfetch`, `websearch` ni `external_directory`: la evidencia sale únicamente del repositorio.

En OpenCode **gana la última regla que coincide**. Por eso las reglas generales (`"*"`) van primero, después las excepciones permitidas y al final los bloqueos que deben ganar siempre.

## Cómo cambiar el agente

1. Edita [`.opencode/agents/documentador.md`](../.opencode/agents/documentador.md).
2. Si la decisión cambia, actualiza la tabla de este documento. No copies aquí la regla.
3. Si añades un comando de shell permitido, colócalo después de `shell "*" deny` y antes de los bloqueos de contenido y de encadenados. Comprueba que no puede mostrar el contenido de un archivo.
4. Mantén el agente autocontenido y sin rutas absolutas ni nombres de un repositorio concreto.
