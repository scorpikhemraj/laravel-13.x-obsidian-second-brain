# Skill: Laravel Security

## Description

Skill for security implementation in Laravel - authentication, authorization, validation, CSRF protection, and encryption.

## When to Use This Skill

- User needs authentication
- User wants authorization
- User needs input validation
- User wants security audit
- User needs encryption

## Workflow

### 1. Authentication Setup

1. Choose auth method (Fortify, Breeze, Jetstream)
2. Install: `composer require laravel/fortify`
3. Configure auth handlers
4. Set up login/register flows

### 2. Authorization (Policies)

1. Create policy: `php artisan make:policy`
2. Define abilities
3. Use in controllers
4. Use in Blade templates

### 3. Input Validation

1. Use FormRequest validation
2. Add rule lists
3. Validate arrays
4. Custom validation rules

### 4. Security Headers

1. Use HTTPS everywhere
2. Set security headers
3. Configure CORS
4. Enable CSRF protection

### 5. Password Security

1. Use bcrypt hashing
2. Implement password reset
3. Add password strength rules
4. Use rate limiting

## Laravel Commands

```bash
# Install Laravel Fortify
composer require laravel/fortify
php artisan vendor:publish --provider="Laravel\Fortify\FortifyServiceProvider"

# Create policy
php artisan make:policy ResourcePolicy

# Use auth middleware
->middleware('auth')
```

## Delegates To

- Agent: security-engineer
- Skill: [[.agents/security-engineer/agent.md]]

## Laravel Docs References

- [[laravel-13.x/06-security/01-authentication.md|Authentication]]
- [[laravel-13.x/06-security/02-authorization.md|Authorization]]
- [[laravel-13.x/06-security/06-password-reset.md|Password Reset]]
- [[laravel-13.x/04-the-basics/12-validation.md|Validation]]

## Related Skills

- [[.skills/laravel-backend/SKILL.md|laravel-backend]] - Before security
- [[.skills/laravel-frontend/SKILL.md|laravel-frontend]] - Parallel with security
- [[.skills/laravel-testing/SKILL.md|laravel-testing]] - After security