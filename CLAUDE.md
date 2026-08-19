# CLAUDE.md — Playbook del agente de Marketing Cloud de Addi

> **Para Claude Code:** este repositorio contiene un subagente (`marketing-cloud`) y su cerebro
> compartido (`knowledge/mc-map.md`), además de la configuración de un proyecto **mcdev**
> (Accenture SFMC DevTools) para hablar con Salesforce Marketing Cloud. Los colegas que lo
> instalan **no saben de git ni de línea de comandos** — tú haces todo y ellos solo **aprueban**
> los comandos. Sé explícito sobre qué vas a correr y por qué. Responde en español.

## Convenciones fijas (no las cambies)

- **Ubicación estándar del repo clonado:** `<HOME>/.claude/marketing-cloud-agent`
  - `<HOME>` = `$env:USERPROFILE` (Windows) o `$HOME` (macOS/Linux). Resuélvelo en runtime.
  - Este es el `<REPO>`. **Todos los comandos `mcdev` se corren desde aquí** (mcdev lee `.mcdevrc.json` del directorio actual).
- **Nombre de credencial mcdev (obligatorio):** `addi`. El agente se refiere a las BUs como `addi/<NombreBU>`.
- **Config sin secretos (se versiona):** `<REPO>/.mcdevrc.json`.
- **Secretos (NUNCA se versionan):** `<REPO>/.mcdev-auth.json` — ya está en `.gitignore`.
- **Cerebro compartido:** `<REPO>/knowledge/mc-map.md`. Se sincroniza por git, push directo a `main`.

---

## 1. Instalación (una sola vez por colega)

Cuando un colega diga *"instala el agente de Marketing Cloud"*, ejecuta estos pasos (pide
aprobación de cada comando; explica en una línea qué hace).

### 1a. Prerrequisitos
- **Claude Code** (ya lo tiene si te está hablando).
- **Node.js** (v18+) — verifica con `node --version`. Trae `npm`.
- **mcdev** (Accenture SFMC DevTools) — verifica con `mcdev --version`. Si falta, instálalo global:
  `npm install -g mcdev`.
  - ⚠️ **safe-chain:** en máquinas de Addi `npm`/`npx` pueden estar envueltos por `safe-chain`
    y su binario roto (error `Cannot find module ...\npm`). Si pasa, llama a npm real directo:
    `& "C:\Program Files\nodejs\node.exe" "C:\Program Files\nodejs\node_modules\npm\bin\npm-cli.js" install -g mcdev`.
    El `mcdev.cmd` resultante NO pasa por safe-chain, así que luego `mcdev ...` funciona normal.
- **git** — verifica con `git --version`. Si falta, instálalo (`winget install Git.Git` / `brew install git`).
- **Acceso al repo en GitHub** — lectura/escritura (push directo requiere escritura). Si git pide
  credenciales, guía a `gh auth login` (si tiene GitHub CLI) o un Personal Access Token.

### 1b. Clonar el repo a la ubicación estándar
```
git clone <URL_DEL_REPO> "<HOME>/.claude/marketing-cloud-agent"
```
Si ya existe: `git -C "<HOME>/.claude/marketing-cloud-agent" pull`.

### 1c. Enlazar el agente en el `~/.claude` personal (para que funcione en cualquier proyecto)
Preferido — **symlink** (así `git pull` también actualiza el comportamiento del agente):
- Windows (PowerShell, puede requerir Modo Desarrollador o admin):
  ```
  New-Item -ItemType SymbolicLink -Path "$env:USERPROFILE\.claude\agents\marketing-cloud.md" -Target "$env:USERPROFILE\.claude\marketing-cloud-agent\.claude\agents\marketing-cloud.md"
  ```
- macOS/Linux:
  ```
  ln -sf "$HOME/.claude/marketing-cloud-agent/.claude/agents/marketing-cloud.md" "$HOME/.claude/agents/marketing-cloud.md"
  ```
