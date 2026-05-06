# Skill: Laravel Deployment

## Description

Skill for Laravel deployment - CI/CD pipelines, Forge/Vapor, server setup, and production deployment.

## When to Use This Skill

- User wants to deploy to production
- User needs CI/CD setup
- User wants Forge/Vapor deployment
- User needs server setup
- User wants SSL/HTTPS setup

## Workflow

### 1. Pre-deployment Checklist

1. Run tests locally
2. Check environment config
3. Optimize production
4. Generate keys

### 2. Server Setup

1. Choose provider (Forge, Vapor, DigitalOcean, AWS)
2. Set up server
3. Configure PHP
4. Set up database

### 3. CI/CD Pipeline

1. Create GitHub Actions
2. Set up deployment workflow
3. Configure environment
4. Set up notifications

### 4. Deployment

1. Deploy to server
2. Run migrations
3. Set up queues
4. Configure caching

### 5. Post-deployment

1. Verify deployment
2. Set up monitoring
3. Configure backups
4. Set up logging

## Environment Configuration

```bash
# Generate app key
php artisan key:generate

# Generate JWT key
php artisan jwt:secret

# Clear caches
php artisan config:clear
php artisan cache:clear
php artisan route:clear
```

## GitHub Actions Example

```yaml
name: Deploy
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy
        run: |
          php artisan migrate --force
          php artisan config:cache
```

## Delegates To

- Agent: devops-engineer
- Skill: [[.agents/devops-engineer/agent.md]]

## Laravel Docs References

- [[laravel-13.x/02-getting-started/07-deployment.md|Deployment]]
- [[laravel-13.x/11-packages/07-laravel-homestead.md|Laravel Forge]]
- [[laravel-13.x/11-packages/10-laravel-octane.md|Laravel Vapor]]

## Related Skills

- [[.skills/laravel-docker/SKILL.md|laravel-docker]] - Before deployment
- [[.skills/laravel-code-review/SKILL.md|laravel-code-review]] - Before deployment