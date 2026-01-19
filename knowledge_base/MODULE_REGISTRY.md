# Module Registry

> Master index of all major modules in VAANI Engine.
> 
> **Update this file** when creating new modules or deprecating existing ones.

---

## How to Use

1. **Before creating a module**: Check if similar functionality exists
2. **After creating a module**: Add entry below with all required fields
3. **When deprecating**: Change status, add successor reference

---

## Module Index

| Module | Status | Description |
|--------|--------|-------------|
| *(No modules registered yet)* | - | - |

---

## Detailed Registry

<!--
### [module_name]

| Field | Value |
|-------|-------|
| **Path** | `/path/to/module` |
| **Status** | Active / Deprecated / Experimental |
| **Created** | YYYY-MM-DD |
| **Session** | Link to session log entry |
| **Purpose** | One-line description |
| **Dependencies** | List of modules this depends on |
| **Dependents** | List of modules that depend on this |
| **Key Interfaces** | Main classes/functions exposed |
| **Owner** | Person/team responsible |

#### Notes
- Additional context, caveats, or usage notes

---
-->

<!-- Modules will be added below as they are created -->

---

## Module Categories

As the codebase grows, modules should fall into these categories:

| Category | Description |
|----------|-------------|
| `core/` | Foundation: config, logging, common utilities |
| `voice/` | Voice pipeline: STT, TTS, VAD, audio processing |
| `api/` | HTTP/WebSocket endpoints |
| `domain/` | Business logic: appointments, patients, etc. |
| `infra/` | Infrastructure: database, cache, queues |
| `tools/` | AI function calling tools |
| `admin/` | Admin dashboard and monitoring |

---

## Health Legend

| Status | Meaning |
|--------|---------|
| 🟢 Active | Production-ready, maintained |
| 🟡 Experimental | In development, API may change |
| 🟠 Maintenance | Stable but not actively developed |
| 🔴 Deprecated | Scheduled for removal |
