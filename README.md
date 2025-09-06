# Complete Migration Guide: Remix v2 to React Router v7

## 📋 Table of Contents
1. [Prerequisites](#prerequisites)
2. [Pre-Migration Analysis](#pre-migration-analysis)
3. [Migration Process](#migration-process)
4. [Configuration Updates](#configuration-updates)
5. [Testing & Verification](#testing--verification)
6. [Troubleshooting](#troubleshooting)
7. [Post-Migration Tasks](#post-migration-tasks)
8. [Key Differences](#key-differences)
9. [Resources](#resources)

## 🔧 Prerequisites

### System Requirements
- **Node.js**: >= 20.0.0
- **npm**: Latest version
- **Git**: For version control and rollback capability

### Project Requirements
- Existing Remix v2 project
- All v3 future flags enabled in `vite.config.ts`
- Clean git working directory (recommended)

### Verify Prerequisites
```bash
# Check Node.js version
node --version
# Should output: v20.x.x or higher

# Check npm version
npm --version

# Check git status
git status
# Should show clean working directory
```

## 🔍 Pre-Migration Analysis

### 1. Document Current State

```bash
# Navigate to your project directory
cd /path/to/your/project

# Create a backup branch
git checkout -b backup-before-react-router-migration
git push origin backup-before-react-router-migration

# Switch back to main branch
git checkout main

# Document current dependencies
cat package.json | grep -E "@remix-run|react-router" > current-deps.txt
```

### 2. Verify Current Remix Setup

```bash
# Check current Remix version
grep "@remix-run" package.json

# Verify vite.config.ts has v3 future flags
cat vite.config.ts
```

**Expected `vite.config.ts` structure:**
```typescript
import { vitePlugin as remix } from "@remix-run/dev";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [
    remix({
      future: {
        v3_fetcherPersist: true,
        v3_relativeSplatPath: true,
        v3_throwAbortReason: true,
        v3_singleFetch: true,
        v3_lazyRouteDiscovery: true,
      },
    }),
    tsconfigPaths(),
  ],
});
```

### 3. Analyze Project Structure

```bash
# Document current project structure
tree app/ -I node_modules > project-structure.txt

# Count route files
find app/routes -name "*.tsx" -o -name "*.ts" | wc -l

# List all route files for later reference
ls -la app/routes/ > route-files.txt
```

## 🚀 Migration Process

### Step 1: Run Official React Router 7 Codemod

```bash
# Install and run the official codemod
npx codemod remix/2/react-router/upgrade

# When prompted about uncommitted changes, type 'y' to proceed
# The codemod will process all files automatically
```

**What the codemod does:**
- Updates `package.json` dependencies and scripts
- Updates `vite.config.ts` to use React Router plugin
- Updates all import statements across the codebase
- Updates entry files (`app/entry.client.tsx`, `app/entry.server.tsx`)
- Updates all route files and components

**Expected output:**
```
[1/2] 🔍 Resolving package from registry...
[2/2] 🏁 Running codemod: remix/2/react-router/upgrade
✔ Successfully fetched "remix/2/react-router/upgrade"
Execution progress | ████████████████████████████████████████ | 100% || 235/235 files
```

### Step 2: Clean Dependencies

```bash
# Remove old dependencies and lock file
rm -rf node_modules package-lock.json

# Install fresh dependencies
npm install
```

**If dependency conflicts occur:**
```bash
# Check for remaining @remix-run dependencies
grep -r "@remix-run" package.json

# Remove incompatible packages (example)
npm uninstall @nasa-gcn/remix-seo

# Try installation again
npm install
```

## 📝 Configuration Updates

### Step 3: Update TypeScript Configuration

```bash
# Edit tsconfig.json
nano tsconfig.json
```

**Update `tsconfig.json`:**
```json
{
  "include": [
    "**/*.ts",
    "**/*.tsx",
    "**/.server/**/*.ts",
    "**/.server/**/*.tsx",
    "**/.client/**/*.ts",
    "**/.client/**/*.tsx",
    ".react-router/types/**/*"
  ],
  "compilerOptions": {
    "lib": ["DOM", "DOM.Iterable", "ES2022"],
    "types": ["@react-router/node", "vite/client"],
    "isolatedModules": true,
    "esModuleInterop": true,
    "jsx": "react-jsx",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "resolveJsonModule": true,
    "target": "ES2022",
    "strict": true,
    "allowJs": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "baseUrl": ".",
    "paths": {
      "~/*": ["./app/*"]
    },
    "rootDirs": [".", "./.react-router/types"],
    "noEmit": true
  }
}
```

### Step 4: Create React Router Configuration

```bash
# Create react-router.config.ts in project root
touch react-router.config.ts
```

**Add to `react-router.config.ts`:**
```typescript
import type { Config } from "@react-router/dev/config";

export default {
  ssr: true,
} satisfies Config;
```

### Step 5: Create Route Configuration

```bash
# Create app/routes.ts
touch app/routes.ts
```

**Generate route list:**
```bash
# List all route files to help build configuration
find app/routes -name "*.tsx" -o -name "*.ts" | sort
```

**Add to `app/routes.ts`:**
```typescript
import { type RouteConfig, index, route } from "@react-router/dev/routes";

export default [
  index("routes/_index.tsx"),
  route("tools", "routes/tools.tsx", [
    index("routes/tools._index.tsx"),
    // Add all your tool routes here based on the file list above
    route("affiliate-agreement-generator", "routes/tools.affiliate-agreement-generator.tsx"),
    route("age-calculator", "routes/tools.age-calculator.tsx"),
    route("ascii-converter", "routes/tools.ascii-converter.tsx"),
    // ... continue with all your routes
  ]),
  route("categories", "routes/categories.tsx"),
  route("privacy-policy", "routes/privacy-policy.tsx"),
  route("terms-conditions", "routes/terms-conditions.tsx"),
  route("robots.txt", "routes/robots[.]txt.tsx"),
  route("sitemap.xml", "routes/sitemap[.]xml.tsx"),
  route("*", "routes/$.tsx"), // Catch-all route
] satisfies RouteConfig;
```

### Step 6: Update Vite Configuration

```bash
# Edit vite.config.ts to remove old Remix module declaration
nano vite.config.ts
```

**Clean up `vite.config.ts`:**
```typescript
import { reactRouter } from "@react-router/dev/vite";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig({
  plugins: [
    reactRouter({
      future: {
        v3_fetcherPersist: true,
        v3_relativeSplatPath: true,
        v3_throwAbortReason: true,
        v3_singleFetch: true,
        v3_lazyRouteDiscovery: true,
      },
    }),
    tsconfigPaths(),
  ],
});
```

### Step 7: Fix Manual Import Issues

```bash
# Check if root.tsx needs manual fixes
grep -n "@remix-run" app/root.tsx
```

**If needed, update `app/root.tsx`:**
```typescript
import {
  Links,
  Meta,
  Outlet,
  Scripts,
  ScrollRestoration,
  useLoaderData,
  data,
} from "react-router";
import type { LinksFunction } from "react-router";
```

### Step 8: Verify Package.json Updates

```bash
# Check updated scripts
grep -A 10 '"scripts"' package.json
```

**Expected scripts:**
```json
{
  "scripts": {
    "build": "react-router build",
    "dev": "react-router dev",
    "start": "react-router-serve ./build/server/index.js",
    "typecheck": "react-router typegen && tsc"
  }
}
```

**Check dependencies:**
```bash
# Verify React Router dependencies are installed
grep -E "@react-router|react-router" package.json
```

**Expected dependencies:**
```json
{
  "dependencies": {
    "@react-router/node": "^7.0.0",
    "@react-router/serve": "^7.0.0",
    "react-router": "^7.0.0"
  },
  "devDependencies": {
    "@react-router/dev": "^7.0.0"
  }
}
```

## 🧪 Testing & Verification

### Step 9: Test Development Server

```bash
# Start development server
npm run dev
```

**Expected output:**
```
> react-router dev

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

**Test in browser:**
```bash
# Open browser (or manually navigate)
open http://localhost:5173
```

### Step 10: Test Production Build

```bash
# Stop dev server (Ctrl+C) and build for production
npm run build
```

**Expected output:**
```
> react-router build

vite v6.x.x building for production...
✓ xxxx modules transformed.
✓ built in xx.xxs
```

**Verify build files:**
```bash
# Check build directory
ls -la build/
ls -la build/client/
ls -la build/server/
```

### Step 11: Test Production Server

```bash
# Start production server
npm start
```

**Expected output:**
```
> react-router-serve ./build/server/index.js

[react-router-serve] http://localhost:3000 (http://192.168.x.x:3000)
```

**Test production build:**
```bash
# Open production server (or manually navigate)
open http://localhost:3000
```

### Step 12: Run Type Checking

```bash
# Run TypeScript type checking
npm run typecheck
```

**Expected output:**
```
> react-router typegen && tsc

Route types generated successfully
```

### Step 13: Verify All Routes Work

```bash
# Test key routes (replace with your actual routes)
curl -I http://localhost:3000/
curl -I http://localhost:3000/tools
curl -I http://localhost:3000/categories
curl -I http://localhost:3000/robots.txt
curl -I http://localhost:3000/sitemap.xml

# All should return 200 OK
```

## 🔧 Troubleshooting

### Common Issue 1: "Route config file not found"

**Error:**
```
Error: Route config file not found at "app/routes.ts"
```

**Solution:**
```bash
# Ensure app/routes.ts exists and is properly configured
ls -la app/routes.ts
cat app/routes.ts
```

### Common Issue 2: Dependency Conflicts

**Error:**
```
npm error ERESOLVE could not resolve
```

**Solution:**
```bash
# Clean everything and reinstall
rm -rf node_modules package-lock.json
npm cache clean --force
npm install

# If still failing, check for conflicting packages
npm ls | grep -E "@remix-run|react-router"
```

### Common Issue 3: TypeScript Errors

**Error:**
```
Cannot find module 'react-router'
```

**Solution:**
```bash
# Verify React Router packages are installed
npm list @react-router/dev @react-router/node react-router

# Reinstall if missing
npm install @react-router/dev @react-router/node @react-router/serve react-router

# Update tsconfig.json as shown in Step 3
```

### Common Issue 4: Import Errors

**Error:**
```
Module '"react-router"' has no exported member 'data'
```

**Solution:**
```bash
# Check correct import locations
grep -r "import.*data" app/

# Update imports - data should be imported from "react-router"
# If still failing, check React Router documentation for correct imports
```

### Common Issue 5: Build Failures

**Error:**
```
Build failed with errors
```

**Solution:**
```bash
# Check for syntax errors
npm run typecheck

# Check for missing dependencies
npm install

# Check vite.config.ts is properly configured
cat vite.config.ts
```

## 📋 Post-Migration Tasks

### Step 14: Commit Changes

```bash
# Add all changes
git add .

# Commit with descriptive message
git commit -m "Migrate from Remix v2 to React Router v7

- Updated package.json dependencies from @remix-run to @react-router packages
- Updated vite.config.ts to use reactRouter plugin
- Updated tsconfig.json with React Router types and root directories
- Updated entry.client.tsx and entry.server.tsx to use React Router 7 components
- Updated all import statements throughout the codebase
- Created app/routes.ts for explicit route configuration
- Created react-router.config.ts for framework configuration
- Removed incompatible dependencies
- All tests pass: dev server, build, and production server work correctly"

# Push to remote
git push origin main
```

### Step 15: Update CI/CD Pipeline

```bash
# Update your CI/CD scripts to use new commands
# Example GitHub Actions workflow update:
```

**Update `.github/workflows/deploy.yml` (if exists):**
```yaml
# Replace old Remix commands
- name: Build
  run: npm run build  # This now runs "react-router build"

- name: Start
  run: npm start      # This now runs "react-router-serve"
```

### Step 16: Update Documentation

```bash
# Update README.md
nano README.md
```

**Update development commands in README:**
```markdown
## Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Type checking
npm run typecheck
```
```

### Step 17: Clean Up Temporary Files

```bash
# Remove temporary documentation files
rm -f current-deps.txt project-structure.txt route-files.txt

# Remove backup branch if everything works
git branch -D backup-before-react-router-migration
git push origin --delete backup-before-react-router-migration
```

### Step 18: Consider SEO Solution

```bash
# If you removed @nasa-gcn/remix-seo, consider alternatives
# Research React Router 7 compatible SEO solutions
```

**SEO alternatives to research:**
- Custom meta tag management
- React Helmet Async
- Next SEO (if compatible)
- Custom implementation using React Router's meta exports

## 📊 Key Differences: Remix v2 vs React Router v7

| Aspect | Remix v2 | React Router v7 |
|--------|----------|-----------------|
| **Main Package** | `@remix-run/react` | `react-router` |
| **Node Package** | `@remix-run/node` | `@react-router/node` |
| **Dev Package** | `@remix-run/dev` | `@react-router/dev` |
| **Serve Package** | `@remix-run/serve` | `@react-router/serve` |
| **Dev Command** | `remix vite:dev` | `react-router dev` |
| **Build Command** | `remix vite:build` | `react-router build` |
| **Serve Command** | `remix-serve` | `react-router-serve` |
| **Route Config** | File-based only | Explicit config in `app/routes.ts` |
| **Entry Client** | `RemixBrowser` | `HydratedRouter` |
| **Entry Server** | `RemixServer` | `ServerRouter` |
| **Vite Plugin** | `vitePlugin as remix` | `reactRouter` |
| **Types Location** | `@remix-run/node` | `@react-router/node` |

## 🎯 Migration Checklist

### Pre-Migration
- [ ] ✅ Node.js >= 20.0.0 installed
- [ ] ✅ Git repository is clean
- [ ] ✅ Created backup branch
- [ ] ✅ Documented current dependencies
- [ ] ✅ Verified v3 future flags enabled

### Migration Steps
- [ ] ✅ Ran official codemod (`npx codemod remix/2/react-router/upgrade`)
- [ ] ✅ Cleaned and reinstalled dependencies
- [ ] ✅ Updated `tsconfig.json`
- [ ] ✅ Created `react-router.config.ts`
- [ ] ✅ Created `app/routes.ts`
- [ ] ✅ Updated `vite.config.ts`
- [ ] ✅ Fixed manual import issues
- [ ] ✅ Removed incompatible packages

### Testing
- [ ] ✅ Development server runs (`npm run dev`)
- [ ] ✅ Production build succeeds (`npm run build`)
- [ ] ✅ Production server runs (`npm start`)
- [ ] ✅ Type checking passes (`npm run typecheck`)
- [ ] ✅ All routes accessible
- [ ] ✅ Core functionality works

### Post-Migration
- [ ] ✅ Committed changes to git
- [ ] ✅ Updated CI/CD pipeline
- [ ] ✅ Updated documentation
- [ ] ✅ Cleaned up temporary files
- [ ] ✅ Considered SEO solution replacement

## 🚨 Important Notes & Best Practices

### File Structure Changes
```bash
# New files created during migration:
.react-router/types/          # Generated TypeScript types
app/routes.ts                 # Route configuration
react-router.config.ts        # Framework configuration

# Modified files:
package.json                  # Dependencies and scripts
vite.config.ts               # Vite plugin configuration
tsconfig.json                # TypeScript configuration
app/entry.client.tsx         # Client entry point
app/entry.server.tsx         # Server entry point
app/root.tsx                 # Root component (imports)
app/routes/*.tsx             # All route files (imports)
```

### Performance Considerations
```bash
# Monitor bundle sizes after migration
npm run build
ls -lah build/client/assets/

# Large chunks may need code splitting
# Consider dynamic imports for heavy components
```

### Environment Variables
```bash
# Verify environment variables still work
echo $NODE_ENV
cat .env

# Test in both development and production
npm run dev    # Check dev environment
npm run build && npm start  # Check production environment
```

### Deployment Updates

**Vercel deployment:**
```json
{
  "buildCommand": "npm run build",
  "outputDirectory": "build/client",
  "installCommand": "npm install"
}
```

**Netlify deployment:**
```toml
[build]
  command = "npm run build"
  publish = "build/client"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

**Docker deployment:**
```dockerfile
# Update Dockerfile if you have one
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

## 📚 Resources & References

### Official Documentation
- [React Router v7 Documentation](https://reactrouter.com/)
- [Migration Guide](https://reactrouter.com/upgrading/remix)
- [React Router v7 GitHub](https://github.com/remix-run/react-router)

### Community Resources
- [React Router Discord](https://discord.gg/remix)
- [Stack Overflow - React Router](https://stackoverflow.com/questions/tagged/react-router)
- [Reddit - r/reactjs](https://reddit.com/r/reactjs)

### Useful Commands Reference
```bash
# Development
npm run dev                    # Start development server
npm run build                  # Build for production
npm start                      # Start production server
npm run typecheck             # Run TypeScript checking

# Debugging
npm ls                        # List installed packages
npm outdated                  # Check for outdated packages
npm audit                     # Security audit
npm run build -- --debug     # Debug build process

# Git operations
git status                    # Check working directory
git diff                      # See changes
git log --oneline -10         # Recent commits
git checkout backup-branch    # Switch to backup if needed
```

### Troubleshooting Commands
```bash
# Clean everything
rm -rf node_modules package-lock.json .react-router
npm cache clean --force
npm install

# Check for conflicts
npm ls | grep -E "remix|react-router"
grep -r "@remix-run" . --exclude-dir=node_modules

# Verify installation
node --version
npm --version
npm list react-router @react-router/dev @react-router/node
```

## 🎉 Success Indicators

Your migration is successful when:

1. ✅ **Development server starts**: `npm run dev` works without errors
2. ✅ **Production build completes**: `npm run build` finishes successfully
3. ✅ **Production server runs**: `npm start` serves the application
4. ✅ **All routes work**: Navigation and direct URL access work
5. ✅ **TypeScript compiles**: `npm run typecheck` passes
6. ✅ **No console errors**: Browser console is clean
7. ✅ **Functionality intact**: All features work as before
8. ✅ **Performance maintained**: No significant performance regression

## 🔄 Rollback Plan

If migration fails and you need to rollback:

```bash
# Switch to backup branch
git checkout backup-before-react-router-migration

# Force reset main branch (CAUTION: This will lose migration work)
git checkout main
git reset --hard backup-before-react-router-migration
git push --force-with-lease origin main

# Or create a new branch from backup
git checkout -b main-restored backup-before-react-router-migration
git push origin main-restored
```

---

**Migration completed successfully!** 🎊

Your Remix v2 application is now running on React Router v7 with improved performance, better type safety, and future-proof architecture.
