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

## Cómo conseguir las 4 llaves (Installed Package)

Necesitas **Client ID**, **Client Secret**, **Authentication Base URI** y el **MID** de la BU
padre (EID). Se obtienen creando un *Installed Package* en Marketing Cloud. Requiere el permiso
**Installed Package | Administer** (roles Administrator / Marketing Cloud Administrator).

**Atajo de un clic** — abre directo la página de Installed Packages (stack s13):

- Windows (PowerShell): `Start-Process "https://mc.s13.exacttarget.com/cloud/#app/Setup/InstalledPackages"`
- macOS: `open "https://mc.s13.exacttarget.com/cloud/#app/Setup/InstalledPackages"`
- Si ese enlace no cae en la página exacta: entra a Marketing Cloud → clic en tu **usuario**
  (arriba a la derecha) → **Setup** → **Platform Tools** → **Apps** → **Installed Packages**.

**Pasos:**

1. **New** → ponle nombre (ej. `mcdev-cli`) y descripción → **Save**.
2. Dentro del paquete: **Add Component** → **API Integration** → **Next**.
3. Tipo **Server-to-Server** → **Next**.
4. Marca los **scopes/permisos** que el agente necesitará (mínimo lectura/escritura de:
   Data Extensions, Automations, Journeys, Email, Content Builder / Assets, Data Extract,
   File Locations, Tracking). Puedes ampliarlos luego. → **Save**.
5. En el detalle del componente copia:
   - **Client Id** → `client_id`
   - **Client Secret** → `client_secret`
   - **Authentication Base URI** (ej. `https://XXXXX.auth.marketingcloudapis.com/`) → `auth_url`
   - **MID** de tu Business Unit padre (arriba, junto al nombre de la cuenta, o en Account Settings)
     → `account_id` / EID

> ⚠️ **El Client Secret caduca cada 180 días** (y Salesforce fuerza rotación de secretos de S2S
> desde 2026). Cuando caduque, genera uno nuevo aquí mismo y vuelve a conectar (`mcdev init addi`).

Con esos 4 datos, Claude (o tú, con `mcdev init addi`) deja mcdev conectado creando el
`.mcdev-auth.json`, que **nunca** se sube a git porque está en `.gitignore`.

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
