# BACKEND.md

This file documents the setup and configuration of the Strapi backend for the domedog-pro project.

## Admin Account

- **Admin Email**: 123kupola@gmail.com
- **Created**: January 2026
- **Access**: http://localhost:1337/admin

*Note: Password is stored securely and not committed to version control.*

## Initial Setup

### Official Strapi Creation Command

```bash
npx create-strapi-app@latest .
```

Run this command **in the `backend/` directory**.

### Interactive Setup Prompts

When running the command above, you'll be prompted with the following questions. Here are the recommended responses for development:

```
? Please log in or sign up.
→ Skip

? Do you want to use the default database (sqlite) ?
→ Yes

? Start with an example structure & data?
→ No

? Start with Typescript?
→ Yes

? Install dependencies with npm?
→ Yes

? Initialize a git repository?
→ No (we already have git at the root level)

? Participate in anonymous A/B testing (to improve Strapi)?
→ No
```

### Expected Output

After successful installation, you should see:

```
✓ Dependencies installed

Strapi   Your application was created!
          Available commands in your project:

          Start Strapi in watch mode. (Changes in Strapi project files will trigger a server restart)
          npm run develop

          Start Strapi without watch mode.
          npm run start

          Build Strapi admin panel.
          npm run build

          Deploy Strapi project.
          npm run deploy

          Display all available commands.
          npm run strapi
```

## Available Commands

Once Strapi is set up, you have these commands available in the `backend/` directory:

### Development

```bash
npm run develop
```
Starts Strapi in watch mode with hot reloading. Access admin panel at `http://localhost:1337/admin`

### Production

```bash
npm run start
```
Starts Strapi without watch mode (for production-like testing)

### Build

```bash
npm run build
```
Builds the Strapi admin panel

### Deployment

```bash
npm run deploy
```
Deploy Strapi project (configure deployment targets first)

### All Available Commands

```bash
npm run strapi
```
Display all available Strapi CLI commands

## Project Structure

After creation, the `backend/` directory contains:

```
backend/
├── src/
│   ├── api/               # Content types and API routes
│   ├── admin/             # Admin panel customizations
│   ├── plugins/           # Custom plugins
│   ├── config/            # Configuration files
│   └── extensions/        # Strapi extensions
├── public/                # Static files and media uploads
├── .env                   # Environment variables (git ignored)
├── .env.example           # Environment variables template
├── package.json           # Dependencies and scripts
├── tsconfig.json          # TypeScript configuration
└── strapi-admin.js        # Admin customization entry point
```

## Database

By default, this setup uses **SQLite** for development:
- Database file: `backend/.tmp/data.db`
- This file is git ignored and created automatically

To switch databases or configure production database:
- Update `backend/config/database.js`
- Modify environment variables in `backend/.env`

## Environment Variables

Key environment variables for the backend:

```
NODE_ENV=development
HOST=0.0.0.0
PORT=1337
APP_KEYS=                    # Generated automatically, keep secure
API_TOKEN_SALT=              # Generated automatically, keep secure
ADMIN_JWT_SECRET=            # Generated automatically, keep secure
JWT_SECRET=                  # Generated automatically, keep secure
```

## Known Issues

### npm audit warnings

Fresh Strapi installations show warnings about deprecated and vulnerable packages in transitive dependencies. This is a known issue with Strapi v5.34.0 and doesn't affect local development.

See the main AGENTS.md file for security considerations.

## Getting Started After Installation

1. Start the development server:
   ```bash
   npm run develop
   ```

2. Access the admin panel: `http://localhost:1337/admin`

3. Create an admin account (first time only)

4. Start creating content types and managing content

## Resources

- [Official Strapi Documentation](https://strapi.io/documentation/)
- [Strapi API Documentation](https://docs.strapi.io/dev-docs/api/rest)
- [Strapi Community](https://strapi.io/community)
