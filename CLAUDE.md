# Claude Code Guidelines for Overleaf Development

This document provides guidelines and context for Claude when working on this Overleaf Community Edition local development setup.

> **IMPORTANT:** When making changes to this project, always update:
> 1. `CHANGELOG.md` - Document all changes with dates and file references
> 2. `CLAUDE.md` (this file) - Update when adding features, services, or changing project structure

## Project Overview

This is a local development setup for [Overleaf](https://github.com/overleaf/overleaf), the open-source online LaTeX editor. The setup uses Docker Compose to run all services locally.

## Directory Structure

```
overleaf/
├── develop/                    # Local development configuration
│   ├── docker-compose.yml      # Main Docker Compose configuration
│   ├── dev.env                 # Environment variables
│   ├── git-bridge-config.json  # Git Bridge configuration
│   └── webpack.config.dev-env.js
├── services/                   # Microservices
│   ├── chat/                   # Chat service
│   ├── clsi/                   # Compile LaTeX Service Interface
│   ├── contacts/               # Contacts service
│   ├── docstore/               # Document storage
│   ├── document-updater/       # Real-time document updates
│   ├── filestore/              # File storage
│   ├── git-bridge/             # Git synchronization bridge
│   ├── history-v1/             # Version history service
│   ├── notifications/          # Notification service
│   ├── project-history/        # Project history tracking
│   ├── real-time/              # WebSocket real-time service
│   └── web/                    # Main web application
├── libraries/                  # Shared libraries
├── CHANGELOG.md                # Change log (MUST be updated)
└── CLAUDE.md                   # This file
```

## Key Configuration Files

### Docker Compose
- **Location:** `develop/docker-compose.yml`
- **Services:** chat, clsi, contacts, docstore, document-updater, filestore, git-bridge, history-v1, mongo, notifications, project-history, real-time, redis, web, webpack

### Web Service Settings
- **Location:** `services/web/config/settings.defaults.js`
- **Purpose:** Default configuration for the web service
- **Key settings:** `defaultFeatures`, `enabledLinkedFileTypes`, `enableGitBridge`

### History-v1 Persistor
- **Location:** `services/history-v1/storage/lib/persistor.js`
- **Important:** Uses `useSubdirectories` for nested file storage

## Development Commands

```bash
# Navigate to develop directory
cd develop

# Build a service
docker compose build <service-name>

# Start services
docker compose up -d

# View logs
docker compose logs <service-name>
docker compose logs --tail=50 <service-name>

# Restart a service
docker compose restart <service-name>

# Execute command in container
docker compose exec <service-name> <command>

# MongoDB shell
docker compose exec mongo mongosh sharelatex
```

## Important Conventions

### 1. Dockerfile Context
All Dockerfiles in `services/` use the repository root as context (`context: ..` in docker-compose.yml). COPY commands must use paths relative to the root:
```dockerfile
# Correct
COPY services/git-bridge/start.sh start.sh

# Incorrect (will fail)
COPY start.sh start.sh
```

### 2. Environment Variables
- Define in `develop/docker-compose.yml` under `environment:`
- Use `process.env.VAR_NAME` in settings files
- Boolean conversions: `process.env.VAR === 'true'`

### 3. Feature Flags
- Check `services/web/app/src/infrastructure/Features.mjs` for feature detection
- Features are controlled by settings and user subscription features
- Default features in `settings.defaults.js` under `defaultFeatures`

### 4. API Routes
- Routes defined in `services/web/app/src/router.mjs`
- Controllers in `services/web/app/src/Features/<Feature>/`
- Use `expressify()` wrapper for async handlers

### 5. MongoDB Operations
```javascript
// Add OAuth application
db.oauthApplications.updateOne(
  { id: "app-id" },
  { $set: { /* fields */ }},
  { upsert: true }
)

// Query projects
db.projects.find({ owner_ref: ObjectId("...") })
```

## Known Limitations

### Server Pro Features (Not Available)
These are proprietary and cannot be enabled in Community Edition:
- GitHub OAuth Integration
- Dropbox Integration
- Mendeley/Zotero/Papers Integration
- Some advanced collaboration features

### Local Development Specifics
- Redis must be version 6.2+ (currently v7) for `getdel` command
- History-v1 uses filesystem storage with nested directories (`USE_SUBDIRECTORIES=true`)
- CLSI uses custom Dockerfile with TeX Live (`Dockerfile.texlive`)

## Changelog Maintenance

**CRITICAL:** Always update `CHANGELOG.md` when making changes:

1. Add entries under `## [Unreleased]`
2. Use categories: `### Added`, `### Changed`, `### Fixed`, `### Removed`
3. Include date in format (YYYY-MM-DD)
4. Reference modified files
5. Provide brief but clear descriptions

Example:
```markdown
### Added

- **Feature Name** (2026-01-29)
  - Description of what was added
  - Files modified: `path/to/file.js`
```

## CLAUDE.md Maintenance

**CRITICAL:** This file (`CLAUDE.md`) must also be kept up-to-date:

### When to Update

1. **New Feature Added**
   - Add to "Key Configuration Files" if new config files created
   - Add to "Known Limitations" if feature has restrictions
   - Add to "Troubleshooting" if common issues expected

2. **New Service Added**
   - Update "Directory Structure" tree
   - Add service to Docker Compose services list
   - Document any new conventions or configurations

3. **Structure Changes**
   - Update "Directory Structure" section
   - Update file paths in examples

4. **New Conventions Established**
   - Add to "Important Conventions" section
   - Provide code examples

5. **New Environment Variables**
   - Document in relevant configuration section

### Update Checklist

When making significant changes, verify:
- [ ] Directory structure is accurate
- [ ] All services listed in Docker Compose section
- [ ] Key configuration files documented
- [ ] New conventions explained with examples
- [ ] Troubleshooting section covers new potential issues

## Troubleshooting

### PDF Compilation Fails
- Check CLSI logs: `docker compose logs clsi`
- Verify compiler setting (pdflatex vs latex)
- Ensure TeX Live is installed in CLSI container

### History Not Loading
- Verify `USE_SUBDIRECTORIES=true` in history-v1 environment
- Check if files need migration from flat to nested structure
- Check history-v1 logs: `docker compose logs history-v1`

### Redis Command Errors
- Ensure Redis version is 6.2 or higher
- Check: `docker compose logs redis`

### Service Connection Issues
- Verify all services are running: `docker compose ps`
- Check service dependencies in docker-compose.yml
- Ensure internal hostnames match service names

## Code Style

- JavaScript/TypeScript: Follow existing patterns in codebase
- Use async/await for asynchronous operations
- Express handlers should use `expressify()` wrapper
- Prefer `.mjs` extension for ES modules

## Testing Changes

After making changes:
1. Rebuild affected service: `docker compose build <service>`
2. Restart service: `docker compose up -d <service>`
3. Check logs for errors: `docker compose logs <service>`
4. Test functionality in browser at `http://localhost`
