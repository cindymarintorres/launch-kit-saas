# 🚀 LaunchKit

**Startup SaaS landing construida con Astro 6, React Islands y Tailwind CSS v4.** Un sitio híbrido (SSG + SSR) para una startup ficticia que vende herramientas para lanzar productos: waitlists, feedback y analytics. Incluye una miniconferencia anual llamada **LaunchConf**.

> Proyecto de portafolio orientado al aprendizaje de Astro 6: Islands Architecture, Content Collections, SSR híbrido, API Routes y deploy en Vercel.

---

## ✨ Características principales

- 🏝️ **Islands Architecture** — React solo donde se necesita JS, el resto es HTML estático puro.
- 📦 **Content Collections** — Changelog y speakers tipados con Zod y Markdown.
- 🔀 **Modo híbrido** — SSG para la mayoría de páginas, SSR para formularios y datos dinámicos.
- 🔌 **API Routes** — Endpoints serverless para waitlist y contador de registrados.
- ⚡ **SubscriberCounter** — Island con polling cada 10s a `/api/subscribers` sin recargar la página.
- 💱 **PricingToggle** — Switch mensual/anual que cambia los precios en tiempo real.
- 🎟️ **LaunchConf** — Sección de conferencia ficticia con speakers, agenda, countdown y registro.
- 🌗 **Dark/Light mode** — ThemeToggle island con persistencia en `localStorage`.
- 🚀 **Deploy en Vercel** — SSR y API Routes como serverless functions automáticamente.

---

## 🛠️ Stack tecnológico

| Capa | Tecnología | Detalles |
|:---|:---|:---|
| **Framework** | 🚀 Astro 6 | `output: hybrid` — SSG + SSR en el mismo proyecto |
| **UI Components** | ⚛️ React 18 | Solo para Islands interactivas |
| **Estilos** | 🎨 Tailwind CSS v4 | CSS-first config, `@theme` tokens, sin `tailwind.config.js` |
| **Lenguaje** | 📘 TypeScript | Strict mode |
| **Validación** | ✅ Zod | En API routes |
| **Content** | 📝 Content Collections | Changelog y speakers en Markdown con schemas Zod |
| **Deploy** | ☁️ Vercel | `@astrojs/vercel` adapter — serverless functions automáticas |
| **Routing** | 📁 File-based routing | `src/pages/` define las rutas automáticamente |

---

## 💡 Idea del Proyecto

LaunchKit es una startup ficticia que vende una herramienta SaaS para que otras startups lancen sus productos rápidamente. El sitio combina una landing de producto con una miniconferencia anual (**LaunchConf**), dando el balance perfecto de páginas estáticas y dinámicas para practicar todos los conceptos de Astro.

**¿Por qué este proyecto y no otro?**

- Es un tipo de sitio muy común en el mundo real — toda startup tiene uno.
- Tiene mezcla natural de contenido estático (hero, features, pricing) con partes dinámicas (waitlist, contador).
- Permite demostrar Astro, React, Tailwind y Vercel en un solo repositorio.
- Se ve creíble en un portafolio — no parece un tutorial de blog.
- La sección LaunchConf agrega variedad visual y un segundo flujo SSR sin forzarlo.

---

## 🏝️ Islands de React — Mapa Completo

| Componente | Directiva | Dónde se usa | Por qué necesita JS |
|:---|:---|:---|:---|
| `WaitlistForm.tsx` | `client:load` | `/waitlist` | Validación Zod, fetch, estados loading/error |
| `SubscriberCounter.tsx` | `client:load` | Home (`/`) | Polling cada 10s a `/api/subscribers` |
| `PricingToggle.tsx` | `client:load` | `/pricing` | Switch mensual/anual — estado React |
| `FAQAccordion.tsx` | `client:idle` | `/` | Acordeón — solo necesita JS al interactuar |
| `ThemeToggle.tsx` | `client:load` | Header (todas) | Dark/light — necesita `localStorage` |
| `ConferenceTimer.tsx` | `client:load` | `/conf` | Countdown — actualiza cada segundo |
| `ConfRegisterForm.tsx` | `client:load` | `/conf/register` | Formulario de registro |

---

## 📦 Content Collections

Content Collections es la forma nativa de Astro para manejar contenido estructurado (Markdown, MDX, JSON) con type safety via Zod. Reemplaza lo que harías con un CMS simple.

### Colecciones del proyecto

| Colección | Archivos | Schema |
|:---|:---|:---|
| `changelog` | `src/content/changelog/*.md` | `title`, `date`, `version`, `type` (feature/fix/breaking) |
| `speakers` | `src/content/speakers/*.md` | `name`, `role`, `company`, `bio`, `talk` |

### Ejemplo — `config.ts`

```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

const changelog = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    version: z.string(),
    date: z.coerce.date(),
    type: z.enum(['feature', 'fix', 'breaking']),
  }),
});

const speakers = defineCollection({
  type: 'content',
  schema: z.object({
    name: z.string(),
    role: z.string(),
    company: z.string(),
    bio: z.string(),
    talk: z.string(),
  }),
});

export const collections = { changelog, speakers };
```

### Consumir una colección en una página

```astro
---
// src/pages/changelog/index.astro
import { getCollection } from 'astro:content';

const entries = await getCollection('changelog');
const sorted = entries.sort((a, b) => b.data.date.getTime() - a.data.date.getTime());
---

{sorted.map(entry => (
  <article>
    <h2>{entry.data.title}</h2>
    <span>{entry.data.version}</span>
  </article>
))}
```

---

## 🔌 API Routes

Las API routes son archivos en `src/pages/api/` que exportan funciones HTTP. En Vercel, cada una se convierte en una serverless function automáticamente.

