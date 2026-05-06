---
title: Eloquent: API Resources
description: Learn how to transform Eloquent models and collections into JSON responses using API Resources.
url: https://laravel.com/docs/13.x/eloquent-resources
tags: [data]
cssclasses:
  - data
  - color-green
color: green
---

# Eloquent: API Resources

-   [Introduction](#introduction)
-   [Generating Resources](#generating-resources)
-   [Concept Overview](#concept-overview)
    -   [Resource Collections](#resource-collections)
-   [Writing Resources](#writing-resources)
    -   [Data Wrapping](#data-wrapping)
    -   [Pagination](#pagination)
    -   [Conditional Attributes](#conditional-attributes)
    -   [Conditional Relationships](#conditional-relationships)
    -   [Adding Meta Data](#adding-meta-data)
-   [JSON:API Resources](#jsonapi-resources)
    -   [Generating JSON:API Resources](#generating-jsonapi-resources)
    -   [Defining Attributes](#defining-jsonapi-attributes)
    -   [Defining Relationships](#defining-jsonapi-relationships)
    -   [Resource Type and ID](#jsonapi-resource-type-and-id)
    -   [Sparse Fieldsets and Includes](#jsonapi-sparse-fieldsets-and-includes)
    -   [Links and Meta](#jsonapi-links-and-meta)
-   [Resource Responses](#resource-responses)

## [Introduction](#introduction)

When building an API, you may need a transformation layer that sits between your Eloquent models and the JSON responses that are actually returned to your application's users. For example, you may wish to display certain attributes for a subset of users and not others, or you may wish to always include certain relationships in the JSON representation of your models. Eloquent's resource classes allow you to expressively and easily transform your models and model collections into JSON.

Of course, you may always convert Eloquent models or collections to JSON using their `toJson` methods; however, Eloquent resources provide more granular and robust control over the JSON serialization of your models and their relationships.

## [Generating Resources](#generating-resources)

To generate a resource class, you may use the `make:resource` Artisan command. By default, resources will be placed in the `app/Http/Resources` directory of your application. Resources extend the `Illuminate\Http\Resources\Json\JsonResource` class:

```bash
php artisan make:resource UserResource
```

#### [Resource Collections](#generating-resource-collections)

In addition to generating resources that transform individual models, you may generate resources that are responsible for transforming collections of models. This allows your JSON responses to include links and other meta information that is relevant to an entire collection of a given resource.

To create a resource collection, you should use the `--collection` flag when creating the resource. Or, including the word `Collection` in the resource name will indicate to Laravel that it should create a collection resource. Collection resources extend the `Illuminate\Http\Resources\Json\ResourceCollection` class:

```bash
php artisan make:resource User --collection

php artisan make:resource UserCollection
```

## [Concept Overview](#concept-overview)

This is a high-level overview of resources and resource collections. You are highly encouraged to read the other sections of this documentation to gain a deeper understanding of the customization and power offered to you by resources.

Before diving into all of the options available to you when writing resources, let's first take a high-level look at how resources are used within Laravel. A resource class represents a single model that needs to be transformed into a JSON structure. For example, here is a simple `UserResource` resource class:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

Every resource class defines a `toArray` method which returns the array of attributes that should be converted to JSON when the resource is returned as a response from a route or controller.

Note that we can access model properties directly from the `$this` variable. This is because a resource class will automatically proxy property and method access down to the underlying model for convenient access. Once the resource is defined, it may be returned from a route or controller. The resource accepts the underlying model instance via its constructor:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

For convenience, you may use the model's `toResource` method, which will use framework conventions to automatically discover the model's underlying resource:

```php
return User::findOrFail($id)->toResource();
```

When invoking the `toResource` method, Laravel will attempt to locate a resource that matches the model's name and is optionally suffixed with `Resource` within the `Http\Resources` namespace closest to the model's namespace.

If your resource class doesn't follow this naming convention or is located in a different namespace, you may specify the default resource for the model using the `UseResource` attribute:

```php
<?php

namespace App\Models;

use App\Http\Resources\CustomUserResource;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Attributes\UseResource;

#[UseResource(CustomUserResource::class)]
class User extends Model
{
    // ...
}
```

Alternatively, you may specify resource class by passing it to the `toResource` method:

```php
return User::findOrFail($id)->toResource(CustomUserResource::class);
```

### [Resource Collections](#resource-collections)

If you are returning a collection of resources or a paginated response, you should use the `collection` method provided by your resource class when creating the resource instance in your route or controller:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all());
});
```

Or, for convenience, you may use the Eloquent collection's `toResourceCollection` method, which will use framework conventions to automatically discover the model's underlying resource collection:

```php
return User::all()->toResourceCollection();
```

When invoking the `toResourceCollection` method, Laravel will attempt to locate a resource collection that matches the model's name and is suffixed with `Collection` within the `Http\Resources` namespace closest to the model's namespace.

If your resource collection class doesn't follow this naming convention or is located in a different namespace, you may specify the default resource collection for the model using the `UseResourceCollection` attribute:

```php
<?php

namespace App\Models;

use App\Http\Resources\CustomUserCollection;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Attributes\UseResourceCollection;

#[UseResourceCollection(CustomUserCollection::class)]
class User extends Model
{
    // ...
}
```

Alternatively, you may specify the resource collection class by passing it to the `toResourceCollection` method:

```php
return User::all()->toResourceCollection(CustomUserCollection::class);
```

#### [Custom Resource Collections](#custom-resource-collections)

By default, resource collections do not allow any addition of custom meta data that may need to be returned with your collection. If you would like to customize the resource collection response, you may create a dedicated resource to represent the collection:

```bash
php artisan make:resource UserCollection
```

Once the resource collection class has been generated, you may easily define any meta data that should be included with the response:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @return array<int|string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

After defining your resource collection, it may be returned from a route or controller:

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

Or, for convenience, you may use the Eloquent collection's `toResourceCollection` method, which will use framework conventions to automatically discover the model's underlying resource collection:

```php
return User::all()->toResourceCollection();
```

When invoking the `toResourceCollection` method, Laravel will attempt to locate a resource collection that matches the model's name and is suffixed with `Collection` within the `Http\Resources` namespace closest to the model's namespace.

#### [Preserving Collection Keys](#preserving-collection-keys)

When returning a resource collection from a route, Laravel resets the collection's keys so that they are in numerical order. However, you may use the `PreserveKeys` attribute on your resource class indicating whether a collection's original keys should be preserved:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Attributes\PreserveKeys;
use Illuminate\Http\Resources\Json\JsonResource;

#[PreserveKeys]
class UserResource extends JsonResource
{
    // ...
}
```

When the `preserveKeys` property is set to `true`, collection keys will be preserved when the collection is returned from a route or controller:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all()->keyBy->id);
});
```

#### [Customizing the Underlying Resource Class](#customizing-the-underlying-resource-class)

Typically, the `$this->collection` property of a resource collection is automatically populated with the result of mapping each item of the collection to its singular resource class. The singular resource class is assumed to be the collection's class name without the trailing `Collection` portion of the class name. In addition, depending on your personal preference, the singular resource class may or may not be suffixed with `Resource`.

For example, `UserCollection` will attempt to map the given user instances into the `UserResource` resource. To customize this behavior, you may use the `Collects` attribute on your resource collection:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Attributes\Collects;
use Illuminate\Http\Resources\Json\ResourceCollection;

#[Collects(Member::class)]
class UserCollection extends ResourceCollection
{
    // ...
}
```

## [Writing Resources](#writing-resources)

If you have not read the [concept overview](#concept-overview), you are highly encouraged to do so before proceeding with this documentation.

Resources only need to transform a given model into an array. So, each resource contains a `toArray` method which translates your model's attributes into an API friendly array that can be returned from your application's routes or controllers:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

Once a resource has been defined, it may be returned directly from a route or controller:

```php
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return User::findOrFail($id)->toUserResource();
});
```

#### [Relationships](#relationships)

If you would like to include related resources in your response, you may add them to the array returned by your resource's `toArray` method. In this example, we will use the `PostResource` resource's `collection` method to add the user's blog posts to the resource response:

```php
use App\Http\Resources\PostResource;
use Illuminate\Http\Request;

/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->posts),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

If you would like to include relationships only when they have already been loaded, check out the documentation on [conditional relationships](#conditional-relationships).

#### [Resource Collections](#writing-resource-collections)

While resources transform a single model into an array, resource collections transform a collection of models into an array. However, it is not absolutely necessary to define a resource collection class for each one of your models since all Eloquent model collections provide a `toResourceCollection` method to generate an "ad-hoc" resource collection on the fly:

```php
use App\Models\User;

Route::get('/users', function () {
    return User::all()->toResourceCollection();
});
```

However, if you need to customize the meta data returned with the collection, it is necessary to define your own resource collection:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

Like singular resources, resource collections may be returned directly from routes or controllers:

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

Or, for convenience, you may use the Eloquent collection's `toResourceCollection` method, which will use framework conventions to automatically discover the model's underlying resource collection:

```php
return User::all()->toResourceCollection();
```

When invoking the `toResourceCollection` method, Laravel will attempt to locate a resource collection that matches the model's name and is suffixed with `Collection` within the `Http\Resources` namespace closest to the model's namespace.

### [Data Wrapping](#data-wrapping)

By default, your outermost resource is wrapped in a `data` key when the resource response is converted to JSON. So, for example, a typical resource collection response looks like the following:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "eladio@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "liliana@example.com"
        }
    ]
}
```

If you would like to disable the wrapping of the outermost resource, you should invoke the `withoutWrapping` method on the base `Illuminate\Http\Resources\Json\JsonResource` class. Typically, you should call this method from your `AppServiceProvider` or another [[03-architecture-concepts/03-service-providers.md|service provider]] that is loaded on every request to your application:

```php
<?php

namespace App\Providers;

use Illuminate\Http\Resources\Json\JsonResource;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        JsonResource::withoutWrapping();
    }
}
```

The `withoutWrapping` method only affects the outermost response and will not remove `data` keys that you manually add to your own resource collections.

#### [Wrapping Nested Resources](#wrapping-nested-resources)

You have total freedom to determine how your resource's relationships are wrapped. If you would like all resource collections to be wrapped in a `data` key, regardless of their nesting, you should define a resource collection class for each resource and return the collection within a `data` key.

You may be wondering if this will cause your outermost resource to be wrapped in two `data` keys. Don't worry, Laravel will never let your resources be accidentally double-wrapped, so you don't have to be concerned about the nesting level of the resource collection you are transforming:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class CommentsCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return ['data' => $this->collection];
    }
}
```

#### [Data Wrapping and Pagination](#data-wrapping-and-pagination)

When returning paginated collections via a resource response, Laravel will wrap your resource data in a `data` key even if the `withoutWrapping` method has been called. This is because paginated responses always contain `meta` and `links` keys with information about the paginator's state:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "eladio@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "liliana@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

### [Pagination](#pagination)

You may pass a Laravel paginator instance to the `collection` method of a resource or to a custom resource collection:

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::paginate());
});
```

Or, for convenience, you may use the paginator's `toResourceCollection` method, which will use framework conventions to automatically discover the paginated model's underlying resource collection:

```php
return User::paginate()->toResourceCollection();
```

Paginated responses always contain `meta` and `links` keys with information about the paginator's state:

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "eladio@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "liliana@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

#### [Customizing the Pagination Information](#customizing-the-pagination-information)

If you would like to customize the information included in the `links` or `meta` keys of the pagination response, you may define a `paginationInformation` method on the resource. This method will receive the `$paginated` data and the array of `$default` information, which is an array containing the `links` and `meta` keys:

```php
/**
 * Customize the pagination information for the resource.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  array  $paginated
 * @param  array  $default
 * @return array
 */
public function paginationInformation($request, $paginated, $default)
{
    $default['links']['custom'] = 'https://example.com';

    return $default;
}
```

### [Conditional Attributes](#conditional-attributes)

Sometimes you may wish to only include an attribute in a resource response if a given condition is met. For example, you may wish to only include a value if the current user is an "administrator". Laravel provides a variety of helper methods to assist you in this situation. The `when` method may be used to conditionally add an attribute to a resource response:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'secret' => $this->when($request->user()->isAdmin(), 'secret-value'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

In this example, the `secret` key will only be returned in the final resource response if the authenticated user's `isAdmin` method returns `true`. If the method returns `false`, the `secret` key will be removed from the resource response before it is sent to the client. The `when` method allows you to expressively define your resources without resorting to conditional statements when building the array.

The `when` method also accepts a closure as its second argument, allowing you to calculate the resulting value only if the given condition is `true`:

```php
'secret' => $this->when($request->user()->isAdmin(), function () {
    return 'secret-value';
}),
```

The `whenHas` method may be used to include an attribute if it is actually present on the underlying model:

```php
'name' => $this->whenHas('name'),
```

Additionally, the `whenNotNull` method may be used to include an attribute in the resource response if the attribute is not null:

```php
'name' => $this->whenNotNull($this->name),
```

#### [Merging Conditional Attributes](#merging-conditional-attributes)

Sometimes you may have several attributes that should only be included in the resource response based on the same condition. In this case, you may use the `mergeWhen` method to include the attributes in the response only when the given condition is `true`:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        $this->mergeWhen($request->user()->isAdmin(), [
            'first-secret' => 'value',
            'second-secret' => 'value',
        ]),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

Again, if the given condition is `false`, these attributes will be removed from the resource response before it is sent to the client.

The `mergeWhen` method should not be used within arrays that mix string and numeric keys. Furthermore, it should not be used within arrays with numeric keys that are not ordered sequentially.

### [Conditional Relationships](#conditional-relationships)

In addition to conditionally loading attributes, you may conditionally include relationships on your resource responses based on if the relationship has already been loaded on the model. This allows your controller to decide which relationships should be loaded on the model and your resource can easily include them only when they have actually been loaded. Ultimately, this makes it easier to avoid "N+1" query problems within your resources.

The `whenLoaded` method may be used to conditionally load a relationship. In order to avoid unnecessarily loading relationships, this method accepts the name of the relationship instead of the relationship itself:

```php
use App\Http\Resources\PostResource;

/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->whenLoaded('posts')),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

In this example, if the relationship has not been loaded, the `posts` key will be removed from the resource response before it is sent to the client.

#### [Conditional Relationship Counts](#conditional-relationship-counts)

In addition to conditionally including relationships, you may conditionally include relationship "counts" on your resource responses based on if the relationship's count has been loaded on the model:

```php
new UserResource($user->loadCount('posts'));
```

The `whenCounted` method may be used to conditionally include a relationship's count in your resource response. This method avoids unnecessarily including the attribute if the relationships' count is not present:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts_count' => $this->whenCounted('posts'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

In this example, if the `posts` relationship's count has not been loaded, the `posts_count` key will be removed from the resource response before it is sent to the client.

Other types of aggregates, such as `avg`, `sum`, `min`, and `max` may also be conditionally loaded using the `whenAggregated` method:

```php
'words_avg' => $this->whenAggregated('posts', 'words', 'avg'),
'words_sum' => $this->whenAggregated('posts', 'words', 'sum'),
'words_min' => $this->whenAggregated('posts', 'words', 'min'),
'words_max' => $this->whenAggregated('posts', 'words', 'max'),
```

#### [Conditional Pivot Information](#conditional-pivot-information)

In addition to conditionally including relationship information in your resource responses, you may conditionally include data from the intermediate tables of many-to-many relationships using the `whenPivotLoaded` method. The `whenPivotLoaded` method accepts the name of the pivot table as its first argument. The second argument should be a closure that returns the value to be returned if the pivot information is available on the model:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoaded('role_user', function () {
            return $this->pivot->expires_at;
        }),
    ];
}
```

If your relationship is using a [[08-eloquent-orm/02-eloquent-relationships.md#defining-custom-intermediate-table-models|custom intermediate table model]], you may pass an instance of the intermediate table model as the first argument to the `whenPivotLoaded` method:

```php
'expires_at' => $this->whenPivotLoaded(new Membership, function () {
    return $this->pivot->expires_at;
}),
```

If your intermediate table is using an accessor other than `pivot`, you may use the `whenPivotLoadedAs` method:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoadedAs('subscription', 'role_user', function () {
            return $this->subscription->expires_at;
        }),
    ];
}
```

### [Adding Meta Data](#adding-meta-data)

Some JSON API standards require the addition of meta data to your resource and resource collections responses. This often includes things like `links` to the resource or related resources, or meta data about the resource itself. If you need to return additional meta data about a resource, include it in your `toArray` method. For example, you might include `links` information when transforming a resource collection:

```php
/**
 * Transform the resource into an array.
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'data' => $this->collection,
        'links' => [
            'self' => 'link-value',
        ],
    ];
}
```

When returning additional meta data from your resources, you never have to worry about accidentally overriding the `links` or `meta` keys that are automatically added by Laravel when returning paginated responses. Any additional `links` you define will be merged with the links provided by the paginator.

#### [Top Level Meta Data](#top-level-meta-data)

Sometimes you may wish to only include certain meta data with a resource response if the resource is the outermost resource being returned. Typically, this includes meta information about the response as a whole. To define this meta data, add a `with` method to your resource class. This method should return an array of meta data to be included with the resource response only when the resource is the outermost resource being transformed:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return parent::toArray($request);
    }

    /**
     * Get additional data that should be returned with the resource array.
     *
     * @return array<string, mixed>
     */
    public function with(Request $request): array
    {
        return [
            'meta' => [
                'key' => 'value',
            ],
        ];
    }
}
```

#### [Adding Meta Data When Constructing Resources](#adding-meta-data-when-constructing-resources)

You may also add top-level data when constructing resource instances in your route or controller. The `additional` method, which is available on all resources, accepts an array of data that should be added to the resource response:

```php
return User::all()
    ->load('roles')
    ->toResourceCollection()
    ->additional(['meta' => [
        'key' => 'value',
    ]]);
```

## [JSON:API Resources](#jsonapi-resources)

Laravel ships with `JsonApiResource`, a resource class that produces responses compliant with the [JSON:API specification](https://jsonapi.org/). It extends the standard `JsonResource` class and automatically handles resource object structure, relationships, sparse fieldsets, includes, lazy attribute evaluation, and sets the `Content-Type` header to `application/vnd.api+json`.

Laravel's JSON:API resources handle the serialization of your responses. If you also need to parse incoming JSON:API query parameters such as filters and sorts, [Spatie's Laravel Query Builder](https://spatie.be/docs/laravel-query-builder) is a great companion package.

### [Generating JSON:API Resources](#generating-jsonapi-resources)

To generate a JSON:API resource, use the `make:resource` Artisan command with the `--json-api` flag:

```bash
php artisan make:resource PostResource --json-api
```

The generated class will extend `Illuminate\Http\Resources\JsonApi\JsonApiResource` and include `$attributes` and `$relationships` properties for you to define:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\JsonApi\JsonApiResource;

class PostResource extends JsonApiResource
{
    /**
     * The resource's attributes.
     */
    public $attributes = [
        // ...
    ];

    /**
     * The resource's relationships.
     */
    public $relationships = [
        // ...
    ];
}
```

JSON:API resources may be returned from routes and controllers just like standard resources:

```php
use App\Http\Resources\PostResource;
use App\Models\Post;

Route::get('/api/posts/{post}', function (Post $post) {
    return new PostResource($post);
});
```

### [Defining Attributes](#defining-jsonapi-attributes)

To define the attributes for a JSON:API resource, assign an array of attribute names to the `$attributes` property:

```php
public $attributes = [
    'title',
    'slug',
    'excerpt',
    'content',
    'created_at',
    'updated_at',
];
```

### [Defining Relationships](#defining-jsonapi-relationships)

To define the relationships for a JSON:API resource, assign an array of relationship names to the `$relationships` property:

```php
public $relationships = [
    'author',
    'comments',
    'tags',
];
```

When defining relationships, you may also specify which related resources should be included by default:

```php
public $relationships = [
    'author',
    'comments' => 'comments',
    'tags' => 'tags',
];
```

### [Resource Type and ID](#jsonapi-resource-type-and-id)

By default, the JSON:API resource will use the model's table name as the resource type and the model's primary key as the resource ID. You may customize this by overriding the `type` and `id` methods:

```php
public function type(): string
{
    return 'posts';
}

public function id(): string
{
    return (string) $this->id;
}
```

### [Sparse Fieldsets and Includes](#jsonapi-sparse-fieldsets-and-includes)

JSON:API resources automatically support sparse fieldsets and includes. You may specify which attributes and relationships should be included in the response by using query parameters:

```
GET /api/posts?fields[posts]=title,content&include=author
```

### [Links and Meta](#jsonapi-links-and-meta)

You may customize the links and meta information for JSON:API responses by overriding the `links` and `meta` methods:

```php
public function links(Request $request): array
{
    return [
        'self' => url('/api/posts/' . $this->id),
    ];
}

public function meta(Request $request): array
{
    return [
        'foo' => 'bar',
    ];
}
```

## [Resource Responses](#resource-responses)

Resources may be returned directly from routes and controllers. Laravel will automatically transform the resource into a JSON response:

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

You may also return collections:

```php
Route::get('/users', function () {
    return UserResource::collection(User::all());
});
```

Or use the convenience methods on the model:

```php
Route::get('/user/{id}', function (string $id) {
    return User::findOrFail($id)->toResource();
});

Route::get('/users', function () {
    return User::all()->toResourceCollection();
});
```