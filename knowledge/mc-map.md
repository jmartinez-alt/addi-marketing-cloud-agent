# Mapa de Marketing Cloud de Addi 🗺️

> Cerebro compartido del agente `marketing-cloud`. Se sincroniza por git (push directo a `main`).
> **Reglas:** solo *append* de estructura verificada contra el servidor. **Nunca** metas aquí
> credenciales (`client_secret`, tokens) ni datos con PII de clientes. Mantenlo conciso, en
> tablas/listas. Ante la duda, verifica con `mcdev reloadBUs addi` o un `retrieve`.

## Instancia

- **Stack / Tenant:** s13 (`mc.s13.exacttarget.com`). El `auth_url` del Installed Package es
  `https://<subdominio>.auth.marketingcloudapis.com/`.
- **Credencial mcdev:** `addi` (EID = MID de la BU padre).

## Business Units

_(Se llenan al correr `mcdev reloadBUs addi`. Anota aquí las BUs relevantes y para qué se usan.)_

| BU (mcdev: `addi/<nombre>`) | MID | Uso | Notas |
|---|---|---|---|
| _pendiente_ | | | |

## Data Extensions clave

_(Anota las DE importantes: nombre/key, BU, para qué journey/automation se usan, campos clave.)_

| Data Extension | BU | Uso | Campos clave |
|---|---|---|---|
| _pendiente_ | | | |

## Journeys

_(Journeys activos y qué disparan. Ojo: los journeys envían comunicaciones reales.)_

| Journey | BU | Qué dispara / entry source | Estado |
|---|---|---|---|
| _pendiente_ | | | |

## Automations

_(Automations y su schedule / SQL queries asociadas.)_

| Automation | BU | Qué hace | Schedule |
|---|---|---|---|
| _pendiente_ | | | |

## Convenciones y particularidades

_(Convenciones de nombres, prefijos por BU, scopes del Installed Package, gotchas del org.)_

- _pendiente_
