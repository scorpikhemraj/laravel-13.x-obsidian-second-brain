# Skill: Laravel Development System

## Description

Comprehensive Laravel development agent system for building products from scratch. This skill coordinates 22 specialized agents through a 5-step chain to deliver complete Laravel applications.

## When to Use This Skill

- User wants to build a new Laravel application from scratch
- User wants to add features to existing Laravel app
- User needs help with Laravel architecture, coding, testing, or deployment
- User wants to maintain or optimize existing Laravel application
- User needs Laravel-specific code review or documentation

## Workflow

### Step 1: Planning & Architecture (Parallel)

1. **product-manager** - Creates feature specs, user stories, backlog
2. **system-architect** - Designs technical architecture, DB schema, API design

**Output:** Technical specification document

### Step 2: Database & Backend (Parallel)

3. **database-engineer** - Creates migrations, models, relationships
4. **backend-developer** - Implements controllers, services, APIs, jobs

**Output:** Backend code implemented

### Step 3: Frontend Development

5. **frontend-developer** - Creates UI components, Blade templates, Inertia pages

**Output:** Frontend code implemented

### Step 4: Security & Testing (Parallel)

6. **security-engineer** - implements auth, validation, permissions
7. **test-engineer** - Writes unit, feature, browser tests

**Output:** Secure, tested code

### Step 5: Review & Deploy (Parallel)

8. **code-reviewer** - Reviews code quality, best practices
9. **devops-engineer** - Docker setup, CI/CD, deployment
10. **documentation-writer** - Creates API docs, README

**Output:** Production-ready application

## Additional Agents (As Needed)

| Use Case | Agent |
|---------|-------|
| Third-party integrations | integration-engineer |
| Performance optimization | performance-engineer |
| AI/ML features | ai-engineer |
| Bug fixes, maintenance | maintenance-engineer |
| Mobile API | mobile-engineer |
| Server infrastructure | infrastructure-engineer |
| Analytics, tracking | analytics-engineer |
| Push/email notifications | notification-engineer |
| Real-time features | realtime-engineer |
| Jobs, queues | queue-engineer |
| Monitoring, logs | monitoring-engineer |
| Learn from mistakes | learning-engineer |

## Subagent Chain Configuration

| Chain Step | Primary Agents | Parallel Agents |
|------------|---------------|----------------|
| 1 | product-manager, system-architect | Both parallel |
| 2 | database-engineer, backend-developer | Both parallel |
| 3 | frontend-developer | - |
| 4 | security-engineer, test-engineer | Both parallel |
| 5 | code-reviewer, devops-engineer, documentation-writer | All parallel |

**Maximum chain depth:** 5 steps
**Parallel execution:** Enabled where agents are independent
**Data passing:** JSON context between agents

## Agents Discovery

| Keyword | Agent |
|---------|-------|
| "plan", "feature", "story", "backlog" | product-manager |
| "architecture", "design", "schema" | system-architect |
| "database", "migration", "model" | database-engineer |
| "api", "controller", "service" | backend-developer |
| "ui", "blade", "frontend" | frontend-developer |
| "auth", "security", "permission" | security-engineer |
| "test", "unit", "feature" | test-engineer |
| "review", "quality", "code review" | code-reviewer |
| "docker", "deploy", "ci/cd" | devops-engineer |
| "docs", "documentation" | documentation-writer |
| "integration", "api", "stripe" | integration-engineer |
| "performance", "cache", "optimize" | performance-engineer |
| "ai", "machine learning" | ai-engineer |
| "fix", "bug", "maintain" | maintenance-engineer |
| "mobile", "react native" | mobile-engineer |
| "server", "infrastructure" | infrastructure-engineer |
| "analytics", "tracking" | analytics-engineer |
| "notification", "email", "push" | notification-engineer |
| "real-time", "websocket" | realtime-engineer |
| "queue", "job", "horizon" | queue-engineer |
| "monitor", "log", "telescope" | monitoring-engineer |
| "learn", "improve", "mistake" | learning-engineer |

## Error Handling

- Agent fails → Report error with details
- Allow 1 retry before failing
- Log failure to learning-engineer
- Stop chain on failure

## Laravel Docs References

- [[laravel-13.x/02-getting-started/01-installation.md|Installation]]
- [[laravel-13.x/02-getting-started/04-directory-structure.md|Directory Structure]]
- [[laravel-13.x/03-architecture-concepts|Architecture Concepts]]
- [[laravel-13.x/04-the-basics/01-routing.md|Routing]]
- [[laravel-13.x/04-the-basics/04-controllers.md|Controllers]]
- [[laravel-13.x/07-database|Database]]
- [[laravel-13.x/08-eloquent-orm|Eloquent ORM]]
- [[laravel-13.x/06-security|Security]]
- [[laravel-13.x/10-testing|Testing]]
- [[laravel-13.x/02-getting-started/07-deployment.md|Deployment]]

## Related Skills

- [[.skills/laravel-planning/SKILL.md|laravel-planning]]
- [[.skills/laravel-architecture/SKILL.md|laravel-architecture]]
- [[.skills/laravel-backend/SKILL.md|laravel-backend]]
- [[.skills/laravel-frontend/SKILL.md|laravel-frontend]]
- [[.skills/laravel-database/SKILL.md|laravel-database]]
- [[.skills/laravel-security/SKILL.md|laravel-security]]
- [[.skills/laravel-testing/SKILL.md|laravel-testing]]
- [[.skills/laravel-code-review/SKILL.md|laravel-code-review]]
- [[.skills/laravel-docker/SKILL.md|laravel-docker]]
- [[.skills/laravel-deployment/SKILL.md|laravel-deployment]]