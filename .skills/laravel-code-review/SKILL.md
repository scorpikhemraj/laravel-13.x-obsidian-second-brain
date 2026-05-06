# Skill: Laravel Code Review

## Description

Skill for code review in Laravel - code quality, best practices, PSR standards, and security audit.

## When to Use This Skill

- User needs code review
- User wants security audit
- User needs performance review
- User wants PSR compliance
- User needs refactoring suggestions

## Workflow

### 1. Code Quality Review

1. Check Laravel conventions
2. Verify PSR standards
3. Review naming conventions
4. Check code organization

### 2. Security Review

1. Check for SQL injection
2. Verify XSS prevention
3. Check CSRF protection
4. Review authentication

### 3. Performance Review

1. Check N+1 queries
2. Review caching
3. Check indexing
4. Review queue usage

### 4. Test Coverage Review

1. Check test existence
2. Review test quality
3. Check edge cases
4. Verify coverage

### 5. Documentation Review

1. Check code comments
2. Verify README updates
3. Check API documentation
4. Review changelog

## Code Review Checklist

- [ ] Follows Laravel conventions
- [ ] PSR-12 compliant
- [ ] Proper validation
- [ ] Security best practices
- [ ] No N+1 queries
- [ ] Proper error handling
- [ ] Tests exist and pass
- [ ] Documentation updated

## Tools

- Laravel Pint (code style): `vendor/bin/pint`
- PHPStan (static analysis): `vendor/bin/phpstan`
- PHP CS Fixer (formatting)

## Delegates To

- Agent: code-reviewer
- Skill: [[.agents/code-reviewer/agent.md]]

## Laravel Docs References

- [[laravel-13.x/11-packages/13-laravel-pint.md|Laravel Pint]]
- [[laravel-13.x/04-the-basics/13-error-handling.md|Error Handling]]
- [[laravel-13.x/10-testing/*|Testing]]

## Related Skills

- [[.skills/laravel-testing/SKILL.md|laravel-testing]] - Before code review
- [[.skills/laravel-docker/SKILL.md|laravel-docker]] - After code review
- [[.skills/laravel-deployment/SKILL.md|laravel-deployment]] - After code review