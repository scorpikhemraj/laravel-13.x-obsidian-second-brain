---
title: Starter Kits
description: Learn about Laravel application starter kits for quick setup
url: https://laravel.com/docs/13.x/starter-kits
tags: [framework]
---

# Starter Kits

## Introduction

To give you a head start building your new Laravel application, we are happy to offer application starter kits. These starter kits give you a head start on building your next Laravel application, and include the routes, controllers, and views you need to register and authenticate your application's users. The starter kits use Laravel Fortify to provide authentication.

While you are welcome to use these starter kits, they are not required. You are free to build your own application from the ground up by simply installing a fresh copy of Laravel. Either way, we know you will build something great!

## Creating an Application Using a Starter Kit

To create a new Laravel application using one of our starter kits, you should first install PHP and the Laravel CLI tool. If you already have PHP and Composer installed, you may install the Laravel installer CLI tool via Composer:

```
composer global require laravel/installer
```

Then, create a new Laravel application using the Laravel installer CLI. The Laravel installer will prompt you to select your preferred starter kit:

```
laravel new my-app
```

After creating your Laravel application, you only need to install its frontend dependencies via NPM and start the Laravel development server:

```
cd my-app
npm install && npm run build
composer run dev
```

Once you have started the Laravel development server, your application will be accessible in your web browser at http://localhost:8000.

## Available Starter Kits

### React

Our React starter kit provides a robust, modern starting point for building Laravel applications with a React frontend using Inertia.

Inertia allows you to build modern, single-page React applications using classic server-side routing and controllers. This lets you enjoy the frontend power of React combined with the incredible backend productivity of Laravel and lightning-fast Vite compilation.

The React starter kit utilizes React 19, TypeScript, Tailwind, and the shadcn/ui component library.

### Svelte

Our Svelte starter kit provides a robust, modern starting point for building Laravel applications with a Svelte frontend using Inertia.

Inertia allows you to build modern, single-page Svelte applications using classic server-side routing and controllers. This lets you enjoy the frontend power of Svelte combined with the incredible backend productivity of Laravel and lightning-fast Vite compilation.

The Svelte starter kit utilizes Svelte 5, TypeScript, Tailwind, and the shadcn-svelte component library.

### Vue

Our Vue starter kit provides a great starting point for building Laravel applications with a Vue frontend using Inertia.

Inertia allows you to build modern, single-page Vue applications using classic server-side routing and controllers. This lets you enjoy the frontend power of Vue combined with the incredible backend productivity of Laravel and lightning-fast Vite compilation.

The Vue starter kit utilizes the Vue Composition API, TypeScript, Tailwind, and the shadcn-vue component library.

### Livewire

Our Livewire starter kit provides the perfect starting point for building Laravel applications with a Laravel Livewire frontend.

Livewire is a powerful way of building dynamic, reactive, frontend UIs using just PHP. It's a great fit for teams that primarily use Blade templates and are looking for a simpler alternative to JavaScript-driven SPA frameworks like React, Svelte, and Vue.

The Livewire starter kit utilizes Livewire, Tailwind, and the Flux UI component library.

## Starter Kit Customization

### React

Our React starter kit is built with Inertia 2, React 19, Tailwind 4, and shadcn/ui. As with all of our starter kits, all of the backend and frontend code exists within your application to allow for full customization.

The majority of the frontend code is located in the `resources/js` directory. You are free to modify any of the code to customize the appearance and behavior of your application:

```
resources/js/
├── components/    # Reusable React components
├── hooks/         # React hooks
├── layouts/       # Application layouts
├── lib/           # Utility functions and configuration
├── pages/         # Page components
└── types/         # TypeScript definitions
```

To publish additional shadcn components, first find the component you want to publish. Then, publish the component using `npx`:

```
npx shadcn@latest add switch
```

In this example, the command will publish the Switch component to `resources/js/components/ui/switch.tsx`. Once the component has been published, you can use it in any of your pages:

```tsx
import { Switch } from "@/components/ui/switch"

const MyPage = () => {
  return (
    <div>
      <Switch />
    </div>
  );
};

export default MyPage;
```

#### Available Layouts

The React starter kit includes two different primary layouts for you to choose from: a "sidebar" layout and a "header" layout. The sidebar layout is the default, but you can switch to the header layout by modifying the layout that is imported at the top of your application's `resources/js/layouts/app-layout.tsx` file:

```tsx
import AppLayoutTemplate from '@/layouts/app/app-sidebar-layout';
// or
import AppLayoutTemplate from '@/layouts/app/app-header-layout';
```

#### Sidebar Variants

The sidebar layout includes three different variants: the default sidebar variant, the "inset" variant, and the "floating" variant. You may choose the variant you like best by modifying the `resources/js/components/app-sidebar.tsx` component:

```tsx
<Sidebar collapsible="icon" variant="sidebar">
// or
<Sidebar collapsible="icon" variant="inset">
```