Si el symlink falla (permisos en Windows), **copia** como fallback y avisa que tras cada
`git pull` habrá que volver a copiar:
```
Copy-Item "$env:USERPROFILE\.claude\marketing-cloud-agent\.claude\agents\marketing-cloud.md" "$env:USERPROFILE\.claude\agents\marketing-cloud.md" -Force
```
(Crea antes `<HOME>/.claude/agents/` si no existe.)

### 1d. Conectar Marketing Cloud (crear `.mcdev-auth.json`)
mcdev se autentica con un **Installed Package** de Marketing Cloud (Setup → Installed Packages →
componente **API Integration** tipo **Server-to-Server**). De ahí salen 4 datos:
`Client ID`, `Client Secret`, `Authentication Base URI` y el `MID` de la BU padre (EID).

El secreto va en `<REPO>/.mcdev-auth.json` con este formato (nombre de credencial = `addi`):
```json
{
    "addi": {
        "client_id": "<CLIENT_ID>",
        "client_secret": "<CLIENT_SECRET>",
        "auth_url": "https://<SUBDOMINIO>.auth.marketingcloudapis.com/",
        "account_id": <EID_NUMERICO>
    }
}
```
**Manejo del secreto (importante):** no imprimas el `client_secret` en la conversación. Opciones:
  1. **Recomendada:** pide al colega que ejecute la conexión interactiva en SU terminal con el
     prefijo `!` (así el secreto no queda en el chat): `! cd "<REPO>"; mcdev init addi`
     (mcdev preguntará los 4 valores y escribirá el `.mcdev-auth.json` por él).
  2. Alternativa: que el colega pegue los valores en `.mcdev-auth.json` a mano.
  3. Si el colega prefiere que tú lo escribas, hazlo pero recuérdale que el secreto quedará en el
     historial del chat y conviene rotarlo si es sensible.

Luego pon el EID también en `.mcdevrc.json` (`credentials.addi.eid`) y refresca las BUs:
```
cd "<REPO>"; mcdev reloadBUs addi
```
Esto valida la conexión y llena la lista de Business Units en `.mcdevrc.json`.

### 1e. Confirmar
Prueba de solo lectura (no dispara nada): `cd "<REPO>"; mcdev retrieve addi/<BU> dataExtension`
o simplemente revisa que `mcdev reloadBUs addi` haya traído las BUs. Dile al colega que abra
cualquier proyecto en Claude Code y pida algo de Marketing Cloud; el agente `marketing-cloud`
se activa solo.

---

## 2. Cómo funciona el aprendizaje compartido (explícaselo si preguntan)

- El agente lee `knowledge/mc-map.md` al empezar cada tarea, **después** de `git pull`.
- Cuando descubre estructura nueva (BU, Data Extension, Journey, Automation, convención), la
  **agrega** (append), hace commit y **push directo a `main`**.
- El colega solo ve prompts de aprobación de git — no necesita saber git.
- **Nunca** subas `.mcdev-auth.json` ni datos con PII de clientes al cerebro. **Nunca borres**
  contenido del mapa salvo que compruebes contra el servidor que está equivocado.

## 3. Actualizar el agente / el cerebro
```
git -C "<HOME>/.claude/marketing-cloud-agent" pull --rebase --autostash
```
(Si el agente se instaló por copia y no por symlink, vuelve a copiar `marketing-cloud.md` — paso 1c.)

## 4. Resolución de problemas
- **El agente `marketing-cloud` no aparece:** revisa que exista `<HOME>/.claude/agents/marketing-cloud.md`
  (symlink o copia) y reinicia Claude Code.
- **`npm` falla con `Cannot find module ...\npm`:** es safe-chain roto; usa el npm real directo (ver 1a).
- **Error de auth / token / scope:** revisa `.mcdev-auth.json` (client_id/secret/auth_url/account_id)
  y que el Installed Package tenga los permisos/scopes necesarios. Prueba `mcdev reloadBUs addi --verbose`.
- **`git push` rechazado:** `git -C "<REPO>" pull --rebase --autostash` y reintenta; si es por
  credenciales, re-autentica (`gh auth login` / PAT).
