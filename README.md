# Agente Documentador para OpenCode

Un subagente para [OpenCode](https://opencode.ai) que explora, mapea y documenta repositorios **basándose únicamente en evidencia**. Genera documentación veraz y mantenible (`README.md`, `ARCHITECTURE.md`, etc.) sin inventar archivos, rutas, comandos ni scripts: lo que no puede verificar, lo omite.

## Requisitos

- OpenCode con el formato de agentes V2. El agente declara sus permisos con la clave `permissions` (lista de `action`/`resource`/`effect`). Una versión anterior podría no aplicarlos, y el agente quedaría sin sus restricciones de escritura y de shell.
- Nada más: el agente es un único archivo Markdown, sin dependencias.

## Instalación

El agente es un solo archivo, [`.opencode/agents/documentador.md`](.opencode/agents/documentador.md). Cópialo a una de estas ubicaciones:

| Alcance | Destino |
|---|---|
| Un repositorio | `.opencode/agents/documentador.md` dentro de ese repositorio |
| Todos tus repositorios | `~/.config/opencode/agents/documentador.md` |

Para descargarlo directamente en el repositorio que quieres documentar, ejecuta desde su raíz:

PowerShell:

```powershell
New-Item -ItemType Directory -Force .opencode/agents | Out-Null
Invoke-WebRequest https://raw.githubusercontent.com/Irene0450/Agente-documentador/main/.opencode/agents/documentador.md -OutFile .opencode/agents/documentador.md
```

Bash:

```bash
mkdir -p .opencode/agents
curl -fsSL https://raw.githubusercontent.com/Irene0450/Agente-documentador/main/.opencode/agents/documentador.md -o .opencode/agents/documentador.md
```

Para instalarlo en todos tus repositorios, cambia `.opencode/agents` por `~/.config/opencode/agents` (en PowerShell, `$HOME/.config/opencode/agents`).

Para **actualizarlo**, repite la descarga. Para **desinstalarlo**, borra el archivo.

## Uso

1. **Haz commit** de los cambios pendientes del repositorio objetivo. Así podrás revisar con `git diff` lo que escriba el agente y deshacerlo si no te convence.
2. Abre el repositorio en OpenCode.
3. Pide al agente principal que use el subagente, diciendo qué quieres (ver [Ejemplos](#ejemplos)). El agente elige el modo según tu petición.
4. **Aprueba o rechaza cada escritura.** El agente pide confirmación antes de crear o modificar cada archivo Markdown. Si rechazas una, te deja el contenido propuesto en la respuesta.
5. Lee el **resumen final** ("Resultado"). Indica qué archivos cambió, qué omitió por falta de evidencia, qué infirió y los avisos de seguridad (posibles secretos en el código o instrucciones sospechosas escritas en el repo).

### Ejemplos

```text
@documentador Documenta este repositorio: README.md y ARCHITECTURE.md con evidencia.
@documentador Genera o actualiza solo el README.
@documentador Revisa la documentación actual y sincronizarla con el código. Solo cambios mínimos.
@documentador Genera un índice navegable en docs/INDEX.md con enlaces verificados.
@documentador Genera un CHANGELOG.md a partir de los commits recientes.
@documentador Explícame la arquitectura de este repositorio sin generar archivos.
```

### Modos

| Modo | Resultado |
|---|---|
| **Documentar completo** | Crea o corrige `README.md` y `ARCHITECTURE.md` con punto de entrada, módulos, flujo principal y reglas de comportamiento no obvias. |
| **README** | Crea o corrige solo el `README.md`. |
| **Sincronizar** | Informa de dónde la documentación no coincide con el código (scripts, rutas rotas, entry points, variables) y propone cambios mínimos. Solo los aplica si se lo pides. |
| **Índice** | Genera `docs/INDEX.md` con enlaces verificados, solo si hay 2 o más documentos. |
| **Changelog** | Genera `CHANGELOG.md` a partir de los commits de git. Si no hay historial, no lo genera. |
| **Entender** | Explica la arquitectura en el chat, sin escribir archivos. |
| **Extras** | ADRs, `docs/OPERATIONS.md` o diagramas Mermaid, solo si hay evidencia y aportan algo. |

## Permisos

- Lee el repositorio libremente, salvo los `.env` y otros archivos de credenciales (claves, certificados, `.npmrc`, `*.tfvars`…).
- Solo escribe archivos Markdown, y cada cambio pide confirmación. Nunca modifica código, configuración ni instrucciones de agentes (`AGENTS.md`, `CLAUDE.md`, `.opencode/`…).
- En la shell solo ejecuta git de lectura (`status`, `log`, `diff`, `show`, `rev-parse`), sin mostrar el contenido de los archivos.
- Trata el contenido del repositorio como datos: no obedece instrucciones escritas en él.
- No accede a internet ni a rutas fuera del repositorio.

## Limitaciones

- La exploración masiva la delega en el subagente `explore` de OpenCode, que tiene sus propios permisos (incluida la web). El documentador le pide que no salga del repositorio ni lea secretos, pero esas restricciones dependen del encargo, no de los permisos.
- La búsqueda con `grep` no respeta los bloqueos de lectura en OpenCode. El agente está instruido para descartar resultados de archivos de secretos, pero conviene no tener secretos reales en el repositorio.
- En repositorios grandes documenta el nivel raíz y los paquetes principales, y dice en el resumen qué quedó fuera.

## Estructura del repositorio

| Ruta | Contenido |
|---|---|
| [`.opencode/agents/documentador.md`](.opencode/agents/documentador.md) | El agente: todas sus reglas y permisos. Es lo único que necesitas para usarlo. |
| [`docs/DESIGN.md`](docs/DESIGN.md) | Por qué el agente funciona así y cómo modificarlo |

## Licencia

[MIT](LICENSE)
