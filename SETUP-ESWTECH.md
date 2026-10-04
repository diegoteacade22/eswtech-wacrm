# eswtech-wacrm — Guía de configuración

Fork propio de [ArnasDon/wacrm](https://github.com/ArnasDon/wacrm) (MIT, template de CRM
para WhatsApp Business API oficial). Verificado: `npm run typecheck` ✅, `npm run build` ✅
(52 rutas). Sólo falta credenciales.

- Repo: https://github.com/diegoteacade22/eswtech-wacrm
- Local: `~/Projects/eswtech-wacrm`
- Stack: Next.js 16 · React 19 · Supabase (Postgres + Auth + RLS) · Meta Cloud API

---

## Paso 1 — Crear el proyecto Supabase

1. https://supabase.com/dashboard → **New project**
2. Nombre: `eswtech-wacrm`. Guardá la password de la DB.
3. **Settings → API** → copiá:
   - Project URL → `NEXT_PUBLIC_SUPABASE_URL`
   - `anon` `public` key → `NEXT_PUBLIC_SUPABASE_ANON_KEY`
   - `service_role` key → `SUPABASE_SERVICE_ROLE_KEY` ⚠️ nunca en el navegador

## Paso 2 — Correr las migraciones (42 archivos)

Opción A — dashboard: SQL Editor, corré `supabase/migrations/001…042` en orden.

Opción B — CLI (recomendada, deja historial):
```bash
cd ~/Projects/eswtech-wacrm
supabase login
supabase link --project-ref <TU_PROJECT_REF>
supabase db push
```

## Paso 3 — Variables de entorno

En `.env.local` (local) y en el panel del hosting (producción):

```bash
NEXT_PUBLIC_SUPABASE_URL=https://<ref>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<anon>
SUPABASE_SERVICE_ROLE_KEY=<service_role>
NEXT_PUBLIC_SITE_URL=https://crm.eswtech.com
NEXT_PUBLIC_APP_LOCALE=es
ENCRYPTION_KEY=<64 hex chars>
META_APP_SECRET=<secret de tu Meta App>
```

Generar `ENCRYPTION_KEY`:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

⚠️ Rotar `ENCRYPTION_KEY` deja huérfano todo token cifrado: hay que re-guardar
la config de WhatsApp de cada cuenta.

## Paso 4 — Deploy

**Hostinger** (el path que testea upstream): hPanel → Websites → Create → Node.js →
conectar el fork → pegar las env vars → push a `main`.

Alternativas: Vercel, Railway, VPS propio. El repo trae `docs/docker.md` si preferís
Docker Compose.

## Paso 5 — Conectar WhatsApp (dentro de la app)

Settings → WhatsApp connection. Por cuenta:

| Campo | De dónde sale |
|---|---|
| Phone Number ID | Meta → WhatsApp → API Setup — **sólo dígitos**, no el número con `+` |
| WABA ID | Meta → WhatsApp → API Setup — **sólo dígitos** |
| Permanent Access Token | Meta Business Settings → System Users → token permanente |
| Verify token | el mismo que configurás en el webhook de Meta |
| Two-step PIN | Meta → WhatsApp → API Setup |

El token temporal de 24h de API Setup **no sirve** (error 190). El permanente necesita
`whatsapp_business_management` + `whatsapp_business_messaging`.

Al guardar, la app verifica en orden: `verify_number` → `waba_phone_numbers` →
`register` (con PIN) → `subscribed_waba`. Si falla te dice en qué paso, con el código
de error de Meta y el trace ID para soporte.

### Webhook en Meta
Callback URL: `https://<tu-dominio>/api/whatsapp/webhook`
Verify token: el mismo que guardaste. Suscribirse al campo `messages`.
`META_APP_SECRET` debe coincidir con el App Secret de la Meta App.

---

## Notas para ESWTech

- **No usa QR.** Es la Cloud API oficial, el mismo camino que ya tenés con Coexistencia
  en `+1 305-464-1625`. No vincula dispositivos.
- **Historial:** Coexistencia sincroniza desde la fecha del alta hacia adelante, no
  importa historial previo. Para respaldar años de chats, exportar desde el celu
  (ver skill `whatsapp-cloud-api-onboarding`, script `wa_chat_export.py`).
- **Broadcasts:** fuera de la ventana de 24h sólo a contactos con opt-in y plantillas
  aprobadas por Meta.
- **MCP server** en `mcp-server/` — read-only por defecto. Sirve para conectar el CRM
  a Hermes/ZEN: `docs/mcp.md`.
- **Knowledge base** con retrieval híbrido (Postgres full-text o pgvector) —
 candidatas para la base de Drive / FAQ de ElectroSurWeb y XionGadget.

## Ya existe `eswtech-crm` en Vercel

Vercel tiene los proyectos `eswtech-crm` (responde, login de Vercel) y
`eswtech-crm-stage-2` (404) con 12 env vars ya cargadas
(`SUPABASE_SERVICE_ROLE_KEY`, `NEXT_PUBLIC_SUPABASE_URL`, `OPENAI_API_KEY`,
`CRM_AUTH_TOKEN`, `CRM_ACCESS_PASSWORD`, `CRM_DISABLE_LOGIN`, etc.).

Antes de poner wacrm en producción, decidir: ¿se reemplaza, o conviven? El
`NEXT_PUBLIC_SUPABASE_URL` que ya está en Vercel es la fuente de verdad del
Supabase actual de ESWTECH.