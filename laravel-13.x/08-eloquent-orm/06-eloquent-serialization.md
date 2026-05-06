---
title: Eloquent: Serialization
description: Learn how to serialize Eloquent models and collections to arrays and JSON.
url: https://laravel.com/docs/13.x/eloquent-serialization
tags: [data]
cssclasses:
  - data
  - color-green
color: green
---

# Eloquent: Serialization

-   [Introduction](#introduction)
-   [Serializing Models and Collections](#serializing-models-and-collections)
    -   [Serializing to Arrays](#serializing-to-arrays)
    -   [Serializing to JSON](#serializing-to-json)
-   [Hiding Attributes From JSON](#hiding-attributes-from-json)
-   [Appending Values to JSON](#appending-values-to-json)
-   [Date Serialization](#date-serialization)

## [Introduction](#introduction)

When building APIs using Laravel, you will often need to convert your models and relationships to arrays or JSON. Eloquent includes convenient methods for making these conversions, as well as controlling which attributes are included in the serialized representation of your models.

For an even more robust way of handling Eloquent model and collection JSON serialization, check out the documentation on [[08-eloquent-orm/05-eloquent-api-resources.md|Eloquent API resources]].

## [Serializing Models and Collections](#serializing-models-and-collections)

### [Serializing to Arrays](#serializing-to-arrays)

To convert a model and its loaded [[08-eloquent-orm/02-eloquent-relationships.md|relationships]] to an array, you should use the `toArray` method. This method is recursive, so all attributes and all relations (including the relations of relations) will be converted to arrays:

```php
use App\Models\User;

$user = User::with('roles')->first();

return $user->toArray();
```

The `attributesToArray` method may be used to convert a model's attributes to an array but not its relationships:

```php
$user = User::first();

return $user->attributesToArray();
```

You may also convert entire [[08-eloquent-orm/03-eloquent-collections.md|collections]] of models to arrays by calling the `toArray` method on the collection instance:

```php
$users = User::all();

return $users->toArray();
```

### [Serializing to JSON](#serializing-to-json)

To convert a model to JSON, you should use the `toJson` method. Like `toArray`, the `toJson` method is recursive, so all attributes and relations will be converted to JSON. You may also specify any JSON encoding options that are [supported by PHP](https://secure.php.net/manual/en/function.json-encode.php):

```php
use App\Models\User;

$user = User::find(1);

return $user->toJson();

return $user->toJson(JSON_PRETTY_PRINT);
```

Alternatively, you may cast a model or collection to a string, which will automatically call the `toJson` method on the model or collection:

```php
return (string) User::find(1);
```

Since models and collections are converted to JSON when cast to a string, you can return Eloquent objects directly from your application's routes or controllers. Laravel will automatically serialize your Eloquent models and collections to JSON when they are returned from routes or controllers:

```php
Route::get('/users', function () {
    return User::all();
});
```

#### [Relationships](#relationships)

When an Eloquent model is converted to JSON, its loaded relationships will automatically be included as attributes on the JSON object. Also, though Eloquent relationship methods are defined using "camel case" method names, a relationship's JSON attribute will be "snake case".

## [Hiding Attributes From JSON](#hiding-attributes-from-json)

Sometimes you may wish to limit the attributes, such as passwords, that are included in your model's array or JSON representation. To do so, you may use the `Hidden` attribute on your model. Attributes that are listed in the `Hidden` attribute will not be included in the serialized representation of your model:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Hidden;
use Illuminate\Database\Eloquent\Model;

#[Hidden(['password'])]
class User extends Model
{
    // ...
}
```

To hide relationships, add the relationship's method name to your Eloquent model's `Hidden` attribute.

Alternatively, you may use the `Visible` attribute to define an "allow list" of attributes that should be included in your model's array and JSON representation. All attributes that are not present in the `Visible` attribute will be hidden when the model is converted to an array or JSON:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Visible;
use Illuminate\Database\Eloquent\Model;

#[Visible(['first_name', 'last_name'])]
class User extends Model
{
    // ...
}
```

#### [Temporarily Modifying Attribute Visibility](#temporarily-modifying-attribute-visibility)

If you would like to make some typically hidden attributes visible on a given model instance, you may use the `makeVisible` or `mergeVisible` methods. The `makeVisible` method returns the model instance:

```php
return $user->makeVisible('attribute')->toArray();

return $user->mergeVisible(['name', 'email'])->toArray();
```

Likewise, if you would like to hide some attributes that are typically visible, you may use the `makeHidden` or `mergeHidden` methods:

```php
return $user->makeHidden('attribute')->toArray();

return $user->mergeHidden(['name', 'email'])->toArray();
```

If you wish to temporarily override all of the visible or hidden attributes, you may use the `setVisible` and `setHidden` methods respectively:

```php
return $user->setVisible(['id', 'name'])->toArray();

return $user->setHidden(['email', 'password', 'remember_token'])->toArray();
```

## [Appending Values to JSON](#appending-values-to-json)

Occasionally, when converting models to arrays or JSON, you may wish to add attributes that do not have a corresponding column in your database. To do so, first define an [[08-eloquent-orm/04-eloquent-mutators-casts.md|accessor]] for the value:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Determine if the user is an administrator.
     */
    protected function isAdmin(): Attribute
    {
        return new Attribute(
            get: fn () => 'yes',
        );
    }
}
```

If you would like the accessor to always be appended to your model's array and JSON representations, you may use the `Appends` attribute on your model. Note that attribute names are typically referenced using their "snake case" serialized representation, even though the accessor's PHP method is defined using "camel case":

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Appends;
use Illuminate\Database\Eloquent\Model;

#[Appends(['is_admin'])]
class User extends Model
{
    // ...
}
```

Once the attribute has been added to the `appends` list, it will be included in both the model's array and JSON representations. Attributes in the `appends` array will also respect the `visible` and `hidden` settings configured on the model.

#### [Appending at Run Time](#appending-at-run-time)

At runtime, you may instruct a model instance to append additional attributes using the `append` or `mergeAppends` methods. Or, you may use the `setAppends` method to override the entire array of appended properties for a given model instance:

```php
return $user->append('is_admin')->toArray();

return $user->mergeAppends(['is_admin', 'status'])->toArray();

return $user->setAppends(['is_admin'])->toArray();
```

Likewise, if you would like to remove all appended properties from a model, you may use the `withoutAppends` method:

```php
return $user->withoutAppends()->toArray();
```

## [Date Serialization](#date-serialization)

#### [Customizing the Default Date Format](#customizing-the-default-date-format)

You may customize the default serialization format by overriding the `serializeDate` method. This method does not affect how your dates are formatted for storage in the database:

```php
/**
 * Prepare a date for array / JSON serialization.
 */
protected function serializeDate(DateTimeInterface $date): string
{
    return $date->format('Y-m-d');
}
```

#### [Customizing the Date Format per Attribute](#customizing-the-date-format-per-attribute)

You may customize the serialization format of individual Eloquent date attributes by specifying the date format in the model's [[08-eloquent-orm/04-eloquent-mutators-casts.md#attribute-casting|cast declarations]]:

```php
protected function casts(): array
{
    return [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];
}
```