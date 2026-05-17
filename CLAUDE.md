# Project Sentinel — Autonomous Incident Resolution Engine

## 1. Project Overview

Project Sentinel is a demonstration system that simulates autonomous AI-driven DevOps operations. The platform monitors microservice health, detects failures, and executes automated incident resolution workflows.

### System Purpose

The dashboard provides real-time visibility into service health across a distributed system. When services fail, the autonomous agent pipeline kicks in to diagnose, repair, and verify recovery—mimicking enterprise SRE practices without the operational complexity.

### Autonomous Incident Resolution Simulation

The system simulates three AI agents working in concert:

- **Main Agent**: Coordinates the resolution workflow, polls for active incidents, assigns tasks to subagents
- **Debugger Agent**: Analyzes service logs, classifies bug types, applies targeted fixes
- **QA Agent**: Verifies service recovery through health checks, updates incident status

This is a controlled demonstration environment. No real AGI or external AI APIs are involved—agents operate using rule-based logic and deterministic workflows.

### Monitoring Workflow

1. Monitoring service polls all endpoints every 10 seconds
2. Health check failures create incident records in SQLite
3. AI resolution engine detects active incidents
4. Agents execute repair workflow in sequence
5. QA verification confirms service recovery
6. Dashboard reflects updated status in real-time

---

## 2. Repository Structure

```
project-meesho2/
├── app/                    # Next.js dashboard (port 3000)
│   ├── src/
│   │   ├── app/           # App Router pages
│   │   ├── components/    # UI components
│   │   ├── hooks/         # React hooks
│   │   └── lib/           # Utilities
│   ├── package.json
│   ├── tailwind.config.ts
│   └── tsconfig.json

├── services/              # Microservice workspace
│   ├── auth-service/      # Port 3001
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── routes/    # health, status, auth
│   │   │   ├── middleware/
│   │   │   └── utils/
│   │   └── package.json
│   ├── payment-service/   # Port 3002
│   │   └── ...
│   └── notification-service/ # Port 3003
│       └── ...

├── scripts/               # DevOps automation
│   ├── chaos-monkey.ts       # Bug injection
│   ├── monitoring-service.ts # Health polling
│   ├── ai-resolution-engine.ts # Autonomous workflow
│   └── package.json

├── database/             # SQLite storage
│   ├── sentinel.db       # Generated database

├── docs/                 # Documentation
│   └── CLAUDE.md         # This file

├── logs/                 # Generated logs
│   ├── agents.log        # AI agent activities
│   ├── chaos.log          # Chaos monkey actions
│   ├── monitor.log        # Health polling
│   ├── auth.log           # Auth service logs
│   ├── payment.log        # Payment service logs
│   └── notification.log   # Notification service logs

├── .backups/             # Chaos monkey backups
├── package.json          # Root workspace
├── pnpm-workspace.yaml
└── tsconfig.json
```

---

## 3. Coding Standards

### General Principles

- **SIMPLE FIRST**: Prefer simple solutions over clever ones
- **READABLE**: Code should be self-documenting
- **MODULAR**: Extract reusable logic into utility functions
- **NO DUPLICATION**: Extract common patterns, don't repeat

### TypeScript Strict Mode

All services must compile with strict mode enabled. No exceptions.

### Async/Await Usage

- Always use async/await over raw promises
- Never use .then() chains except for parallel execution with Promise.all
- Always handle errors with try/catch at the top level
- Never leave promises unhandled

### Modular Architecture

- One file per route handler
- Group related utilities in dedicated folders
- Keep express route files focused on HTTP handling
- Business logic goes in services/ subdirectory

### File Naming Conventions

- **Files**: kebab-case (e.g., `auth-routes.ts`, `health-check.ts`)
- **Components**: PascalCase (e.g., `ServiceCard.tsx`)
- **Types/Interfaces**: PascalCase with `.types.ts` suffix when co-located

### Folder Naming Conventions

- Use lowercase kebab-case for all directories
- Group by feature, not by file type
- Keep folder depth shallow (max 3 levels)

### Logging Standards

- All services must log to their dedicated log file
- Use structured JSON logging with Winston
- Include timestamps in ISO 8601 format
- Log levels: info, warn, error—never use console.log in production paths

---

## 4. TypeScript Rules

### No `any` Types

Never use `any`. Use `unknown` if type is truly unknown, then narrow with type guards.

```typescript
// BAD
function parse(data: any): any { ... }

// GOOD
function parse(data: string): Record<string, unknown> { ... }
```

