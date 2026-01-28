# Astro + Strapi Integration Implementation Plan

## Phase 1: Project Initialization

### Step 1.1: Create Astro Project
- Run `npm create astro@latest astro` in the root directory
- Choose template: "Empty" or "Blog" (depends on use case)
- Select package manager: npm
- Initialize Git repository if needed

### Step 1.2: Create Strapi Project
- Run `npm create strapi-app@latest strapi` in the root directory
- Choose database type (SQLite for development, PostgreSQL recommended for production)
- This creates a full Strapi project with admin panel

### Step 1.3: Install Integration Dependencies
- Install `@sensinum/astro-strapi-loader` in Astro for type-safe content collections
- This provides TypeScript types for Strapi content

## Phase 2: Strapi Backend Setup

### Step 2.1: Start Strapi Development Server
- Run `cd strapi && npm install && npm run develop`
- Access admin panel at `http://localhost:1337/admin`
- Create initial admin user

### Step 2.2: Define Content Types
Identify your site's content needs and create content types. Examples:
- **Blog Posts**: title, slug, content, author, published date, featured image
- **Pages**: title, slug, content, meta description
- **Authors**: name, email, bio, avatar
- **Categories**: name, slug, description

Create each content type in Settings → Content Types Builder

### Step 2.3: Configure API Permissions
- Access Roles & Permissions in Settings
- For "Public" role, enable read access to published content only
- This secures your API while allowing frontend to access published content

### Step 2.4: Test API Endpoints
- Create test content in Strapi
- Verify REST API is working: `GET http://localhost:1337/api/posts?populate=*`
- This confirms Strapi is properly configured

## Phase 3: Astro Frontend Setup

### Step 3.1: Configure Environment Variables
Create `astro/.env`:
```
STRAPI_URL=http://localhost:1337
PUBLIC_STRAPI_URL=http://localhost:1337
```

### Step 3.2: Create API Client
Create `astro/src/lib/strapi.ts`:
```typescript
const API_URL = import.meta.env.STRAPI_URL || 'http://localhost:1337';

export async function fetchAPI(path: string) {
  const res = await fetch(`${API_URL}/api${path}`);
  if (!res.ok) {
    throw new Error(`Strapi API error: ${res.statusText}`);
  }
  return res.json();
}

export async function getPosts() {
  return fetchAPI('/posts?populate=*');
}

export async function getPostBySlug(slug: string) {
  const res = await fetchAPI(`/posts?filters[slug][$eq]=${slug}&populate=*`);
  return res.data[0];
}
```

### Step 3.3: Create Dynamic Routes
Create `astro/src/pages/posts/[slug].astro`:
```typescript
import { getPostBySlug, getPosts } from '@/lib/strapi';

export async function getStaticPaths() {
  const posts = await getPosts();
  return posts.data.map((post: any) => ({
    params: { slug: post.attributes.slug },
    props: { post },
  }));
}

interface Props {
  post: any;
}

const { post } = Astro.props;
const { title, content } = post.attributes;
```

### Step 3.4: Create Listing Pages
Create `astro/src/pages/posts.astro` to display all posts:
```typescript
import { getPosts } from '@/lib/strapi';

const result = await getPosts();
const posts = result.data;
```

## Phase 4: Content Collections (Optional but Recommended)

### Step 4.1: Set Up Content Collections
Install and configure Astro content collections for type safety:
- Create `astro/src/content/config.ts`
- Define collection schema for Strapi content types
- This provides TypeScript intellisense and validation

### Step 4.2: Implement Astro Strapi Loader
- Configure `@sensinum/astro-strapi-loader` to automatically load Strapi collections
- This maps Strapi content types to Astro content collections
- Provides full type safety without manual schema definition

## Phase 5: Styling & Components

### Step 5.1: Choose CSS Framework
Options:
- Tailwind CSS (recommended for quick development)
- CSS Modules
- Styled Components
- Plain CSS

Example with Tailwind:
```bash
cd astro && npm install -D tailwindcss
npx tailwindcss init
```

### Step 5.2: Create Reusable Components
- `src/components/Header.astro`
- `src/components/Footer.astro`
- `src/components/PostCard.astro`
- `src/components/Navigation.astro`

### Step 5.3: Create Layouts
- `src/layouts/BaseLayout.astro` (main layout)
- `src/layouts/PostLayout.astro` (for blog posts)
- `src/layouts/PageLayout.astro` (for static pages)

## Phase 6: Image Handling

### Step 6.1: Configure Image Optimization
- Use Astro's built-in `<Image>` component for automatic optimization
- Configure image service in `astro.config.mjs`

### Step 6.2: Store Images in Strapi
- Upload images via Strapi admin panel
- Reference images in content types
- Fetch image URLs from API responses
- Display using Astro Image component

## Phase 7: Build & Deployment Preparation

### Step 7.1: Configure Build Settings
In `astro/astro.config.mjs`:
```javascript
export default defineConfig({
  output: 'static', // For static site generation
  // or 'hybrid' for SSR on demand
});
```

### Step 7.2: Set Up Environment for Production
Create `astro/.env.production`:
```
STRAPI_URL=https://your-strapi-domain.com
```

### Step 7.3: Test Production Build
```bash
npm run build
npm run preview
```

## Phase 8: Webhook Configuration

### Step 8.1: Add Build Webhook to Strapi
In Strapi admin:
1. Settings → Webhooks
2. Create webhook with:
   - URL: Your build service webhook (Netlify/Vercel)
   - Events: `entry.publish`, `entry.unpublish`
   - Content types: Select your content types

### Step 8.2: Deploy to Hosting Platform
- Push repository to GitHub
- Connect to Netlify or Vercel
- Configure environment variables
- Enable automatic deployments on push
- Configure webhook from Strapi to trigger builds

## Testing & Validation

### Step 9.1: Development Testing
- Test dynamic routes generate correctly
- Verify content displays properly
- Check image loading and optimization
- Validate responsive design

### Step 9.2: Production Testing
- Test static build completes without errors
- Verify all pages are generated
- Check performance metrics (Core Web Vitals)
- Test webhook triggering builds on content changes

## Optional Enhancements

### Search Functionality
- Implement client-side search using Lunr or similar
- Or use Strapi search plugins

### Caching Strategy
- Configure CDN caching headers
- Use Astro middleware for custom caching rules

### Analytics & Monitoring
- Add analytics service (Plausible, Google Analytics)
- Monitor build times and deployment status

### API Rate Limiting
- Implement rate limiting for Strapi API
- Use API tokens for authentication if needed

## Common Issues & Solutions

1. **API Timeouts**: Increase fetch timeout, implement pagination for large datasets
2. **Image Loading**: Ensure Strapi media path is accessible, use proper CORS headers
3. **Build Performance**: Optimize Strapi queries, implement pagination and filtering
4. **Content Updates**: Set up webhooks correctly, test webhook delivery in Strapi

## Success Criteria

✓ Astro development server runs without errors
✓ Strapi admin panel is accessible and populated with content
✓ Dynamic pages are generated from Strapi content
✓ Static build completes successfully
✓ Content updates trigger automatic rebuilds
✓ Site loads with good Core Web Vitals scores
