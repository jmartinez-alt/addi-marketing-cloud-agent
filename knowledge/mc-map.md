# Mapa de Marketing Cloud de Addi 🗺️

> Cerebro compartido del agente `marketing-cloud`. Se sincroniza por git (push directo a `main`).
> **Reglas:** solo *append* de estructura verificada contra el servidor. **Nunca** metas aquí
> credenciales (`client_secret`, tokens) ni datos con PII de clientes. Mantenlo conciso, en
> tablas/listas. Ante la duda, verifica con `mcdev reloadBUs addi` o un `retrieve`.

## Instancia

- **Stack / Tenant:** s13 (`mc.s13.exacttarget.com`). El `auth_url` del Installed Package es
  `https://mc1vr5zw7cydtlvpy2j89--pg470.auth.marketingcloudapis.com/`.
- **Credencial mcdev:** `addi`. **EID / MID padre:** `546003659`.
- **Cuenta:** Adelante Soluciones Financieras SAS (NIT: 9012167) — la razón social de Addi.

## Business Units

_(Verificado con `mcdev reloadBUs addi` el 2026-08-19.)_

| BU (mcdev: `addi/<nombre>`) | MID | Uso | Notas |
|---|---|---|---|
| `_ParentBU_` (Adelante Soluciones Financieras SAS) | 546003659 | BU padre / cuenta principal | Único BU visible con el Installed Package actual. Si se necesitan BUs hijas, ampliar el acceso del package y correr `mcdev reloadBUs addi`. |

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

## Data Extensions clave (verificado 2026-08-19)

- Total: **508 DEs**, 448 sendable (todas con Subscriber Key). Muchas basura: ~53 `*_SimulationSupportDE_*`, `Prueba*`, `test*`.
- Volumen dominado por tracking, NO por contactos: `DE_Reports_Tracking` = 33.8M filas.
- `Contactos sin correo ni celular` (key GUID `0A86171F-...`) = 17,702 filas, solo campos `Contact Key`+`Fuente` → lista curada de candidatos a borrar (sin email ni phone).
- `DE_Import_Whatsapp_ALLC` = 97,237 filas con Phone → posible maestra de teléfonos.
- `DE_Contacs_Email_Phone_NOTEMPTY` = **vacía (0 filas)** pese al nombre.
- Las DEs sincronizadas de Salesforce (`Contact_Salesforce`, `User_Salesforce`, `Lead_Salesforce`) dan **403** en el endpoint REST customobjectdata (no se leen por ahí). ~243/508 DEs dieron 403.

## Análisis de contactos / límite de facturación (2026-08-19)

**Problema:** contrato 500k contactos; están ~600k. Objetivo: identificar borrables (sin email/phone) y duplicados.

- **All Contacts (facturable, leído de Contact Builder UI 2026-08-19) = 582,515.** Contrato: 500k → sobran **82,515**. NO hay API de conteo de All Contacts (Contacts API da 404 en este package); la fuente autoritativa es la pantalla All Contacts.
- **All Subscribers (padrón email) = 328,518** (56% del total). 0 emails vacíos, 0 inválidos (todos los email-subs tienen email válido).
- **No-email (por diferencia) = 582,515 − 328,518 = 253,997** (44%): SMS/WhatsApp/push. Ahí viven los "sin email". No verificado por SQL (usuario prefirió no crear objetos).
- **Duplicados por email = 99,309 registros extra (30.2%)** → mismo email, distinto SubscriberKey (= ID de Salesforce). Consolidar a 1/email deja 229,209.
- Desuscritos: 4,761 (1.4%).
- **All Contacts (facturable, ~600k) > 328k email-subs.** El gap (~270k) son contactos NO-email (SMS/WhatsApp/push) → ahí viven los "sin email". Falta medir con SQL sobre `_MobileAddress` / `_Subscribers` (Query Activity).
- **Causa raíz de duplicados = Salesforce** (Contacts duplicados con mismo email que se sincronizan). Fix real: dedupe en SF o cambiar estrategia de SubscriberKey. Conecta con el agente de Salesforce.
- **SubscriberKey = IDs de Salesforce** (`006...` = Contact/Account).

