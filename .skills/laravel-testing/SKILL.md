# Skill: Laravel Testing

## Description

Skill for testing in Laravel - unit tests, feature tests, browser tests (Dusk), and test automation.

## When to Use This Skill

- User wants test coverage
- User needs unit tests
- User wants feature tests
- User needs browser automation
- User wants test-driven development

## Workflow

### 1. Test Planning

1. Identify test scenarios
2. Plan test structure
3. Set up testing environment
4. Configure test database

### 2. Unit Tests

1. Create test: `php artisan make:test`
2. Test models/services
3. Use Pest or PHPUnit
4. Mock dependencies

### 3. Feature Tests

1. Test HTTP endpoints
2. Test CRUD operations
3. Test authentication
4. Test authorization

### 4. Browser Tests (Dusk)

1. Install Dusk: `composer require --dev laravel/dusk`
2. Create Dusk test
3. Test browser interactions
4. Run on CI/CD

### 5. Test Coverage

1. Run tests: `php artisan test`
2. Check coverage
3. Fix failures
4. Maintain coverage

## Laravel Commands

```bash
# Run all tests
php artisan test

# Run specific test
php artisan test --filter=TestName

# Create test
php artisan make:test ResourceTest

# Create Dusk test
php artisan dusk:make ResourceTest

# Run Dusk
php artisan dusk

# Run with coverage
php artisan test --coverage
```

## Testing Best Practices

- Test one thing per test
- Use descriptive test names
- Follow AAA pattern (Arrange, Act, Assert)
- Mock external services
- Use factories for data

## Delegates To

- Agent: test-engineer
- Skill: [[.agents/test-engineer/agent.md]]

## Laravel Docs References

- [[laravel-13.x/10-testing/01-testing-getting-started.md|Testing Getting Started]]
- [[laravel-13.x/10-testing/02-http-tests.md|HTTP Tests]]
- [[laravel-13.x/10-testing/05-database-testing.md|Database Testing]]
- [[laravel-13.x/10-testing/04-laravel-dusk.md|Laravel Dusk]]

## Related Skills

- [[.skills/laravel-backend/SKILL.md|laravel-backend]] - Before testing
- [[.skills/laravel-security/SKILL.md|laravel-security]] - Parallel with testing
- [[.skills/laravel-code-review/SKILL.md|laravel-code-review]] - After testing