| Endpoint | Método | Descripción | Validación |
|:---|:---|:---|:---|
| `/api/waitlist` | `POST` | Guarda el email de registro | Zod: email válido |
| `/api/subscribers` | `GET` | Devuelve `{ count: number }` | Sin auth |
| `/api/conf-register` | `POST` | Registro a LaunchConf | Zod: nombre, email, ticket type |

### Ejemplo — API Route con Zod

```typescript
// src/pages/api/waitlist.ts
import type { APIRoute } from 'astro';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
});

export const POST: APIRoute = async ({ request }) => {
  const body = await request.json();
  const result = schema.safeParse(body);

  if (!result.success) {
    return new Response(JSON.stringify({ error: 'Email inválido' }), { status: 400 });
  }

  // Guardar en DB o archivo JSON...
  return new Response(JSON.stringify({ ok: true }), { status: 201 });
};
```

---

## 🏗️ Arquitectura de Carpetas

```
launchkit/
├── src/
│   ├── components/
│   │   ├── Header.astro              ← HTML estático — 0 JS
│   │   ├── Footer.astro              ← HTML estático
│   │   ├── Hero.astro                ← HTML estático
│   │   ├── PricingCard.astro         ← HTML estático
│   │   ├── SpeakerCard.astro         ← HTML estático
│   │   ├── WaitlistForm.tsx          ← 🏝️ Island React
│   │   ├── SubscriberCounter.tsx     ← 🏝️ Island React
│   │   ├── PricingToggle.tsx         ← 🏝️ Island React
│   │   ├── FAQAccordion.tsx          ← 🏝️ Island React
│   │   ├── ThemeToggle.tsx           ← 🏝️ Island React
│   │   ├── ConferenceTimer.tsx       ← 🏝️ Island React
│   │   └── ConfRegisterForm.tsx      ← 🏝️ Island React
│   │
│   ├── content/
│   │   ├── config.ts                 ← Schemas Zod de colecciones
│   │   ├── changelog/
│   │   │   ├── v1.0.0.md
│   │   │   └── v1.1.0.md
│   │   └── speakers/
│   │       ├── speaker-1.md
│   │       └── speaker-2.md
│   │
│   ├── layouts/
│   │   ├── BaseLayout.astro          ← Layout base con <slot />
│   │   └── ConfLayout.astro          ← Layout especial para LaunchConf
│   │
│   ├── pages/
│   │   ├── index.astro               ← SSG: Home
│   │   ├── pricing.astro             ← SSG: Precios
│   │   ├── about.astro               ← SSG: About
│   │   ├── changelog/
│   │   │   ├── index.astro           ← SSG: Lista changelog
│   │   │   └── [version].astro       ← SSG: Detalle de versión
│   │   ├── conf/
│   │   │   ├── index.astro           ← SSG: LaunchConf landing
│   │   │   └── register.astro        ← SSR: Formulario de registro
│   │   ├── waitlist.astro            ← SSR: Página de waitlist
│   │   └── api/
│   │       ├── waitlist.ts           ← API Route POST
│   │       ├── subscribers.ts        ← API Route GET
│   │       └── conf-register.ts      ← API Route POST
│   │
│   └── styles/
│       └── global.css                ← @import 'tailwindcss' + @theme tokens
│
├── public/
│   └── og-image.png                  ← Open Graph para SEO
│
├── astro.config.mjs                  ← output: hybrid + vercel adapter
├── tsconfig.json
└── package.json
```

---

---

## 🚀 Setup Inicial — Paso a Paso

### Prerrequisitos

- Node.js v20+ (LTS recomendado)
- `pnpm` o `npm` v9+
- Cuenta en Vercel (gratis) — [vercel.com](https://vercel.com)
- Cuenta en GitHub para el deploy automático


### 1️⃣ Clonar el repositorio

```bash
git clone https://github.com/cindymarintorres/launch-kit-saas.git
cd launchkit-saas
```

### 2️⃣ Instalar dependencias

```bash
pnpm install
```

### 3️⃣ Arrancar el dev server

```bash
pnpm run dev                # localhost:4321
```
---

## 🛑 Comandos de Referencia Rápida

```bash
# Desarrollo
pnpm run dev                  # Dev server en localhost:4321

# Build
pnpm run build                # Build de producción → genera dist/
pnpm run preview              # Previsualizar el dist/ localmente

# Type checking
npx astro check              # Verifica tipos en archivos .astro

# Agregar integraciones
npx astro add react
npx astro add tailwind
npx astro add vercel
npx astro add sitemap        # Genera sitemap.xml automáticamente

# Deploy
git push origin main         # Vercel hace redeploy automático
```

---

## 🗺️ Rutas de la Aplicación

| Ruta | Archivo | Tipo | Protegida |
|:---|:---|:---|:---|
| `/` | `index.astro` | SSG | No |
| `/pricing` | `pricing.astro` | SSG | No |
| `/about` | `about.astro` | SSG | No |
| `/changelog` | `changelog/index.astro` | SSG | No |
| `/changelog/[version]` | `changelog/[version].astro` | SSG | No |
| `/conf` | `conf/index.astro` | SSG | No |
| `/conf/register` | `conf/register.astro` | SSR | No |
| `/waitlist` | `waitlist.astro` | SSR | No |
| `/api/waitlist` | `api/waitlist.ts` | API Route POST | No |
| `/api/subscribers` | `api/subscribers.ts` | API Route GET | No |
| `/api/conf-register` | `api/conf-register.ts` | API Route POST | No |

---

## 📋 Estado del proyecto

Proyecto de portafolio en desarrollo como parte de un proceso de aprendizaje de Astro 6 con foco en Islands Architecture, Content Collections, SSR híbrido y deploy en Vercel.