# Changelog

All notable changes to this Overleaf Community Edition local development setup will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Added

- **Project Documentation** (2026-01-29)
  - Created `CHANGELOG.md` for tracking all project changes
  - Created `CLAUDE.md` with project guidelines, structure, and conventions
  - Added CLAUDE.md maintenance guidelines for keeping documentation current

- **Git Bridge Integration** (2026-01-29)
  - Added `git-bridge` service to `develop/docker-compose.yml`
  - Created `develop/git-bridge-config.json` configuration file
  - Added Git Bridge OAuth application to MongoDB
  - Modified `services/git-bridge/Dockerfile` to work with monorepo context
  - Added `GIT_BRIDGE_ENABLED`, `GIT_BRIDGE_HOST`, `GIT_BRIDGE_PORT` environment variables to web service
  - Added `enableGitBridge` and `gitBridgePublicBaseUrl` settings to `services/web/config/settings.defaults.js`

- **Review Panel / Track Changes** (2026-01-29)
  - Enabled `trackChangesAvailable: true` in `services/web/app/src/Features/Project/ProjectEditorHandler.mjs`
  - Set `trackChanges: true` in default features
  - Added `getThreads` endpoint to `services/web/app/src/Features/Chat/ChatController.mjs`
  - Added `/project/:project_id/threads` route to `services/web/app/src/router.mjs`
  - Added `/project/:project_id/changes/users` stub route (returns empty array for Community Edition)
  - Added `/project/:project_id/track_changes` POST endpoint for toggling track changes per user
  - Imports added: `EditorRealTimeController`, `Project` model in router.mjs
  - Added thread comment endpoints to ChatController and router:
    - `POST /project/:project_id/thread/:thread_id/messages` - send comment
    - `POST /project/:project_id/thread/:thread_id/messages/:message_id/edit` - edit comment
    - `DELETE /project/:project_id/thread/:thread_id/messages/:message_id` - delete comment
    - `POST /project/:project_id/thread/:thread_id/resolve` - resolve thread
    - `POST /project/:project_id/thread/:thread_id/reopen` - reopen thread
    - `DELETE /project/:project_id/thread/:thread_id` - delete thread
  - Added duplicate routes with `/doc/:doc_id/` path segment (used by review panel)

- **URL Linked Files** (2026-01-29)
  - Added `url` to `ENABLED_LINKED_FILE_TYPES` in `develop/docker-compose.yml`

### Changed

- **Redis Upgrade** (2026-01-29)
  - Upgraded Redis from version 5 to version 7 in `develop/docker-compose.yml`
  - Fixes `getdel` command not supported error

### Fixed

- **History Loading** (2026-01-29)
  - Added `useSubdirectories` configuration conversion in `services/history-v1/storage/lib/persistor.js`
  - Added `useSubdirectories` to `services/history-v1/config/custom-environment-variables.json`
  - Migrated existing blob/chunk files from flat to nested directory structure

- **Filestore/History-v1 Blob Storage** (2026-01-28)
  - Configured `USE_SUBDIRECTORIES=true` for history-v1 service
  - Set up shared volume `history-v1-buckets` between filestore and history-v1
  - Configured bucket paths for blobs, chunks, project_blobs, analytics, zips

- **CLSI TeX Live** (2026-01-28)
  - Created custom `services/clsi/Dockerfile.texlive` with full TeX Live installation
  - Enabled PDF compilation with pdflatex, xelatex, lualatex

## Notes

### Server Pro Features (Not Available)

The following integrations are proprietary Server Pro features and not available in Community Edition:
- GitHub OAuth Integration
- Dropbox Integration
- Mendeley Integration
- Zotero Integration
- Papers Integration

### Git Bridge Usage

```bash
# Clone a project
git clone http://localhost:8000/git/PROJECT_ID

# Authenticate with your Overleaf credentials
```

### Project Compiler Setting

If PDF compilation fails, check the project's compiler setting in the editor menu. Change from `latex` to `pdflatex` if needed.