#### Authentication Page Layout Variants

The authentication pages included with the React starter kit, such as the login page and registration page, also offer three different layout variants: "simple", "card", and "split".

### Svelte

Our Svelte starter kit is built with Inertia 2, Svelte 5, Tailwind, and shadcn-svelte. As with all of our starter kits, all of the backend and frontend code exists within your application to allow for full customization.

### Vue

Our Vue starter kit is built with Inertia 2, Vue 3 Composition API, Tailwind, and shadcn-vue. As with all of our starter kits, all of the backend and frontend code exists within your application to allow for full customization.

### Livewire

Our Livewire starter kit is built with Livewire 4, Tailwind, and Flux UI. As with all of our starter kits, all of the backend and frontend code exists within your application to allow for full customization.

## Authentication

All starter kits use Laravel Fortify to handle authentication. Fortify provides routes, controllers, and logic for login, registration, password reset, email verification, and more.

Fortify automatically registers the following authentication routes based on the features that are enabled in your application's `config/fortify.php` configuration file:

| Route | Method | Description |
|-------|--------|-------------|
| `/login` | GET | Display login form |
| `/login` | POST | Authenticate user |
| `/logout` | POST | Log user out |
| `/register` | GET | Display registration form |
| `/register` | POST | Create new user |
| `/forgot-password` | GET | Display password reset request form |
| `/forgot-password` | POST | Send password reset link |
| `/reset-password/{token}` | GET | Display password reset form |
| `/reset-password` | POST | Update password |
| `/email/verify` | GET | Display email verification notice |
| `/email/verify/{id}/{hash}` | GET | Verify email address |
| `/email/verification-notification` | POST | Resend verification email |
| `/user/confirm-password` | GET | Display password confirmation form |
| `/user/confirm-password` | POST | Confirm password |
| `/two-factor-challenge` | GET | Display 2FA challenge form |
| `/two-factor-challenge` | POST | Verify 2FA code |

The `php artisan route:list` Artisan command can be used to display all of the routes in your application.

### Enabling and Disabling Features

You can control which Fortify features are enabled in your application's `config/fortify.php` configuration file:

```php
use Laravel\Fortify\Features;

'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],
```

To disable a feature, comment out or remove that feature entry from the `features` array.

### Customizing User Creation and Password Reset

When a user registers or resets their password, Fortify invokes action classes located in your application's `app/Actions/Fortify` directory:

| File | Description |
|------|-------------|
| `CreateNewUser.php` | Validates and creates new users |
| `ResetUserPassword.php` | Validates and updates user passwords |
| `PasswordValidationRules.php` | Defines password validation rules |

### Two-Factor Authentication

Starter kits include built-in two-factor authentication (2FA), allowing users to secure their accounts using any TOTP-compatible authenticator app. 2FA is enabled by default via `Features::twoFactorAuthentication()` in your application's `config/fortify.php` configuration file.

### Rate Limiting

Rate limiting prevents brute-forcing and repeated login attempts from overwhelming your authentication endpoints. You can customize Fortify's rate limiting behavior in your application's `FortifyServiceProvider`.

## Teams

The React, Svelte, Vue, and Livewire starter kits may also be generated with team support. When the teams feature is enabled, each user belongs to one or more teams and has a current team.

## WorkOS AuthKit Authentication

By default, the React, Svelte, Vue, and Livewire starter kits all utilize Laravel's built-in authentication system. In addition, we also offer a WorkOS AuthKit powered variant of each starter kit that offers:

- Social authentication (Google, Microsoft, GitHub, and Apple)
- Passkey authentication
- Email based "Magic Auth"
- SSO

## Inertia SSR

The React, Svelte, and Vue starter kits are compatible with Inertia's server-side rendering capabilities. To build an Inertia SSR compatible bundle for your application, run the `build:ssr` command:

```
npm run build:ssr
```

For convenience, a `composer dev:ssr` command is also available:

```
composer dev:ssr
```

## Community Maintained Starter Kits

When creating a new Laravel application using the Laravel installer, you may provide any community maintained starter kit available on Packagist to the `--using` flag:

```
laravel new my-app --using=example/starter-kit
```

## Frequently Asked Questions

### How do I upgrade?

Every starter kit gives you a solid starting point for your next application. With full ownership of the code, you can tweak, customize, and build your application exactly as you envision. However, there is no need to update the starter kit itself.

### How do I enable email verification?

Email verification can be added by uncommenting the `MustVerifyEmail` import in your `App/Models/User.php` model and ensuring the model implements the `MustVerifyEmail` interface.

After registration, users will receive a verification email. To restrict access to certain routes until the user's email address is verified, add the `verified` middleware to the routes.

### How do I modify the default email template?

You may want to customize the default email template to better align with your application's branding. To modify this template, you should publish the email views to your application with the following command:

```
php artisan vendor:publish --tag=laravel-mail
```

This will generate several files in `resources/views/vendor/mail`.