### Explicit Return Types

Always declare return types for exported functions.

```typescript
// BAD
export function getStatus() { ... }

// GOOD
export function getStatus(): ServiceStatus { ... }
```

### Strict Null Checks

Enable strict null checks in tsconfig. Never use `!` assertion operator.

```typescript
// BAD
const value = data!.name;

// GOOD
const value = data?.name ?? 'unknown';
```

### Typed Interfaces

Define interfaces for all data structures.

```typescript
interface Incident {
  id: number;
  service_name: string;
  severity: 'critical' | 'warning' | 'info';
  status: 'active' | 'resolved';
  created_at: string;
  resolved_at?: string;
}
```

### Typed API Responses

Wrap all HTTP responses in typed objects.

```typescript
interface ApiResponse<T> {
  data: T;
  success: boolean;
  error?: string;
}
```

---

## 5. Naming Conventions

### Files

- Route handlers: `{feature}-routes.ts`
- Utilities: `{purpose}.ts`
- Types: `{name}.types.ts` or `index.ts` in type folders
- Tests: `{feature}.test.ts` (when added)

### Functions

- Use verb-noun pattern: `getServiceStatus()`, `createIncident()`
- Use camelCase
- Keep names descriptive but concise

### Variables

- Use camelCase
- Prefer meaningful names over single letters (except loop counters)
- Boolean variables should be prefixed with `is`, `has`, `should`

### Constants

- Use UPPER_SNAKE_CASE for true constants
- Group related constants in objects or enums

### Components

- Use PascalCase
- Include component purpose in name: `ServiceStatusCard`
- Event handlers: `handle{Action}` (e.g., `handleRefresh`)

### Database Tables

- Use snake_case: `service_health`, `incidents`
- Use singular names for entities
- Primary key: `id` (INTEGER AUTOINCREMENT)

---

## 6. Monitoring Rules

### Health Polling Intervals

| Service | Interval | Timeout |
|---------|----------|---------|
| All services | 10 seconds | 5 seconds |
| Dashboard sync | 10 seconds | 2 seconds |

### Service Status Lifecycle

```
HEALTHY → (timeout) → WARNING → (timeout) → CRITICAL
HEALTHY ← (recovery) ← WARNING ← (recovery) ← CRITICAL
```

Status values:
- **HEALTHY**: Response < 2000ms, status code 200
- **WARNING**: Response > 2000ms, status code 200
- **CRITICAL**: Timeout, connection refused, or status code != 200

### Incident Severity Levels

| Level | Trigger | Response Time |
|--------|---------|---------------|
| critical | Service unreachable | Immediate |
| warning | Response time > 2s | Within 30s |
| info | Minor anomalies | Batch |

### Logging Requirements

- Monitor logs to `/logs/monitor.log`
- Include service name, response time, status code
- Log all status transitions
- Preserve log history (append mode, no rotation needed for demo)

### SQLite Update Rules

- Update service health on every poll
- Create incident on first CRITICAL detection
- Mark incident resolved only after QA verification
- Never delete historical incidents

---

## 7. Subagent Responsibilities

### Human's Role: Commands Claude to Act

When a service is CRITICAL, the human commands:
```
Sentinel Agent, fix auth-service
```

Claude then spawns subagents to handle the work.

### Main Agent (Claude Code)

**Responsibilities:**
- Coordinate incident resolution workflow
- Spawn subagents to analyze and fix
- Use Claude's /subagent feature for parallel work
- Log all workflow transitions

**Commands to use:**
```bash
/subagent analyze auth-service     # Spawn debugger
/plan                               # Plan the fix
Shift+Tab x2                        # Open Plan Mode
```

### Subagent Alpha: The Debugger

**Responsibilities:**
- Read service logs: `logs/auth.log`
- Analyze error patterns
- Identify root cause
- Write intelligent fix (NOT just backup restore)

**Example workflow:**
```
1. Read: services/auth-service/src/routes/health.ts
2. Find: res.status(500) instead of 200
3. Fix: Change to res.status(200)
4. Test: curl http://localhost:3001/health
5. Log: Document fix in incident-history.log
```

### Subagent Beta: The QA

**Responsibilities:**
- Verify fix works
- Run health check
- Confirm recovery
- Update incident status

**Verification:**
```bash
curl http://localhost:3001/health
# Must return: {"status":"healthy","service":"auth-service"}
```

### The Assignment Point

Claude MUST analyze logs and write fixes like a real AI would:
- **NOT** just restore from backup
- **YES** to intelligent pattern matching
- **YES** to explaining the bug
- **YES** to writing the fix

