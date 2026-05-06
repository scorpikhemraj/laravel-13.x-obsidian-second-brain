# Skill: Laravel Backend Development

## Description

Skill for backend development in Laravel - controllers, services, APIs, jobs, events, and business logic implementation.

## When to Use This Skill

- User wants to create API endpoints
- User needs controller development
- User needs service layer implementation
- User wants job/queue setup
- User needs event handling

## Workflow

### 1. Route Definition

1. Define routes in routes/api.php or routes/web.php
2. Set HTTP method, endpoint, middleware
3. Name routes for URL generation

### 2. Controller Creation

1. Generate controller: `php artisan make:controller`
2. Define resource methods (index, show, store, update, destroy)
3. UseFormRequest for validation
4. Return JSON or view responses

### 3. Service Layer

1. Create service class for business logic
2. Implement repository pattern if needed
3. Use dependency injection
4. Keep controllers thin

### 4. Request Validation

1. Create FormRequest: `php artisan make:request`
2. Define validation rules
3. Add authorization logic
4. Use in controller

### 5. API Resources

1. Create API Resource: `php artisan make:resource`
2. Define data transformations
3. Handle relationships
4. Return resource responses

## Laravel Commands

```bash
# Create controller
php artisan make:controller Api/ResourceController

# Create resource
php artisan make:resource ResourceName

# Create request
php artisan make:request StoreResourceRequest

# Create service
# Manual: app/Services/ResourceService.php

# Create job
php artisan make:job ProcessResourceJob
```

## Delegates To

- Agent: backend-developer
- Skill: [[.agents/backend-developer/agent.md]]

## Laravel Docs References

- [[laravel-13.x/04-the-basics/01-routing.md|Routing]]
- [[laravel-13.x/04-the-basics/04-controllers.md|Controllers]]
- [[laravel-13.x/04-the-basics/12-validation.md|Validation]]
- [[laravel-13.x/04-the-basics/05-http-requests.md|HTTP Requests]]
- [[laravel-13.x/08-eloquent-orm/05-eloquent-api-resources.md|API Resources]]

## Related Skills

- [[.skills/laravel-architecture/SKILL.md|laravel-architecture]] - Before backend
- [[.skills/laravel-database/SKILL.md|laravel-database]] - Parallel with backend
- [[.skills/laravel-security/SKILL.md|laravel-security]] - After backend
- [[.skills/laravel-testing/SKILL.md|laravel-testing]] - After backend