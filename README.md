# The-Capstone-Project-Sentinel-by-Meesho
Project Sentinel is an autonomous AI-driven incident resolution platform that monitors microservices, detects failures in real time, and automatically diagnoses, fixes, tests, and deploys solutions using Claude subagents. Built with Next.js, Node.js, MCP integration, Chaos Engineering, and multi-agent orchestration.


Monorepo Structure
```
Project-Sentinel/
├── app/                    # Next.js dashboard (port 3000)
├── services/               # 3 Microservices
│   ├── auth-service/       # Port 3001
│   ├── payment-service/   # Port 3002
│   └── notification-service/ # Port 3003
├── scripts/                # Automation scripts
├── database/               # SQLite storage
├── logs/                   # Service logs
├── .backups/               # Chaos Monkey backups
└── docs/                  # Documentation


# Project Sentinel - Setup & Usage Guide

## Quick Start

### Step 1: Start All Services (3 terminals)

```bash
# Terminal 1 - Dashboard
cd app && npm install && npm run dev

# Terminal 2 - Auth Service
cd services/auth-service && npm install && npm run dev

# Terminal 3 - Payment Service  
cd services/payment-service && npm install && npm run dev

# Terminal 4 - Notification Service
cd services/notification-service && npm install && npm run dev

# Terminal 5 - Monitoring
cd scripts && npm install && npm run monitor
```

### Step 2: Open Dashboard
Navigate to http://localhost:3000 to see the real-time dashboard.

---

## The Autonomous Workflow

### Phase 1: Inject Chaos
```bash
cd scripts && npm run chaos
```
This breaks one of the services.

### Phase 2: Detection (Automatic)
The monitoring service automatically:
- Detects the CRITICAL service
- Creates an incident in SQLite
- Updates dashboard status

### Phase 3: Autonomous Fix (THE ASSIGNMENT CORE)

**When you see an incident in the dashboard, you must command Claude:**

```
Sentinel Agent, identify why auth-service is failing. Access the /services/logs, find the error, and implement a fix that follows our CLAUDE.md standards.
```

Claude will then:
1. Use subagent to analyze logs
2. Identify the root cause
3. Write a fix
4. Run tests
5. Commit the fix

### Claude Subagent Commands

Use these Claude Code commands:

```bash
# Analyze logs for a specific service
/subagent analyze auth-service

# Plan a fix strategy
Shift+Tab x2

# Fix the service
Sentinel Agent, fix auth-service

# Verify the fix worked
curl http://localhost:3001/health
```

---

## Dashboard Features

| Feature | Update Interval |
|---------|----------------|
| System Health Cards | Every 2 seconds |
| Active Incidents | Every 2 seconds |
| Resolved Incidents | Every 2 seconds |
| Service Grid | Every 2 seconds |

---

## Manual Commands

```bash
# Restore a broken service
cd scripts && npm run restore auth-service

# Check chaos status
cd scripts && npm run status

# See all logs
cat logs/monitor.log
cat logs/agents.log
cat logs/chaos.log
```

---

## Architecture

```
User runs chaos → Service crashes → Monitor detects → Incident created
                                                              ↓
                                              User commands Claude to fix:
                                              "Sentinel Agent, fix auth-service"
                                                              ↓
                                              Claude analyzes → writes fix → commits
                                                              ↓
                                              QA verifies → Dashboard updates
```

---

## Verification Checklist

- [ ] Dashboard loads at http://localhost:3000
- [ ] All 3 services show HEALTHY initially
- [ ] Running `npm run chaos` breaks a service
- [ ] Monitoring detects CRITICAL within 10 seconds
- [ ] Incident appears in dashboard Active Incidents
- [ ] User commands "Sentinel Agent, fix..."
- [ ] Claude analyzes and fixes the issue
- [ ] Service returns to HEALTHY
- [ ] Incident marked as resolved
