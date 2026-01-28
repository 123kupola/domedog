# AGENTS.md

This file provides guidance to AI agents and developers when working with code in this repository.

## Project Information

- **Project Name**: domedog-pro
- **Creator**: 123kupola@gmail.com
- **Repository**: https://github.com/123kupola/domedog
- **License**: BSD 3-Clause License
- **Created**: January 2026

## Project Overview

This is a static site generator built with **Astro** as the frontend framework and **Strapi** as the headless CMS backend. The architecture separates content management (Strapi) from content presentation (Astro), allowing for fast static HTML generation with dynamic content management.

## Architecture Overview

### Frontend: Astro
- **Type**: Static Site Generator with optional SSR
- **Purpose**: Server-first rendering with minimal JavaScript delivery (islands architecture)
- **Performance**: Ships lightweight HTML with zero unnecessary JavaScript overhead
- **Content Source**: Fetches content from Strapi API at build time
- **Framework Flexibility**: Can integrate React, Vue, Svelte, or other UI frameworks when needed

### Backend: Strapi
- **Type**: Headless CMS (Node.js/TypeScript based)
- **Purpose**: Manage and deliver content via REST/GraphQL APIs
- **Content Editors**: Provides intuitive admin panel for non-developers
- **Deployment**: Can be self-hosted or deployed to various platforms

### Integration Pattern
1. **Build Time**: Astro fetches content from Strapi API using `getStaticPaths()` to generate static pages
2. **Updates**: Webhooks from Strapi trigger new Astro builds when content changes
3. **API Access**: Content is accessed via Strapi's REST API (GraphQL available as alternative)
4. **Type Safety**: Consider using `@sensinum/astro-strapi-loader` for typed content collections

## Project Structure

```
domedog-pro/
├── astro/                    # Astro frontend project
│   ├── src/
│   │   ├── pages/           # File-based routing (generates static pages)
│   │   ├── layouts/         # Reusable page layouts
│   │   ├── components/      # Astro and UI framework components
│   │   ├── lib/             # Utility functions and API clients
│   │   └── styles/          # Global styles
│   ├── public/              # Static assets
│   ├── astro.config.mjs     # Astro configuration
│   └── package.json
│
└── backend/                  # Strapi backend project
    ├── src/
    │   ├── api/             # Content types and routes
    │   ├── admin/           # Admin panel customizations
    │   └── plugins/         # Custom plugins
    ├── public/              # Media files
    ├── .env.example         # Environment variables template
    └── package.json
```

## Setup Commands

### Initial Setup

```bash
# Create Astro project
npm create astro@latest astro

# Create Strapi project (in backend directory)
mkdir backend && cd backend
npm create strapi-app@latest . --dbclient=sqlite

# Install Astro Strapi loader (for type-safe content collections)
cd astro
npm install @sensinum/astro-strapi-loader
```

### Astro Development

```bash
# Install dependencies
cd astro && npm install

# Start development server (http://localhost:3000)
npm run dev

# Build static site
npm run build

# Preview production build
npm run preview

# Check for type errors
npm run type-check

# Lint code
npm run lint
```

### Backend (Strapi) Development

```bash
# Navigate to backend directory
cd backend

# Start Strapi admin panel (http://localhost:1337)
npm run develop

# Build Strapi for production
npm run build

# Start production server
npm run start
```

## Key Development Tasks

### Adding New Content Types in Strapi
1. Access Strapi admin panel at `http://localhost:1337/admin`
2. Create or modify content types in Settings → Content Types Builder
3. Define fields and relationships
4. Publish changes to make API available

### Creating Dynamic Pages in Astro
1. Create a dynamic route file in `src/pages/` (e.g., `[slug].astro`)
2. Use `getStaticPaths()` to fetch all slugs from Strapi
3. Use `getStaticProps()` to fetch individual page content
4. Render content using the fetched data

### Connecting to Strapi API
```javascript
// Typical API call pattern
const API_URL = import.meta.env.STRAPI_URL || 'http://localhost:1337';

async function getStrapiData(endpoint) {
  const response = await fetch(`${API_URL}/api${endpoint}`);
  return response.json();
}
```

## Build & Deployment

### Environment Variables
Create `.env` files in both `astro/` and `backend/` directories:

**astro/.env**
```
STRAPI_URL=http://localhost:1337
```

**backend/.env**
```
NODE_ENV=development
HOST=0.0.0.0
PORT=1337
DATABASE_URL=...
```

### Webhook Configuration
Set up Strapi webhook to trigger Astro builds on content changes:
1. In Strapi admin: Settings → Webhooks
2. Create webhook pointing to your build service (Netlify, Vercel, etc.)
3. Trigger on: `publish` and `unpublish` events for your content types

### Hosting Options
- **Frontend (Astro)**: Vercel, Netlify, Cloudflare Pages, GitHub Pages
- **Backend (Strapi)**: Heroku, Railway, Koyeb, DigitalOcean App Platform, self-hosted

## Performance Considerations

- Astro's islands architecture ensures minimal JavaScript reaches the browser
- Static HTML pages are pre-generated and serve instantly
- Images should be optimized using Astro's built-in image component
- Content updates require rebuilding the static site (use webhooks for automation)
- Cache Strapi responses appropriately to minimize API calls during build

## Resources

- [Astro Documentation](https://docs.astro.build/)
- [Strapi Documentation](https://strapi.io/)
- [Astro & Strapi Integration Guide](https://docs.astro.build/en/guides/cms/strapi/)
- [Strapi Astro Integration Page](https://strapi.io/integrations/astro)
