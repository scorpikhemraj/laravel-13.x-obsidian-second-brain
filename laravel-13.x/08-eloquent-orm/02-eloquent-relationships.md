---
title: Eloquent: Relationships
description: Learn how to define and work with database relationships in Laravel Eloquent ORM.
url: https://laravel.com/docs/13.x/eloquent-relationships
tags: [data]
cssclasses:
  - data
  - color-green
color: green
---

# Eloquent: Relationships

-   [Introduction](#introduction)
-   [Defining Relationships](#defining-relationships)
    -   [One to One / Has One](#one-to-one)
    -   [One to Many / Has Many](#one-to-many)
    -   [One to Many (Inverse) / Belongs To](#one-to-many-inverse)
    -   [Has One of Many](#has-one-of-many)
    -   [Has One Through](#has-one-through)
    -   [Has Many Through](#has-many-through)
-   [Scoped Relationships](#scoped-relationships)
-   [Many to Many Relationships](#many-to-many)
    -   [Retrieving Intermediate Table Columns](#retrieving-intermediate-table-columns)
    -   [Filtering Queries via Intermediate Table Columns](#filtering-queries-via-intermediate-table-columns)
    -   [Ordering Queries via Intermediate Table Columns](#ordering-queries-via-intermediate-table-columns)
    -   [Defining Custom Intermediate Table Models](#defining-custom-intermediate-table-models)
-   [Polymorphic Relationships](#polymorphic-relationships)
    -   [One to One](#one-to-one-polymorphic-relations)
    -   [One to Many](#one-to-many-polymorphic-relations)
    -   [One of Many](#one-of-many-polymorphic-relations)
    -   [Many to Many](#many-to-many-polymorphic-relations)
    -   [Custom Polymorphic Types](#custom-polymorphic-types)
-   [Dynamic Relationships](#dynamic-relationships)
-   [Querying Relations](#querying-relations)
    -   [Relationship Methods vs. Dynamic Properties](#relationship-methods-vs-dynamic-properties)
    -   [Querying Relationship Existence](#querying-relationship-existence)
    -   [Querying Relationship Absence](#querying-relationship-absence)
    -   [Querying Morph To Relationships](#querying-morph-to-relationships)
-   [Aggregating Related Models](#aggregating-related-models)
    -   [Counting Related Models](#counting-related-models)
    -   [Other Aggregate Functions](#other-aggregate-functions)
    -   [Counting Related Models on Morph To Relationships](#counting-related-models-on-morph-to-relationships)
-   [Eager Loading](#eager-loading)
    -   [Constraining Eager Loads](#constraining-eager-loads)
    -   [Lazy Eager Loading](#lazy-eager-loading)
    -   [Automatic Eager Loading](#automatic-eager-loading)
    -   [Preventing Lazy Loading](#preventing-lazy-loading)
-   [Inserting and Updating Related Models](#inserting-and-updating-related-models)
    -   [The `save` Method](#the-save-method)
    -   [The `create` Method](#the-create-method)
    -   [Belongs To Relationships](#updating-belongs-to-relationships)
    -   [Many to Many Relationships](#updating-many-to-many-relationships)
-   [Touching Parent Timestamps](#touching-parent-timestamps)

## [Introduction](#introduction)

Database tables are often related to one another. For example, a blog post may have many comments or an order could be related to the user who placed it. Eloquent makes managing and working with these relationships easy, and supports a variety of common relationships:

-   [One To One](#one-to-one)
-   [One To Many](#one-to-many)
-   [Many To Many](#many-to-many)
-   [Has One Through](#has-one-through)
-   [Has Many Through](#has-many-through)
-   [One To One (Polymorphic)](#one-to-one-polymorphic-relations)
-   [One To Many (Polymorphic)](#one-to-many-polymorphic-relations)
-   [Many To Many (Polymorphic)](#many-to-many-polymorphic-relations)

## [Defining Relationships](#defining-relationships)

Eloquent relationships are defined as methods on your Eloquent model classes. Since relationships also serve as powerful [[07-database/02-query-builder.md|query builders]], defining relationships as methods provides powerful method chaining and querying capabilities. For example, we may chain additional query constraints on this `posts` relationship:

```php
$user->posts()->where('active', 1)->get();
```

But, before diving too deep into using relationships, let's learn how to define each type of relationship supported by Eloquent.

### [One to One / Has One](#one-to-one)

A one-to-one relationship is a very basic type of database relationship. For example, a `User` model might be associated with one `Phone` model. To define this relationship, we will place a `phone` method on the `User` model. The `phone` method should call the `hasOne` method and return its result. The `hasOne` method is available to your model via the model's `Illuminate\Database\Eloquent\Model` base class:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOne;

class User extends Model
{
    /**
     * Get the phone associated with the user.
     */
    public function phone(): HasOne
    {
        return $this->hasOne(Phone::class);
    }
}
```

The first argument passed to the `hasOne` method is the name of the related model class. Once the relationship is defined, we may retrieve the related record using Eloquent's dynamic properties. Dynamic properties allow you to access relationship methods as if they were properties defined on the model:

```php
$phone = User::find(1)->phone;
```

Eloquent determines the foreign key of the relationship based on the parent model name. In this case, the `Phone` model is automatically assumed to have a `user_id` foreign key. If you wish to override this convention, you may pass a second argument to the `hasOne` method:

```php
return $this->hasOne(Phone::class, 'foreign_key');
```

Additionally, Eloquent assumes that the foreign key should have a value matching the primary key column of the parent. In other words, Eloquent will look for the value of the user's `id` column in the `user_id` column of the `Phone` record. If you would like the relationship to use a primary key value other than `id` or your model's primary key, you may pass a third argument to the `hasOne` method:

```php
return $this->hasOne(Phone::class, 'foreign_key', 'local_key');
```

#### [Defining the Inverse of the Relationship](#one-to-one-defining-the-inverse-of-the-relationship)

So, we can access the `Phone` model from our `User` model. Next, let's define a relationship on the `Phone` model that will let us access the user that owns the phone. We can define the inverse of a `hasOne` relationship using the `belongsTo` method:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Phone extends Model
{
    /**
     * Get the user that owns the phone.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

When invoking the `user` method, Eloquent will attempt to find a `User` model that has an `id` which matches the `user_id` column on the `Phone` model.

Eloquent determines the foreign key name by examining the name of the relationship method and suffixing the method name with `_id`. So, in this case, Eloquent assumes that the `Phone` model has a `user_id` column. However, if the foreign key on the `Phone` model is not `user_id`, you may pass a custom key name as the second argument to the `belongsTo` method:

```php
/**
 * Get the user that owns the phone.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key');
}
```

If the parent model does not use `id` as its primary key, or you wish to find the associated model using a different column, you may pass a third argument to the `belongsTo` method specifying the parent table's custom key:

```php
/**
 * Get the user that owns the phone.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key', 'owner_key');
}
```

### [One to Many / Has Many](#one-to-many)

A one-to-many relationship is used to define relationships where a single model is the parent to one or more child models. For example, a blog post may have an infinite number of comments. Like all other Eloquent relationships, one-to-many relationships are defined by defining a method on your Eloquent model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * Get the comments for the blog post.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
```

Remember, Eloquent will automatically determine the proper foreign key column for the `Comment` model. By convention, Eloquent will take the "snake case" name of the parent model and suffix it with `_id`. So, in this example, Eloquent will assume the foreign key column on the `Comment` model is `post_id`.

Once the relationship method has been defined, we can access the [[08-eloquent-orm/03-eloquent-collections.md|collection]] of related comments by accessing the `comments` property. Remember, since Eloquent provides "dynamic relationship properties", we can access relationship methods as if they were defined as properties on the model:

```php
use App\Models\Post;

$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    // ...
}
```

Since all relationships also serve as query builders, you may add further constraints to the relationship query by calling the `comments` method and continuing to chain conditions onto the query:

```php
$comment = Post::find(1)->comments()
    ->where('title', 'foo')
    ->first();
```

Like the `hasOne` method, you may also override the foreign and local keys by passing additional arguments to the `hasMany` method:

```php
return $this->hasMany(Comment::class, 'foreign_key');

return $this->hasMany(Comment::class, 'foreign_key', 'local_key');
```

#### [Automatically Hydrating Parent Models on Children](#automatically-hydrating-parent-models-on-children)

Even when utilizing Eloquent eager loading, "N + 1" query problems can arise if you try to access the parent model from a child model while looping through the child models:

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->post->title;
    }
}
```

In the example above, an "N + 1" query problem has been introduced because, even though comments were eager loaded for every `Post` model, Eloquent does not automatically hydrate the parent `Post` on each child `Comment` model.

If you would like Eloquent to automatically hydrate parent models onto their children, you may invoke the `chaperone` method when defining a `hasMany` relationship:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * Get the comments for the blog post.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class)->chaperone();
    }
}
```

> [!IMPORTANT]
> **Laravel Expert Note: Chaperoning & N+1**
> The `chaperone()` method is one of the most underrated features in Laravel 13.x. It elegantly solves the "circular N+1" problem where accessing a parent model from a child (e.g., `$comment->post`) triggers a new query even if you eager-loaded the children. Use it by default for primary relationships to keep your database interactions efficient.

Or, if you would like to opt-in to automatic parent hydration at run time, you may invoke the `chaperone` model when eager loading the relationship:

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

### [One to Many (Inverse) / Belongs To](#one-to-many-inverse)

Now that we can access all of a post's comments, let's define a relationship to allow a comment to access its parent post. To define the inverse of a `hasMany` relationship, define a relationship method on the child model which calls the `belongsTo` method:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    /**
     * Get the post that owns the comment.
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

Once the relationship has been defined, we can retrieve a comment's parent post by accessing the `post` "dynamic relationship property":

```php
use App\Models\Comment;

$comment = Comment::find(1);

return $comment->post->title;
```

In the example above, Eloquent will attempt to find a `Post` model that has an `id` which matches the `post_id` column on the `Comment` model.

Eloquent determines the default foreign key name by examining the name of the relationship method and suffixing the method name with a `_` followed by the name of the parent model's primary key column. So, in this example, Eloquent will assume the `Post` model's foreign key on the `comments` table is `post_id`.

However, if the foreign key for your relationship does not follow these conventions, you may pass a custom foreign key name as the second argument to the `belongsTo` method:

```php
/**
 * Get the post that owns the comment.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key');
}
```

If your parent model does not use `id` as its primary key, or you wish to find the associated model using a different column, you may pass a third argument to the `belongsTo` method specifying your parent table's custom key:

```php
/**
 * Get the post that owns the comment.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key', 'owner_key');
}
```

#### [Default Models](#default-models)

The `belongsTo`, `hasOne`, `hasOneThrough`, and `morphOne` relationships allow you to define a default model that will be returned if the given relationship is `null`. This pattern is often referred to as the [Null Object pattern](https://en.wikipedia.org/wiki/Null_Object_pattern) and can help remove conditional checks in your code. In the following example, the `user` relation will return an empty `App\Models\User` model if no user is attached to the `Post` model:

```php
/**
 * Get the author of the post.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault();
}
```

To populate the default model with attributes, you may pass an array or closure to the `withDefault` method:

```php
/**
 * Get the author of the post.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault([
        'name' => 'Guest Author',
    ]);
}

/**
 * Get the author of the post.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault(function (User $user, Post $post) {
        $user->name = 'Guest Author';
    });
}
```

#### [Querying Belongs To Relationships](#querying-belongs-to-relationships)

When querying for the children of a "belongs to" relationship, you may manually build the `where` clause to retrieve the corresponding Eloquent models:

```php
use App\Models\Post;

$posts = Post::where('user_id', $user->id)->get();
```

However, you may find it more convenient to use the `whereBelongsTo` method, which will automatically determine the proper relationship and foreign key for the given model:

```php
$posts = Post::whereBelongsTo($user)->get();
```

You may also provide a [[08-eloquent-orm/03-eloquent-collections.md|collection]] instance to the `whereBelongsTo` method. When doing so, Laravel will retrieve models that belong to any of the parent models within the collection:

```php
$users = User::where('vip', true)->get();

$posts = Post::whereBelongsTo($users)->get();
```

By default, Laravel will determine the relationship associated with the given model based on the class name of the model; however, you may specify the relationship name manually by providing it as the second argument to the `whereBelongsTo` method:

```php
$posts = Post::whereBelongsTo($user, 'author')->get();
```

### [Has One of Many](#has-one-of-many)

Sometimes a model may have many related models, yet you want to easily retrieve the "latest" or "oldest" related model of the relationship. For example, a `User` model may be related to many `Order` models, but you want to define a convenient way to interact with the most recent order the user has placed. You may accomplish this using the `hasOne` relationship type combined with the `ofMany` methods:

```php
/**
 * Get the user's most recent order.
 */
public function latestOrder(): HasOne
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```

Likewise, you may define a method to retrieve the "oldest", or first, related model of a relationship:

```php
/**
 * Get the user's oldest order.
 */
public function oldestOrder(): HasOne
{
    return $this->hasOne(Order::class)->oldestOfMany();
}
```

By default, the `latestOfMany` and `oldestOfMany` methods will retrieve the latest or oldest related model based on the model's primary key, which must be sortable. However, sometimes you may wish to retrieve a single model from a larger relationship using a different sorting criteria.

For example, using the `ofMany` method, you may retrieve the user's most expensive order. The `ofMany` method accepts the sortable column as its first argument and which aggregate function (`min` or `max`) to apply when querying for the related model:

```php
/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->hasOne(Order::class)->ofMany('price', 'max');
}
```

Because PostgreSQL does not support executing the `MAX` function against UUID columns, it is not currently possible to use one-of-many relationships in combination with PostgreSQL UUID columns.

#### [Converting "Many" Relationships to Has One Relationships](#converting-many-relationships-to-has-one-relationships)

Often, when retrieving a single model using the `latestOfMany`, `oldestOfMany`, or `ofMany` methods, you already have a "has many" relationship defined for the same model. For convenience, Laravel allows you to easily convert this relationship into a "has one" relationship by invoking the `one` method on the relationship:

```php
/**
 * Get the user's orders.
 */
public function orders(): HasMany
{
    return $this->hasMany(Order::class);
}

/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->orders()->one()->ofMany('price', 'max');
}
```

You may also use the `one` method to convert `HasManyThrough` relationships to `HasOneThrough` relationships:

```php
public function latestDeployment(): HasOneThrough
{
    return $this->deployments()->one()->latestOfMany();
}
```

#### [Advanced Has One of Many Relationships](#advanced-has-one-of-many-relationships)

It is possible to construct more advanced "has one of many" relationships. For example, a `Product` model may have many associated `Price` models that are retained in the system even after new pricing is published. In addition, new pricing data for the product may be able to be published in advance to take effect at a future date via a `published_at` column.

So, in summary, we need to retrieve the latest published pricing where the published date is not in the future. In addition, if two prices have the same published date, we will prefer the price with the greatest ID. To accomplish this, we must pass an array to the `ofMany` method that contains the sortable columns which determine the latest price. In addition, a closure will be provided as the second argument to the `ofMany` method. This closure will be responsible for adding additional publish date constraints to the relationship query:

```php
/**
 * Get the current pricing for the product.
 */
public function currentPricing(): HasOne
{
    return $this->hasOne(Price::class)->ofMany([
        'published_at' => 'max',
        'id' => 'max',
    ], function (Builder $query) {
        $query->where('published_at', '<', now());
    });
}
```

### [Has One Through](#has-one-through)

The "has-one-through" relationship defines a one-to-one relationship with another model. However, this relationship indicates that the declaring model can be matched with one instance of another model by proceeding *through* a third model.

For example, in a vehicle repair shop application, each `Mechanic` model may be associated with one `Car` model, and each `Car` model may be associated with one `Owner` model. While the mechanic and the owner have no direct relationship within the database, the mechanic can access the owner *through* the `Car` model. Let's look at the tables necessary to define this relationship:

```
mechanics
    id - integer
    name - string

cars
    id - integer
    model - string
    mechanic_id - integer

owners
    id - integer
    name - string
    car_id - integer
```

Now that we have examined the table structure for the relationship, let's define the relationship on the `Mechanic` model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOneThrough;

class Mechanic extends Model
{
    /**
     * Get the car's owner.
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(Owner::class, Car::class);
    }
}
```

The first argument passed to the `hasOneThrough` method is the name of the final model we wish to access, while the second argument is the name of the intermediate model.

Or, if the relevant relationships have already been defined on all of the models involved in the relationship, you may fluently define a "has-one-through" relationship by invoking the `through` method and supplying the names of those relationships. For example, if the `Mechanic` model has a `cars` relationship and the `Car` model has an `owner` relationship, you may define a "has-one-through" relationship connecting the mechanic and the owner like so:

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

#### [Key Conventions](#has-one-through-key-conventions)

Typical Eloquent foreign key conventions will be used when performing the relationship's queries. If you would like to customize the keys of the relationship, you may pass them as the third and fourth arguments to the `hasOneThrough` method. The third argument is the name of the foreign key on the intermediate model. The fourth argument is the name of the foreign key on the final model. The fifth argument is the local key, while the sixth argument is the local key of the intermediate model:

```php
class Mechanic extends Model
{
    /**
     * Get the car's owner.
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(
            Owner::class,
            Car::class,
            'mechanic_id', // Foreign key on the cars table...
            'car_id', // Foreign key on the owners table...
            'id', // Local key on the mechanics table...
            'id' // Local key on the cars table...
        );
    }
}
```

Or, as discussed earlier, if the relevant relationships have already been defined on all of the models involved in the relationship, you may fluently define a "has-one-through" relationship by invoking the `through` method and supplying the names of those relationships. This approach offers the advantage of reusing the key conventions already defined on the existing relationships:

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

### [Has Many Through](#has-many-through)

The "has-many-through" relationship provides a convenient way to access distant relations via an intermediate relation. For example, let's assume we are building a deployment platform like [Laravel Cloud](https://cloud.laravel.com). An `Application` model might access many `Deployment` models through an intermediate `Environment` model. Using this example, you could easily gather all deployments for a given application. Let's look at the tables required to define this relationship:

```
applications
    id - integer
    name - string

environments
    id - integer
    application_id - integer
    name - string

deployments
    id - integer
    environment_id - integer
    commit_hash - string
```

Now that we have examined the table structure for the relationship, let's define the relationship on the `Application` model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasManyThrough;

class Application extends Model
{
    /**
     * Get all of the deployments for the application.
     */
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(Deployment::class, Environment::class);
    }
}
```

The first argument passed to the `hasManyThrough` method is the name of the final model we wish to access, while the second argument is the name of the intermediate model.

Or, if the relevant relationships have already been defined on all of the models involved in the relationship, you may fluently define a "has-many-through" relationship by invoking the `through` method and supplying the names of those relationships. For example, if the `Application` model has a `environments` relationship and the `Environment` model has a `deployments` relationship, you may define a "has-many-through" relationship connecting the application and the deployments like so:

```php
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```

Though the `Deployment` model's table does not contain a `application_id` column, the `hasManyThrough` relation provides access to an application's deployments via `$application->deployments`. To retrieve these models, Eloquent inspects the `application_id` column on the intermediate `Environment` model's table. After finding the relevant environment IDs, they are used to query the `Deployment` model's table.

#### [Key Conventions](#has-many-through-key-conventions)

Typical Eloquent foreign key conventions will be used when performing the relationship's queries. If you would like to customize the keys of the relationship, you may pass them as the third and fourth arguments to the `hasManyThrough` method. The third argument is the name of the foreign key on the intermediate model. The fourth argument is the name of the foreign key on the final model. The fifth argument is the local key, while the sixth argument is the local key of the intermediate model:

```php
class Application extends Model
{
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(
            Deployment::class,
            Environment::class,
            'application_id', // Foreign key on the environments table...
            'environment_id', // Foreign key on the deployments table...
            'id', // Local key on the applications table...
            'id' // Local key on the environments table...
        );
    }
}
```

Or, as discussed earlier, if the relevant relationships have already been defined on all of the models involved in the relationship, you may fluently define a "has-many-through" relationship by invoking the `through` method and supplying the names of those relationships. This approach offers the advantage of reusing the key conventions already defined on the existing relationships:

```php
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```

### [Scoped Relationships](#scoped-relationships)

It's common to add additional methods to models that constrain relationships. For example, you might add a `featuredPosts` method to a `User` model which constrains the broader `posts` relationship with an additional `where` constraint:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get the user's posts.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class)->latest();
    }

    /**
     * Get the user's featured posts.
     */
    public function featuredPosts(): HasMany
    {
        return $this->posts()->where('featured', true);
    }
}
```

However, if you attempt to create a model via the `featuredPosts` method, its `featured` attribute would not be set to `true`. If you would like to create models via relationship methods and also specify attributes that should be added to all models created via that relationship, you may use the `withAttributes` method when building the relationship query:

```php
/**
 * Get the user's featured posts.
 */
public function featuredPosts(): HasMany
{
    return $this->posts()->withAttributes(['featured' => true]);
}
```

The `withAttributes` method will add `where` conditions to the query using the given attributes, and it will also add the given attributes to any models created via the relationship method:

```php
$post = $user->featuredPosts()->create(['title' => 'Featured Post']);

$post->featured; // true
```

To instruct the `withAttributes` method to not add `where` conditions to the query, you may set the `asConditions` argument to `false`:

```php
return $this->posts()->withAttributes(['featured' => true], asConditions: false);
```

## [Many to Many Relationships](#many-to-many)

Many-to-many relations are slightly more complicated than `hasOne` and `hasMany` relationships. An example of a many-to-many relationship is a user that has many roles and those roles are also shared by other users in the application. For example, a user may be assigned the role of "Author" and "Editor"; however, those roles may also be assigned to other users as well. So, a user has many roles and a role has many users.

#### [Table Structure](#many-to-many-table-structure)

To define this relationship, three database tables are needed: `users`, `roles`, and `role_user`. The `role_user` table is derived from the alphabetical order of the related model names and contains `user_id` and `role_id` columns. This table is used as an intermediate table linking the users and roles.

Remember, since a role can belong to many users, we cannot simply place a `user_id` column on the `roles` table. This would mean that a role could only belong to a single user. In order to provide support for roles being assigned to multiple users, the `role_user` table is needed. We can summarize the relationship's table structure like so:

```
users
    id - integer
    name - string

roles
    id - integer
    name - string

role_user
    user_id - integer
    role_id - integer
```

#### [Model Structure](#many-to-many-model-structure)

Many-to-many relationships are defined by writing a method that returns the result of the `belongsToMany` method. The `belongsToMany` method is provided by the `Illuminate\Database\Eloquent\Model` base class that is used by all of your application's Eloquent models. For example, let's define a `roles` method on our `User` model. The first argument passed to this method is the name of the related model class:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    /**
     * The roles that belong to the user.
     */
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}
```

Once the relationship is defined, you may access the user's roles using the `roles` dynamic relationship property:

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    // ...
}
```

Since all relationships also serve as query builders, you may add further constraints to the relationship query by calling the `roles` method and continuing to chain conditions onto the query:

```php
$roles = User::find(1)->roles()->orderBy('name')->get();
```

To determine the table name of the relationship's intermediate table, Eloquent will join the two related model names in alphabetical order. However, you are free to override this convention. You may do so by passing a second argument to the `belongsToMany` method:

```php
return $this->belongsToMany(Role::class, 'role_user');
```

In addition to customizing the name of the intermediate table, you may also customize the column names of the keys on the table by passing additional arguments to the `belongsToMany` method. The third argument is the foreign key name of the model on which you are defining the relationship, while the fourth argument is the foreign key name of the model that you are joining to:

```php
return $this->belongsToMany(Role::class, 'role_user', 'user_id', 'role_id');
```

#### [Defining the Inverse of the Relationship](#many-to-many-defining-the-inverse-of-the-relationship)

To define the "inverse" of a many-to-many relationship, you should define a method on the related model which also returns the result of the `belongsToMany` method. To complete our user / role example, let's define the `users` method on the `Role` model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * The users that belong to the role.
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }
}
```

As you can see, the relationship is defined exactly the same as its `User` model counterpart with the exception of referencing the `App\Models\User` model. Since we're reusing the `belongsToMany` method, all of the usual table and key customization options are available when defining the "inverse" of many-to-many relationships.

### [Retrieving Intermediate Table Columns](#retrieving-intermediate-table-columns)

As you have already learned, working with many-to-many relations requires the presence of an intermediate table. Eloquent provides some very helpful ways of interacting with this table. For example, let's assume our `User` model has many `Role` models that it is related to. After accessing this relationship, we may access the intermediate table using the `pivot` attribute on the models:

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```

Notice that each `Role` model we retrieve is automatically assigned a `pivot` attribute. This attribute contains a model representing the intermediate table.

By default, only the model keys will be present on the `pivot` model. If your intermediate table contains extra attributes, you must specify them when defining the relationship:

```php
return $this->belongsToMany(Role::class)->withPivot('active', 'created_by');
```

If you would like your intermediate table to have `created_at` and `updated_at` timestamps that are automatically maintained by Eloquent, call the `withTimestamps` method when defining the relationship:

```php
return $this->belongsToMany(Role::class)->withTimestamps();
```

Intermediate tables that utilize Eloquent's automatically maintained timestamps are required to have both `created_at` and `updated_at` timestamp columns.

#### [Customizing the `pivot` Attribute Name](#customizing-the-pivot-attribute-name)

As noted previously, attributes from the intermediate table may be accessed on models via the `pivot` attribute. However, you are free to customize the name of this attribute to better reflect its purpose within your application.

For example, if your application contains users that may subscribe to podcasts, you likely have a many-to-many relationship between users and podcasts. If this is the case, you may wish to rename your intermediate table attribute to `subscription` instead of `pivot`. This can be done using the `as` method when defining the relationship:

```php
return $this->belongsToMany(Podcast::class)
    ->as('subscription')
    ->withTimestamps();
```

Once the custom intermediate table attribute has been specified, you may access the intermediate table data using the customized name:

```php
$users = User::with('podcasts')->get();

foreach ($users->flatMap->podcasts as $podcast) {
    echo $podcast->subscription->created_at;
}
```

### [Filtering Queries via Intermediate Table Columns](#filtering-queries-via-intermediate-table-columns)

You may filter the results returned by `belongsToMany` using the `where` method on the pivot table:

```php
$roles = User::find(1)->roles()->wherePivot('active', 1)->get();
```

### [Ordering Queries via Intermediate Table Columns](#ordering-queries-via-intermediate-table-columns)

You may order the results returned by `belongsToMany` using the `orderByPivot` method:

```php
$roles = User::find(1)->roles()->orderByPivot('created_at', 'desc')->get();
```

### [Defining Custom Intermediate Table Models](#defining-custom-intermediate-table-models)

If you would like to define a model to represent the intermediate table, you may define a "pivot" model. Pivot models extend the `Illuminate\Database\Eloquent\Relations\Pivot` class:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
    //
}
```

When defining a `belongsToMany` relationship that uses a custom pivot model, you may call the `using` method:

```php
return $this->belongsToMany(Role::class)->using(RoleUser::class);
```

Custom pivot models may have any relationships you need:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

## [Polymorphic Relationships](#polymorphic-relationships)

A polymorphic relationship allows the child model to belong to more than one type of model using a single association. For example, imagine you are building an application that allows users to share both posts and videos. In a typical relational database, you might need separate `comments` table for posts and another for videos. However, with polymorphic relationships, you can use a single `comments` table for both.

### [One to One](#one-to-one-polymorphic-relations)

#### [Table Structure](#one-to-one-polymorphic-table-structure)

A one-to-one polymorphic relationship is similar to a typical one-to-one relationship; however, the child model can belong to more than one type of model on a single association. For example, a blog `Post` and a `User` may share a polymorphic relation to an `Image` model. Using polymorphic relationships, you can have a single table of images that may be associated with either posts or users.

```
posts
    id - integer
    title - string
    body - text

users
    id - integer
    name - string
    avatar - string

images
    id - integer
    url - string
    imageable_id - integer
    imageable_type - string
```

#### [Model Structure](#one-to-one-polymorphic-model-structure)

Next, let's examine the model definitions needed to build this relationship:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class Image extends Model
{
    public function imageable(): MorphTo
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}

class User extends Model
{
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}
```

#### [Retrieving a Polymorphic Relationship](#retrieving-a-polymorphic-relationship)

Once your database table and models are defined, you may access the relationships:

```php
$post = Post::find(1);

$image = $post->image;
```

You may also retrieve the parent of the polymorphic model from the polymorphic model by accessing the method name that defines the relationship:

```php
$image = Image::find(1);

$imageable = $image->imageable;
```

### [One to Many](#one-to-many-polymorphic-relations)

#### [Table Structure](#one-to-many-polymorphic-table-structure)

One-to-many polymorphic relations are similar to typical one-to-many relations; however, the child model can belong to more than one type of model on a single association. For example, imagine users can "comment" on posts and videos. Using polymorphic relationships, you may use a single `comments` table for both of these scenarios.

```
posts
    id - integer
    title - string
    body - text

videos
    id - integer
    title - string
    url - string

comments
    id - integer
    body - text
    commentable_id - integer
    commentable_type - string
```

#### [Model Structure](#one-to-many-polymorphic-model-structure)

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Comment extends Model
{
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

class Video extends Model
{
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

#### [Retrieving a Polymorphic Relationship](#retrieving-a-polymorphic-relationship)

Once your database table and models are defined, you may access the relationships:

```php
$post = Post::find(1);

$comments = $post->comments;
```

### [One of Many](#one-of-many-polymorphic-relations)

Like "has one of many" relationships, you may also define "morph one of many" relationships that retrieve a single related model from a polymorphic relationship that may have many related models.

For example, a `Post` model might have many `Image` models, but you want to define a convenient way to access the "featured" image. You may use the `morphOne` relationship combined with the `ofMany` method:

```php
public function featuredImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->ofMany('is_featured', 'max');
}
```

For more information on the "one of many" relationship, see the [Has One of Many](#has-one-of-many) documentation.

### [Many to Many](#many-to-many-polymorphic-relations)

#### [Table Structure](#many-to-many-polymorphic-table-structure)

Polymorphic many-to-many relationships are similar to typical many-to-many relationships. For example, a blog `Post` and a `Video` model could share a polymorphic relation to a `Tag` model. Using polymorphic many-to-many relationships, you can have a single table of tags that are associated with posts or videos.

```
posts
    id - integer
    title - string
    body - text

videos
    id - integer
    title - string
    url - string

tags
    id - integer
    name - string

taggables
    tag_id - integer
    taggable_id - integer
    taggable_type - string
```

#### [Model Structure](#many-to-many-polymorphic-model-structure)

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Tag extends Model
{
    public function posts(): MorphToMany
    {
        return $this->morphedByMany(Post::class, 'taggable');
    }

    public function videos(): MorphToMany
    {
        return $this->morphedByMany(Video::class, 'taggable');
    }
}
```

#### [Retrieving a Polymorphic Relationship](#retrieving-a-polymorphic-relationship)

Once your database table and models are defined, you may access the relationships:

```php
$post = Post::find(1);

$tags = $post->tags;
```

You may also retrieve the taggable models from the `Tag` model:

```php
$tag = Tag::find(1);

$posts = $tag->posts;
```

#### [Custom Polymorphic Types](#custom-polymorphic-types)

By default, Laravel will use the fully qualified class name of the model to store the type of the related model. For example, a `Comment` that belongs to a `Post` model might have a `commentable_type` value of `App\Models\Post`.

However, you may wish to decouple your database from your application's internal structure. You may define a "morph alias" for each model that you would like to use instead of the class name:

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::enforceMorphMap([
    'post' => Post::class,
    'video' => Video::class,
]);
```

Typically, you should call this in the `boot` method of your `AppServiceProvider`. This will allow Laravel to auto-detect the model corresponding to the alias when resolving the polymorphic relationship.

#### [Dynamic Relationships](#dynamic-relationships)

The `Relation::morphMap` method allows you to define the relationship mappings outside of the model classes. However, you may also use the `HasRelationships` trait to define dynamic polymorphic relationships at runtime.

For example, if you have a `Podcast` model with multiple types of content, you may wish to define a "episodes" relationship dynamically:

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Podcast::resolveRelationUsing('episodes', function ($model) {
    return $model->morphMany(Episode::class, 'episodic');
});
```

## [Querying Relations](#querying-relations)

### [Relationship Methods vs. Dynamic Properties](#relationship-methods-vs-dynamic-properties)

If you do not need to add additional query constraints, you may access relationships as if they were public properties. For example:

```php
$user = User::find(1);

foreach ($user->posts as $post) {
    // ...
}
}
```

However, if you want to add additional constraints to the relationship query, you must call the relationship method and continue chaining query builder methods:

```php
$user = User::find(1);

$posts = $user->posts()->where('active', 1)->get();
```

### [Querying Relationship Existence](#querying-relationship-existence)

When retrieving model records, you may wish to limit your results based on whether the related models exist. For example, imagine you want to retrieve all blog posts that have at least one comment. To do so, you may pass the name of the relationship to the `has` method:

```php
use App\Models\Post;

$posts = Post::has('comments')->get();
```

You may also specify an operator and count to further customize the query:

```php
$posts = Post::has('comments', '>=', 3)->get();
```

If you need even more power, you may use the `whereHas` method to put "where" conditions on your `has` queries. These methods allow you to add customized constraints to a relationship constraint:

```php
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('status', 'approved');
})->get();
```

If you want to count the number of related models without actually loading them, use the `withCount` method:

```php
$posts = Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}
```

### [Querying Relationship Absence](#querying-relationship-absence)

When retrieving model records, you may wish to limit your results based on the absence of the related model. For example, imagine you want to retrieve all blog posts that don't have any comments. To do so, you may pass the name of the relationship to the `doesntHave` method:

```php
$posts = Post::doesntHave('comments')->get();
```

### [Querying Morph To Relationships](#querying-morph-to-relationships)

To query relationships with polymorphic types, use the `ofType` method:

```php
use App\Models\Comment;

$comments = Comment::ofType('post')->get();
```

## [Aggregating Related Models](#aggregating-related-models)

### [Counting Related Models](#counting-related-models)

If you want to count the number of related models without actually loading them, use the `withCount` method:

```php
$posts = Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}
```

### [Other Aggregate Functions](#other-aggregate-functions)

Like the `withCount` method, you may use `withMin`, `withMax`, `withAvg`, `withSum`, and `withExists` to aggregate related models:

```php
$posts = Post::withMax('comments', 'created_at')->get();

foreach ($posts as $post) {
    echo $post->comments_max_created_at;
}
```

### [Counting Related Models on Morph To Relationships](#counting-related-models-on-morph-to-relationships)

The `withExists` method may also be used to count related models on polymorphic relationships:

```php
use App\Models\Image;

$images = Image::withExists('imageable')->get();
```

## [Eager Loading](#eager-loading)

Eager loading loads an entity's relationships at the same time as the entity is retrieved:

```php
$books = Book::with('author')->get();
```

You may also eager load multiple relationships:

```php
$books = Book::with(['author', 'publisher'])->get();
```

### [Constraining Eager Loads](#constraining-eager-loads)

```php
$users = User::with(['posts' => function (Builder $query) {
    $query->where('title', 'like', '%foo%');
}])->get();
```

### [Lazy Eager Loading](#lazy-eager-loading)

```php
$books = Book::all();

$books->load('author');
```

### [Automatic Eager Loading](#automatic-eager-loading)

By default, all relationships will be automatically eager loaded when they are accessed:

```php
$user = User::find(1);

$user->posts()->first(); // Auto-loaded
```

You may disable automatic eager loading using the `preventAutomaticLoading` method:

```php
use Illuminate\Database\Eloquent\Model;

Model::preventAutomaticLoading(false);
```

### [Preventing Lazy Loading](#preventing-lazy-loading)

Laravel's `preventLazyLoading` method may be used to prevent lazy loading:

```php
Model::preventLazyLoading();
```

## [Inserting and Updating Related Models](#inserting-and-updating-related-models)

### [The `save` Method](#the-save-method)

To associate a new related model with a parent model, use the `save` method:

```php
$comment = new Comment(['message' => 'A new comment.']);

$post = Post::find(1);

$post->comments()->save($comment);
```

### [The `create` Method](#the-create-method)

```php
$post = Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
```

### [Belongs To Relationships](#updating-belongs-to-relationships)

If you would like to assign a child model to a new parent model, you may use the `associate` method:

```php
$post = Post::find(1);

$user = User::find(1);

$post->user()->associate($user);

$post->save();
```

If you would like to disassociate a child model from its parent model, use the `dissociate` method:

```php
$post->user()->dissociate();

$post->save();
```

### [Many to Many Relationships](#updating-many-to-many-relationships)

To attach a related model to a many-to-many relationship, use the `attach` method:

```php
$user = User::find(1);

$user->roles()->attach($roleId);
```

To detach a related model from a many-to-many relationship, use the `detach` method:

```php
$user->roles()->detach($roleId);
```

To sync the associations for a many-to-many relationship, use the `sync` method:

```php
$user->roles()->sync([1, 2, 3]);
```

## [Touching Parent Timestamps](#touching-parent-timestamps)

When a model belongs to another, it may be helpful to automatically update the parent's timestamp when the child model is updated. For example, when a `Comment` model is updated, you may want to automatically touch the `updated_at` timestamp of the owning `Post`. To accomplish this, add a `touches` property to the child model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    protected $touches = ['post'];

    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

Now, when you update a `Comment`, the parent `Post`'s `updated_at` column will also be updated:

```php
$comment = Comment::find(1);

$comment->text = 'Edit to this comment!';

$comment->save();
```