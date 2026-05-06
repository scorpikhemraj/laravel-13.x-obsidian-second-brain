---
title: Blade Templates
description: Laravel's templating engine for rendering views with components, directives, and layout control
url: https://laravel.com/docs/13.x/blade
tags: [framework]
cssclasses:
  - framework
  - color-blue
color: blue
---

# Blade Templates

-   [Introduction](#introduction)
    -   [Supercharging Blade With Livewire](#supercharging-blade-with-livewire)
-   [Displaying Data](#displaying-data)
    -   [HTML Entity Encoding](#html-entity-encoding)
    -   [Blade and JavaScript Frameworks](#blade-and-javascript-frameworks)
-   [Blade Directives](#blade-directives)
    -   [If Statements](#if-statements)
    -   [Switch Statements](#switch-statements)
    -   [Loops](#loops)
    -   [The Loop Variable](#the-loop-variable)
    -   [Conditional Classes](#conditional-classes)
    -   [Additional Attributes](#additional-attributes)
    -   [Including Subviews](#including-subviews)
    -   [The `@once` Directive](#the-once-directive)
    -   [Raw PHP](#raw-php)
    -   [Comments](#comments)
-   [Components](#components)
    -   [Rendering Components](#rendering-components)
    -   [Index Components](#index-components)
    -   [Passing Data to Components](#passing-data-to-components)
    -   [Component Attributes](#component-attributes)
    -   [Reserved Keywords](#reserved-keywords)
    -   [Slots](#slots)
    -   [Inline Component Views](#inline-component-views)
    -   [Dynamic Components](#dynamic-components)
    -   [Manually Registering Components](#manually-registering-components)
-   [Anonymous Components](#anonymous-components)
    -   [Anonymous Index Components](#anonymous-index-components)
    -   [Data Properties / Attributes](#data-properties-attributes)
    -   [Accessing Parent Data](#accessing-parent-data)
    -   [Anonymous Components Paths](#anonymous-component-paths)
-   [Building Layouts](#building-layouts)
    -   [Layouts Using Components](#layouts-using-components)
    -   [Layouts Using Template Inheritance](#layouts-using-template-inheritance)
-   [Forms](#forms)
    -   [CSRF Field](#csrf-field)
    -   [Method Field](#method-field)
    -   [Validation Errors](#validation-errors)
-   [Stacks](#stacks)
-   [Service Injection](#service-injection)
-   [Rendering Inline Blade Templates](#rendering-inline-blade-templates)
-   [Rendering Blade Fragments](#rendering-blade-fragments)
-   [Extending Blade](#extending-blade)
    -   [Custom Echo Handlers](#custom-echo-handlers)
    -   [Custom If Statements](#custom-if-statements)

## [Introduction](#introduction)

Blade is the simple, yet powerful templating engine that is included with Laravel. Unlike some PHP templating engines, Blade does not restrict you from using plain PHP code in your templates. In fact, all Blade templates are compiled into plain PHP code and cached until they are modified, meaning Blade adds essentially zero overhead to your application. Blade template files use the `.blade.php` file extension and are typically stored in the `resources/views` directory.

Blade views may be returned from routes or controllers using the global `view` helper. Of course, as mentioned in the documentation on [[04-the-basics/07-views.md|views]], data may be passed to the Blade view using the `view` helper's second argument:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'Finn']);
});
```

### [Supercharging Blade With Livewire](#supercharging-blade-with-livewire)

Want to take your Blade templates to the next level and build dynamic interfaces with ease? Check out [Laravel Livewire](https://livewire.laravel.com). Livewire allows you to write Blade components that are augmented with dynamic functionality that would typically only be possible via frontend frameworks like React, Svelte, or Vue, providing a great approach to building modern, reactive frontends without the complexities, client-side rendering, or build steps of many JavaScript frameworks.

## [Displaying Data](#displaying-data)

You may display data that is passed to your Blade views by wrapping the variable in curly braces. For example, given the following route:

```php
Route::get('/', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

You may display the contents of the `name` variable like so:

```blade
Hello, {{ $name }}.
```

Blade's `{{ }}` echo statements are automatically sent through PHP's `htmlspecialchars` function to prevent XSS attacks.

You are not limited to displaying the contents of the variables passed to the view. You may also echo the results of any PHP function. In fact, you can put any PHP code you wish inside of a Blade echo statement:

```blade
The current UNIX timestamp is {{ time() }}.
```

### [HTML Entity Encoding](#html-entity-encoding)

By default, Blade (and the Laravel `e` function) will double encode HTML entities. If you would like to disable double encoding, call the `Blade::withoutDoubleEncoding` method from the `boot` method of your `AppServiceProvider`:

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::withoutDoubleEncoding();
    }
}
```

#### [Displaying Unescaped Data](#displaying-unescaped-data)

By default, Blade `{{ }}` statements are automatically sent through PHP's `htmlspecialchars` function to prevent XSS attacks. If you do not want your data to be escaped, you may use the following syntax:

```blade
Hello, {!! $name !!}.
```

Be very careful when echoing content that is supplied by users of your application. You should typically use the escaped, double curly brace syntax to prevent XSS attacks when displaying user supplied data.

### [Blade and JavaScript Frameworks](#blade-and-javascript-frameworks)

Since many JavaScript frameworks also use "curly" braces to indicate a given expression should be displayed in the browser, you may use the `@` symbol to inform the Blade rendering engine an expression should remain untouched. For example:

```blade
<h1>Laravel</h1>

Hello, @{{ name }}.
```

In this example, the `@` symbol will be removed by Blade; however, the `{{ name }}` expression will remain untouched by the Blade engine, allowing it to be rendered by your JavaScript framework.

The `@` symbol may also be used to escape Blade directives:

```blade
{{-- Blade template --}}
@@if()

<!-- HTML output -->
@if()
```

#### [Rendering JSON](#rendering-json)

Sometimes you may pass an array to your view with the intention of rendering it as JSON in order to initialize a JavaScript variable. For example:

```php
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

However, instead of manually calling `json_encode`, you may use the `Illuminate\Support\Js::from` method. The `from` method accepts the same arguments as PHP's `json_encode` function; however, it will ensure that the resulting JSON has been properly escaped for inclusion within HTML quotes. The `from` method will return a string `JSON.parse` JavaScript statement that will convert the given object or array into a valid JavaScript object:

```php
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

The latest versions of the Laravel application skeleton include a `Js` facade, which provides convenient access to this functionality within your Blade templates:

```php
<script>
    var app = {{ Js::from($array) }};
</script>
```

You should only use the `Js::from` method to render existing variables as JSON. The Blade templating is based on regular expressions and attempts to pass a complex expression to the directive may cause unexpected failures.

#### [The `@verbatim` Directive](#the-at-verbatim-directive)

If you are displaying JavaScript variables in a large portion of your template, you may wrap the HTML in the `@verbatim` directive so that you do not have to prefix each Blade echo statement with an `@` symbol:

```blade
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

## [Blade Directives](#blade-directives)

In addition to template inheritance and displaying data, Blade also provides convenient shortcuts for common PHP control structures, such as conditional statements and loops. These shortcuts provide a very clean, terse way of working with PHP control structures while also remaining familiar to their PHP counterparts.

### [If Statements](#if-statements)

You may construct `if` statements using the `@if`, `@elseif`, `@else`, and `@endif` directives. These directives function identically to their PHP counterparts:

```blade
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

For convenience, Blade also provides an `@unless` directive:

```blade
@unless (Auth::check())
    You are not signed in.
@endunless
```

In addition to the conditional directives already discussed, the `@isset` and `@empty` directives may be used as convenient shortcuts for their respective PHP functions:

```blade
@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

#### [Authentication Directives](#authentication-directives)

The `@auth` and `@guest` directives may be used to quickly determine if the current user is [[06-security/01-authentication.md|authenticated]] or is a guest:

```blade
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

If needed, you may specify the authentication guard that should be checked when using the `@auth` and `@guest` directives:

```blade
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

#### [Environment Directives](#environment-directives)

You may check if the application is running in the production environment using the `@production` directive:

```blade
@production
    // Production specific content...
@endproduction
```

Or, you may determine if the application is running in a specific environment using the `@env` directive:

```blade
@env('staging')
    // The application is running in "staging"...
@endenv

@env(['staging', 'production'])
    // The application is running in "staging" or "production"...
@endenv
```

#### [Section Directives](#section-directives)

You may determine if a template inheritance section has content using the `@hasSection` directive:

```blade
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

You may use the `sectionMissing` directive to determine if a section does not have content:

```blade
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

#### [Session Directives](#session-directives)

The `@session` directive may be used to determine if a [[04-the-basics/11-http-session.md|session]] value exists. If the session value exists, the template contents within the `@session` and `@endsession` directives will be evaluated. Within the `@session` directive's contents, you may echo the `$value` variable to display the session value:

```blade
@session('status')
    <div class="p-4 bg-green-100">
        {{ $value }}
    </div>
@endsession
```

#### [Context Directives](#context-directives)

The `@context` directive may be used to determine if a [[05-digging-deeper/06-context.md|context]] value exists. If the context value exists, the template contents within the `@context` and `@endcontext` directives will be evaluated. Within the `@context` directive's contents, you may echo the `$value` variable to display the context value:

```blade
@context('canonical')
    <link href="{{ $value }}" rel="canonical">
@endcontext
```

### [Switch Statements](#switch-statements)

Switch statements can be constructed using the `@switch`, `@case`, `@break`, `@default` and `@endswitch` directives:

```blade
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

### [Loops](#loops)

In addition to conditional statements, Blade provides simple directives for working with PHP's loop structures. Again, each of these directives functions identically to their PHP counterparts:

```blade
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

While iterating through a `foreach` loop, you may use the [loop variable](#the-loop-variable) to gain valuable information about the loop, such as whether you are in the first or last iteration through the loop.

When using loops you may also skip the current iteration or end the loop using the `@continue` and `@break` directives:

```blade
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

You may also include the continuation or break condition within the directive declaration:

```blade
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

### [The Loop Variable](#the-loop-variable)

While iterating through a `foreach` loop, a `$loop` variable will be available inside of your loop. This variable provides access to some useful bits of information such as the current loop index and whether this is the first or last iteration through the loop:

```blade
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

If you are in a nested loop, you may access the parent loop's `$loop` variable via the `parent` property:

```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is the first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

The `$loop` variable also contains a variety of other useful properties:

| Property | Description |
| --- | --- |
| `$loop->index` | The index of the current loop iteration (starts at 0). |
| `$loop->iteration` | The current loop iteration (starts at 1). |
| `$loop->remaining` | The iterations remaining in the loop. |
| `$loop->count` | The total number of items in the array being iterated. |
| `$loop->first` | Whether this is the first iteration through the loop. |
| `$loop->last` | Whether this is the last iteration through the loop. |
| `$loop->even` | Whether this is an even iteration through the loop. |
| `$loop->odd` | Whether this is an odd iteration through the loop. |
| `$loop->depth` | The nesting level of the current loop. |
| `$loop->parent` | When in a nested loop, the parent's loop variable. |

### [Conditional Classes & Styles](#conditional-classes)

The `@class` directive conditionally compiles a CSS class string. The directive accepts an array of classes where the array key contains the class or classes you wish to add, while the value is a boolean expression. If the array element has a numeric key, it will always be included in the rendered class list:

```blade
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

<span class="p-4 text-gray-500 bg-red"></span>
```

Likewise, the `@style` directive may be used to conditionally add inline CSS styles to an HTML element:

```blade
@php
    $isActive = true;
@endphp

<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>

<span style="background-color: red; font-weight: bold;"></span>
```

### [Additional Attributes](#additional-attributes)

For convenience, you may use the `@checked` directive to easily indicate if a given HTML checkbox input is "checked". This directive will echo `checked` if the provided condition evaluates to `true`:

```blade
<input
    type="checkbox"
    name="active"
    value="active"
    @checked(old('active', $user->active))
/>
```

Likewise, the `@selected` directive may be used to indicate if a given select option should be "selected":

```blade
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

Additionally, the `@disabled` directive may be used to indicate if a given element should be "disabled":

```blade
<button type="submit" @disabled($errors->isNotEmpty())>Submit</button>
```

Moreover, the `@readonly` directive may be used to indicate if a given element should be "readonly":

```blade
<input
    type="email"
    name="email"
    value="[email protected]"
    @readonly($user->isNotAdmin())
/>
```

In addition, the `@required` directive may be used to indicate if a given element should be "required":

```blade
<input
    type="text"
    name="title"
    value="title"
    @required($user->isAdmin())
/>
```

### [Including Subviews](#including-subviews)

While you're free to use the `@include` directive, Blade [components](#components) provide similar functionality and offer several benefits over the `@include` directive such as data and attribute binding.

Blade's `@include` directive allows you to include a Blade view from within another view. All variables that are available to the parent view will be made available to the included view:

```blade
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

Even though the included view will inherit all data available in the parent view, you may also pass an array of additional data that should be made available to the included view:

```blade
@include('view.name', ['status' => 'complete'])
```

If you attempt to `@include` a view which does not exist, Laravel will throw an error. If you would like to include a view that may or may not be present, you should use the `@includeIf` directive:

```blade
@includeIf('view.name', ['status' => 'complete'])
```

If you would like to `@include` a view if a given boolean expression evaluates to `true` or `false`, you may use the `@includeWhen` and `@includeUnless` directives:

```blade
@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

To include the first view that exists from a given array of views, you may use the `includeFirst` directive:

```blade
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

If you would like to include a view without inheriting any variables from the parent view, you may use the `@includeIsolated` directive. The included view will only have access to variables you explicitly pass:

```blade
@includeIsolated('view.name', ['user' => $user])
```

You should avoid using the `__DIR__` and `__FILE__` constants in your Blade views, since they will refer to the location of the cached, compiled view.

#### [Rendering Views for Collections](#rendering-views-for-collections)

You may combine loops and includes into one line with Blade's `@each` directive:

```blade
@each('view.name', $jobs, 'job')
```

The `@each` directive's first argument is the view to render for each element in the array or collection. The second argument is the array or collection you wish to iterate over, while the third argument is the variable name that will be assigned to the current iteration within the view. So, for example, if you are iterating over an array of `jobs`, typically you will want to access each job as a `job` variable within the view. The array key for the current iteration will be available as the `key` variable within the view.

You may also pass a fourth argument to the `@each` directive. This argument determines the view that will be rendered if the given array is empty.

```blade
@each('view.name', $jobs, 'job', 'view.empty')
```

Views rendered via `@each` do not inherit the variables from the parent view. If the child view requires these variables, you should use the `@foreach` and `@include` directives instead.

### [The `@once` Directive](#the-once-directive)

The `@once` directive allows you to define a portion of the template that will only be evaluated once per rendering cycle. This may be useful for pushing a given piece of JavaScript into the page's header using [stacks](#stacks). For example, if you are rendering a given [component](#components) within a loop, you may wish to only push the JavaScript to the header the first time the component is rendered:

```blade
@once
    @push('scripts')
        <script>
            // Your custom JavaScript...
        </script>
    @endpush
@endonce
```

Since the `@once` directive is often used in conjunction with the `@push` or `@prepend` directives, the `@pushOnce` and `@prependOnce` directives are available for your convenience:

```blade
@pushOnce('scripts')
    <script>
        // Your custom JavaScript...
    </script>
@endPushOnce
```

If you are pushing duplicate content from two separate Blade templates, you should provide a unique identifier as the second argument to the `@pushOnce` directive to ensure the content is only rendered once:

```blade
<!-- pie-chart.blade.php -->
@pushOnce('scripts', 'chart.js')
    <script src="/chart.js"></script>
@endPushOnce

<!-- line-chart.blade.php -->
@pushOnce('scripts', 'chart.js')
    <script src="/chart.js"></script>
@endPushOnce
```

### [Raw PHP](#raw-php)

In some situations, it's useful to embed PHP code into your views. You can use the Blade `@php` directive to execute a block of plain PHP within your template:

```blade
@php
    $counter = 1;
@endphp
```

Or, if you only need to use PHP to import a class, you may use the `@use` directive:

```blade
@use('App\Models\Flight')
```

A second argument may be provided to the `@use` directive to alias the imported class:

```blade
@use('App\Models\Flight', 'FlightModel')
```

If you have multiple classes within the same namespace, you may group the imports of those classes:

```blade
@use('App\Models\{Flight, Airport}')
```

The `@use` directive also supports importing PHP functions and constants by prefixing the import path with the `function` or `const` modifiers:

```blade
@use(function App\Helpers\format_currency)
@use(const App\Constants\MAX_ATTEMPTS)
```

Just like class imports, aliases are supported for functions and constants as well:

```blade
@use(function App\Helpers\format_currency, 'formatMoney')
@use(const App\Constants\MAX_ATTEMPTS, 'MAX_TRIES')
```

Grouped imports are also supported with both function and const modifiers, allowing you to import multiple symbols from the same namespace in a single directive:

```blade
@use(function App\Helpers\{format_currency, format_date})
@use(const App\Constants\{MAX_ATTEMPTS, DEFAULT_TIMEOUT})
```

### [Comments](#comments)

Blade also allows you to define comments in your views. However, unlike HTML comments, Blade comments are not included in the HTML returned by your application:

```blade
{{-- This comment will not be present in the rendered HTML --}}
```

## [Components](#components)

Components and slots provide similar benefits to sections, layouts, and includes; however, some may find the mental model of components and slots easier to understand. There are two approaches to writing components: class-based components and anonymous components.

To create a class-based component, you may use the `make:component` Artisan command. To illustrate how to use components, we will create a simple `Alert` component. The `make:component` command will place the component in the `app/View/Components` directory:

```bash
php artisan make:component Alert
```

The `make:component` command will also create a view template for the component. The view will be placed in the `resources/views/components` directory. When writing components for your own application, components are automatically discovered within the `app/View/Components` directory and `resources/views/components` directory, so no further component registration is typically required.

You may also create components within subdirectories:

```bash
php artisan make:component Forms/Input
```

The command above will create an `Input` component in the `app/View/Components/Forms` directory and the view will be placed in the `resources/views/components/forms` directory.

#### [Manually Registering Package Components](#manually-registering-package-components)

When writing components for your own application, components are automatically discovered within the `app/View/Components` directory and `resources/views/components` directory.

However, if you are building a package that utilizes Blade components, you will need to manually register your component class and its HTML tag alias. You should typically register your components in the `boot` method of your package's service provider:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::component('package-alias', Alert::class);
}
```

Once your component has been registered, it may be rendered using its tag alias:

```blade
<x-package-alert/>
```

Alternatively, you may use the `componentNamespace` method to autoload component classes by convention. For example, a `Nightshade` package might have `Calendar` and `ColorPicker` components that reside within the `Package\Views\Components` namespace:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

This will allow the usage of package components by their vendor namespace using the `package-name::` syntax:

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade will automatically detect the class that's linked to this component by pascal-casing the component name. Subdirectories are also supported using "dot" notation.

### [Rendering Components](#rendering-components)

To display a component, you may use a Blade component tag within one of your Blade templates. Blade component tags start with the string `x-` followed by the kebab case name of the component class:

```blade
<x-alert/>

<x-user-profile/>
```

If the component class is nested deeper within the `app/View/Components` directory, you may use the `.` character to indicate directory nesting. For example, if we assume a component is located at `app/View/Components/Inputs/Button.php`, we may render it like so:

```blade
<x-inputs.button/>
```

If you would like to conditionally render your component, you may define a `shouldRender` method on your component class. If the `shouldRender` method returns `false` the component will not be rendered:

```php
use Illuminate\Support\Str;

/**
 * Whether the component should be rendered
 */
public function shouldRender(): bool
{
    return Str::length($this->message) > 0;
}
```

### [Index Components](#index-components)

Sometimes components are part of a component group and you may wish to group the related components within a single directory. For example, imagine a "card" component with the following class structure:

```
App\Views\Components\Card\Card
App\Views\Components\Card\Header
App\Views\Components\Card\Body
```

Since the root `Card` component is nested within a `Card` directory, you might expect that you would need to render the component via `<x-card.card>`. However, when a component's file name matches the name of the component's directory, Laravel automatically assumes that component is the "root" component and allows you to render the component without repeating the directory name:

```blade
<x-card>
    <x-card.header>...</x-card.header>
    <x-card.body>...</x-card.body>
</x-card>
```

### [Passing Data to Components](#passing-data-to-components)

You may pass data to Blade components using HTML attributes. Hard-coded, primitive values may be passed to the component using simple HTML attribute strings. PHP expressions and variables should be passed to the component via attributes that use the `:` character as a prefix:

```blade
<x-alert type="error" :message="$message"/>
```

You should define all of the component's data attributes in its class constructor. All public properties on a component will automatically be made available to the component's view. It is not necessary to pass the data to the view from the component's `render` method:

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;
use Illuminate\View\View;

class Alert extends Component
{
    /**
     * Create the component instance.
     */
    public function __construct(
        public string $type,
        public string $message,
    ) {}

    /**
     * Get the view / contents that represent the component.
     */
    public function render(): View
    {
        return view('components.alert');
    }
}
```

When your component is rendered, you may display the contents of your component's public variables by echoing the variables by name:

```blade
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

#### [Casing](#casing)

Component constructor arguments should be specified using `camelCase`, while `kebab-case` should be used when referencing the argument names in your HTML attributes. For example, given the following component constructor:

```php
/**
 * Create the component instance.
 */
public function __construct(
    public string $alertType,
) {}
```

The `$alertType` argument may be provided to the component like so:

```blade
<x-alert alert-type="danger" />
```

#### [Short Attribute Syntax](#short-attribute-syntax)

When passing attributes to components, you may also use a "short attribute" syntax. This is often convenient since attribute names frequently match the variable names they correspond to:

```blade
{{-- Short attribute syntax... --}}
<x-profile :$userId :$name />

{{-- Is equivalent to... --}}
<x-profile :user-id="$userId" :name="$name" />
```

#### [Escaping Attribute Rendering](#escaping-attribute-rendering)

Since some JavaScript frameworks such as Alpine.js also use colon-prefixed attributes, you may use a double colon (`::`) prefix to inform Blade that the attribute is not a PHP expression. For example, given the following component:

```blade
<x-button ::class="{ danger: isDeleting }">
    Submit
</x-button>
```

The following HTML will be rendered by Blade:

```blade
<button :class="{ danger: isDeleting }">
    Submit
</button>
```

#### [Component Methods](#component-methods)

In addition to public variables being available to your component template, any public methods on the component may be invoked. For example, imagine a component that has an `isSelected` method:

```php
/**
 * Determine if the given option is the currently selected option.
 */
public function isSelected(string $option): bool
{
    return $option === $this->selected;
}
```

You may execute this method from your component template by invoking the variable matching the name of the method:

```blade
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

#### [Accessing Attributes and Slots Within Component Classes](#using-attributes-slots-within-component-class)

Blade components also allow you to access the component name, attributes, and slot inside the class's render method. However, in order to access this data, you should return a closure from your component's `render` method:

```php
use Closure;

/**
 * Get the view / contents that represent the component.
 */
public function render(): Closure
{
    return function () {
        return '<div {{ $attributes }}>Components content</div>';
    };
}
```

The closure returned by your component's `render` method may also receive a `$data` array as its only argument. This array will contain several elements that provide information about the component:

```php
return function (array $data) {
    // $data['componentName'];
    // $data['attributes'];
    // $data['slot'];

    return '<div {{ $attributes }}>Components content</div>';
}
```

The elements in the `$data` array should never be directly embedded into the Blade string returned by your `render` method, as doing so could allow remote code execution via malicious attribute content.

The `componentName` is equal to the name used in the HTML tag after the `x-` prefix. So `<x-alert />`'s `componentName` will be `alert`. The `attributes` element will contain all of the attributes that were present on the HTML tag. The `slot` element is an `Illuminate\Support\HtmlString` instance with the contents of the component's slot.

The closure should return a string. If the returned string corresponds to an existing view, that view will be rendered; otherwise, the returned string will be evaluated as an inline Blade view.

#### [Additional Dependencies](#additional-dependencies)

If your component requires dependencies from Laravel's [[03-architecture-concepts/02-service-container.md|service container]], you may list them before any of the component's data attributes and they will automatically be injected by the container:

```php
use App\Services\AlertCreator;

/**
 * Create the component instance.
 */
public function __construct(
    public AlertCreator $creator,
    public string $type,
    public string $message,
) {}
```

#### [Hiding Attributes / Methods](#hiding-attributes-and-methods)

If you would like to prevent some public methods or properties from being exposed as variables to your component template, you may add them to an `$except` array property on your component:

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    /**
     * The properties / methods that should not be exposed to the component template.
     *
     * @var array
     */
    protected $except = ['type'];

    /**
     * Create the component instance.
     */
    public function __construct(
        public string $type,
    ) {}
}
```

### [Component Attributes](#component-attributes)

We've already examined how to pass data attributes to a component; however, sometimes you may need to specify additional HTML attributes, such as `class`, that are not part of the data required for a component to function. Typically, you want to pass these additional attributes down to the root element of the component template. For example, imagine we want to render an `alert` component like so:

```blade
<x-alert type="error" :message="$message" class="mt-4"/>
```

All of the attributes that are not part of the component's constructor will automatically be added to the component's "attribute bag". This attribute bag is automatically made available to the component via the `$attributes` variable. All of the attributes may be rendered within the component by echoing this variable:

```blade
<div {{ $attributes }}>
    <!-- Component content -->
</div>
```

Using directives such as `@env` within component tags is not supported at this time. For example, `<x-alert :live="@env('production')"/>` will not be compiled.

#### [Default / Merged Attributes](#default-merged-attributes)

Sometimes you may need to specify default values for attributes or merge additional values into some of the component's attributes. To accomplish this, you may use the attribute bag's `merge` method. This method is particularly useful for defining a set of default CSS classes that should always be applied to a component:

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

If we assume this component is utilized like so:

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

The final, rendered HTML of the component will appear like the following:

```blade
<div class="alert alert-error mb-4">
    <!-- Contents of the $message variable -->
</div>
```

#### [Conditionally Merge Classes](#conditionally-merge-classes)

Sometimes you may want to merge additional CSS classes if a given condition is true. You may accomplish this by chaining the `class` method onto the attribute bag's `merge` method:

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type])->class(['bg-white' => $isDarkMode]) }}>
    {{ $message }}
</div>
```

#### [Non-Class Attribute Differences](#non-class-attribute-differences)

When merging attributes that are not class attributes, if you want to merge the attribute values as opposed to overwriting the existing value, you may use the `merge` method's `prepend` or `append` modifiers:

```blade
<x-alert type="error" :message="$message" data-initial-value="{{ $message }}"/>
```

```blade
<div {{ $attributes->merge(['data-initial-value' => $message], prepend: 'data-') }}>
    {{ $message }}
</div>
```

#### [Attributes Without Values](#attributes-without-values)

Some attributes may not require an associated value, such as `disabled`. Blade passes a boolean variable for these attributes. To conditionally include such an attribute based on a boolean value, you may use the `attribute` method:

```blade
<div {{ $attributes->attribute(':disabled', $isDisabled) }}>
    {{ $message }}
</div>
```

#### [Retrieving Attributes](#retrieving-attributes)

If needed, you may retrieve the entire attribute bag as a raw PHP array using the `attributes->all()` method. This method is particularly useful when building auxiliary HTML extensions:

```php
public function render(): View
{
    return view('components.alert', [
        'attributes' => $this->attributes->all(),
    ]);
}
```

#### [Reserved Keywords](#reserved-keywords)

By default, public methods and properties on your component are exposed to the view. However, you may explicitly exclude properties and methods from being exposed as data to the component view by declaring them on the `$except` property of your component:

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    public string $type = 'info';

    protected array $except = [
        'type',
    ];

    public function __construct(
        public string $message,
    ) {}
}
```

### [Slots](#slots)

You will often need to pass additional content to your component via "slots". Component slots are rendered by echoing the `$slot` variable:

```blade
<div class="alert">
    {{ $slot }}
</div>
```

So, assuming our component is defined like above, you may pass content to the component like so:

```blade
<x-alert>
    <p>This is an alert message!</p>
</x-alert>
```

If you want to define multiple slots for a component, you may define a named slot. Any other content that is not within a named slot will be passed to the `$slot` variable:

```blade
<x-alert>
    <x-slot:title>
        {{ $title }}
    </x-slot>

    <p>{{ $message }}</p>
</x-alert>
```

You may display the content of a named slot by echoing the slot's name as a variable:

```blade
<div class="alert">
    <div class="title">
        {{ $title }}
    </div>

    {{ $slot }}
</div>
```

#### [Scoped Slots](#scoped-slots)

If you have used a JavaScript framework such as Vue, you may be familiar with "scoped slots", which allow you to access data from the component within a slot. Likewise, you may pass data to a slot from your component by using the `$attributes` bag:

```blade
<x-alert>
    <x-slot:title slot-name="title">
        {{ $title }} <span class="text-light">&copy; 2025</span>
    </x-slot>

    {{ $slot }}
</x-alert>
```

```blade
<div class="alert">
    <div class="title">
        {{ $title }} - {{ $attributes->get('slot-name') }}
    </div>

    {{ $slot }}
</div>
```

### [Inline Component Views](#inline-component-views)

For very small components, it may feel cumbersome to move the component class and its corresponding view to different files. For these situations, Laravel allows you to return the view directly from the component's `render` method:

```php
use Illuminate\View\Component;

class Alert extends Component
{
    public function __construct(
        public string $type,
        public string $message,
    ) {}

    public function render(): View
    {
        return <<<'blade'
<div class='alert alert-{{ $type }}'>
    {{ $message }}
</div>
blade;
    }
}
```

### [Dynamic Components](#dynamic-components)

Sometimes you may need to render a component but not know which component should be rendered until runtime. In this situation, you may use the `dynamic-component` method to render the component based on a variable that contains the component's class name:

```php
use Illuminate\Support\Facades\Blade;

$component = $type === 'error' ? Alert::class : Notice::class;

// ...

return Blade::dynamicComponent($component)->render();
```

Or, in your Blade template:

```blade
{{-- For a variable containing the component class --}}
<x-dynamic-component :component="$component" type="error" :message="$message"/>
```

### [Manually Registering Components](#manually-registering-components)

To manually register components for your application for a given namespace, use the `component` method. This is typically done in the `boot` method of your `AppServiceProvider`:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Blade::component('components.alert', Alert::class);
    Blade::component('inputs.field', Field::class);
}
```

When a component is manually registered, the string passed as the first argument will serve as the HTML tag alias. So, for the example above, you may render the component like so:

```blade
<x-alert type="error" :message="$message"/>

<x-field type="text" name="title"/>
```

## [Anonymous Components](#anonymous-components)

Similar to class-based components, anonymous components offer the same benefits; however, they are defined using a single file that contains only the Blade template and its PHP logic.

Anonymous components are stored in the `resources/views/components` directory. To create an anonymous component, create a new Blade template inside the `resources/views/components` directory:

```bash
resources/views/components/alert.blade.php
```

You may also place anonymous components within subdirectories within the `resources/views/components` directory:

```bash
resources/views/components/inputs/field.blade.php
```

### [Anonymous Index Components](#anonymous-index-components)

Sometimes a component may consist of many smaller partials that are grouped together in a single directory. A component that corresponds to a directory may be rendered using a single "index" component. For example, a component at `resources/views/components/card/index.blade.php` may be rendered like so:

```blade
<x-card>
    <x-card.header>...</x-card.header>
    <x-card.body>...</x-card.body>
</x-card>
```

However, the component at the given path may also receive data from the parent component. To learn more about this feature, please consult the documentation on [rendering components](#rendering-components) and [component attributes](#component-attributes).

### [Data Properties / Attributes](#data-properties-attributes)

Since there is no backing class for anonymous components, you may use the `@props` Blade directive to declare which attributes should be treated as data properties:

```blade
@props(['type', 'message'])

<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

By default, all public properties defined on an anonymous component will be automatically added to the props array. However, you may explicitly declare which PHP types the props should have:

```blade
@props(['type' => 'string', 'message' => 'string'])

<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

You may also use PHP union types or other PHP type declarations:

```blade
@props(['user' => \App\Models\User|null, 'isActive' => 'bool'])
```

#### [Accessing Parent Data](#accessing-parent-data)

In some cases, you may want to access data from the parent component (the component that is rendering the current component). You may access this data using the `$attributes` attribute bag's `get` method:

```blade
@foreach ($attributes->get('users') as $user)
    {{ $user }}
@endforeach
```

### [Anonymous Components Paths](#anonymous-component-paths)

If you would like to change the directory where anonymous components are stored, you may do so from the `boot` method in `AppServiceProvider`:

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Blade::anonymousComponentNamespace(resource_path('views/components'), 'components');
}
```

## [Building Layouts](#building-layouts)

### [Layouts Using Components](#layouts-using-components)

Laravel makes it easy to build component-driven layouts. Using Laravel's "component" approach to layouts, you may use a Blade component as the layout container for your page.

#### [Defining the Layout Component](#defining-the-layout-component)

First, let's define a layout component. This component will serve as the parent component for our page's layout. We'll use slots to inject content into the layout:

```blade
{{-- resources/views/components/layout.blade.php --}}

<html>
    <head>
        <title>{{ $title }}</title>
    </head>

    <body>
        <header>
            {{ $header }}
        </header>

        <main>
            {{ $slot }}
        </main>
    </body>
</html>
```

#### [Defining the Child Component](#defining-the-child-component)

Now that we have defined a layout component using slots, we can now build a page component that uses this layout:

```blade
{{-- resources/views/post/create.blade.php --}}

<x-layout>
    <x-slot:title>
        Create Post
    </x-slot>

    <form method="POST" action="/posts">
        <!-- Form Contents -->
    </form>
</x-layout>
```

This example demonstrates the flexibility of both components and slots; however, if desired, you can also pass an array of attributes to a layout to be treated as default attributes. To learn more about passing default attributes to layouts, please consult the documentation on [component attributes](#component-attributes).

### [Layouts Using Template Inheritance](#layouts-using-template-inheritance)

Before components were introduced, Blade has also allowed you to define layouts via "template inheritance". This works by declaring a parent Blade template, and then using the `@extends` directive to inherit from that template into a child view. While this technique is still supported, using [components](#components) provides similar functionality and is generally considered the preferred approach.

#### [Defining a Layout Template](#defining-a-layout-template)

For this example, we will use the `@yield` directive to define the content sections of our layout template. Typically, this layout will be stored in the `resources/views/layouts` directory:

```blade
{{-- resources/views/layouts/app.blade.php --}}

<!DOCTYPE html>
<html>
    <head>
        <title>@yield('title')</title>
    </head>

    <body>
        @yield('content')
    </body>
</html>
```

#### [Extending the Layout](#extending-the-layout)

When using template inheritance, child views use the `@extends` directive to specify which layout they are inheriting from. Child views may then inject content into the layout using the `@section` directive. As seen in this example, the `@section` directive's content will be injected into the `@yield('content')` call in our layout:

```blade
{{-- resources/views/post/create.blade.php --}}

@extends('layouts.app')

@section('title', 'Create Post')

@section('content')
<form method="POST" action="/posts">
    <!-- Form Contents -->
</form>
@endsection
```

## [Forms](#forms)

In addition to templating and layout features, Blade provides useful shortcuts for common tasks when building forms, such as rendering CSRF field inputs, method field inputs, and validation error display.

### [CSRF Field](#csrf-field)

Laravel applications include the web middlewareGroup that will automatically verify that the content of the request matches the token stored in the session. This middleware is automatically added to the `web` middlewareGroup in your application's `bootstrap/app.php` file.

As part of automatically verifying that requests include a token in the session, Laravel provides the `@csrf` Blade directive for conveniently generating the token field:

```blade
<form method="POST" action="/profile">
    @csrf

    <!-- ... -->
</form>
```

### [Method Field](#method-field)

HTML forms do not support `PUT`, `PATCH`, or `DELETE` actions. So, when defining `PUT`, `PATCH`, or `DELETE` routes that are called from an HTML form, you may add a `_method` field to the form to spoof the HTTP verb.

Laravel provides the `@method` Blade directive to generate this field:

```blade
<form method="POST" action="/profile">
    @method('PATCH')

    <!-- ... -->
</form>
```

### [Validation Errors](#validation-errors)

You may use the `@error` directive to quickly check if [[04-the-basics/12-validation.md|validation error messages]] exist for a given attribute. Within an `@error` directive, you may echo the `$message` variable to display the error message:

```blade
<label for="title">Post Title</label>

<input
    id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror"
/>

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

## [Stacks](#stacks)

Blade allows you to push content to named stacks which can be rendered somewhere else in your layout. This is particularly useful for specifying JavaScript libraries required by your child views:

```blade
@push('scripts')
    <script>
        // Your custom JavaScript...
    </script>
@endpush
```

You may push to a stack multiple times. To push content to the beginning of a stack, use the `@prepend` directive:

```blade
@prepend('scripts')
    <script>
        // Your custom JavaScript...
    </script>
@endprepend
```

You can render the stack using the `@stack` directive:

```blade
<head>
    @stack('scripts')
</head>
```

## [Service Injection](#service-injection)

The `@inject` directive may be used to retrieve a service from the Laravel [[03-architecture-concepts/02-service-container.md|service container]]. The first argument passed to `@inject` is the name of the variable the service will be placed in, while the second argument is the class or interface name of the service you wish to resolve:

```blade
@inject('app\Http\Services\Analytics', 'App\Services\Analytics')
<div>
    Monthly Revenue: ${{ $analytics->getRevenue() }}
</div>
```

## [Rendering Inline Blade Templates](#rendering-inline-blade-templates)

Sometimes you may need to render a raw string of Blade template code from the database. To accomplish this, you may use the `Blade::render` method provided by the `Blade` facade:

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian']);
```

If needed, you may also provide an environment for the Blade renderer using the third argument to the `render` method:

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian'],
    true // preserve quotes...
);
```

## [Rendering Blade Fragments](#rendering-blade-fragments)

When building components, you may sometimes find it useful to define a smaller Blade fragment, or "fragment", and render it from the parent component. To accomplish this, you may use the `$fragment` variable available in components:

```blade
{{ $fragment('alert', ['type' => 'error', 'message' => $message]) }}
```

The first argument is the name of the fragment (the path to the view under `resources/views/components`) and the second argument is an array of data that should be passed to the fragment:

```blade
{{-- resources/views/components/alert.blade.php --}}

<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

## [Extending Blade](#extending-blade)

### [Custom Echo Handlers](#custom-echo-handlers)

If you attempt to echo an object using Blade's double curly syntax, the object's `__toString` method will be called. However, you may implement a custom `ToHtmlInterface` to specify the HTML that should be generated when the object is echoed:

```php
namespace App\Markdown;

use Illuminate\Contracts\Support\Htmlable;
use Stringable;

class Markdown implements Htmlable, Stringable
{
    /**
     * Get the HTML content of the object.
     */
    public function toHtml(): string
    {
        return htmlspecialchars($this->__toString(), ENT_QUOTES, 'UTF-8');
    }

    /**
     * Convert the object to its string representation.
     */
    public function __toString(): string
    {
        return "<p>Rendering...</p>";
    }
}
```

### [Custom If Statements](#custom-if-statements)

When compiling Blade templates, the directive `@if(...)` is an abstraction that checks the expression passed to it as a PHP variable and echoes the contents of the block if the variable evaluates to `true`. However, sometimes it can be cumbersome to define the same check repeatedly.

Blade allows you to define custom conditional directives via the ` Blade::if` method. For example, let's define a conditional that checks for an application environment:

```php
use Illuminate\Support\Facades\Blade;

/*
 * Bootstrap any application services.
 */
public function boot(): void
{
    Blade::if('env', function ($environment) {
        return app()->environment($environment);
    });
}
```

Now you may use your custom conditional directive within your Blade templates:

```blade
@env('production')
    <!-- Production specific content... -->
@endenv

@env(['production', 'staging'])
    <!-- ... -->
@endenv
```