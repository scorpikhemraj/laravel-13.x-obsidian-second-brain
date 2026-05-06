# Skill: Laravel Frontend Development

## Description

Skill for frontend development in Laravel - Blade templates, Inertia pages, Vue/React components, and UI implementation.

## When to Use This Skill

- User wants to create UI pages
- User needs Blade component development
- User wants Inertia integration
- User needs Vue/React components
- User wants Tailwind styling

## Workflow

### 1. Page Structure

1. Identify page requirements
2. Choose approach (Blade or Inertia)
3. Define page layout
4. Plan components

### 2. Blade Development

1. Create/edit Blade template
2. Use components (@component, @slot)
3. Include assets (CSS, JS)
4. Use Laravel directives (@auth, @guest)

### 3. Component Creation

1. Create Blade component: `php artisan make:component`
2. Define props and slots
3. Add styling (Tailwind)
4. Add interactivity (Alpine.js)

### 4. Inertia Integration

1. Install Inertia adapter
2. Create Inertia controller
3. Use Inertia::render()
4. Share data with pages

### 5. Asset Management

1. Use Vite for bundling
2. Configure scripts/styles
3. Use Laravel Mix if needed

## Laravel Commands

```bash
# Create Blade component
php artisan make:component ComponentName

# Install Inertia
composer require inertiajs/inertia-laravel

# Install Vue adapter
npm install @inertiajs/vue3

# Install Tailwind
composer require wire-elements/modal
```

## Delegates To

- Agent: frontend-developer
- Skill: [[.agents/frontend-developer/agent.md]]

## Laravel Docs References

- [[laravel-13.x/04-the-basics/08-blade-templates.md|Blade Templates]]
- [[laravel-13.x/04-the-basics/07-views.md|Views]]
- [[laravel-13.x/04-the-basics/09-asset-bundling-vite.md|Asset Bundling (Vite)]]
- [[laravel-13.x/02-getting-started/05-frontend.md|Frontend]]

## Related Skills

- [[.skills/laravel-backend/SKILL.md|laravel-backend]] - Before frontend
- [[.skills/laravel-database/SKILL.md|laravel-database]] - Parallel with backend
- [[.skills/laravel-security/SKILL.md|laravel-security]] - After frontend