---

## 8. Incident Resolution Workflow

The autonomous workflow follows this sequence:

### Step 1: Detect Failure

Monitoring service identifies CRITICAL status, creates incident in SQLite with status='active'.

### Step 2: Classify Issue

Main Agent polls active incidents, passes to Debugger Agent for analysis.

### Step 3: Inspect Logs

Debugger Agent reads service-specific log file from `/logs/{service}.log`.

### Step 4: Apply Repair

Debugger Agent identifies bug type and applies targeted fix:
- Restores original files from `.backups/` directory
- Resolves syntax errors, type mismatches, logic errors

### Step 5: Restart Service

Service process is restarted to apply changes.

### Step 6: Verify Service Recovery

QA Agent sends HTTP request to service `/health` endpoint.

### Step 7: Confirm Health

If health check returns 200, service is recovered. Else, retry repair.

### Step 8: Update Incident Database

QA Agent executes SQL update to set status='resolved', resolved_at=CURRENT_TIMESTAMP.

### Step 9: Notify Dashboard

System status JSON is updated with new service health data.

---

## 9. Resolution Protocol

**THIS SECTION IS MANDATORY FOR ALL AUTONOMOUS OPERATIONS.**

### Before Applying Any Fix

1. **Read service logs** — Check `/logs/{service}.log` for error patterns
2. **Read incident history** — Check for similar past incidents
3. **Verify backup exists** — Check `.backups/{service}/` directory
4. **Assess fix strategy** — Choose between backup restore or manual repair

### If Similar Fix Failed Previously

1. **Enter Thinking Mode** — Pause and evaluate alternative approaches
2. **Avoid repeating failed strategy** — If backup restore failed, try manual repair
3. **Attempt alternative solution** — Use chaos-monkey backup directly
4. **Escalate severity** — If recovery fails twice, log warning and mark for manual review

### Resolution Decision Tree

```
Incident detected
       ↓
Backup exists? → YES → Restore from backup → Restart → QA Verify
       ↓ NO
Manual repair attempt
       ↓
Fix successful? → YES → QA Verify → Mark Resolved
       ↓ NO
Retry once more
       ↓
Still failing? → YES → Log escalation → Manual intervention required
```

---

## 10. Debugging Workflow

### Reading Logs

All service logs are stored in `/logs/{service}.log`. The monitoring service appends to `/logs/monitor.log`. Agent activity is logged to `/logs/agents.log`.

To read logs:
```typescript
import { promises as fs } from 'fs';
const logs = await fs.readFile('./logs/auth.log', 'utf-8');
```

### Identifying Root Cause

Common error patterns in logs:
- `ECONNREFUSED` → Service not running or wrong port
- `timeout` → Service unresponsive, possible deadlock
- `SYNTAX` → Code parsing error, check TypeScript compilation
- `Cannot find module` → Missing import or typo in path

### Validating Fixes

After applying repair:
1. Restart the service process
2. Run service health endpoint
3. Check response status and time
4. Verify logs show healthy status

### Rollback Verification

If fix fails:
1. Read backup from `.backups/{service}/`
2. Restore original file content
3. Verify service returns to healthy status
4. Log rollback action

---

## 11. Testing Requirements

### Post-Fix Health Checks

Every repair must be followed by:
1. HTTP GET to `/health` endpoint
2. Verify status code === 200
3. Verify response time < 5000ms
4. Log verification result

### Endpoint Verification

| Service | Endpoint | Expected |
|---------|----------|----------|
| auth-service | GET /health | `{ status: "healthy", service: "auth-service" }` |
| payment-service | GET /health | `{ status: "healthy", service: "payment-service" }` |
| notification-service | GET /health | `{ status: "healthy", service: "notification-service" }` |

### Service Recovery Confirmation

QA Agent must confirm:
1. Service responds to health check
2. Incident record updated in SQLite
3. Resolution logged to agents.log

### SQLite Validation

After resolution:
```sql
SELECT status, resolved_at FROM incidents WHERE id = ?
-- Expected: status='resolved', resolved_at = CURRENT_TIMESTAMP
```

---

## 12. Rollback Policy

### Restore from Backup

The chaos-monkey script maintains backup of original file content before mutation. Use the restore function:

```typescript
import { restore } from './chaos-monkey.js';
await restore('auth-service');
```

### Revert Failed Repairs

If QA verification fails after repair attempt:
1. Debugger Agent logs failure
2. Main Agent triggers rollback
3. Original file content restored from `.backups/`
4. Service restarted

