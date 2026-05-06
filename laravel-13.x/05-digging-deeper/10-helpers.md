---
title: Helpers
description: Laravel Helpers documentation - global helper PHP functions included with Laravel
url: https://laravel.com/docs/13.x/helpers
tags: [logic]
cssclasses:
  - logic
  - color-orange
color: orange
---

# Helpers

- [Introduction](#introduction)
- [Available Methods](#available-methods)
- [Other Utilities](#other-utilities)

## Introduction

Laravel includes a variety of global "helper" PHP functions. Many of these functions are used by the framework itself; however, you are free to use them in your own applications if you find them convenient.

## Available Methods

### Arrays & Objects

- [`Arr::accessible()`](#arraccessible) - Determine if the given value is array accessible
- [`Arr::add()`](#arradd) - Add a key/value pair to an array if it doesn't exist
- [`Arr::array()`](#arrarray) - Retrieve value, throwing if not an array
- [`Arr::boolean()`](#arrboolean) - Retrieve value, throwing if not a boolean
- [`Arr::collapse()`](#arrcollapse) - Collapse an array of arrays into a single array
- [`Arr::crossJoin()`](#arrcrossjoin) - Cross join arrays returning Cartesian product
- [`Arr::divide()`](#arrdivide) - Return arrays containing keys and values
- [`Arr::dot()`](#arrdot) - Flatten multi-dimensional array using dot notation
- [`Arr::every()`](#arrevery) - Ensure all values pass a given truth test
- [`Arr::except()`](#arrexcept) - Remove key/value pairs from an array
- [`Arr::exists()`](#arrexists) - Check if key exists in array
- [`Arr::first()`](#arrfirst) - Return first element passing truth test
- [`Arr::flatten()`](#arrflatten) - Flatten multi-dimensional array
- [`Arr::float()`](#arrfloat) - Retrieve value, throwing if not a float
- [`Arr::forget()`](#arrforget) - Remove key/value pairs using dot notation
- [`Arr::get()`](#arrget) - Retrieve value using dot notation
- [`Arr::has()`](#arrhas) - Check if item exists using dot notation
- [`Arr::hasAll()`](#arrhasall) - Check if all keys exist
- [`Arr::hasAny()`](#arrhasany) - Check if any key exists
- [`Arr::isAssoc()`](#arrisassoc) - Check if array is associative
- [`Arr::isList()`](#arrislist) - Check if array keys are sequential integers
- [`Arr::join()`](#arrjoin) - Join array elements with a string
- [`Arr::keyBy()`](#arrkeyby) - Key array by given key
- [`Arr::last()`](#arrlast) - Return last element passing truth test
- [`Arr::map()`](#arrmap) - Iterate and modify array values
- [`Arr::only()`](#arronly) - Return only specified key/value pairs
- [`Arr::pluck()`](#arrpluck) - Retrieve all values for given key
- [`Arr::prepend()`](#arrprepend) - Push item to beginning of array
- [`Arr::pull()`](#arrpull) - Return and remove key/value pair
- [`Arr::push()`](#arrpush) - Push item into array using dot notation
- [`Arr::random()`](#arrrandom) - Return random value from array
- [`Arr::reject()`](#arrreject) - Remove items using closure
- [`Arr::set()`](#arrset) - Set value using dot notation
- [`Arr::shuffle()`](#arrshuffle) - Randomly shuffle array
- [`Arr::some()`](#arrsome) - Check if any value passes truth test
- [`Arr::sort()`](#arrsort) - Sort array by values
- [`Arr::toCssClasses()`](#arrtocssclasses) - Compile CSS class string conditionally
- [`Arr::toCssStyles()`](#arrtocssstyles) - Compile CSS style string conditionally
- [`Arr::undot()`](#arrundot) - Expand dot notation to multi-dimensional array
- [`Arr::where()`](#arrwhere) - Filter array using closure
- [`Arr::wrap()`](#arrwrap) - Wrap value in array

### Paths

- [`app_path()`](#apppath) - Get the path to the app directory
- [`base_path()`](#basepath) - Get the path to the base directory
- [`config_path()`](#configpath) - Get the path to the config directory
- [`database_path()`](#databasepath) - Get the path to the database directory
- [`lang_path()`](#langpath) - Get the path to the lang directory
- [`public_path()`](#publicpath) - Get the path to the public directory
- [`resource_path()`](#resourcepath) - Get the path to the resources directory
- [`storage_path()`](#storagepath) - Get the path to the storage directory

### URLs

- [`action()`](#action) - Generate URL to controller action
- [`asset()`](#asset) - Generate URL to asset
- [`route()`](#route) - Generate URL for named route
- [`url()`](#url) - Get full URL

### Miscellaneous

- [`abort()`](#abort) - Throw HTTP exception
- [`app()`](#app) - Get container instance
- [`auth()`](#auth) - Get auth instance
- [`back()`](#back) - Generate redirect back URL
- [`bcrypt()`](#bcrypt) - Hash value using bcrypt
- [`cache()`](#cache) - Get/set cache values
- [`config()`](#config) - Get/set config values
- [`cookie()`](#cookie) - Create cookie instance
- [`csrf_token()`](#csrftoken) - Get CSRF token
- [`dd()`](#dd) - Dump and die
- [`dispatch()`](#dispatch) - Dispatch job
- [`encrypt()`](#encrypt) - Encrypt value
- [`env()`](#env) - Get environment variable
- [`event()`](#event) - Dispatch event
- [`filled()`](#filled) - Check if value is filled
- [`info()`](#info) - Log info message
- [`logger()`](#logger) - Log message
- [`now()`](#now) - Get current date/time
- [`old()`](#old) - Get old input value
- [`optional()`](#optional) - Get value or null
- [`redirect()`](#redirect) - Create redirect response
- [`report()`](#report) - Report exception
- [`request()`](#request) - Get request instance
- [`rescue()`](#rescue) - Rescue closure
- [`resolve()`](#resolve) - Resolve from container
- [`response()`](#response) - Create response
- [`retry()`](#retry) - Retry closure
- [`session()`](#session) - Get/set session
- [`tap()`](#tap) - Tap and return
- [`throw_if()`](#throwif) - Throw if condition
- [`today()`](#today) - Get today's date
- [`transform()`](#transform) - Transform value
- [`validator()`](#validator) - Create validator
- [`value()`](#value) - Get value or return callable
- [`view()`](#view) - Create view response
- [`with()`](#with) - Return value

## Arrays & Objects

### `Arr::accessible()`

```php
use Illuminate\Support\Arr;
use Illuminate\Support\Collection;

$isAccessible = Arr::accessible(['a' => 1, 'b' => 2]);
// true

$isAccessible = Arr::accessible(new Collection);
// true

$isAccessible = Arr::accessible('abc');
// false
```

### `Arr::add()`

```php
use Illuminate\Support\Arr;

$array = Arr::add(['name' => 'Desk'], 'price', 100);
// ['name' => 'Desk', 'price' => 100]
```

### `Arr::get()`

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$price = Arr::get($array, 'products.desk.price');
// 100

$discount = Arr::get($array, 'products.desk.discount', 0);
// 0
```

### `Arr::has()`

```php
use Illuminate\Support\Arr;

$array = ['product' => ['name' => 'Desk', 'price' => 100]];

$contains = Arr::has($array, 'product.name');
// true
```

### `Arr::pluck()`

```php
use Illuminate\Support\Arr;

$array = [
    ['developer' => ['id' => 1, 'name' => 'Taylor']],
    ['developer' => ['id' => 2, 'name' => 'Abigail']],
];

$names = Arr::pluck($array, 'developer.name');
// ['Taylor', 'Abigail']
```

### `Arr::random()`

```php
use Illuminate\Support\Arr;

$array = [1, 2, 3, 4, 5];

$random = Arr::random($array);
// 4 - (retrieved randomly)

$items = Arr::random($array, 2);
// [2, 5] - (retrieved randomly)
```

### `Arr::toCssClasses()`

```php
use Illuminate\Support\Arr;

$isActive = false;
$hasError = true;

$array = ['p-4', 'font-bold' => $isActive, 'bg-red' => $hasError];

$classes = Arr::toCssClasses($array);
// 'p-4 bg-red'
```

## Paths

### `app_path()`

```php
$path = app_path();

// /home/laravel/app
```

### `base_path()`

```php
$path = base_path();

// /home/laravel
```

### `config_path()`

```php
$path = config_path();

// /home/laravel/config
```

### `public_path()`

```php
$path = public_path();

// /home/laravel/public
```

### `storage_path()`

```php
$path = storage_path();

// /home/laravel/storage
```

## URLs

### `action()`

```php
$url = action('HomeController@index');
$url = action(['HomeController@index', 'slug' => 'about']);
```

### `asset()`

```php
$url = asset('js/app.js');
// http://example.com/js/app.js
```

### `route()`

```php
$url = route('profile');
$url = route('profile', ['username' => 'john']);
```

### `url()`

```php
$url = url('posts');
$url = url('posts', ['page' => 2]);
$url = url(); // full URL for current page
```

## Miscellaneous

### `app()`

```php
$container = app();

// Get service from container
$translator = app('translator');
$translator = app(\Illuminate\Contracts\Translation\Translator::class);
```

### `auth()`

```php
$user = auth()->user();

// Check authentication
if (auth()->check()) { /* ... */ }

// Use guard
$user = auth('admin')->user();
```

### `back()`

```php
return back();
return back()->withInput();
```

### `bcrypt()`

```php
$hash = bcrypt('my-password');
// $2y$10$...
```

### `cache()`

```php
cache(['key' => 'value'], 600);
// Set value for 10 minutes

$value = cache('key');
// Get value

cache()->put('key', 'value', 600);
```

### `config()`

```php
$value = config('app.timezone');
$value = config('app.timezone', 'UTC');

config(['app.debug' => true]);
```

### `dd()`

```php
dd($value);
dd($value1, $value2);
```

### `dispatch()`

```php
dispatch(new ProcessPodcast($podcast));
```

### `encrypt()`

```php
$encrypted = encrypt('hidden message');
```

### `env()`

```php
$debug = env('APP_DEBUG', false);
```

### `now()`

```php
$now = now();
$tomorrow = now()->addDay();
```

### `old()`

```php
$name = old('name');
$name = old('name', 'Default');
```

### `optional()`

```php
$name = optional($user)->name;
// null if $user is null
```

### `redirect()`

```php
return redirect('/home');
return redirect()->route('profile');
```

### `report()`

```php
report(new Exception('Something went wrong'));
```

### `request()`

```php
$request = request();
$name = request('name');
$name = request()->input('name');
```

### `rescue()`

```php
$result = rescue(function () {
    return $this->process();
});
```

### `retry()`

```php
return retry(5, function () {
    // attempt 5 times
}, 100);
```

### `session()`

```php
session(['key' => 'value']);
$value = session('key');
```

### `tap()`

```php
$data = tap($user, function ($user) {
    $user->name = 'John';
});
// returns $user
```

### `throw_if()`

```php
throw_if($condition, new Exception('Error'));
throw_if($condition, 'Error message');
```

### `today()`

```php
$today = today();
$tomorrow = today()->addDay();
```

### `transform()`

```php
$value = transform(null, function () {
    return 'default';
});
// 'default'

$value = transform(5, function ($value) {
    return $value * 2;
});
// 10
```

### `validator()`

```php
$validator = validator($data, $rules);
```

### `value()`

```php
$value = value(function () {
    return now()->toDateTimeString();
});
```

### `view()`

```php
return view('home');
return view('home', ['name' => 'John']);
```

### `with()`

```php
$value = with($object, function ($object) {
    return $object->getName();
});
```

## Other Utilities

### Benchmarking

```php
use Illuminate\Support\Benchmark;

Benchmark::dd(fn () => Post::all()); // Display execution time
Benchmark::dd(fn () => Post::all(), 1); // Run 1 iteration
Benchmark::measure(fn () => Post::all()); // Return duration in milliseconds
```

### Dates and Time

Laravel provides Carbon date handling through the `now()` and `today()` helpers.

### Deferred Functions

```php
// Functions are lazy loaded until actually needed
// See documentation for specific use cases
```

### Lottery

```php
use Illuminate\Support\Lottery;

// Check if won (1 in 1000 chance)
if (Lottery::odds(1, 1000)->win()) { /* ... */ }
```

### Pipeline

```php
use Illuminate\Support\Pipeline;

$result = $result
    ->pipe(through(new StripTags))
    ->pipe(through(new ConvertToString));
```

### Sleep

```php
use function Illuminate\Support\sleep;

sleep(1); // Sleep for 1 second
sleep(1000); // Sleep for 1000 milliseconds
```

### Timebox

```php
use Illuminate\Support\Timebox;

$result = timebox(fn () => expensiveOperation(), 100);
```

### URI

```php
use Illuminate\Support\Facades\URI;

$uri = URI::of('/api/v1/users')
    ->withQuery(['page' => 1])
    ->__toString();
// /api/v1/users?page=1
```