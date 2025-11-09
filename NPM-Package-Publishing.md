# NPM Package Publishing Cheatsheet

> A practical guide for JavaScript/TypeScript developers publishing packages to npmjs.com

---

## Table of Contents

1. [Semantic Versioning (SemVer)](#1-semantic-versioning-semver)
2. [Package.json Essentials](#2-packagejson-essentials)
3. [Publishing Workflow](#3-publishing-workflow)
4. [Pre-release Versions](#4-pre-release-versions)
5. [Production Preparation](#5-production-preparation)
6. [Git & npm Best Practices](#6-git--npm-best-practices)
7. [npm Commands Reference](#7-npm-commands-reference)
8. [TypeScript-Specific Setup](#8-typescript-specific-setup)
9. [Security & Maintenance](#9-security--maintenance)
10. [Troubleshooting](#10-troubleshooting)

---

## 1) Semantic Versioning (SemVer)

**Format:** `MAJOR.MINOR.PATCH` (e.g., `1.4.2`)

### Version Components

- **MAJOR** (1.x.x): Breaking changes (incompatible API changes)
- **MINOR** (x.1.x): New features (backwards-compatible)
- **PATCH** (x.x.1): Bug fixes (backwards-compatible)

### Examples

```txt
1.0.0  → 1.0.1   # Bug fix
1.0.1  → 1.1.0   # New feature
1.1.0  → 2.0.0   # Breaking change
```

### Pre-release Tags

```txt
0.1.0-alpha.1   # Early testing, unstable
0.1.0-beta.1    # Feature-complete, testing phase
0.1.0-rc.1      # Release candidate, final testing
1.0.0           # Stable release
```

### Version 0.x.x (Pre-1.0)

- **0.x.x** signals "not stable yet"
- Breaking changes allowed in MINOR version
- Use until API is stable

### Incrementing Versions

```bash
npm version patch   # 1.0.0 → 1.0.1
npm version minor   # 1.0.1 → 1.1.0
npm version major   # 1.1.0 → 2.0.0

# Pre-release
npm version prerelease --preid=alpha   # 1.0.0 → 1.0.1-alpha.0
npm version prerelease                 # 1.0.1-alpha.0 → 1.0.1-alpha.1
```

---

## 2) Package.json Essentials

### Required Fields

```json
{
  "name": "my-package",
  "version": "1.0.0",
  "description": "Brief description of what the package does",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "author": "Your Name <email@example.com>",
  "license": "MIT"
}
```

### Recommended Fields

```json
{
  "name": "my-package",
  "version": "1.0.0",
  "description": "TypeScript utilities for Web3",
  "keywords": ["web3", "ethereum", "typescript"],
  "author": "Your Name",
  "license": "MIT",
  "repository": {
    "type": "git",
    "url": "git+https://github.com/username/repo.git"
  },
  "bugs": {
    "url": "https://github.com/username/repo/issues"
  },
  "homepage": "https://github.com/username/repo#readme",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": [
    "dist/",
    "README.md",
    "LICENSE"
  ],
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "prepublishOnly": "npm run build && npm test"
  },
  "engines": {
    "node": ">=18.0.0"
  },
  "publishConfig": {
    "access": "public"
  }
}
```

### Files Field (What Gets Published)

```json
{
  "files": [
    "dist/",           // Compiled JavaScript
    "README.md",       // Documentation
    "LICENSE",         // License file
    "package.json"     // Automatically included
  ]
}
```

**What NOT to include:**
- `src/` (source TypeScript files)
- `node_modules/`
- `.env` files
- Test files (`*.test.ts`)
- Build configs (`tsconfig.json`, `jest.config.js`)

### Entry Points

```json
{
  "main": "dist/index.js",           // CommonJS entry
  "types": "dist/index.d.ts",        // TypeScript declarations
  "exports": {
    ".": {
      "import": "./dist/index.mjs",  // ESM entry
      "require": "./dist/index.js",  // CJS entry
      "types": "./dist/index.d.ts"   // Types entry
    }
  }
}
```

---

## 3) Publishing Workflow

### First-Time Setup

```bash
# 1. Create npm account
# Visit https://www.npmjs.com/signup

# 2. Login in terminal
npm login

# 3. Verify login
npm whoami
```

### Publishing Steps

```bash
# 1. Update version
npm version patch  # or minor/major

# 2. Build package
npm run build

# 3. Run tests
npm test

# 4. Preview what will be published
npm pack --dry-run

# 5. Publish
npm publish

# For scoped packages (@username/package)
npm publish --access public
```

### Automated Pre-publish

Add to `package.json`:

```json
{
  "scripts": {
    "prepublishOnly": "npm run build && npm test"
  }
}
```

This runs automatically before `npm publish`.

---

## 4) Pre-release Versions

### Publishing Alpha/Beta Versions

```bash
# Set pre-release version
npm version prerelease --preid=alpha
# Result: 1.0.0 → 1.0.1-alpha.0

# Publish with tag
npm publish --tag alpha

# Users install with:
npm install my-package@alpha
```

### Version Progression

```
0.1.0-alpha.1   # First alpha
0.1.0-alpha.2   # Second alpha
0.1.0-beta.1    # First beta
0.1.0-rc.1      # Release candidate
1.0.0           # Stable release
```

### Promoting to Stable

```bash
# Remove pre-release tag
npm version 1.0.0

# Publish as latest
npm publish

# Add latest tag to existing version
npm dist-tag add my-package@1.0.0 latest
```

### Dist Tags

```bash
# List all tags
npm dist-tag ls my-package

# Add custom tag
npm dist-tag add my-package@1.2.3 stable

# Remove tag
npm dist-tag rm my-package beta
```

---

## 5) Production Preparation

### Pre-publish Checklist

- [ ] All tests pass (`npm test`)
- [ ] README.md is complete with examples
- [ ] LICENSE file exists
- [ ] Version is updated (`npm version`)
- [ ] Build artifacts are current (`npm run build`)
- [ ] `.gitignore` excludes build files
- [ ] `files` field in package.json is correct
- [ ] `prepublishOnly` script runs build + test
- [ ] No secrets in code or `.env` files
- [ ] Dependencies are production-ready (no `latest` tags)
- [ ] TypeScript declarations (`.d.ts`) are generated

### Security Audit

```bash
# Check for vulnerabilities
npm audit

# Fix automatically (if possible)
npm audit fix

# Force fix (may break dependencies)
npm audit fix --force
```

### .npmignore vs files Field

**Option 1: `.npmignore` (blacklist)**

```
# .npmignore
src/
tests/
*.test.ts
tsconfig.json
jest.config.js
.github/
.env
```

**Option 2: `files` field (whitelist, recommended)**

```json
{
  "files": ["dist/", "README.md", "LICENSE"]
}
```

### Verify Package Contents

```bash
# See what will be published
npm pack --dry-run

# Create tarball (test install)
npm pack
tar -tzf my-package-1.0.0.tgz

# Test install from tarball
npm install ./my-package-1.0.0.tgz
```

---

## 6) Git & npm Best Practices

### What to Commit to Git

✅ **Commit:**
- `src/` (source code)
- `package.json`
- `package-lock.json`
- `tsconfig.json`
- `README.md`
- `LICENSE`
- `.gitignore`

❌ **Don't commit:**
- `dist/` (build artifacts)
- `node_modules/` (dependencies)
- `.env` (secrets)
- `coverage/` (test coverage)

### .gitignore Template

```gitignore
# Dependencies
node_modules/

# Build outputs
dist/
build/
*.tsbuildinfo

# Test coverage
coverage/
.nyc_output/

# Environment
.env
.env.local
.env.*.local

# Logs
logs/
*.log
npm-debug.log*

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
*.swp
```

### Git Workflow for Releases

```bash
# 1. Update version (creates git tag)
npm version patch -m "Release v%s"

# 2. Push code + tags
git push && git push --tags

# 3. Publish to npm
npm publish

# 4. Create GitHub release (optional)
gh release create v1.0.1 --generate-notes
```

---

## 7) npm Commands Reference

### Installation

```bash
# Install dependencies
npm install

# Install and save to package.json
npm install package-name
npm install -D package-name  # devDependencies

# Install specific version
npm install package@1.2.3
npm install package@latest

# Install from Git
npm install user/repo
npm install git+https://github.com/user/repo.git
```

### Package Management

```bash
# List installed packages
npm list
npm list --depth=0  # Top-level only

# Check for outdated packages
npm outdated

# Update packages
npm update
npm update package-name

# Uninstall package
npm uninstall package-name
```

### Publishing & Versioning

```bash
# Update version
npm version patch|minor|major
npm version prerelease --preid=alpha

# Publish
npm publish
npm publish --tag beta
npm publish --access public  # For scoped packages

# Unpublish (within 72 hours)
npm unpublish package-name@version

# Deprecate version
npm deprecate package-name@version "Deprecated message"
```

### Package Info

```bash
# View package info
npm view package-name
npm view package-name version
npm view package-name versions

# Show package metadata
npm show package-name

# Search packages
npm search keyword
```

### Scripts

```bash
# Run script
npm run script-name

# List available scripts
npm run

# Run test
npm test  # or npm t

# Run build
npm run build
```

---

## 8) TypeScript-Specific Setup

### tsconfig.json for Libraries

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "declaration": true,
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "types": ["node"]
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.test.ts"]
}
```

### Build Script

```json
{
  "scripts": {
    "build": "tsc",
    "build:watch": "tsc --watch",
    "clean": "rm -rf dist",
    "prebuild": "npm run clean"
  }
}
```

### Type Declarations

```typescript
// src/index.ts
export { wallet } from "./wallet";
export { providers } from "./providers";

// dist/index.d.ts (auto-generated)
export { wallet } from "./wallet";
export { providers } from "./providers";
```

### ESM + CJS Dual Package

```json
{
  "type": "commonjs",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js",
      "types": "./dist/index.d.ts"
    }
  }
}
```

---

## 9) Security & Maintenance

### Security Best Practices

- **Never commit secrets:** Use `.env` files (add to `.gitignore`)
- **Audit dependencies:** Run `npm audit` regularly
- **Use lockfiles:** Commit `package-lock.json`
- **Minimal dependencies:** Fewer deps = smaller attack surface
- **2FA on npm:** Enable two-factor auth on your npm account
- **Scoped packages:** Use `@username/package` for better control

### Dependency Management

```bash
# Check for security issues
npm audit

# Update vulnerable packages
npm audit fix

# Check for outdated packages
npm outdated

# Update to latest (respecting semver)
npm update

# Update to absolute latest (breaking)
npm install package@latest
```

### npm Tokens

```bash
# Create token (for CI/CD)
npm token create --read-only

# List tokens
npm token list

# Revoke token
npm token revoke <token-id>
```

### Package Maintenance

```bash
# Transfer ownership
npm owner add username package-name
npm owner rm username package-name

# List collaborators
npm owner ls package-name

# Deprecate old version
npm deprecate package@version "Use v2.0.0 instead"

# Unpublish (only if <72hrs old)
npm unpublish package@version
```

---

## 10) Troubleshooting

### Common Issues

**"You do not have permission to publish"**

```bash
# Login again
npm logout
npm login

# For scoped packages, use --access public
npm publish --access public
```

**"Version already exists"**

```bash
# Increment version
npm version patch
npm publish
```

**"Cannot find module after publish"**

```bash
# Check files field in package.json
npm pack --dry-run

# Ensure dist/ is included
{
  "files": ["dist/"]
}
```

**"Missing TypeScript declarations"**

```json
{
  "compilerOptions": {
    "declaration": true  // Add this to tsconfig.json
  }
}
```

**"ENOENT: no such file or directory"**

```bash
# Build before publishing
npm run build
npm publish
```

### Debugging

```bash
# Verbose logging
npm publish --loglevel verbose

# Check registry URL
npm config get registry

# Set registry (if using private)
npm config set registry https://registry.npmjs.org/

# Clear cache
npm cache clean --force
```

---

## Quick Reference Card

### Version Bump

```bash
npm version patch  # 1.0.0 → 1.0.1
npm version minor  # 1.0.1 → 1.1.0
npm version major  # 1.1.0 → 2.0.0
```

### Publish

```bash
npm publish              # Latest
npm publish --tag alpha  # Pre-release
```

### Pre-publish Checklist

```bash
npm run build  # Build
npm test       # Test
npm pack --dry-run  # Verify contents
```

### Essential Scripts

```json
{
  "scripts": {
    "build": "tsc",
    "test": "jest",
    "prepublishOnly": "npm run build && npm test"
  }
}
```

---

## Further Reading

- [npm Documentation](https://docs.npmjs.com/)
- [Semantic Versioning](https://semver.org/)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)

---

Built with ❤️ for the JavaScript/TypeScript community
