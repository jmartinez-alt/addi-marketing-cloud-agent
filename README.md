# Agente de Marketing Cloud de Addi 📧⚡

Un subagente de **Claude Code** que ayuda al equipo a trabajar con la instancia de
**Salesforce Marketing Cloud** (Engagement / ExactTarget) de Addi desde la terminal:
descargar (retrieve) y desplegar (deploy) metadata entre Business Units, gestionar
Data Extensions, Journeys, Automations, Emails, SQL Queries, AMPscript y más.

Usa el CLI **[mcdev](https://github.com/Accenture/sfmc-devtools)** (Accenture SFMC DevTools),
que es el equivalente de facto al `sf` CLI pero para Marketing Cloud (Salesforce no tiene
un CLI oficial para MC).

Es el hermano del [agente de Salesforce](https://github.com/esmartinez10/addi-salesforce-agent):
misma arquitectura, mismo estilo. Lo especial: **aprende con el uso**. Mantiene un "mapa de
Marketing Cloud" (`knowledge/mc-map.md`) que se comparte por git — cuando un colega descubre
algo nuevo (una BU, una Data Extension, un Journey), el agente lo guarda y lo comparte con todos.

## Requisitos

- [Claude Code](https://claude.com/claude-code)
- [Node.js](https://nodejs.org) v18+ (trae `npm`)
- [mcdev](https://github.com/Accenture/sfmc-devtools) — `npm install -g mcdev`
- `git`
- Acceso a este repositorio en GitHub (lectura + escritura)
- Un **Installed Package** en Marketing Cloud (Server-to-Server) para obtener `Client ID`,
  `Client Secret`, `Auth Base URI` y el `MID` de la BU padre (EID)

## Instalación (fácil — la hace la IA por ti)

**No necesitas saber git ni comandos.** Solo:

1. Clona o descarga este repositorio (o pídele el link a alguien del equipo).
2. Abre la carpeta en **Claude Code**.
3. Escribe: **`instala el agente de Marketing Cloud`**
4. Claude te irá pidiendo **aprobar** unos comandos (instalar mcdev, enlazar el agente,
   conectar Marketing Cloud). Para conectar necesitarás las 4 credenciales del Installed Package.

> Los detalles técnicos del instalador viven en [`CLAUDE.md`](./CLAUDE.md) — es lo que Claude Code
> lee para saber cómo instalar, autenticar y sincronizar el aprendizaje.

## Cómo conectar Marketing Cloud (Installed Package)

1. En Marketing Cloud → **Setup → Platform Tools → Apps → Installed Packages → New**.
2. Crea un paquete y añade un componente **API Integration** de tipo **Server-to-Server**.
3. Asigna los scopes/permisos que necesites (leer/escribir Data Extensions, Automations,
   Journeys, Email, etc.).
4. Copia **Client ID**, **Client Secret**, **Authentication Base URI** y el **MID** de la BU padre.

Con esos 4 datos, Claude deja mcdev conectado (crea el `.mcdev-auth.json`, que **nunca** se sube
a git porque está en `.gitignore`).

## Cómo funciona el aprendizaje compartido

- Al empezar una tarea, el agente actualiza el mapa (`git pull`) y lo lee.
- Cuando descubre estructura nueva, la **agrega** al mapa y la **comparte** (`git push`).
  Tú solo apruebas el prompt.
- El mapa es una foto que puede desactualizarse: el agente **siempre verifica contra el servidor
  real** (retrieve / reloadBUs) antes de dar algo por cierto.

## Reglas de seguridad del agente

- **Retrieve/backup primero:** antes de modificar algo, descarga el estado actual.
- **Deploy y acciones que disparan comunicaciones** (`deploy`, `execute`, `publish`, `schedule`,
  `stop`, `delete`) **piden tu confirmación explícita** antes de ejecutarse — un Journey o
  Automation puede enviar correos reales a clientes.
- **Nunca** subir el `.mcdev-auth.json` ni datos con PII de clientes al repo.

## Estructura del repo

```
.
├─ README.md                            # este archivo
├─ CLAUDE.md                            # playbook que Claude Code usa para instalar/autenticar/sincronizar
├─ .mcdevrc.json                        # config de mcdev (BUs, EID) — SIN secretos, se versiona
├─ .gitignore                           # excluye el secreto (.mcdev-auth.json) y los datos de mcdev
├─ .claude/agents/marketing-cloud.md    # el agente (comportamiento + reglas)
└─ knowledge/mc-map.md                  # el cerebro compartido (se sincroniza por git)
```
