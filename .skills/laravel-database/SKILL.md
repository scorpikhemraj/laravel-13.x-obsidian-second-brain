# Skill: Laravel Database

## Description

Skill for database development in Laravel - migrations, models, relationships, and queries.

## When to Use This Skill

- User wants database schema
- User needs migrations
- User needs Eloquent models
- User wants relationships
- User needs query optimization

## Workflow

### 1. Migration Creation

1. Create migration: `php artisan make:migration`
2. Define table structure
3. Add columns and types
4. Add indexes and foreign keys

### 2. Model Creation

1. Create model: `php artisan make:model`
2. Define fillable/guarded
3. Set table name if custom
4. Add relationships

### 3. Relationship Definition

1. Define hasOne/hasMany
2. Define belongsTo
3. Define many-to-many (BelongsToMany)
4. Define polymorphic relations

### 4. Seeding

1. Create seeder: `php artisan make:seeder`
2. Define sample data
3. Use factories
4. Run seeding

### 5. Query Optimization

1. Use eager loading
2. Use query scopes
3. Add indexes
4. Use database transactions

## Laravel Commands

```bash
# Create migration
php artisan make:migration create_resources_table

# Create model
php artisan make:model Resource

# Create model with migration
php artisan make:model Resource -m

# Create seeder
php artisan make:seeder ResourceSeeder

# Run migrations
php artisan migrate

# Rollback migrations
php artisan migrate:rollback
```

## Delegates To

- Agent: database-engineer
- Skill: [[.agents/database-engineer/agent.md]]

## Laravel Docs References

- [[laravel-13.x/07-database/01-database-getting-started.md|Database Getting Started]]
- [[laravel-13.x/07-database/04-migrations.md|Migrations]]
- [[laravel-13.x/07-database/02-query-builder.md|Query Builder]]
- [[laravel-13.x/08-eloquent-orm/01-eloquent-getting-started.md|Eloquent Getting Started]]
- [[laravel-13.x/08-eloquent-orm/02-eloquent-relationships.md|Eloquent Relationships]]

## Related Skills

- [[.skills/laravel-architecture/SKILL.md|laravel-architecture]] - Before database
- [[.skills/laravel-backend/SKILL.md|laravel-backend]] - Parallel with database
- [[.skills/laravel-security/SKILL.md|laravel-security]] - After database