### Preserve Incident History

- Never delete incident records
- Always set resolved_at timestamp on resolution
- Log all rollback actions to agents.log
- Maintain complete audit trail

### Rollback Logging

Every rollback must log:
- Timestamp
- Service name
- Original bug type
- Reason for rollback
- Next action (retry/escalate)

---

## 13. Logging Standards

### Timestamp Format

All logs use ISO 8601 format:
```
YYYY-MM-DDTHH:mm:ss.sssZ
```

Example: `2026-05-14T14:32:45.123Z`

### Structured Logs

Logs should be JSON-formatted with Winston:
```typescript
logger.info('Health check completed', { service: 'auth-service', responseTime: 45 });
```

### Agent Activity Logs

Location: `/logs/agents.log`

Format:
```
[timestamp] [AGENT_NAME] action | target_service | result
```

Example:
```
[2026-05-14T14:32:45.123Z] [MAIN] SCAN | all | No active incidents
[2026-05-14T14:32:46.456Z] [DEBUGGER] ANALYZE | auth-service | Reading logs
[2026-05-14T14:32:47.789Z] [DEBUGGER] CLASSIFY | auth-service | syntax-typo
[2026-05-14T14:32:49.012Z] [QA] VERIFY | auth-service | Health check passed
[2026-05-14T14:32:50.345Z] [QA] RESOLVE | auth-service | Incident resolved
```

### Chaos-Monkey Logs

Location: `/logs/chaos.log`

Format: `[timestamp] CHAOS: details`

### Monitor Logs

Location: `/logs/monitor.log`

Format: `[timestamp] SERVICE_NAME: STATUS (responseTimems) - message`

---

## 14. Security & Safety Rules

### Never Modify Unrelated Services

Fixes must target only the service specified in the incident. Do not modify other services, dashboard code, or shared packages.

### Only Patch Targeted Files

Modify only files within `/services/{service}/src/`. Never modify:
- Configuration files outside service directory
- Database schema files
- Dashboard application code
- Shared packages unless explicitly required

### Preserve Backups Before Edits

All file modifications must maintain backup capability. The chaos-monkey script handles this automatically. If manually editing, copy original first.

### Never Delete Incident History

All incidents must remain in SQLite. Do not DELETE from incidents table. Use UPDATE to change status only.

### Restricted Operations

**NEVER perform:**
- `DROP TABLE` operations
- File deletions outside /logs/
- Network requests to external services
- Modifications to .git directory
- Creation of new files outside specified paths

### Safe Rollback

Always prefer restoration over deletion. If in doubt, revert to last known good state.

---

## 15. Deployment Notes

### Local Development Ports

| Component | Port | URL |
|-----------|------|-----|
| Dashboard | 3000 | http://localhost:3000 |
| auth-service | 3001 | http://localhost:3001 |
| payment-service | 3002 | http://localhost:3002 |
| notification-service | 3003 | http://localhost:3003 |

### Environment Variables

No environment variables required for local development. All configuration is in code.

### Dashboard Startup

```bash
cd app
npm install
npm run dev
# Dashboard available at http://localhost:3000
```

### Service Startup

```bash
# Start all services (separate terminals)
cd services/auth-service && npm run dev
cd services/payment-service && npm run dev
cd services/notification-service && npm run dev
```

### Monitoring Startup

```bash
cd scripts
npm run monitor
# Polls services every 10 seconds
```

### AI Resolution Startup

```bash
cd scripts
npm run ai
# Resolves incidents every 15 seconds
```

### Chaos Monkey

```bash
# Inject bug
cd scripts && npm run chaos

# Restore service
cd scripts && npm run restore auth-service

# Check status
cd scripts && npm run status
```

### Quick Start (All Components)

```bash
# Terminal 1: Dashboard
cd app && npm run dev

# Terminal 2: Services
cd services/auth-service && npm run dev
cd services/payment-service && npm run dev
cd services/notification-service && npm run dev

# Terminal 3: Monitoring
cd scripts && npm run monitor

# Terminal 4: AI Engine
cd scripts && npm run ai
```

---

## Quick Reference

| Command | Description |
|---------|-------------|
| `npm run chaos` | Inject random bug |
| `npm run restore <service>` | Restore service to original |
| `npm run monitor` | Start health monitoring |
| `npm run ai` | Start AI resolution engine |

---

*Last updated: 2026-05-14*
*Version: 1.0.0*
*Autonomous Operations Manual*
*Project Sentinel*