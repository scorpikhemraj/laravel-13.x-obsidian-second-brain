# Skill: Laravel Architecture

## Description

Skill for system architecture, technical design, database schema design, and technology decisions for Laravel projects.

## When to Use This Skill

- User needs technical architecture design
- User wants database schema design
- User needs tech stack recommendations
- User wants API design
- User needs architecture patterns

## Workflow

### 1. Requirements Analysis

1. Review feature specifications
2. Identify technical requirements
3. Define constraints
4. Assess scale needs

### 2. Architecture Design

1. Choose application pattern (modular, monolith, microservices)
2. Define service layers (API, business logic, data)
3. Design component interactions
4. Document architecture decisions

### 3. Database Schema Design

1. Identify entities and relationships
2. Design schema (tables, columns, indexes)
3. Define foreign keys
4. Plan migrations

### 4. API Design

1. Design RESTful endpoints
2. Define request/response formats
3. Plan authentication
4. Document API spec

### 5. Technology Stack

1. Choose PHP version
2. Select packages (Sanctum, Horizon, etc.)
3. Choose frontend stack
4. Choose infrastructure

## Architecture Output Template

```markdown
# Technical Specification

## Architecture Overview
[Description]

## Database Schema
### Entities
- [Entity 1]: [fields]
- [Entity 2]: [fields]

### Relationships
- [Relationship descriptions]

## API Endpoints
| Method | Endpoint | Description |
|--------|---------|-------------|
| GET | /api/resource | List |
| POST | /api/resource | Create |

## Technology Stack
- PHP: 8.3
- Laravel: 13.x
- Database: PostgreSQL
- Frontend: Inertia + Vue
```

## Delegates To

- Agent: system-architect
- Skill: [[.agents/system-architect/agent.md]]

## Laravel Docs References

- [[laravel-13.x/03-architecture-concepts/*|Architecture Concepts]]
- [[laravel-13.x/08-eloquent-orm/02-eloquent-relationships.md|Eloquent Relationships]]
- [[laravel-13.x/11-packages/*|Packages]]

## Related Skills

- [[.skills/laravel-planning/SKILL.md|laravel-planning]] - Before architecture
- [[.skills/laravel-database/SKILL.md|laravel-database]] - For DB implementation
- [[.skills/laravel-backend/SKILL.md|laravel-backend]] - For API implementation