---
title: HTTP Tests
description: Testing HTTP requests and responses in Laravel applications
url: https://laravel.com/docs/13.x/http-tests
tags: [testing]
cssclasses:
  - ai
  - color-purple
color: purple
---

# HTTP Tests

- [Introduction](#introduction)
- [Making Requests](#making-requests)
    - [Customizing Request Headers](#customizing-request-headers)
    - [Cookies](#cookies)
    - [Session / Authentication](#session--authentication)
    - [Debugging Responses](#debugging-responses)
    - [Exception Handling](#exception-handling)
- [Testing JSON APIs](#testing-json-apis)
    - [Fluent JSON Testing](#fluent-json-testing)
- [Testing File Uploads](#testing-file-uploads)
- [Testing Views](#testing-views)
    - [Rendering Blade and Components](#rendering-blade-and-components)
- [Caching Routes](#caching-routes)
- [Available Assertions](#available-assertions)

## Introduction

Laravel provides a very fluent API for making HTTP requests to your application and examining the responses. For example, take a look at the feature test defined below:

```php
test('the application returns a successful response', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
});
```

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_the_application_returns_a_successful_response(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

The `get` method makes a `GET` request into the application, while the `assertStatus` method asserts that the returned response should have the given HTTP status code. In addition to this simple assertion, Laravel also contains a variety of assertions for inspecting the response headers, content, JSON structure, and more.

## Making Requests

To make a request to your application, you may invoke the `get`, `post`, `put`, `patch`, or `delete` methods within your test. These methods do not actually issue a "real" HTTP request to your application. Instead, the entire network request is simulated internally.

Instead of returning an `Illuminate\Http\Response` instance, test request methods return an instance of `Illuminate\Testing\TestResponse`, which provides a variety of helpful assertions that allow you to inspect your application's responses:

```php
test('basic request', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
});
```

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_a_basic_request(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

In general, each of your tests should only make one request to your application. Unexpected behavior may occur if multiple requests are executed within a single test method.

For convenience, the CSRF middleware is automatically disabled when running tests.

### Customizing Request Headers

You may use the `withHeaders` method to customize the request's headers before it is sent to the application:

```php
test('interacting with headers', function () {
    $response = $this->withHeaders([
        'X-Header' => 'Value',
    ])->post('/user', ['name' => 'Sally']);

    $response->assertStatus(201);
});
```

### Cookies

You may use the `withCookie` or `withCookies` methods to set cookie values before making a request:

```php
test('interacting with cookies', function () {
    $response = $this->withCookie('color', 'blue')->get('/');

    $response = $this->withCookies([
        'color' => 'blue',
        'name' => 'Taylor',
    ])->get('/');

    //
});
```

### Session / Authentication

Laravel provides several helpers for interacting with the session during HTTP testing. First, you may set the session data to a given array using the `withSession` method:

```php
test('interacting with the session', function () {
    $response = $this->withSession(['banned' => false])->get('/');

    //
});
```

Laravel's session is typically used to maintain state for the currently authenticated user. Therefore, the `actingAs` helper method provides a simple way to authenticate a given user as the current user:

```php
use App\Models\User;

test('an action that requires authentication', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->withSession(['banned' => false])
        ->get('/');

    //
});
```

```php
<?php

namespace Tests\Feature;

use App\Models\User;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_an_action_that_requires_authentication(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)
            ->withSession(['banned' => false])
            ->get('/');

        //
    }
}
```

You may also specify which guard should be used to authenticate the given user by passing the guard name as the second argument to the `actingAs` method:

```php
$this->actingAs($user, 'web');
```

If you would like to ensure the request is unauthenticated, you may use the `actingAsGuest` method:

```php
$this->actingAsGuest();
```

### Debugging Responses

After making a test request to your application, the `dump`, `dumpHeaders`, and `dumpSession` methods may be used to examine and debug the response contents:

```php
test('basic test', function () {
    $response = $this->get('/');

    $response->dump();
    $response->dumpHeaders();
    $response->dumpSession();
});
```

Alternatively, you may use the `dd`, `ddHeaders`, `ddBody`, `ddJson`, and `ddSession` methods to dump information about the response and then stop execution:

```php
test('basic test', function () {
    $response = $this->get('/');

    $response->dd();
    $response->ddHeaders();
    $response->ddBody();
    $response->ddJson();
    $response->ddSession();
});
```

### Exception Handling

Sometimes you may need to test that your application is throwing a specific exception. To accomplish this, you may "fake" the exception handler via the `Exceptions` facade:

```php
use App\Exceptions\InvalidOrderException;
use Illuminate\Support\Facades\Exceptions;

test('exception is thrown', function () {
    Exceptions::fake();

    $response = $this->get('/order/1');

    // Assert an exception was thrown...
    Exceptions::assertReported(InvalidOrderException::class);

    // Assert against the exception...
    Exceptions::assertReported(function (InvalidOrderException $e) {
        return $e->getMessage() === 'The order was invalid.';
    });
});
```

```php
<?php

namespace Tests\Feature;

use App\Exceptions\InvalidOrderException;
use Illuminate\Support\Facades\Exceptions;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_exception_is_thrown(): void
    {
        Exceptions::fake();

        $response = $this->get('/');

        // Assert an exception was thrown...
        Exceptions::assertReported(InvalidOrderException::class);

        // Assert against the exception...
        Exceptions::assertReported(function (InvalidOrderException $e) {
            return $e->getMessage() === 'The order was invalid.';
        });
    }
}
```

The `assertNotReported` and `assertNothingReported` methods may be used to assert that a given exception was not thrown during the request or that no exceptions were thrown:

```php
Exceptions::assertNotReported(InvalidOrderException::class);

Exceptions::assertNothingReported();
```

You may totally disable exception handling for a given request by invoking the `withoutExceptionHandling` method before making your request:

```php
$response = $this->withoutExceptionHandling()->get('/');
```

In addition, if you would like to ensure that your application is not utilizing features that have been deprecated, you may invoke the `withoutDeprecationHandling` method:

```php
$response = $this->withoutDeprecationHandling()->get('/');
```

The `assertThrows` method may be used to assert that code within a given closure throws an exception of the specified type:

```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    OrderInvalid::class
);
```

If you would like to inspect and make assertions against the exception that is thrown, you may provide a closure as the second argument to the `assertThrows` method:

```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    fn (OrderInvalid $e) => $e->orderId() === 123;
);
```

The `assertDoesntThrow` method may be used to assert that the code within a given closure does not throw any exceptions:

```php
$this->assertDoesntThrow(fn () => (new ProcessOrder)->execute());
```

## Testing JSON APIs

Laravel also provides several helpers for testing JSON APIs and their responses. For example, the `json`, `getJson`, `postJson`, `putJson`, `patchJson`, `deleteJson`, and `optionsJson` methods may be used to issue JSON requests with various HTTP verbs:

```php
test('making an api request', function () {
    $response = $this->postJson('/api/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertJson([
            'created' => true,
        ]);
});
```

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_making_an_api_request(): void
    {
        $response = $this->postJson('/api/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJson([
                'created' => true,
            ]);
    }
}
```

In addition, JSON response data may be accessed as array variables on the response:

```php
expect($response['created'])->toBeTrue();
```

```php
$this->assertTrue($response['created']);
```

The `assertJson` method converts the response to an array to verify that the given array exists within the JSON response returned by the application.

### Asserting Exact JSON Matches

If you would like to verify that a given array **exactly matches** the JSON returned by your application, you should use the `assertExactJson` method:

```php
test('asserting an exact json match', function () {
    $response = $this->postJson('/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertExactJson([
            'created' => true,
        ]);
});
```

### Asserting on JSON Paths

If you would like to verify that the JSON response contains the given data at a specified path, you should use the `assertJsonPath` method:

```php
test('asserting a json path value', function () {
    $response = $this->postJson('/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertJsonPath('team.owner.name', 'Darian');
});
```

The `assertJsonPath` method also accepts a closure:

```php
$response->assertJsonPath('team.owner.name', fn (string $name) => strlen($name) >= 3);
```

If you need to assert multiple JSON paths at once, you may use the `assertJsonPaths` method:

```php
$response->assertJsonPaths([
    'team.owner.name' => 'Darian',
    'team.owner.email' => fn (string $email) => str($email)->is('*@laravel.com'),
    'team.members.0.name' => 'Sally',
]);
```

You may use the `assertJsonMissingPaths` method to assert that multiple JSON paths are missing from the response:

```php
$response->assertJsonMissingPaths([
    'team.owner.password',
    'team.members.0.api_token',
]);
```

### Fluent JSON Testing

Laravel also offers a beautiful way to fluently test your application's JSON responses:

```php
use Illuminate\Testing\Fluent\AssertableJson;

test('fluent json', function () {
    $response = $this->getJson('/users/1');

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->where('id', 1)
                ->where('name', 'Victoria Faith')
                ->where('email', fn (string $email) => str($email)->is('*@laravel.com'))
                ->whereNot('status', 'pending')
                ->missing('password')
                ->etc()
        );
});
```

#### Understanding the `etc` Method

In the example above, you may have noticed we invoked the `etc` method at the end of our assertion chain. This method informs Laravel that there may be other attributes present on the JSON object. If the `etc` method is not used, the test will fail if other attributes that you did not make assertions against exist on the JSON object.

#### Asserting Attribute Presence / Absence

To assert that an attribute is present or absent, you may use the `has` and `missing` methods:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->has('data')
        ->missing('message')
);
```

In addition, the `hasAll` and `missingAll` methods allow asserting the presence or absence of multiple attributes simultaneously:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->hasAll(['status', 'data'])
        ->missingAll(['message', 'code'])
);
```

You may use the `hasAny` method to determine if at least one of a given list of attributes is present:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->has('status')
        ->hasAny('data', 'message', 'code')
);
```

#### Asserting Against JSON Collections

Often, your route will return a JSON response that contains multiple items:

```php
Route::get('/users', function () {
    return User::all();
});
```

We may use the fluent JSON object's `has` method to make assertions against the users included in the response:

```php
$response
    ->assertJson(fn (AssertableJson $json) =>
        $json->has(3)
            ->first(fn (AssertableJson $json) =>
                $json->where('id', 1)
                    ->where('name', 'Victoria Faith')
                    ->missing('password')
                    ->etc()
            )
    );
```

If you would like to make the same assertions against every item in a JSON collection, you may use the `each` method:

```php
$response
    ->assertJson(fn (AssertableJson $json) =>
        $json->has(3)
            ->each(fn (AssertableJson $json) =>
                $json->whereType('id', 'integer')
                    ->whereType('name', 'string')
                    ->missing('password')
                    ->etc()
            )
    );
```

#### Asserting JSON Types

You may only want to assert that the properties in the JSON response are of a certain type:

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->whereType('id', 'integer')
        ->whereAllType([
            'users.0.name' => 'string',
            'meta' => 'array'
        ])
);
```

The `whereType` and `whereAllType` methods recognize the following types: `string`, `integer`, `double`, `boolean`, `array`, and `null`.

## Testing File Uploads

The `Illuminate\Http\UploadedFile` class provides a `fake` method which may be used to generate dummy files or images for testing:

```php
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

test('avatars can be uploaded', function () {
    Storage::fake('avatars');

    $file = UploadedFile::fake()->image('avatar.jpg');

    $response = $this->post('/avatar', [
        'avatar' => $file,
    ]);

    Storage::disk('avatars')->assertExists($file->hashName());
});
```

```php
<?php

namespace Tests\Feature;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_avatars_can_be_uploaded(): void
    {
        Storage::fake('avatars');

        $file = UploadedFile::fake()->image('avatar.jpg');

        $response = $this->post('/avatar', [
            'avatar' => $file,
        ]);

        Storage::disk('avatars')->assertExists($file->hashName());
    }
}
```

If you would like to assert that a given file does not exist, you may use the `assertMissing` method:

```php
Storage::fake('avatars');

// ...

Storage::disk('avatars')->assertMissing('missing.jpg');
```

#### Fake File Customization

When creating files using the `fake` method, you may specify the width, height, and size of the image:

```php
UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);
```

In addition to creating images, you may create files of any other type using the `create` method:

```php
UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);
```

## Testing Views

Laravel also allows you to render a view without making a simulated HTTP request to the application. To accomplish this, you may call the `view` method within your test:

```php
test('a welcome view can be rendered', function () {
    $view = $this->view('welcome', ['name' => 'Taylor']);

    $view->assertSee('Taylor');
});
```

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_a_welcome_view_can_be_rendered(): void
    {
        $view = $this->view('welcome', ['name' => 'Taylor']);

        $view->assertSee('Taylor');
    }
}
```

The `TestView` class provides the following assertion methods: `assertSee`, `assertSeeInOrder`, `assertSeeText`, `assertSeeTextInOrder`, `assertDontSee`, and `assertDontSeeText`.

If needed, you may get the raw, rendered view contents by casting the `TestView` instance to a string:

```php
$contents = (string) $this->view('welcome');
```

#### Sharing Errors

Some views may depend on errors shared in the global error bag. To hydrate the error bag with error messages, you may use the `withViewErrors` method:

```php
$view = $this->withViewErrors([
    'name' => ['Please provide a valid name.']
])->view('form');

$view->assertSee('Please provide a valid name.');
```

### Rendering Blade and Components

If necessary, you may use the `blade` method to evaluate and render a raw Blade string:

```php
$view = $this->blade(
    '<x-component :name="$name" />',
    ['name' => 'Taylor']
);

$view->assertSee('Taylor');
```

You may use the `component` method to evaluate and render a Blade component:

```php
$view = $this->component(Profile::class, ['name' => 'Taylor']);

$view->assertSee('Taylor');
```

## Caching Routes

Before a test runs, Laravel boots a fresh instance of the application, including collecting all defined routes. If your applications have many route files, you may wish to add the `Illuminate\Foundation\Testing\WithCachedRoutes` trait to your test cases:

```php
<?php

use App\Http\Controllers\UserController;
use Illuminate\Foundation\Testing\WithCachedRoutes;

pest()->use(WithCachedRoutes::class);

test('basic example', function () {
    $this->get(action([UserController::class, 'index']));

    // ...
});
```

```php
<?php

namespace Tests\Feature;

use App\Http\Controllers\UserController;
use Illuminate\Foundation\Testing\WithCachedRoutes;
use Tests\TestCase;

class BasicTest extends TestCase
{
    use WithCachedRoutes;

    public function test_basic_example(): void
    {
        $response = $this->get(action([UserController::class, 'index']));

        // ...
    }
}
```

## Available Assertions

### Response Assertions

Laravel's `Illuminate\Testing\TestResponse` class provides a variety of custom assertion methods:

```php
// Assert the response status code...
$response->assertStatus(200);

// Assert the response has a valid status code...
$response->assertSuccessful();

// Assert the response has an invalid status code...
$response->assertStatus(400)->assertForbidden();

// Assert the response redirects to a given URI...
$response->assertRedirect('/example');

// Assert the response view contains data...
$response->assertViewHas('name', 'Taylor');

// Assert the session contains data...
$response->assertSessionHas('name', 'Taylor');

// Assert the session has an error bag...
$response->assertSessionHasErrors();

// Assert the session does not have errors...
$response->assertSessionDoesntHaveErrors();

// Assert the cookie exists and is not expired...
$response->assertCookie('token');

// Assert the cookie does not exist...
$response->assertCookieExpired('token');

// Assert the cookie is encrypted...
$response->assertCookieSigned('token');

// Assert the header exists...
$response->assertHeader('name');

// Assert the header value matches...
$response->assertHeader('name', 'Taylor');
```

### Authentication Assertions

```php
// Assert the user is authenticated...
$response->assertAuthenticated();

// Assert the user is not authenticated...
$response->assertGuest();
```

### Validation Assertions

```php
// Assert that validation messages are present for a given field...
$response->assertValidationErrors(['field']);

// Assert that no validation errors are present...
$response->assertValidationErrorsIn('email');

// Assert that a specific validation error is present...
$response->assertValidationErrorsFor('email');
```