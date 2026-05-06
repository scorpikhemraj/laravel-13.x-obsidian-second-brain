# Skill: Laravel Docker

## Description

Skill for Docker containerization in Laravel - Docker setup, docker-compose, Laravel Sail, and container deployment.

## When to Use This Skill

- User wants Docker setup
- User needs Docker Compose
- User wants Laravel Sail
- User needs local development
- User wants container deployment

## Workflow

### 1. Dockerfile Creation

1. Use official PHP image
2. Install extensions
3. Copy application
4. Set up entrypoint

### 2. Docker Compose

1. Define services (PHP, Nginx, MySQL, Redis)
2. Configure volumes
3. Set up networks
4. Configure environment

### 3. Laravel Sail

1. Install Sail: `composer require laravel/sail --dev`
2. Publish: `php artisan vendor:publish --tag=sail`
3. Create alias: `alias sail='./vendor/bin/sail'`
4. Start: `./sail up`

### 4. Development Setup

1. Set up development environment
2. Configure hot reload
3. Set up debugging
4. Configure testing

### 5. Production Build

1. Optimize Dockerfile
2. Build production image
3. Set up CI/CD
4. Configure monitoring

## Dockerfile Example

```dockerfile
FROM php:8.3-fpm

# Install extensions
RUN docker-php-ext-install pdo_mysql

# Copy application
COPY . /var/www/html

# Set up permissions
RUN chown -R www-data:www-data /var/www/html/storage

# Expose port
EXPOSE 8000

# Start application
CMD [[.skills/laravel-docker/"php", "artisan", "serve", "--host=0.0.0.0"]]
```

## Laravel Sail Commands

```bash
# Start Sail
./sail up

# Stop Sail
./sail down

# Run Artisan
./sail artisan make:model

# Run tests
./sail test
```

## Delegates To

- Agent: devops-engineer
- Skill: [[.agents/devops-engineer/agent.md]]

## Laravel Docs References

- [[laravel-13.x/11-packages/18-laravel-sail.md|Laravel Sail]]
- [[laravel-13.x/02-getting-started/07-deployment.md|Deployment]]

## Related Skills

- [[.skills/laravel-code-review/SKILL.md|laravel-code-review]] - Before Docker
- [[.skills/laravel-deployment/SKILL.md|laravel-deployment]] - After Docker