## Cómo se sacó (scripts read-only)

- Carpeta local (fuera del repo, gitignored): `C:\Users\jmartinez_addi\mc-analysis\`.
- Token OAuth v2 (client_credentials) → REST `data/v1/customobjectdata/key/{key}/rowset?$pageSize=1` para row counts; SOAP `Retrieve` de `Subscriber` (paginado 2500, ContinueRequest) para el padrón.
- El Installed Package tiene scopes amplios (data_extensions_read, list_and_subscribers_read, etc.).

## Journey + Automation "AutomatizacionM0" (analizado 2026-08-20)

Nurture de onboarding/activación SMB (MBs), multicanal (~30 días). Creado por sguillen (Comms Dayana).

**Flujo de datos (Automation `AutomatizacionM0`, scheduled, cada 15 min, HOY EN PAUSA):**
1. Query `AutomatizacionM0` (Update) → DE entrada `AutomatizacionM0` (key F348506D). Fuente: `Opportunity_Salesforce` + joins a Account/Contact (Associated + Legal Rep). Filtros: Checkin_Date_Activation NOT NULL, rank=1, Cluster='SMB', Acquisition_Channel='MBs', StageName IN (Activation,Onboarding,Churned,Early life cycle). Normaliza Phone a '57...'.
2. Query `AutomatizacionM02` (Update) → `DE_Import_Whatsapp_ALLC` (FFE4485E): solo nuevos (LEFT JOIN ... WHERE D.ContactKey IS NULL), Id as ContactKey, Phone, 'co' Locale.
3. ImportFile `Import_WhatsApp_v2` → importa a `DE_Import_Whatsapp_ALLC` (misma DE ⇒ posible redundancia).

**Journey (Draft, NUNCA publicado; entryMode=MultipleEntries):** 120 actividades (54 WAIT, 45 decisiones MultiCriteria, 9 WhatsApp, 6 Email, 6 SMS). Entrada = DE audience `AutomatizacionM0`. Splits por Segmento/Stage/Flujo/Dias con cadencia Día 8/11/17/21/24/27/30 y variantes A/B. Sin goals ni exit.

**Cambios aplicados (2026-08-20, vía mcdev deploy, sin --schedule = sigue en pausa):**
- Schedule 15 min → **cada 4 h** (`FREQ=HOURLY;INTERVAL=4`).
- Notificación de **error** a `sguillen@addi.com` (formato mcdev: `notifications:[{email:[...],message:"",type:"Error"}]`; se aplica vía `/legacy/v1/beta/automations/notifications/{programId}`).
- Método: editar meta en `deploy/`, `mcdev deploy addi/_ParentBU_ automation <key>`. mcdev NO activa salvo `--schedule`. mcdev sí gestiona notifications; NO gestiona goals de journey.

**WhatsApp: `defaultContactKey` es lo correcto (revertido 2026-08-25).** Historia: el 2026-08-20 se cambiaron las 9 WA a `mobileNumberExpression=phone` (por pedido del usuario, advirtiendo el riesgo). Journey Builder lanzó el error *"The whatsApp mobile Expression configuration is invalid. Modify whatsApp Settings to include a valid attribute"*. Causa: el binding del atributo quedó **incompleto** al setearlo por JSON/mcdev (JB necesita que el atributo se seleccione desde la UI para generar todos los campos internos) + el patrón real de la cuenta es enviar por el móvil **registrado** del contacto (`defaultContactKey` + registro en `DE_Import_Whatsapp_ALLC`). **Fix aplicado: revertir las 9 WA a `defaultContactKey`** (deploy mcdev). **LECCIÓN: NO editar el número de WhatsApp por JSON; usar la UI. Y en esta cuenta WhatsApp SIEMPRE va por `defaultContactKey`.** Backup en `_backup_M0_20260820/`. Journey en Draft.
**Goal:** descartado por el usuario (no se agregó).
**Deploy de journey:** vía `mcdev deploy addi/_ParentBU_ journey <key>` (soporte beta, funcionó; update atómico, actualiza el Draft in-place sin nueva versión).

**Otros bugs/riesgos detectados:**
- WhatsApp usa `mobileNumberExpression=defaultContactKey` (= ID de Salesforce, no el phone) → no entregaría. CRÍTICO, verificar.
- Normalización de phone: `REPLACE(...,'57','')` borra TODO '57' del número (no solo el prefijo) → corrompe números válidos.
- Sin phone → Phone='57' (inválido) en vez de excluir.
- Sin goal/exit → los que activan siguen recibiendo los 30 días.
- MultipleEntries + refresh cada 15 min → riesgo de re-entrada/envíos duplicados.
- Schedule cada 15 min excesivo para onboarding (mejor horario/diario).

## Acceso usuario / Content Builder (2026-08-20)

Caso `nlizarazo@addi.com`: error UNAUTHORIZED en Content Builder. Diagnóstico por API: usuario activo, no bloqueado, BU Adelante, roles **Administrator + Marketing Cloud Administrator** (idéntico a un admin que sí entra); el rol Administrator (`SYS_DEF_ADMIN`) incluye Content Builder/Content Builder Admin; sin "Individual role for..." (sin denies individuales). → NO era provisión ni permisos: era **propagación/caché de permisos**. **Solución que funcionó:** re-guardar el rol Administrator en UI (Setup→Users→quitar y re-marcar Administrator→Save) + relogin. NO se puede hacer por API/mcdev: el rol "Marketing Cloud Administrator" NO es asignable por API y se borraría al actualizar el usuario. El agente no controla el navegador → este fix es UI-only.

## DEs de comms fed-by-file: goalcomms, Merchant_Intelligence_SBs (2026-08-20)

Migración a Marketing Cloud Next. Hallazgo: **NO se alimentan del CRM vía Automation Studio.**
- `goalcomms` (key 6274A1E2, Comms Jose): campos Id/Name/Email/Phone/Variable/Variable_2..4. Se llena por **importFile `goalcomms.csv`** (automation "comms goal" 27482B8A, tipo triggered). Los `Variable*` son contenido de campaña, no CRM.
- `Merchant_Intelligence_SBs` (key BC5F8F34, COMMS_Nicolas): campos id/phone/email/name/segmento/canal/variable/variable_2/3. **Nada la referencia** en MC → carga **manual**.
- Patrón recurrente: las queries "goalcomms"/"premioscomms"/etc NO llenan su DE; leen la DE de comms y registran ContactKey+Phone en `DE_Import_Whatsapp_ALLC` (FFE4485E) para habilitar WhatsApp.

**Cómo el org SÍ deriva segmento/canal desde CRM (ej. query AutomatizacionM0, fuente `Opportunity_Salesforce`+Account+Contact):**
- **canal** ← `Channel__c` + `Paylink_Type__c` (PAY_LINK+PHYSICAL STORE=Tienda Física, +SOCIAL SELLING=Redes sociales, E_COMMERCE=Página web, MARKETPLACE=Marketplace).
- **segmento** ← `DATEDIFF(Checkin_Date_Activation__c, hoy)` = Days → S0(1-7)/S1(8-14)/S2(15-21)/S3(22-30)/>30 fuera.
- **variable*** = contenido de campaña, NO campo CRM.
- Otros: `Vertical__c`, `Ally_Cluster__c`, `Acquisition_Channel__c`, `StageName`. Query `DIAG_Canal_CRM`: canal='CRM - Contact/Lead/User' según objeto origen.

## Convenciones y particularidades

- SubscriberKey = ID de Salesforce (contactos sincronizados vía Marketing Cloud Connect).
- Keys de muchas DEs están en formato GUID (mcdev avisa que conviene renombrarlas).
