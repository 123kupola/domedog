# SECURITY.md

This document outlines security best practices and implementation strategies for the domedog-pro project. It covers data validation, authentication, API security, and other security concerns we'll encounter as we develop.

## Table of Contents

- [Data Validation with Zod](#data-validation-with-zod)
- [Authentication & Login](#authentication--login) *(coming)*
- [API Security](#api-security) *(coming)*
- [Session Management](#session-management) *(coming)*
- [Security Checklist](#security-checklist)
- [Resources](#resources)

---

## Data Validation with Zod

## Why Zod + astro-strapi-loader?

**astro-strapi-loader** handles:
- Fetching data from Strapi API
- Type generation from Strapi schema

**Zod** handles:
- Runtime validation (catches bad data)
- Security checks (prevents malformed data)
- Type coercion and transformation

Together they provide:
- ✅ **Static Type Safety** (TypeScript compile-time)
- ✅ **Runtime Validation** (Zod at runtime)
- ✅ **Security** (prevent unexpected data shapes)

## Installation

Install Zod in the `astro/` directory:

```bash
npm install zod
```

## Security Best Practices with Zod

### 1. Schema Validation Pattern

Always validate data from **external sources** (APIs, user input, databases):

```typescript
// src/lib/schemas.ts
import { z } from 'zod';

// Define exact shape of post data from Strapi
export const PostSchema = z.object({
  id: z.number(),
  title: z.string().min(1).max(255), // Prevent empty/too long titles
  slug: z.string().regex(/^[a-z0-9-]+$/).max(100), // Only alphanumeric + hyphens
  content: z.string().min(1), // Ensure content exists
  publishedAt: z.string().datetime(), // Ensure valid date format
  author: z.object({
    id: z.number(),
    name: z.string().min(1).max(100),
    email: z.string().email(), // Validate email format
  }),
});

export type Post = z.infer<typeof PostSchema>;
```

### 2. Sanitization for Security

Never trust data from APIs. Sanitize and validate:

```typescript
// src/lib/strapi.ts
import { z } from 'zod';
import { PostSchema } from './schemas';

const API_URL = import.meta.env.STRAPI_URL || 'http://localhost:1337';

export async function getStrapiPosts() {
  try {
    const response = await fetch(`${API_URL}/api/posts?populate=*`);

    if (!response.ok) {
      throw new Error(`API error: ${response.status}`);
    }

    const data = await response.json();

    // Validate each post against schema
    // This will throw if data doesn't match
    const validatedPosts = z.array(PostSchema).parse(data.data);

    return validatedPosts;
  } catch (error) {
    if (error instanceof z.ZodError) {
      console.error('Data validation failed:', error.errors);
      throw new Error('Invalid data from Strapi');
    }
    throw error;
  }
}
```

### 3. Input Sanitization

If accepting user input (search, filters), always validate:

```typescript
// src/lib/user-input.ts
import { z } from 'zod';

// Validation for user search input
export const SearchQuerySchema = z.object({
  query: z.string()
    .min(1)
    .max(100)
    .regex(/^[a-zA-Z0-9\s-]+$/, 'Only letters, numbers, spaces, and hyphens allowed'),
  category: z.string().optional(),
  limit: z.number().int().min(1).max(50), // Prevent DOS via huge results
});

export type SearchQuery = z.infer<typeof SearchQuerySchema>;

// Use in Astro endpoint
export async function validateSearchInput(input: unknown) {
  try {
    return SearchQuerySchema.parse(input);
  } catch (error) {
    if (error instanceof z.ZodError) {
      return { error: 'Invalid search parameters' };
    }
    throw error;
  }
}
```

### 4. Strict Content Type Checking

Prevent XSS by validating content types:

```typescript
// src/lib/content-validator.ts
import { z } from 'zod';

// Strict validation for user-generated content
export const UserCommentSchema = z.object({
  author: z.string().min(1).max(50),
  email: z.string().email(),
  comment: z.string()
    .min(1)
    .max(1000)
    .refine((val) => !containsScriptTags(val), 'HTML/Script tags not allowed'),
});

function containsScriptTags(str: string): boolean {
  const dangerous = /<script|<iframe|<embed|javascript:/i;
  return dangerous.test(str);
}
```

### 5. Environment Variables Validation

Validate your config at startup:

```typescript
// src/lib/env.ts
import { z } from 'zod';

const EnvSchema = z.object({
  STRAPI_URL: z.string().url('Invalid Strapi URL'),
  PUBLIC_STRAPI_URL: z.string().url('Invalid public Strapi URL'),
  NODE_ENV: z.enum(['development', 'production']),
});

export const env = EnvSchema.parse({
  STRAPI_URL: import.meta.env.STRAPI_URL,
  PUBLIC_STRAPI_URL: import.meta.env.PUBLIC_STRAPI_URL,
  NODE_ENV: import.meta.env.MODE,
});
```

## Implementation Roadmap

### Phase 1: Core Schemas (Week 1)
- [ ] Create `src/lib/schemas.ts` with post, author, category schemas
- [ ] Create `src/lib/strapi.ts` with validated API calls
- [ ] Test validation with sample data from Strapi

### Phase 2: User Input Validation (Week 2)
- [ ] Add search/filter validation
- [ ] Create API endpoints with input validation
- [ ] Test with invalid inputs

### Phase 3: Error Handling (Week 3)
- [ ] Create error handling middleware
- [ ] Log validation errors (but don't expose to users)
- [ ] Provide user-friendly error messages

### Phase 4: Production Security (Week 4)
- [ ] Add rate limiting
- [ ] Enable CORS restrictions
- [ ] Review and test all validation rules

## Security Checklist

### Data Validation
- [ ] All Strapi API responses validated with Zod
- [ ] User input validated before use
- [ ] String length limits enforced
- [ ] Email format validated
- [ ] URLs validated with z.string().url()
- [ ] Numbers validated with min/max bounds

### XSS Prevention
- [ ] No HTML content accepted from untrusted sources
- [ ] Script tags explicitly blocked
- [ ] User input sanitized before rendering

### Injection Prevention
- [ ] Database queries use Strapi (prevents SQL injection)
- [ ] URL parameters validated
- [ ] Search terms restricted to safe characters
- [ ] No dynamic code execution from user input

### API Security
- [ ] Strapi API only accessed from server-side (not client)
- [ ] Sensitive data never exposed in client code
- [ ] API rate limiting configured on Strapi
- [ ] CORS properly configured

### Configuration Security
- [ ] Environment variables validated at startup
- [ ] Secrets never committed to git
- [ ] .env file in .gitignore
- [ ] Use .env.example for safe defaults

## Common Patterns

### Safe API Fetch with Validation

```typescript
export async function safeApiCall<T>(
  schema: z.ZodSchema<T>,
  endpoint: string,
): Promise<T> {
  try {
    const response = await fetch(`${API_URL}${endpoint}`);

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }

    const raw = await response.json();
    return schema.parse(raw); // Throws if invalid
  } catch (error) {
    // Log full error for debugging
    console.error(`API call failed for ${endpoint}:`, error);
    // But only return safe error to user
    throw new Error('Failed to load content');
  }
}
```

### Safe Rendering in Templates

```astro
---
// posts.astro
import { getStrapiPosts } from '../lib/strapi';
import { PostSchema } from '../lib/schemas';

let posts: typeof PostSchema[] = [];
let error: string | null = null;

try {
  posts = await getStrapiPosts();
} catch (err) {
  error = 'Failed to load posts';
}
---

{error && <p class="error">{error}</p>}
{posts.map((post) => (
  <article>
    <h1>{post.title}</h1> <!-- Safe: validated by Zod -->
    <p>{post.content}</p>
  </article>
))}
```

## Testing Validation

Test your schemas with invalid data:

```typescript
// src/lib/schemas.test.ts
import { describe, it, expect } from 'vitest';
import { PostSchema } from './schemas';

describe('PostSchema', () => {
  it('rejects missing title', () => {
    expect(() => PostSchema.parse({
      id: 1,
      slug: 'test',
      content: 'test',
      publishedAt: new Date().toISOString(),
      author: { id: 1, name: 'Test', email: 'test@example.com' },
      // missing title
    })).toThrow();
  });

  it('rejects invalid slug format', () => {
    expect(() => PostSchema.parse({
      id: 1,
      title: 'Test',
      slug: 'Test Post!', // Invalid characters
      content: 'test',
      publishedAt: new Date().toISOString(),
      author: { id: 1, name: 'Test', email: 'test@example.com' },
    })).toThrow();
  });

  it('accepts valid post', () => {
    const valid = PostSchema.parse({
      id: 1,
      title: 'Valid Post',
      slug: 'valid-post',
      content: 'Valid content',
      publishedAt: new Date().toISOString(),
      author: { id: 1, name: 'Author', email: 'author@example.com' },
    });
    expect(valid.title).toBe('Valid Post');
  });
});
```

---

## Authentication & Login

*This section will cover:*
- Setting up admin authentication in Strapi
- JWT token management
- Secure password hashing
- Session handling
- Protected routes in Astro

*Status: To be implemented in Phase 3*

---

## API Security

*This section will cover:*
- CORS configuration between Astro and Strapi
- Rate limiting
- API key management
- HTTPS/TLS requirements
- Hiding sensitive API endpoints

*Status: To be implemented in Phase 4*

---

## Session Management

*This section will cover:*
- Token refresh strategies
- Token expiration handling
- Logout mechanism
- Remember-me functionality
- Cross-site request forgery (CSRF) prevention

*Status: To be implemented in Phase 5*

---

## Security Checklist

### ✅ Data Validation (Implemented)
- [x] All Strapi API responses validated with Zod
- [x] User input validation patterns documented
- [x] String length limits enforced
- [x] Email format validation
- [x] URL validation with z.string().url()
- [x] XSS prevention patterns

### ⏳ Authentication (Coming)
- [ ] Strapi JWT configuration
- [ ] Password strength requirements
- [ ] Account lockout after failed login
- [ ] Two-factor authentication option

### ⏳ API Security (Coming)
- [ ] CORS properly configured
- [ ] Rate limiting on Strapi API
- [ ] API versioning strategy
- [ ] Sensitive data never exposed

### ⏳ Infrastructure (Coming)
- [ ] HTTPS enforced in production
- [ ] Environment variables validated
- [ ] Secrets never committed to git
- [ ] Security headers configured

---

## Resources

- [Zod Documentation](https://zod.dev)
- [Zod Security Guide](https://zod.dev?id=security)
- [OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- [Astro Security Guide](https://docs.astro.build/en/guides/security/)
