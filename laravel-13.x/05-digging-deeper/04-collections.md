---
title: Collections
description: A fluent, convenient wrapper for working with arrays of data in Laravel.
url: https://laravel.com/docs/13.x/collections
tags: [logic]
cssclasses:
  - logic
  - color-orange
color: orange
---

# Collections

-   [Introduction](#introduction)
    -   [Creating Collections](#creating-collections)
    -   [Extending Collections](#extending-collections)
-   [Available Methods](#available-methods)
-   [Higher Order Messages](#higher-order-messages)
-   [Lazy Collections](#lazy-collections)
    -   [Introduction](#lazy-collection-introduction)
    -   [Creating Lazy Collections](#creating-lazy-collections)
    -   [The Enumerable Contract](#the-enumerable-contract)
    -   [Lazy Collection Methods](#lazy-collection-methods)

## [Introduction](#introduction)

The `Illuminate\Support\Collection` class provides a fluent, convenient wrapper for working with arrays of data. For example, check out the following code. We'll use the `collect` helper to create a new collection instance from the array, run the `strtoupper` function on each element, and then remove all empty elements:

```php
$collection = collect(['Taylor', 'Abigail', null])->map(function (?string $name) {
    return strtoupper($name);
})->reject(function (string $name) {
    return empty($name);
});
```

As you can see, the `Collection` class allows you to chain its methods to perform fluent mapping and reducing of the underlying array. In general, collections are immutable, meaning every `Collection` method returns an entirely new `Collection` instance.

### [Creating Collections](#creating-collections)

As mentioned above, the `collect` helper returns a new `Illuminate\Support\Collection` instance for the given array. So, creating a collection is as simple as:

```php
$collection = collect([1, 2, 3]);
```

You may also create a collection using the [make](#method-make) and [fromJson](#method-fromjson) methods.

The results of [[08-eloquent-orm/01-eloquent-getting-started.md|Eloquent]] queries are always returned as `Collection` instances.

### [Extending Collections](#extending-collections)

Collections are "macroable", which allows you to add additional methods to the `Collection` class at run time. The `Illuminate\Support\Collection` class' `macro` method accepts a closure that will be executed when your macro is called. The macro closure may access the collection's other methods via `$this`, just as if it were a real method of the collection class. For example, the following code adds a `toUpper` method to the `Collection` class:

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function (string $value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```

Typically, you should declare collection macros in the `boot` method of a [[03-architecture-concepts/03-service-providers.md|service provider]].

#### [Macro Arguments](#macro-arguments)

If necessary, you may define macros that accept additional arguments:

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\Lang;

Collection::macro('toLocale', function (string $locale) {
    return $this->map(function (string $value) use ($locale) {
        return Lang::get($value, [], $locale);
    });
});

$collection = collect(['first', 'second']);

$translated = $collection->toLocale('es');

// ['primero', 'segundo'];
```

## [Available Methods](#available-methods)

For the majority of the remaining collection documentation, we'll discuss each method available on the `Collection` class. Remember, all of these methods may be chained to fluently manipulate the underlying array. Furthermore, almost every method returns a new `Collection` instance, allowing you to preserve the original copy of the collection when necessary:

[after](#method-after) [all](#method-all) [average](#method-average) [avg](#method-avg) [before](#method-before) [chunk](#method-chunk) [chunkWhile](#method-chunkwhile) [collapse](#method-collapse) [collapseWithKeys](#method-collapsewithkeys) [collect](#method-collect) [combine](#method-combine) [concat](#method-concat) [contains](#method-contains) [containsStrict](#method-containsstrict) [count](#method-count) [countBy](#method-countBy) [crossJoin](#method-crossjoin) [dd](#method-dd) [diff](#method-diff) [diffAssoc](#method-diffassoc) [diffAssocUsing](#method-diffassocusing) [diffKeys](#method-diffkeys) [doesntContain](#method-doesntcontain) [doesntContainStrict](#method-doesntcontainstrict) [dot](#method-dot) [dump](#method-dump) [duplicates](#method-duplicates) [duplicatesStrict](#method-duplicatesstrict) [each](#method-each) [eachSpread](#method-eachspread) [ensure](#method-ensure) [every](#method-every) [except](#method-except) [filter](#method-filter) [first](#method-first) [firstOrFail](#method-first-or-fail) [firstWhere](#method-first-where) [flatMap](#method-flatmap) [flatten](#method-flatten) [flip](#method-flip) [forget](#method-forget) [forPage](#method-forpage) [fromJson](#method-fromjson) [get](#method-get) [groupBy](#method-groupby) [has](#method-has) [hasAny](#method-hasany) [hasMany](#method-hasmany) [hasSole](#method-hassole) [implode](#method-implode) [intersect](#method-intersect) [intersectUsing](#method-intersectusing) [intersectAssoc](#method-intersectAssoc) [intersectAssocUsing](#method-intersectassocusing) [intersectByKeys](#method-intersectbykeys) [isEmpty](#method-isempty) [isNotEmpty](#method-isnotempty) [join](#method-join) [keyBy](#method-keyby) [keys](#method-keys) [last](#method-last) [lazy](#method-lazy) [macro](#method-macro) [make](#method-make) [map](#method-map) [mapInto](#method-mapinto) [mapSpread](#method-mapspread) [mapToGroups](#method-maptogroups) [mapWithKeys](#method-mapwithkeys) [max](#method-max) [median](#method-median) [merge](#method-merge) [mergeRecursive](#method-mergerecursive) [min](#method-min) [mode](#method-mode) [multiply](#method-multiply) [nth](#method-nth) [only](#method-only) [pad](#method-pad) [partition](#method-partition) [percentage](#method-percentage) [pipe](#method-pipe) [pipeInto](#method-pipeinto) [pipeThrough](#method-pipethrough) [pluck](#method-pluck) [pop](#method-pop) [prepend](#method-prepend) [pull](#method-pull) [push](#method-push) [put](#method-put) [random](#method-random) [range](#method-range) [reduce](#method-reduce) [reduceSpread](#method-reduce-spread) [reject](#method-reject) [replace](#method-replace) [replaceRecursive](#method-replacerecursive) [reverse](#method-reverse) [search](#method-search) [select](#method-select) [shift](#method-shift) [shuffle](#method-shuffle) [skip](#method-skip) [skipUntil](#method-skipuntil) [skipWhile](#method-skipwhile) [slice](#method-slice) [sliding](#method-sliding) [sole](#method-sole) [some](#method-some) [sort](#method-sort) [sortBy](#method-sortby) [sortByDesc](#method-sortbydesc) [sortDesc](#method-sortdesc) [sortKeys](#method-sortkeys) [sortKeysDesc](#method-sortkeysdesc) [sortKeysUsing](#method-sortkeysusing) [splice](#method-splice) [split](#method-split) [splitIn](#method-splitin) [sum](#method-sum) [take](#method-take) [takeUntil](#method-takeuntil) [takeWhile](#method-takewhile) [tap](#method-tap) [times](#method-times) [toArray](#method-toarray) [toJson](#method-tojson) [toPrettyJson](#method-to-pretty-json) [transform](#method-transform) [undot](#method-undot) [union](#method-union) [unique](#method-unique) [uniqueStrict](#method-uniquestrict) [unless](#method-unless) [unlessEmpty](#method-unlessempty) [unlessNotEmpty](#method-unlessnotempty) [unwrap](#method-unwrap) [value](#method-value) [values](#method-values) [when](#method-when) [whenEmpty](#method-whenempty) [whenNotEmpty](#method-whennotempty) [where](#method-where) [whereStrict](#method-wherestrict) [whereBetween](#method-wherebetween) [whereIn](#method-wherein) [whereInStrict](#method-whereinstrict) [whereInstanceOf](#method-whereinstanceof) [whereNotBetween](#method-wherenotbetween) [whereNotIn](#method-wherenotin) [whereNotInStrict](#method-wherenotinstrict) [whereNotNull](#method-wherenotnull) [whereNull](#method-wherenull) [wrap](#method-wrap) [zip](#method-zip)

## [Method Listing](#method-listing)

#### [`after()`](#method-after)

The `after` method returns the item after the given item. `null` is returned if the given item is not found or is the last item:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->after(3);

// 4

$collection->after(5);

// null
```

This method searches for the given item using "loose" comparison, meaning a string containing an integer value will be considered equal to an integer of the same value. To use "strict" comparison, you may provide the `strict` argument to the method:

```php
collect([2, 4, 6, 8])->after('4', strict: true);

// null
```

Alternatively, you may provide your own closure to search for the first item that passes a given truth test:

```php
collect([2, 4, 6, 8])->after(function (int $item, int $key) {
    return $item > 5;
});

// 8
```

#### [`all()`](#method-all)

The `all` method returns the underlying array represented by the collection:

```php
collect([1, 2, 3])->all();

// [1, 2, 3]
```

#### [`average()`](#method-average)

Alias for the [avg](#method-avg) method.

#### [`avg()`](#method-avg)

The `avg` method returns the [average value](https://en.wikipedia.org/wiki/Average) of a given key:

```php
$average = collect([
    ['foo' => 10],
    ['foo' => 10],
    ['foo' => 20],
    ['foo' => 40]
])->avg('foo');

// 20

$average = collect([1, 1, 2, 4])->avg();

// 2
```

#### [`before()`](#method-before)

The `before` method is the opposite of the [after](#method-after) method. It returns the item before the given item. `null` is returned if the given item is not found or is the first item:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->before(3);

// 2

$collection->before(1);

// null

collect([2, 4, 6, 8])->before('4', strict: true);

// null

collect([2, 4, 6, 8])->before(function (int $item, int $key) {
    return $item > 5;
});

// 4
```

#### [`chunk()`](#method-chunk)

The `chunk` method breaks the collection into multiple, smaller collections of a given size:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7]);

$chunks = $collection->chunk(4);

$chunks->all();

// [[1, 2, 3, 4], [5, 6, 7]]
```

This method is especially useful in [[04-the-basics/07-views.md|views]] when working with a grid system such as [[08-eloquent-orm/01-eloquent-getting-started.md|Bootstrap](https://getbootstrap.com/docs/5.3/layout/grid/). For example, imagine you have a collection of [Eloquent]] models you want to display in a grid:

```blade
@foreach ($products->chunk(3) as $chunk)
    <div class="row">
        @foreach ($chunk as $product)
            <div class="col-xs-4">{{ $product->name }}</div>
        @endforeach
    </div>
@endforeach
```

#### [`chunkWhile()`](#method-chunkwhile)

The `chunkWhile` method breaks the collection into multiple, smaller collections based on the evaluation of the given callback. The `$chunk` variable passed to the closure may be used to inspect the previous element:

```php
$collection = collect(str_split('AABBCCCD'));

$chunks = $collection->chunkWhile(function (string $value, int $key, Collection $chunk) {
    return $value === $chunk->last();
});

$chunks->all();

// [['A', 'A'], ['B', 'B'], ['C', 'C', 'C'], ['D']]
```

#### [`collapse()`](#method-collapse)

The `collapse` method collapses a collection of arrays or collections into a single, flat collection:

```php
$collection = collect([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]);

$collapsed = $collection->collapse();

$collapsed->all();

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### [`collapseWithKeys()`](#method-collapsewithkeys)

The `collapseWithKeys` method flattens a collection of arrays or collections into a single collection, keeping the original keys intact. If the collection is already flat, it will return an empty collection:

```php
$collection = collect([
    ['first'  => collect([1, 2, 3])],
    ['second' => [4, 5, 6]],
    ['third'  => collect([7, 8, 9])]
]);

$collapsed = $collection->collapseWithKeys();

$collapsed->all();

// [
//     'first'  => [1, 2, 3],
//     'second' => [4, 5, 6],
//     'third'  => [7, 8, 9],
// ]
```

#### [`collect()`](#method-collect)

The `collect` method returns a new `Collection` instance with the items currently in the collection:

```php
$collectionA = collect([1, 2, 3]);

$collectionB = $collectionA->collect();

$collectionB->all();

// [1, 2, 3]
```

The `collect` method is primarily useful for converting [lazy collections](#lazy-collections) into standard `Collection` instances:

```php
$lazyCollection = LazyCollection::make(function () {
    yield 1;
    yield 2;
    yield 3;
});

$collection = $lazyCollection->collect();

$collection::class;

// 'Illuminate\Support\Collection'

$collection->all();

// [1, 2, 3]
```

The `collect` method is especially useful when you have an instance of `Enumerable` and need a non-lazy collection instance. Since `collect()` is part of the `Enumerable` contract, you can safely use it to get a `Collection` instance.

#### [`combine()`](#method-combine)

The `combine` method combines the values of the collection, as keys, with the values of another array or collection:

```php
$collection = collect(['name', 'age']);

$combined = $collection->combine(['George', 29]);

$combined->all();

// ['name' => 'George', 'age' => 29]
```

#### [`concat()`](#method-concat)

The `concat` method appends the given array or collection's values onto the end of another collection:

```php
$collection = collect(['John Doe']);

$concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

$concatenated->all();

// ['John Doe', 'Jane Doe', 'Johnny Doe']
```

The `concat` method numerically reindexes keys for items concatenated onto the original collection. To maintain keys in associative collections, see the [merge](#method-merge) method.

#### [`contains()`](#method-contains)

The `contains` method determines whether the collection contains a given item. You may pass a closure to the `contains` method to determine if an element exists in the collection matching a given truth test:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->contains(function (int $value, int $key) {
    return $value > 5;
});

// false
```

Alternatively, you may pass a string to the `contains` method to determine whether the collection contains a given item value:

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->contains('Desk');

// true

$collection->contains('New York');

// false
```

You may also pass a key / value pair to the `contains` method, which will determine if the given pair exists in the collection:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->contains('product', 'Bookcase');

// false
```

The `contains` method uses "loose" comparisons when checking item values, meaning a string with an integer value will be considered equal to an integer of the same value. Use the [containsStrict](#method-containsstrict) method to filter using "strict" comparisons.

For the inverse of `contains`, see the [doesntContain](#method-doesntcontain) method.

#### [`containsStrict()`](#method-containsstrict)

This method has the same signature as the [contains](#method-contains) method; however, all values are compared using "strict" comparisons.

This method's behavior is modified when using [[08-eloquent-orm/03-eloquent-collections.md#method-contains|Eloquent Collections]].

#### [`count()`](#method-count)

The `count` method returns the total number of items in the collection:

```php
$collection = collect([1, 2, 3, 4]);

$collection->count();

// 4
```

#### [`countBy()`](#method-countBy)

The `countBy` method counts the occurrences of values in the collection. By default, the method counts the occurrences of every element, allowing you to count certain "types" of elements in the collection:

```php
$collection = collect([1, 2, 2, 2, 3]);

$counted = $collection->countBy();

$counted->all();

// [1 => 1, 2 => 3, 3 => 1]
```

You may pass a closure to the `countBy` method to count all items by a custom value:

```php
$collection = collect(['[email protected]', '[email protected]', '[email protected]']);

$counted = $collection->countBy(function (string $email) {
    return substr(strrchr($email, '@'), 1);
});

$counted->all();

// ['gmail.com' => 2, 'yahoo.com' => 1]
```

#### [`crossJoin()`](#method-crossjoin)

The `crossJoin` method cross joins the collection's values among the given arrays or collections, returning a Cartesian product with all possible permutations:

```php
$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b']);

$matrix->all();

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

$matrix->all();

/*
    [
        [1, 'a', 'I'],
        [1, 'a', 'II'],
        [1, 'b', 'I'],
        [1, 'b', 'II'],
        [2, 'a', 'I'],
        [2, 'a', 'II'],
        [2, 'b', 'I'],
        [2, 'b', 'II'],
    ]
*/
```

#### [`dd()`](#method-dd)

The `dd` method dumps the collection's items and ends execution of the script:

```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dd();

/*
    array:2 [
        0 => "John Doe"
        1 => "Jane Doe"
    ]
*/
```

If you do not want to stop executing the script, use the [dump](#method-dump) method instead.

#### [`diff()`](#method-diff)

The `diff` method compares the collection against another collection or a plain PHP `array` based on its values. This method will return the values in the original collection that are not present in the given collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$diff = $collection->diff([2, 4, 6, 8]);

$diff->all();

// [1, 3, 5]
```

This method's behavior is modified when using [[08-eloquent-orm/03-eloquent-collections.md#method-diff|Eloquent Collections]].

#### [`diffAssoc()`](#method-diffassoc)

The `diffAssoc` method compares the collection against another collection or a plain PHP `array` based on its keys and values. This method will return the key / value pairs in the original collection that are not present in the given collection:

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$diff = $collection->diffAssoc([
    'color' => 'yellow',
    'type' => 'fruit',
    'remain' => 3,
    'used' => 6,
]);

$diff->all();

// ['color' => 'orange', 'remain' => 6]
```

#### [`diffAssocUsing()`](#method-diffassocusing)

Unlike `diffAssoc`, `diffAssocUsing` accepts a user supplied callback function for the indices comparison:

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$diff = $collection->diffAssocUsing([
    'Color' => 'yellow',
    'Type' => 'fruit',
    'Remain' => 3,
], 'strnatcasecmp');

$diff->all();

// ['color' => 'orange', 'remain' => 6]
```

The callback must be a comparison function that returns an integer less than, equal to, or greater than zero. For more information, refer to the PHP documentation on [array\_diff\_uassoc](https://www.php.net/array_diff_uassoc#refsect1-function.array-diff-uassoc-parameters), which is the PHP function that the `diffAssocUsing` method utilizes internally.

#### [`diffKeys()`](#method-diffkeys)

The `diffKeys` method compares the collection against another collection or a plain PHP `array` based on its keys. This method will return the key / value pairs in the original collection that are not present in the given collection:

```php
$collection = collect([
    'one' => 10,
    'two' => 20,
    'three' => 30,
    'four' => 40,
    'five' => 50,
]);

$diff = $collection->diffKeys([
    'two' => 2,
    'four' => 4,
    'six' => 6,
    'eight' => 8,
]);

$diff->all();

// ['one' => 10, 'three' => 30, 'five' => 50]
```

#### [`doesntContain()`](#method-doesntcontain)

The `doesntContain` method determines whether the collection does not contain a given item. You may pass a closure to the `doesntContain` method to determine if an element does not exist in the collection matching a given truth test:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->doesntContain(function (int $value, int $key) {
    return $value < 5;
});

// false
```

Alternatively, you may pass a string to the `doesntContain` method to determine whether the collection does not contain a given item value:

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->doesntContain('Table');

// true

$collection->doesntContain('Desk');

// false
```

You may also pass a key / value pair to the `doesntContain` method, which will determine if the given pair does not exist in the collection:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->doesntContain('product', 'Bookcase');

// true
```

The `doesntContain` method uses "loose" comparisons when checking item values, meaning a string with an integer value will be considered equal to an integer of the same value.

#### [`doesntContainStrict()`](#method-doesntcontainstrict)

This method has the same signature as the [doesntContain](#method-doesntcontain) method; however, all values are compared using "strict" comparisons.

#### [`dot()`](#method-dot)

The `dot` method flattens a multi-dimensional collection into a single level collection that uses "dot" notation to indicate depth:

```php
$collection = collect(['products' => ['desk' => ['price' => 100]]]);

$flattened = $collection->dot();

$flattened->all();

// ['products.desk.price' => 100]
```

#### [`dump()`](#method-dump)

The `dump` method dumps the collection's items:

```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dump();

/*
    array:2 [
        0 => "John Doe"
        1 => "Jane Doe"
    ]
*/
```

If you want to stop executing the script after dumping the collection, use the [dd](#method-dd) method instead.

#### [`duplicates()`](#method-duplicates)

The `duplicates` method retrieves and returns duplicate values from the collection:

```php
$collection = collect(['a', 'b', 'a', 'c', 'b']);

$collection->duplicates();

// [2 => 'a', 4 => 'b']
```

If the collection contains arrays or objects, you can pass the key of the attributes that you wish to check for duplicate values:

```php
$employees = collect([
    ['email' => '[email protected]', 'position' => 'Developer'],
    ['email' => '[email protected]', 'position' => 'Designer'],
    ['email' => '[email protected]', 'position' => 'Developer'],
]);

$employees->duplicates('position');

// [2 => 'Developer']
```

#### [`duplicatesStrict()`](#method-duplicatesstrict)

This method has the same signature as the [duplicates](#method-duplicates) method; however, all values are compared using "strict" comparisons.

#### [`each()`](#method-each)

The `each` method iterates over the items in the collection and passes each item to a closure:

```php
$collection = collect([1, 2, 3, 4]);

$collection->each(function (int $item, int $key) {
    // ...
});
```

If you would like to stop iterating through the items, you may return `false` from your closure:

```php
$collection->each(function (int $item, int $key) {
    if (/* condition */) {
        return false;
    }
});
```

#### [`eachSpread()`](#method-eachspread)

The `eachSpread` method iterates over the collection's items, passing each nested item value into the given callback:

```php
$collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

$collection->eachSpread(function (string $name, int $age) {
    // ...
});
```

You may stop iterating through the items by returning `false` from the callback:

```php
$collection->eachSpread(function (string $name, int $age) {
    return false;
});
```

#### [`ensure()`](#method-ensure)

The `ensure` method may be used to verify that all elements of a collection are of a given type or list of types. Otherwise, an `UnexpectedValueException` will be thrown:

```php
return $collection->ensure(User::class);

return $collection->ensure([User::class, Customer::class]);
```

Primitive types such as `string`, `int`, `float`, `bool`, and `array` may also be specified:

```php
return $collection->ensure('int');
```

The `ensure` method does not guarantee that elements of different types will not be added to the collection at a later time.

#### [`every()`](#method-every)

The `every` method may be used to verify that all elements of a collection pass a given truth test:

```php
collect([1, 2, 3, 4])->every(function (int $value, int $key) {
    return $value > 2;
});

// false
```

If the collection is empty, the `every` method will return true:

```php
$collection = collect([]);

$collection->every(function (int $value, int $key) {
    return $value > 2;
});

// true
```

#### [`except()`](#method-except)

The `except` method returns all items in the collection except for those with the specified keys:

```php
$collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

$filtered = $collection->except(['price', 'discount']);

$filtered->all();

// ['product_id' => 1]
```

For the inverse of `except`, see the [only](#method-only) method.

This method's behavior is modified when using [[08-eloquent-orm/03-eloquent-collections.md#method-except|Eloquent Collections]].

#### [`filter()`](#method-filter)

The `filter` method filters the collection using the given callback, keeping only those items that pass a given truth test:

```php
$collection = collect([1, 2, 3, 4]);

$filtered = $collection->filter(function (int $value, int $key) {
    return $value > 2;
});

$filtered->all();

// [3, 4]
```

If no callback is supplied, all entries of the collection that are equivalent to `false` will be removed:

```php
$collection = collect([1, 2, 3, null, false, '', 0, []]);

$collection->filter()->all();

// [1, 2, 3]
```

For the inverse of `filter`, see the [reject](#method-reject) method.

#### [`first()`](#method-first)

The `first` method returns the first element in the collection that passes a given truth test:

```php
collect([1, 2, 3, 4])->first(function (int $value, int $key) {
    return $value > 2;
});

// 3
```

You may also call the `first` method with no arguments to get the first element in the collection. If the collection is empty, `null` is returned:

```php
collect([1, 2, 3, 4])->first();

// 1
```

#### [`firstOrFail()`](#method-first-or-fail)

The `firstOrFail` method is identical to the `first` method; however, if no result is found, an `Illuminate\Support\ItemNotFoundException` exception will be thrown:

```php
collect([1, 2, 3, 4])->firstOrFail(function (int $value, int $key) {
    return $value > 5;
});

// Throws ItemNotFoundException...
```

You may also call the `firstOrFail` method with no arguments to get the first element in the collection. If the collection is empty, an `Illuminate\Support\ItemNotFoundException` exception will be thrown:

```php
collect([])->firstOrFail();

// Throws ItemNotFoundException...
```

#### [`firstWhere()`](#method-first-where)

The `firstWhere` method returns the first element in the collection with the given key / value pair:

```php
$collection = collect([
    ['name' => 'Regena', 'age' => null],
    ['name' => 'Linda', 'age' => 14],
    ['name' => 'Diego', 'age' => 23],
    ['name' => 'Linda', 'age' => 84],
]);

$collection->firstWhere('name', 'Linda');

// ['name' => 'Linda', 'age' => 14]
```

You may also call the `firstWhere` method with a comparison operator:

```php
$collection->firstWhere('age', '>=', 18);

// ['name' => 'Diego', 'age' => 23]
```

Like the [where](#method-where) method, you may pass one argument to the `firstWhere` method. In this scenario, the `firstWhere` method will return the first item where the given item key's value is "truthy":

```php
$collection->firstWhere('age');

// ['name' => 'Linda', 'age' => 14]
```

#### [`flatMap()`](#method-flatmap)

The `flatMap` method iterates through the collection and passes each value to the given closure. The closure is free to modify the item and return it, thus forming a new collection of modified items. Then, the array is flattened by one level:

```php
$collection = collect([
    ['name' => 'Sally'],
    ['school' => 'Arkansas'],
    ['age' => 28]
]);

$flattened = $collection->flatMap(function (array $values) {
    return array_map('strtoupper', $values);
});

$flattened->all();

// ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];
```

#### [`flatten()`](#method-flatten)

The `flatten` method flattens a multi-dimensional collection into a single dimension:

```php
$collection = collect([
    'name' => 'Taylor',
    'languages' => [
        'PHP', 'JavaScript'
    ]
]);

$flattened = $collection->flatten();

$flattened->all();

// ['Taylor', 'PHP', 'JavaScript'];
```

If necessary, you may pass the `flatten` method a "depth" argument:

```php
$collection = collect([
    'Apple' => [
        [
            'name' => 'iPhone 6S',
            'brand' => 'Apple'
        ],
    ],
    'Samsung' => [
        [
            'name' => 'Galaxy S7',
            'brand' => 'Samsung'
        ],
    ],
]);

$products = $collection->flatten(1);

$products->values()->all();

/*
    [
        ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
    ]
*/
```

In this example, calling `flatten` without providing the depth would have also flattened the nested arrays, resulting in `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`. Providing a depth allows you to specify the number of levels nested arrays will be flattened.

#### [`flip()`](#method-flip)

The `flip` method swaps the collection's keys with their corresponding values:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

$flipped = $collection->flip();

$flipped->all();

// ['Taylor' => 'name', 'Laravel' => 'framework']
```

#### [`forget()`](#method-forget)

The `forget` method removes an item from the collection by its key:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

// Forget a single key...
$collection->forget('name');

// ['framework' => 'Laravel']

// Forget multiple keys...
$collection->forget(['name', 'framework']);

// []
```

Unlike most other collection methods, `forget` does not return a new modified collection; it modifies and returns the collection it is called on.

#### [`forPage()`](#method-forpage)

The `forPage` method returns a new collection containing the items that would be present on a given page number. The method accepts the page number as its first argument and the number of items to show per page as its second argument:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

$chunk = $collection->forPage(2, 3);

$chunk->all();

// [4, 5, 6]
```

#### [`fromJson()`](#method-fromjson)

The static `fromJson` method creates a new collection instance by decoding a given JSON string using the `json_decode` PHP function:

```php
use Illuminate\Support\Collection;

$json = json_encode([
    'name' => 'Taylor Otwell',
    'role' => 'Developer',
    'status' => 'Active',
]);

$collection = Collection::fromJson($json);
```

#### [`get()`](#method-get)

The `get` method returns the item at a given key. If the key does not exist, `null` is returned:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

$value = $collection->get('name');

// Taylor
```

You may optionally pass a default value as the second argument:

```php
$collection = collect(['name' => 'Taylor', 'framework' => 'Laravel']);

$value = $collection->get('age', 34);

// 34
```

You may even pass a callback as the method's default value. The result of the callback will be returned if the specified key does not exist:

```php
$collection->get('email', function () {
    return '[email protected]';
});

// [email protected]
```

#### [`groupBy()`](#method-groupby)

The `groupBy` method groups the collection's items by a given key:

```php
$collection = collect([
    ['account_id' => 'account-x10', 'product' => 'Chair'],
    ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ['account_id' => 'account-x11', 'product' => 'Desk'],
]);

$grouped = $collection->groupBy('account_id');

$grouped->all();

/*
    [
        'account-x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'account-x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

Instead of passing a string `key`, you may pass a callback. The callback should return the value you wish to key the group by:

```php
$grouped = $collection->groupBy(function (array $item, int $key) {
    return substr($item['account_id'], -3);
});

$grouped->all();

/*
    [
        'x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

Multiple grouping criteria may be passed as an array. Each array element will be applied to the corresponding level within a multi-dimensional array:

```php
$collection = collect([
    ['account_id' => 'account-x10', 'product' => 'Chair'],
    ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ['account_id' => 'account-x11', 'product' => 'Desk'],
]);

$grouped = $collection->groupBy(['account_id', 'product']);

$grouped->all();

/*
    [
        'account-x10' => [
            'Chair' => [
                ['account_id' => 'account-x10', 'product' => 'Chair'],
            ],
            'Bookcase' => [
                ['account_id' => 'account-x10', 'product' => 'Bookcase'],
            ],
        ],
        'account-x11' => [
            'Desk' => [
                ['account_id' => 'account-x11', 'product' => 'Desk'],
            ],
        ],
    ]
*/
```

The `groupBy` method uses "loose" comparisons when checking the key. Use the [groupByStrict](#method-groupbystrict) method to filter using "strict" comparisons.

This method's behavior is modified when using [[08-eloquent-orm/03-eloquent-collections.md#method-groupby|Eloquent Collections]].

#### [`has()`](#method-has)

The `has` method determines whether the collection contains a given key:

```php
$collection = collect(['account_id' => 1, 'amount' => 100]);

$collection->has('amount');

// true

$collection->has('email');

// false
```

#### [`hasAny()`](#method-hasany)

The `hasAny` method determines whether the collection contains any of the given keys:

```php
$collection = collect(['account_id' => 1, 'amount' => 100]);

$collection->hasAny(['account_id', 'email']);

// true

$collection->hasAny(['email', 'first_name']);

// false
```

#### [`hasMany()`](#method-hasmany)

The `hasMany` method returns `true` when the collection has more than one element. Otherwise, the method should return `false`:

```php
$collection = collect([]);

$collection->hasMany();

// false
```

#### [`hasSole()`](#method-hassole)

The `hasSole` method returns `true` when the collection has exactly one element. Otherwise, the method should return `false`:

```php
$collection = collect(['one' => 1]);

$collection->hasSole();

// true
```

#### [`implode()`](#method-implode)

The `implode` method joins the collection's values with a string. For primitive arrays, you may pass a string to use as the separator:

```php
$collection = collect([
    ['account_id' => 1, 'amount' => 50],
    ['account_id' => 2, 'amount' => 100],
]);

$collection->implode('amount', ', ');

// 50, 100
```

You may also pass a closure to format the items being joined:

```php
$collection = collect([
    ['account_id' => 1, 'amount' => 50],
    ['account_id' => 2, 'amount' => 100],
]);

$collection->implode(function (array $item, int $key) {
    return $item['account_id'] . ': ' . $item['amount'];
}, "\n");

// 1: 50
// 2: 100
```

#### [`intersect()`](#method-intersect)

The `intersect` method compares the collection against another collection or a plain PHP `array` based on its values. This method will return the values in the original collection that are also present in the given collection:

```php
$collection = collect(['Desk', 'Chair', 'Bookcase']);

$intersected = $collection->intersect(['Desk', 'Chair', 'Table']);

$intersected->all();

// [0 => 'Desk', 1 => 'Chair']
```

#### [`intersectUsing()`](#method-intersectusing)

Unlike `intersect`, `intersectUsing` accepts a user supplied callback function for the indices comparison:

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$intersected = $collection->intersectUsing([
    'Color' => 'yellow',
    'Type' => 'fruit',
    'Remain' => 3,
], 'strnatcasecmp');

$intersected->all();

// [
//     'type' => 'fruit',
// ]
```

#### [`intersectAssoc()`](#method-intersectAssoc)

The `intersectAssoc` method compares the collection against another collection or array based on its keys and values. This method returns the key/value pairs in the original collection that are also present in the given collection:

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$intersected = $collection->intersectAssoc([
    'color' => 'yellow',
    'type' => 'fruit',
    'remain' => 3,
    'used' => 6,
]);

$intersected->all();

// ['type' => 'fruit']
```

#### [`intersectAssocUsing()`](#method-intersectassocusing)

Unlike `intersectAssoc`, `intersectAssocUsing` accepts a user supplied callback function for the indices comparison:

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$intersected = $collection->intersectAssocUsing([
    'Color' => 'yellow',
    'Type' => 'fruit',
    'Remain' => 3,
], 'strnatcasecmp');

$intersected->all();

// ['type' => 'fruit']
```

#### [`intersectByKeys()`](#method-intersectbykeys)

The `intersectByKeys` method compares the collection against another collection or a plain PHP `array` based on its keys. This method will return the key / value pairs in the original collection that are also present in the given collection:

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6,
]);

$intersected = $collection->intersectByKeys([
    'color' => 'yellow',
    'type' => 'fruit',
    'used' => 6,
]);

$intersected->all();

// ['color' => 'orange', 'type' => 'fruit']
```

#### [`isEmpty()`](#method-isempty)

The `isEmpty` method returns `true` if the collection is empty; otherwise, `false` is returned:

```php
collect([])->isEmpty();

// true
```

#### [`isNotEmpty()`](#method-isnotempty)

The `isNotEmpty` method returns `true` if the collection is not empty; otherwise, `false` is returned:

```php
collect([])->isNotEmpty();

// false
```

#### [`join()`](#method-join)

The `join` method joins the collection's values with a string:

```php
collect(['a', 'b', 'c'])->join(', ');

// a, b, c

collect(['a', 'b', 'c'])->join(', ', ' and ');

// a, b and c
```

#### [`keyBy()`](#method-keyby)

The `keyBy` method keys the collection by the given key. If the key exists on multiple items, only the last item will be retained in the new collection:

```php
$collection = collect([
    ['id' => '213', 'name' => 'Darrell'],
    ['id' => '312', 'name' => 'Meredith'],
]);

$keyed = $collection->keyBy('id');

$keyed->all();

/*
    [
        '213' => ['id' => '213', 'name' => 'Darrell'],
        '312' => ['id' => '312', 'name' => 'Meredith'],
    ]
*/
```

You may also pass a callback. The callback should return the value to key the collection by:

```php
$keyed = $collection->keyBy(function (array $item) {
    return strtoupper($item['name']);
});

$keyed->all();

/*
    [
        'DARRELL' => ['id' => '213', 'name' => 'Darrell'],
        'MEREDITH' => ['id' => '312', 'name' => 'Meredith'],
    ]
*/
```

#### [`keys()`](#method-keys)

The `keys` method returns all of the collection's keys:

```php
$collection = collect([
    'name' => 'Desk',
    'price' => 100,
]);

$keys = $collection->keys();

// ['name', 'price']
```

#### [`last()`](#method-last)

The `last` method returns the last element in the collection that passes a given truth test:

```php
collect([1, 2, 3, 4])->last(function (int $value, int $key) {
    return $value > 2;
});

// 4
```

You may also call the `last` method with no arguments to get the last element in the collection. If the collection is empty, `null` is returned:

```php
collect([1, 2, 3, 4])->last();

// 4
```

#### [`lazy()`](#method-lazy)

The `lazy` method returns a [LazyCollection](#lazy-collections) instance of the collection:

```php
$lazyCollection = collect([1, 2, 3, 4])->lazy();

$lazyCollection::class;

// 'Illuminate\Support\LazyCollection'
```

This method is useful for converting an eager collection to a lazy one in order to reduce memory usage when iterating over large datasets. See the [Lazy Collections](#lazy-collections) section for more information.

#### [`macro()`](#method-macro)

The static `macro` method allows you to add methods to the `Collection` class at runtime. See the section on [Extending Collections](#extending-collections) for more information.

#### [`make()`](#method-make)

The static `make` method creates a new collection instance. See the section on [Creating Collections](#creating-collections) for more information.

#### [`map()`](#method-map)

The `map` method iterates over the collection and passes each value to the given callback. The callback is free to modify the item and return it, thus forming a new collection of modified items:

```php
$collection = collect([1, 2, 3, 4, 5]);

$multiplied = $collection->map(function (int $item, int $key) {
    return $item * 2;
});

$multiplied->all();

// [2, 4, 6, 8, 10]
```

Like most collection methods, `map` returns a new `Collection` instance. It does not modify the original collection. If you want to transform the original collection instead, use the [transform](#method-transform) method.

#### [`mapInto()`](#method-mapinto)

The `mapInto` method iterates over the collection, creating a new instance of the given class for each element:

```php
$collection = collect(['1', '2', '3', '4']);

$collection->mapInto('integer');

/*
    Illuminate\Support\Collection Object
    (
        [0] => 1
        [1] => 2
        [2] => 3
        [3] => 4
    )
*/
```

#### [`mapSpread()`](#method-mapspread)

The `mapSpread` method iterates over the collection, passing each nested item value into the given callback:

```php
$collection = collect([
    [1, 2],
    [3, 4],
]);

$multiplied = $collection->mapSpread(function (int $one, int $two) {
    return $one * $two;
});

$multiplied->all();

// [2, 12]
```

#### [`mapToGroups()`](#method-maptogroups)

The `mapToGroups` method groups the collection by the given callback and then maps each group's values to a single value:

```php
$collection = collect([
    ['id' => 1, 'name' => 'Ruby'],
    ['id' => 2, 'name' => 'Ruby'],
    ['id' => 3, 'name' => 'PHP'],
    ['id' => 4, 'name' => 'PHP'],
]);

$grouped = $collection->mapToGroups(function (array $item, int $key) {
    return [$item['name'] => $item['id']];
});

$grouped->all();

/*
    [
        'Ruby' => [1, 2],
        'PHP' => [3, 4],
    ]
*/
```

#### [`mapWithKeys()`](#method-mapwithkeys)

The `mapWithKeys` method iterates over the collection and passes each value to the given callback. The callback should return an associative array with a single key/value pair:

```php
$collection = collect([
    ['id' => 1, 'name' => 'A'],
    ['id' => 2, 'name' => 'B'],
]);

$collection->mapWithKeys(function (array $item, int $key) {
    return [$item['name'] => $item['id']];
});

$collection->all();

/*
    [
        'A' => 1,
        'B' => 2,
    ]
*/
```

#### [`max()`](#method-max)

The `max` method returns the maximum value of a given key:

```php
$max = collect([
    ['foo' => 10],
    ['foo' => 20],
])->max('foo');

// 20

$max = collect([1, 2, 3])->max();

// 3
```

#### [`median()`](#method-median)

The `median` method returns the [median](https://en.wikipedia.org/wiki/Median) value of a given key:

```php
$median = collect([
    ['foo' => 1],
    ['foo' => 2],
    ['foo' => 3],
    ['foo' => 4],
])->median('foo');

// 2.5

$median = collect([1, 1, 1, 1, 1, 2, 2, 3, 4, 4])->median();

// 2
```

#### [`merge()`](#method-merge)

The `merge` method merges the given array or collection's items with the original collection:

```php
$collection = collect(['product_id' => 1, 'price' => 100]);

$merged = $collection->merge(['price' => 200, 'discount' => false]);

$merged->all();

// ['product_id' => 1, 'price' => 200, 'discount' => false]
```

If the given collection keys are numeric, the values will be appended to the end of the original collection:

```php
$collection = collect([1, 2]);

$merged = $collection->merge([3, 4]);

$merged->all();

// [1, 2, 3, 4]
```

#### [`mergeRecursive()`](#method-mergerecursive)

The `mergeRecursive` method merges the given array or collection recursively with the original collection:

```php
$collection = collect(['products' => ['desk' => ['price' => 100]]]);

$merged = $collection->mergeRecursive(['products' => ['chair' => ['price' => 200]]]);

$merged->all();

// ['products' => ['desk' => ['price' => 100], 'chair' => ['price' => 200]]]
```

#### [`min()`](#method-min)

The `min` method returns the minimum value of a given key:

```php
$min = collect([
    ['foo' => 10],
    ['foo' => 20],
])->min('foo');

// 10

$min = collect([1, 2, 3])->min();

// 1
```

#### [`mode()`](#method-mode)

The `mode` method returns the [mode](https://en.wikipedia.org/wiki/Mode_(statistics)) value of a given key:

```php
$mode = collect([
    ['foo' => 1],
    ['foo' => 2],
    ['foo' => 1],
])->mode('foo');

// [1]

$mode = collect([1, 1, 2, 4])->mode();

// [1]
```

#### [`multiply()`](#method-multiply)

The `multiply` method returns the product of all items in the collection. If the collection contains nested arrays or objects, you should provide the key that should be used to calculate the product:

```php
$multiplied = collect([
    ['name' => 'Desk', 'price' => 100],
    ['name' => 'Table', 'price' => 200],
])->multiply('price');

// 20000

$multiplied = collect([2, 2, 2])->multiply();

// 8
```

You may also provide a "default" value that will be returned if the collection is empty:

```php
$multiplied = collect([])->multiply('price', 0);

// 0

$multiplied = collect([])->multiply();

// 0
```

#### [`nth()`](#method-nth)

The `nth` method creates a new collection consisting of every n-th element:

```php
$collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

$collection->nth(4)->all();

// ['d']
```

#### [`only()`](#method-only)

The `only` method returns the items in the collection with the specified keys:

```php
$collection = collect([
    'product_id' => 1,
    'name' => 'Desk',
    'price' => 100,
    'discount' => false,
]);

$filtered = $collection->only(['product_id', 'name']);

$filtered->all();

// ['product_id' => 1, 'name' => 'Desk']
```

For the inverse of `only`, see the [except](#method-except) method.

#### [`pad()`](#method-pad)

The `pad` method will pad the collection to the specified length with a given value. If the `pad` method is called on the left side of the collection (negative index), the padding will be inserted before the first item:

```php
$collection = collect(['A', 'B', 'C']);

$collection->pad(5, 0)->all();

// [0, 0, 'A', 'B', 'C']

$collection = collect(['A', 'B', 'C']);

$collection->pad(-5, 0)->all();

// ['A', 'B', 'C', 0, 0]
```

#### [`partition()`](#method-partition)

The `partition` method returns two collections, one containing the elements that pass the given truth test and one containing all other elements:

```php
$collection = collect([1, 2, 3, 4, 5, 6]);

[$underThree, $threeOrAbove] = $collection->partition(function (int $item) {
    return $item < 3;
});

$underThree->all();

// [1, 2]

$threeOrAbove->all();

// [3, 4, 5, 6]
```

#### [`percentage()`](#method-percentage)

The `percentage` method calculates the percentage value of all items in the collection:

```php
$result = collect([50, 60, 70, 80, 90])->percentage(
    fn (int $item) => $item >= 60
);

// 60.0

$result = collect([1, 2, 3, 4])->percentage(
    fn (int $item) => $item > 2
);

// 50.0

$result = collect([1, 2, 3, 4])->percentage();

// 100.0
```

If the collection is empty, `0.0` is returned:

```php
$result = collect([])->percentage();

// 0.0
```

#### [`pipe()`](#method-pipe)

The `pipe` method passes the collection to the given callback and returns the result:

```php
$collection = collect([1, 2, 3]);

$piped = $collection->pipe(function (Collection $collection) {
    return $collection->sum();
});

// 6
```

#### [`pipeInto()`](#method-pipeinto)

The `pipeInto` method wraps the collection as the given class and passes it to the given callback:

```php
class Processor
{
    public function __construct(
        public Collection $collection,
    ) {}
}

$collection = collect([1, 2, 3]);

$processor = $collection->pipeInto(Processor::class);

$processor->collection->all();

// [1, 2, 3]
```

#### [`pipeThrough()`](#method-pipethrough)

The `pipeThrough` method passes the collection into an array of callbacks and returns the result:

```php
$collection = collect([1, 2, 3]);

$piped = $collection->pipeThrough([
    function (Collection $collection) {
        return $collection->filter(fn (int $item) => $item >= 2);
    },
    function (Collection $collection) {
        return $collection->values();
    },
]);

$piped->all();

// [2, 3]
```

#### [`pluck()`](#method-pluck)

The `pluck` method retrieves all of the values for a given key:

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$plucked = $collection->pluck('name');

$plucked->all();

// ['Desk', 'Chair']

$plucked = $collection->pluck('name', 'product_id');

$plucked->all();

// ['prod-100' => 'Desk', 'prod-200' => 'Chair']
```

You may also specify how you wish to key the resulting collection by passing a second argument:

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$plucked = $collection->pluck('name', 'product_id');

$plucked->all();

/*
    [
        'prod-100' => 'Desk',
        'prod-200' => 'Chair',
    ]
*/
```

#### [`pop()`](#method-pop)

The `pop` method removes and returns the last item from the collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->pop();

// 5

$collection->all();

// [1, 2, 3, 4]
```

#### [`prepend()`](#method-prepend)

The `prepend` method adds an item to the beginning of the collection:

```php
$collection = collect([1, 2, 3, 4]);

$collection->prepend(0);

$collection->all();

// [0, 1, 2, 3, 4]
```

You may also provide a key to prepend with:

```php
$collection = collect(['first' => 1, 'second' => 2]);

$collection->prepend(0, 'zero');

$collection->all();

// ['zero' => 0, 'first' => 1, 'second' => 2]
```

#### [`pull()`](#method-pull)

The `pull` method removes and returns an item from the collection by its key:

```php
$collection = collect([1, 2, 3, 4]);

$collection->pull(3);

// 4

$collection->all();

// [1, 2]
```

#### [`push()`](#method-push)

The `push` method adds an item to the end of the collection:

```php
$collection = collect(['Desk', 'Table']);

$collection->push('Chair');

$collection->all();

// ['Desk', 'Table', 'Chair']
```

#### [`put()`](#method-put)

The `put` method sets the given key and value in the collection:

```php
$collection = collect(['Desk', 'Table']);

$collection->put(1, 'Chair');

$collection->all();

// [0 => 'Desk', 1 => 'Chair']
```

#### [`random()`](#method-random)

The `random` method returns a random item from the collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->random();

// 4 - (retrieved randomly)
```

You may pass an integer to `random` to specify how many items you wish to randomly retrieve:

```php
$collection->random(3);

// [5, 3, 4] - (retrieved randomly)
```

The collection can return the same value multiple times if the collection has fewer items than the number requested.

#### [`range()`](#method-range)

The `range` method returns a collection of integers between the start and end values:

```php
collect()->range(0, 5);

// [0, 1, 2, 3, 4, 5]

collect()->range(5, 10);

// [5, 6, 7, 8, 9, 10]
```

#### [`reduce()`](#method-reduce)

The `reduce` method reduces the collection to a single value, passing the result of each iteration into the subsequent iteration:

```php
$collection = collect([1, 2, 3]);

$total = $collection->reduce(function (int $carry, int $item) {
    return $carry + $item;
});

// 6
```

The initial value for the carry can be specified as the second argument:

```php
$collection = collect([1, 2, 3]);

$total = $collection->reduce(function (int $carry, int $item) {
    return $carry + $item;
}, 4);

// 10
```

#### [`reduceSpread()`](#method-reducespread)

The `reduceSpread` method reduces the collection to a single value, passing the result of each iteration into the subsequent iteration:

```php
$collection = collect([1, 2, 3, 4]);

[$min, $max] = $collection->reduceSpread(function (int $carry, int $item) {
    return [
        min($carry[0], $item),
        max($carry[1], $item),
    ];
}, [PHP_INT_MAX, PHP_INT_MIN]);

// [1, 4]
```

#### [`reject()`](#method-reject)

The `reject` method filters the collection using the given callback. It removes items that pass a given truth test:

```php
$collection = collect([1, 2, 3, 4]);

$filtered = $collection->reject(function (int $value, int $key) {
    return $value > 2;
});

$filtered->all();

// [1, 2]
```

If no callback is supplied, all entries that are equivalent to `false` will be removed:

```php
$collection = collect([1, 2, 3, null, false, '', 0, []]);

$filtered = $collection->reject();

$filtered->all();

// [1, 2, 3]
```

For the inverse of `reject`, see the [filter](#method-filter) method.

#### [`replace()`](#method-replace)

The `replace` method replaces items in the collection with matching items from the given array or collection:

```php
$collection = collect(['Desk', 'Table']);

$merged = $collection->replace([0 => 'Chair']);

$merged->all();

// ['Chair', 'Table']
```

#### [`replaceRecursive()`](#method-replacerecursive)

This method works like `replace`, but recursively into arrays found within the collection's values:

```php
$collection = collect([
    'products' => [
        'desk' => ['price' => 100],
    ],
]);

$merged = $replaced->replaceRecursive([
    'products' => [
        'desk' => ['price' => 200],
        'chair' => ['price' => 100],
    ],
]);

$merged->all();

/*
    [
        'products' => [
            'desk' => ['price' => 200],
            'chair' => ['price' => 100],
        ],
    ]
*/
```

#### [`reverse()`](#method-reverse)

The `reverse` method reverses the order of the collection's items:

```php
$collection = collect(['Desk', 'Table', 'Chair']);

$reversed = $collection->reverse();

$reversed->all();

// [
//     0 => 'Chair',
//     1 => 'Table',
//     2 => 'Desk',
// ]
```

#### [`search()`](#method-search)

The `search` method searches for a given value in the collection and returns its key if found. If the item is not found, `false` is returned.

```php
$collection = collect(['Desk', 'Table', 'Chair']);

$collection->search('Desk');

// 0

$collection->search('Office')->false();

// false
```

The search uses "loose" comparisons. To use "strict" comparisons, pass `true` as the second argument to the method:

```php
collect([1, 2, 3, 4])->search('4', strict: true);

// 3
```

Alternatively, you may pass your own closure to search for the first item that passes a given truth test:

```php
$collection->search(function (int $item, int $key) {
    return $item > 3;
});

// 3
```

#### [`select()`](#method-select)

The `select` method allows you to select values from an array or collection by keys:

```php
$collection = collect([
    'id' => 1,
    'name' => 'John',
    'email' => 'john@example.com',
    'role' => 'admin',
]);

$selected = $collection->select(['id', 'name']);

$selected->all();

// ['id' => 1, 'name' => 'John']
```

#### [`shift()`](#method-shift)

The `shift` method removes and returns the first item from the collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$shifted = $collection->shift();

$shifted;

// 1

$collection->all();

// [2, 3, 4, 5]
```

#### [`shuffle()`](#method-shuffle)

The `shuffle` method randomly shuffles the items in the collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$shuffled = $collection->shuffle();

$shuffled->all();

// [3, 1, 5, 2, 4] - (randomly shuffled)
```

#### [`skip()`](#method-skip)

The `skip` method returns a new collection without the first given number of items:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$skipped = $collection->skip(4);

$skipped->all();

// [5, 6, 7, 8, 9, 10]
```

#### [`skipUntil()`](#method-skipuntil)

The `skipUntil` method returns a new collection without items before the given condition is met:

```php
$collection = collect([1, 2, 3, 4]);

$skipped = $collection->skipUntil(function (int $item) {
    return $item >= 3;
});

$skipped->all();

// [3, 4]

$collection = collect([1, 2, 3, 4]);

$skipped = $collection->skipUntil(3);

$skipped->all();

// [3, 4]
```

You may also pass a value to `skipUntil` to skip all items until it finds that value:

```php
$collection = collect([1, 2, 3, 4, 5]);

$skipped = $collection->skipUntil(3);

$skipped->all();

// [3, 4, 5]
```

#### [`skipWhile()`](#method-skipwhile)

The `skipWhile` method returns a new collection without items as long as the given condition is met:

```php
$collection = collect([1, 2, 3, 4]);

$skipped = $collection->skipWhile(function (int $item) {
    return $item < 3;
});

$skipped->all();

// [3, 4]
```

#### [`slice()`](#method-slice)

The `slice` method returns a slice of the collection starting at the given index:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$sliced = $collection->slice(4);

$sliced->all();

// [5, 6, 7, 8, 9, 10]
```

If you would like to limit the size of the returned slice, pass the size as the second argument:

```php
$sliced = $collection->slice(4, 2);

$sliced->all();

// [5, 6]
```

#### [`sliding()`](#method-sliding)

The `sliding` method returns a collection of chunks (slices) of the collection, where each chunk is a slided window of a given size:

```php
$collection = collect([1, 2, 3, 4, 5]);

$sliding = $collection->sliding(2);

$sliding->all();

/*
    [
        0 => [1, 2],
        1 => [2, 3],
        2 => [3, 4],
        3 => [4, 5],
    ]
*/
```

This is similar to the [chunk](#method-chunk) method, but the `sliding` method returns overlapping slices:

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunks = $collection->chunk(2);

$chunks->all();

/*
    [
        0 => [1, 2],
        1 => [3, 4],
        2 => [5],
    ]
*/
```

The `sliding` method has a third parameter `step`, which controls how far the window moves each iteration. By default, the step is `1`, meaning each subsequent slice starts one element after the previous slice. To retrieve slices that don't share any elements, pass a step value equal to the window size:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8]);

$sliding = $collection->sliding(3, step: 3);

$sliding->all();

/*
    [
        0 => [1, 2, 3],
        1 => [4, 5, 6],
        2 => [7, 8],
    ]
*/
```

#### [`sole()`](#method-sole)

The `sole` method returns the first element in the collection that passes the given truth test, but only if the test matches exactly one element:

```php
$collection = collect([
    ['id' => 1, 'name' => 'Desk'],
    ['id' => 2, 'name' => 'Chair'],
]);

$collection->sole('id', 1);

// ['id' => 1, 'name' => 'Desk']
```

Alternatively, you may pass a closure to `sole`:

```php
$collection->sole(function (array $item) {
    return $item['id'] > 2;
});

// ['id' => 3, 'name' => 'Table']
```

If no items match, an `Illuminate\Support\ItemNotFoundException` exception will be thrown. If more than one item matches, an `Illuminate\Support\MultipleItemsFoundException` exception will be thrown.

#### [`some()`](#method-some)

Alias for the [contains](#method-contains) method.

#### [`sort()`](#method-sort)

The `sort` method sorts the collection:

```php
$collection = collect([5, 3, 1, 2, 4]);

$sorted = $collection->sort();

$sorted->values()->all();

// [1, 2, 3, 4, 5]
```

If your collection contains arrays or objects, you may pass the key of the attributes that you wish to sort by:

```php
$collection = collect([
    ['id' => 2, 'name' => 'Desk'],
    ['id' => 1, 'name' => 'Chair'],
]);

$ Sorted = $collection->sort('id');

$ Sorted->values()->all();

/*
    [
        ['id' => 1, 'name' => 'Chair'],
        ['id' => 2, 'name' => 'Desk'],
    ]
*/
```

You may also pass a callback to specify how to sort your collection:

```php
$collection = collect([
    ['id' => 2, 'name' => 'Desk'],
    ['id' => 1, 'name' => 'Chair'],
]);

$ Sorted = $collection->sort(function (array $a, array $b) {
    return $a['id'] <=> $b['id'];
});

$ Sorted->values()->all();

/*
    [
        ['id' => 1, 'name' => 'Chair'],
        ['id' => 2, 'name' => 'Desk'],
    ]
*/
```

The `sort` method uses "loose" comparisons when comparing values. If you want items to be sorted using "strict" comparisons, use the [sortStrict](#method-sortstrict) method.

#### [`sortBy()`](#method-sortby)

The `sortBy` method sorts the collection by the given key:

```php
$collection = collect([
    ['id' => 2, 'name' => 'Desk'],
    ['id' => 1, 'name' => 'Chair'],
]);

$ Sorted = $collection->sortBy('id');

$ Sorted->values()->all();

/*
    [
        ['id' => 1, 'name' => 'Chair'],
        ['id' => 2, 'name' => 'Desk'],
    ]
*/
```

You may also pass a callback to specify how to sort your collection:

```php
$ Sorted = $collection->sortBy(function (array $item) {
    return $item['id'];
});

$Sorted->values()->all();

/*
    [
        ['id' => 1, 'name' => 'Chair'],
        ['id' => 2, 'name' => 'Desk'],
    ]
*/
```

The `sortBy` method uses "loose" comparisons when comparing values. If you want items to be sorted using "strict" comparisons, use the [sortByStrict](#method-sortbystrict) method.

#### [`sortByDesc()`](#method-sortbydesc)

This method has the same signature as the [sortBy](#method-sortby) method, but will sort the collection in the opposite order.

#### [`sortDesc()`](#method-sortdesc)

This method will sort the collection in the opposite order of the [sort](#method-sort) method:

```php
$collection = collect([5, 3, 1, 2, 4]);

$sorted = $collection->sortDesc();

$sorted->values()->all();

// [5, 4, 3, 2, 1]
```

#### [`sortKeys()`](#method-sortkeys)

The `sortKeys` method sorts the collection by the keys of the underlying associative array:

```php
$collection = collect([
    'z' => 5,
    'a' => 1,
    'b' => 2,
]);

$ Sorted = $collection->sortKeys();

$ Sorted->keys()->all();

// ['a', 'b', 'z']
```

The `sortKeys` method uses "loose" comparisons when comparing keys. To use strict comparisons, see the [sortKeysStrict](#method-sortkeysstrict) method.

#### [`sortKeysDesc()`](#method-sortkeysdesc)

This method has the same signature as the [sortKeys](#method-sortkeys) method, but will sort the collection in the opposite order.

#### [`sortKeysUsing()`](#method-sortkeysusing)

The `sortKeysUsing` method sorts the collection by the keys of the underlying associative array using the given callback:

```php
$collection = collect([
    'z' => 5,
    'a' => 1,
    'b' => 2,
]);

$ Sorted = $collection->sortKeysUsing('strcmp');

$ Sorted->keys()->all();

// ['a', 'b', 'z']
```

The callback must be a comparison function that returns an integer less than, equal to, or greater than zero. For more information, refer to the PHP documentation on [uasort](https://www.php.net/uasort).

#### [`splice()`](#method-splice)

The `splice` method removes and returns a slice of items starting at the given index:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->splice(2);

// [3, 4, 5]

$collection->all();

// [1, 2]
```

You may pass a second argument to limit the size of the resulting collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->splice(2, 2);

// [3, 4]

$collection->all();

// [1, 2, 5]
```

In addition, you can pass a third parameter containing the replacement items to insert in place of the sliced items:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->splice(2, 2, [10, 11]);

$collection->all();

// [1, 2, 10, 11, 5]
```

#### [`split()`](#method-split)

The `split` method breaks the collection into the given number of groups:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$collection->split(3)->all();

/*
    [
        0 => [1, 2, 3, 4],
        1 => [5, 6, 7],
        2 => [8, 9, 10],
    ]
*/
```

#### [`splitIn()`](#method-splitin)

The `splitIn` method breaks the collection into the given number of groups, filling groups from the beginning:

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$collection->splitIn(3)->all();

/*
    [
        0 => [1, 2, 3, 4],
        1 => [5, 6, 7],
        2 => [8, 9, 10],
    ]
*/
```

#### [`sum()`](#method-sum)

The `sum` method returns the sum of all items in the collection:

```php
collect([1, 2, 3, 4, 5])->sum();

// 15
```

If the collection contains nested arrays or objects, you should provide the key that should be used to calculate the sum:

```php
$collection = collect([
    ['name' => 'Desk', 'price' => 100],
    ['name' => 'Table', 'price' => 200],
]);

$collection->sum('price');

// 300
```

You may also provide your own callback to specify which value to sum:

```php
$collection = collect([
    ['name' => 'Desk', 'price' => 100],
    ['name' => 'Table', 'price' => 200],
]);

$collection->sum(function (array $item) {
    return $item['price'];
});

// 300
```

#### [`take()`](#method-take)

The `take` method returns a new collection with the specified number of items:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->take(2)->all();

// [1, 2]

$collection->take(-2)->all();

// [4, 5]
```

#### [`takeUntil()`](#method-takeuntil)

The `takeUntil` method returns items in the collection until the given condition is met:

```php
$collection = collect([1, 2, 3, 4]);

$taken = $collection->takeUntil(function (int $item) {
    return $item >= 3;
});

$taken->all();

// [1, 2]
```

You may also pass a value to `takeUntil` to retrieve all items until that value is found:

```php
$collection = collect([1, 2, 3, 4, 5]);

$taken = $collection->takeUntil(3);

$taken->all();

// [1, 2]
```

#### [`takeWhile()`](#method-takewhile)

The `takeWhile` method returns items in the collection as long as the given condition is met:

```php
$collection = collect([1, 2, 3, 4]);

$taken = $collection->takeWhile(function (int $item) {
    return $item < 3;
});

$taken->all();

// [1, 2]
```

After the condition fails, no more items are returned.

#### [`tap()`](#method-tap)

The `tap` method passes the collection to the given callback without affecting the collection itself:

```php
collect([1, 2, 3, 4, 5])
    ->tap(function (Collection $collection) {
        Log::debug('Before', $collection->all());
    })
    ->reject(function (int $item) {
        return $item > 2;
    })
    ->tap(function (Collection $collection) {
        Log::debug('After', $collection->all());
    });
```

This is useful for debugging a chain of methods or performing some intermediate operation without altering the chain.

#### [`times()`](#method-times)

The static `times` method creates a new collection by invoking the given callback a specified number of times:

```php
Collection::times(5, function (int $number) {
    return $number * 2;
});

// [2, 4, 6, 8, 10]
```

#### [`toArray()`](#method-toarray)

The `toArray` method converts the collection to a plain PHP `array`:

```php
$collection = collect([
    ['id' => 1, 'name' => 'Desk'],
]);

$collection->toArray();

/*
    [
        ['id' => 1, 'name' => 'Desk'],
    ]
*/
```

> **Note**   
> `toArray` also converts all of the collection's nested objects to arrays. If you want to convert the collection to a raw JSON string, use the [toJson](#method-tojson) method instead.

#### [`toJson()`](#method-tojson)

The `toJson` method converts the collection to a JSON string:

```php
$collection = collect([
    ['id' => 1, 'name' => 'Desk'],
]);

$collection->toJson();

// "[{"id":1,"name":"Desk"}]"
```

#### [`toPrettyJson()`](#method-to-pretty-json)

The `toPrettyJson` method converts the collection to a pretty printed JSON string:

```php
$collection = collect([
    ['id' => 1, 'name' => 'Desk'],
]);

$collection->toPrettyJson();

/*
"[
    {
        "id": 1,
        "name": "Desk"
    }
]"
*/
```

#### [`transform()`](#method-transform)

The `transform` method iterates over the collection and calls the given callback with each item. The result of the callback will replace the item in the collection:

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->transform(function (int $item, int $key) {
    return $item * 2;
});

$collection->all();

// [2, 4, 6, 8, 10]
```

Unlike most other collection methods, `transform` modifies the collection in place. If you want to create a new collection instead, use the [map](#method-map) method.

#### [`undot()`](#method-undot)

The `undot` method expands nested array keys that use "dot" notation:

```php
$collection = collect([
    'products.desk.price' => 100,
    'products.desk.has_colors' => false,
]);

$undot = $collection->undot();

$undot->all();

/*
    [
        'products' => [
            'desk' => [
                'price' => 100,
                'has_colors' => false,
            ],
        ],
    ]
*/
```

#### [`union()`](#method-union)

The `union` method adds the given array to the collection:

```php
$collection = collect([1 => ['id' => 1, 'name' => 'Desk']]);

$union = $collection->union([2 => ['id' => 2, 'name' => 'Chair']]);

$union->all();

/*
    [
        1 => ['id' => 1, 'name' => 'Desk'],
        2 => ['id' => 2, 'name' => 'Chair'],
    ]
*/
```

#### [`unique()`](#method-unique)

The `unique` method returns all of the unique items in the collection:

```php
$collection = collect([1, 1, 2, 2, 3, 4, 4]);

$unique = $collection->unique();

$unique->all();

// [1, 2, 3, 4]
```

If the collection contains arrays or objects, you may specify the key used to determine uniqueness:

```php
$collection = collect([
    ['id' => 1, 'name' => 'Desk'],
    ['id' => 2, 'name' => 'Chair'],
    ['id' => 1, 'name' => 'Desk'],
]);

$unique = $collection->unique('id');

$unique->all();

/*
    [
        ['id' => 1, 'name' => 'Desk'],
        ['id' => 2, 'name' => 'Chair'],
    ]
*/
```

This method's behavior is modified when using [[08-eloquent-orm/03-eloquent-collections.md#method-unique|Eloquent Collections]].

#### [`uniqueStrict()`](#method-uniquestrict)

This method has the same signature as the [unique](#method-unique) method; however, all values are compared using "strict" comparisons.

#### [`unless()`](#method-unless)

The `unless` method executes the given callback when the first argument evaluates to `false`:

```php
$collection = collect([1, 2, 3]);

$collection->unless(false, function (Collection $collection) {
    return $collection->pop();
});

// 3

$collection->unless(true, function (Collection $collection) {
    return $collection->pop();
});

// [1, 2]
```

#### [`unlessEmpty()`](#method-unlessempty)

Alias for the [whenNotEmpty](#method-whennotempty) method.

#### [`unlessNotEmpty()`](#method-unlessnotempty)

Alias for the [whenEmpty](#method-whenempty) method.

#### [`unwrap()`](#method-unwrap)

The static `unwrap` method returns the collection's items if the value is a `Collection` instance:

```php
Collection::unwrap(collect([1, 2, 3]));

// [1, 2, 3]

Collection::unwrap([1, 2, 3]);

// [1, 2, 3]
```

#### [`value()`](#method-value)

The `value` method retrieves the value at the given key:

```php
$collection = collect([
    ['id' => 1, 'name' => 'Desk'],
]);

$value = $collection->value('name');

// Desk
```

Alternatively, you may pass a second argument to use as a default value if the key does not exist:

```php
$collection = collect([]);

$value = $collection->value('name', 'Default');

// Default
```

#### [`values()`](#method-values)

The `values` method returns a new collection with the keys reset to consecutive integers:

```php
$collection = collect([
    'a' => 'Desk',
    'b' => 'Chair',
]);

$values = $collection->values();

$values->all();

/*
    [
        0 => 'Desk',
        1 => 'Chair',
    ]
*/
```

#### [`when()`](#method-when)

The `when` method executes the given callback when the first argument evaluates to `true`:

```php
$collection = collect([1, 2, 3]);

$collection->when(true, function (Collection $collection) {
    return $collection->push(4);
});

// [1, 2, 3, 4]

$collection->when(false, function (Collection $collection) {
    return $collection->push(4);
});

// [1, 2, 3]
```

#### [`whenEmpty()`](#method-whenempty)

The `whenEmpty` method executes the given callback when the collection is empty:

```php
$collection = collect([]);

$collection->whenEmpty(function (Collection $collection) {
    return $collection->push('Desk');
});

// ['Desk']
```

You may pass a second callback that will be executed when the collection is not empty:

```php
$collection = collect(['Chair']);

$collection->whenEmpty(
    function (Collection $collection) {
        return $collection->push('Desk');
    },
    function (Collection $collection) {
        return $collection->push('Table');
    }
);

// ['Chair', 'Table']
```

#### [`whenNotEmpty()`](#method-whennotempty)

The `whenNotEmpty` method executes the given callback when the collection is not empty:

```php
$collection = collect(['Desk']);

$collection->whenNotEmpty(function (Collection $collection) {
    return $collection->push('Table');
});

// ['Desk', 'Table']
```

#### [`where()`](#method-where)

The `where` method filters the collection by a given key / value pair:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
]);

$filtered = $collection->where('price', 100);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 100],
    ]
*/
```

The `where` method uses "loose" comparisons when checking item values. If you want items to be filtered using "strict" comparisons, use the [whereStrict](#method-wherestrict) method.

In addition, you may provide an operator as the second parameter to compare the key's value against:

```php
$filtered = $collection->where('price', '>=', 150);

$filtered->all();

/*
    [
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Bookcase', 'price' => 150],
    ]
*/
```

#### [`whereStrict()`](#method-wherestrict)

This method has the same signature as the [where](#method-where) method; however, all values are compared using "strict" comparisons.

#### [`whereBetween()`](#method-wherebetween)

The `whereBetween` method filters the collection to within a given range:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
]);

$filtered = $collection->whereBetween('price', 100, 150);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
    ]
*/
```

#### [`whereIn()`](#method-wherein)

The `whereIn` method filters the collection to have items that are in the given array:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
]);

$filtered = $collection->whereIn('price', [100, 150]);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Bookcase', 'price' => 150],
    ]
*/
```

> **Note**   
> When filtering by integers, `whereIn` will look for items with an integer type, matching both `"100"` and `100`.

The `whereIn` method uses "loose" comparisons when checking item values. If you want items to be filtered using "strict" comparisons, use the [whereInStrict](#method-whereinstrict) method.

#### [`whereInStrict()`](#method-whereinstrict)

This method has the same signature as the [whereIn](#method-wherein) method; however, all values are compared using "strict" comparisons.

#### [`whereInstanceOf()`](#method-whereinstanceof)

The `whereInstanceOf` method filters the collection to only have items of the given type:

```php
$collection = collect([
    new User,
    new User,
    new Post,
]);

$filtered = $collection->whereInstanceOf(User::class);

$filtered->all();

// [User, User]
```

#### [`whereNotBetween()`](#method-wherenotbetween)

The `whereNotBetween` method filters the collection to not be within a given range:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
]);

$filtered = $collection->whereNotBetween('price', 100, 150);

$filtered->all();

/*
    [
        ['product' => 'Desk', 'price' => 200],
    ]
*/
```

#### [`whereNotIn()`](#method-wherenotin)

The `whereNotIn` method filters the collection to not have items that are in the given array:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
]);

$filtered = $collection->whereNotIn('price', [100, 150]);

$filtered->all();

/*
    [
        ['product' => 'Desk', 'price' => 200],
    ]
*/
```

The `whereNotIn` method uses "loose" comparisons when checking item values. If you want items to be filtered using "strict" comparisons, use the [whereNotInStrict](#method-wherenotinstrict) method.

#### [`whereNotInStrict()`](#method-wherenotinstrict)

This method has the same signature as the [whereNotIn](#method-wherenotin) method; however, all values are compared using "strict" comparisons.

#### [`whereNotNull()`](#method-wherenotnull)

The `whereNotNull` method filters the collection to exclude null values:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair'],
    ['product' => 'Bookcase', 'price' => 150],
]);

$filtered = $collection->whereNotNull('price');

$filtered->all();

/*
    [
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Bookcase', 'price' => 150],
    ]
*/
```

You may also pass an array of keys to exclude null values for any of those keys:

```php
$collection = collect([
    ['name' => 'Desk', 'price' => 200, 'discount' => null],
    ['name' => 'Chair', 'price' => 100],
    ['name' => 'Bookcase', 'price' => 150, 'discount' => false],
]);

$filtered = $collection->whereNotNull(['price', 'discount']);

$filtered->all();

/*
    [
        ['name' => 'Desk', 'price' => 200, 'discount' => null],
        ['name' => 'Chair', 'price' => 100, 'discount' => null],
        ['name' => 'Bookcase', 'price' => 150, 'discount' => false],
    ]
*/
```

#### [`whereNull()`](#method-wherenull)

The `whereNull` method filters the collection to only have null values:

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair'],
    ['product' => 'Bookcase', 'price' => 150],
]);

$filtered = $collection->whereNull('price');

$filtered->all();

/*
    [
        ['product' => 'Chair'],
    ]
*/
```

#### [`wrap()`](#method-wrap)

The static `wrap` method wraps the given value in a collection:

```php
$collection = Collection::wrap('Desk');

$collection->all();

// ['Desk']
```

If the given value is already a `Collection`, it will be returned:

```php
$collection = Collection::wrap('Desk');

Collection::wrap($collection)->all();

// ['Desk']
```

If the given value is an array, wrap it in a collection:

```php
$collection = Collection::wrap(['Desk', 'Table']);

$collection->all();

// ['Desk', 'Table']
```

#### [`zip()`](#method-zip)

The `zip` method merges together the values of the given array with the values of the collection at the corresponding index:

```php
$collection = collect(['Chair', 'Desk']);

$zipped = $collection->zip(['Table', 'Lamp']);

$zipped->all();

/*
    [
        ['Chair', 'Table'],
        ['Desk', 'Lamp'],
    ]
*/
```

## [Higher Order Messages](#higher-order-messages)

Collection has "higher order message" support for convenience shortcuts which allow you to easily execute common collection methods. The higher order messages available are: `avg`, `sum`, `max`, `min`, `sum`, `contains`, `each`, `map`, `filter`, `first`, `reject`, and `partition`.

#### [`average`](#average)

Average of all items:

```php
collect([1, 2, 3, 4, 5])->average();

// 3
```

#### [`avg`](#avg)

Average of all items:

```php
collect([1, 2, 3, 4, 5])->avg();

// 3
```

#### [`contains`](#contains)

Check if collection contains a given item:

```php
collect([1, 2, 3, 4, 5])->contains(3);

// true
```

#### [`count`](#count)

Count items in collection:

```php
collect([1, 2, 3, 4, 5])->count();

// 5
```

#### [`each`](#each)

Iterate over each item:

```php
collect([1, 2, 3, 4, 5])->each(function (int $item) {
    //
});
```

#### [`every`](#every)

Validate every item passes a truth test:

```php
collect([1, 2, 3, 4, 5])->every(fn (int $item) => $item > 0);

// true
```

#### [`filter`](#filter)

Filter items by value:

```php
collect([1, 2, 3, 4, 5])->filter(fn (int $item) => $item > 2);

// [3, 4, 5]
```

#### [`first`](#first)

Get the first item:

```php
collect([1, 2, 3, 4, 5])->first();

// 1
```

#### [`flatMap`](#flatmap)

Map and flatten:

```php
collect([
    ['id' => 1, 'name' => 'Desk'],
    ['id' => 2, 'name' => 'Desk'],
])->flatMap(fn (array $item) => [
    $item['id'] => $item['name'],
]);

// [1 => 'Desk', 2 => 'Desk']
```

#### [`groupBy`](#groupby)

Group by a key:

```php
collect([
    ['id' => 1, 'name' => 'Desk'],
    ['id' => 2, 'name' => 'Desk'],
])->groupBy('name');

// [
//     'Desk' => [
//         ['id' => 1, 'name' => 'Desk'],
//         ['id' => 2, 'name' => 'Desk'],
//     ],
// ]
```

#### [`keyBy`](#keyby)

Key the collection by a key:

```php
collect([
    ['id' => 1, 'name' => 'Desk'],
    ['id' => 2, 'name' => 'Desk'],
])->keyBy('id');

// [
//     1 => ['id' => 1, 'name' => 'Desk'],
//     2 => ['id' => 2, 'name' => 'Desk'],
// ]
```

#### [`map`](#map)

Map each item to a new value:

```php
collect([1, 2, 3, 4, 5])->map(fn (int $item) => $item * 2);

// [2, 4, 6, 8, 10]
```

#### [`max`](#max)

Maximum value of all items:

```php
collect([1, 2, 3, 4, 5])->max();

// 5
```

#### [`min`](#min)

Minimum value of all items:

```php
collect([1, 2, 3, 4, 5])->min();

// 1
```

#### [`partition`](#partition)

Partition the collection:

```php
[$first, $second] = collect([1, 2, 3, 4, 5])->partition(fn (int $item) => $item <= 3);

// $first: [1, 2, 3]
// $second: [4, 5]
```

#### [`reject`](#reject)

Reject items by value:

```php
collect([1, 2, 3, 4, 5])->reject(fn (int $item) => $item > 3);

// [1, 2, 3]
```

#### [`sum`](#sum)

Sum of all items:

```php
collect([1, 2, 3, 4, 5])->sum();

// 15
```

#### [`unique`](#unique)

Get unique items:

```php
collect([1, 2, 2, 3, 3, 4, 5])->unique();

// [1, 2, 3, 4, 5]
```

## [Lazy Collections](#lazy-collections)

### [Introduction](#lazy-collection-introduction)

Lazy collections are a powerful alternative to regular collections that allow you to work with very large datasets with a minimal memory footprint. Instead of storing all the items in memory at once, lazy collections utilize PHP's [generators](https://www.php.net/manual/en/language.generators.overview.php) to only load items into memory as they are needed.

However, there are some subtle differences between lazy collections and regular collections:

- For iteration, [each](#method-each), [eachSpread](#method-eachspread), and [map](#method-map) methods will utilize generators, which may lead to slight differences in behavior compared to standard collections.
- Methods that rely on array access (like [first](#method-first), [last](#method-last), or [search](#method-search)) typically require all items to be evaluated to work correctly.
- The collection must be cast to an array or converted to a string if you need to enumerate the collection multiple times.

### [Creating Lazy Collections](#creating-lazy-collections)

You can create a lazy collection instance by calling the `lazy` method on a standard collection:

```php
$lazyCollection = collect([1, 2, 3])->lazy();

$lazyCollection::class;

// 'Illuminate\Support\LazyCollection'
```

You can also create a lazy collection from a generator using the `LazyCollection::make` method:

```php
$lazyCollection = LazyCollection::make(function () {
    yield 1;
    yield 2;
    yield 3;
});
```

### [The Enumerable Contract](#the-enumerable-contract)

Both `Collection` and `LazyCollection` implement the `Illuminate\Support\Enumerable` contract, which defines the following methods:

- [all](#method-all)
- [average](#method-average)
- [avg](#method-avg)
- [chunk](#method-chunk)
- [chunkWhile](#method-chunkwhile)
- [collapse](#method-collapse)
- [collect](#method-collect)
- [combine](#method-combine)
- [contains](#method-contains)
- [containsStrict](#method-containsstrict)
- [count](#method-count)
- [countBy](#method-countBy)
- [crossJoin](#method-crossjoin)
- [dd](#method-dd)
- [diff](#method-diff)
- [diffAssoc](#method-diffassoc)
- [diffKeys](#method-diffkeys)
- [dump](#method-dump)
- [duplicates](#method-duplicates)
- [duplicatesStrict](#method-duplicatesstrict)
- [each](#method-each)
- [eachSpread](#method-eachspread)
- [every](#method-every)
- [except](#method-except)
- [filter](#method-filter)
- [first](#method-first)
- [firstOrFail](#method-first-or-fail)
- [firstWhere](#method-first-where)
- [flatMap](#method-flatmap)
- [flatten](#method-flatten)
- [flip](#method-flip)
- [forPage](#method-forpage)
- [get](#method-get)
- [groupBy](#method-groupby)
- [has](#method-has)
- [hasAny](#method-hasany)
- [implode](#method-implode)
- [intersect](#method-intersect)
- [intersectAssoc](#method-intersectAssoc)
- [intersectByKeys](#method-intersectbykeys)
- [isEmpty](#method-isempty)
- [isNotEmpty](#method-isnotempty)
- [join](#method-join)
- [keyBy](#method-keyby)
- [keys](#method-keys)
- [last](#method-last)
- [lazy](#method-lazy)
- [map](#method-map)
- [mapInto](#method-mapinto)
- [mapSpread](#method-mapspread)
- [mapToGroups](#method-maptogroups)
- [mapWithKeys](#method-mapwithkeys)
- [max](#method-max)
- [median](#method-median)
- [merge](#method-merge)
- [min](#method-min)
- [mode](#method-mode)
- [nth](#method-nth)
- [only](#method-only)
- [pad](#method-pad)
- [partition](#method-partition)
- [percentage](#method-percentage)
- [pipe](#method-pipe)
- [pluck](#method-pluck)
- [pop](#method-pop)
- [prepend](#method-prepend)
- [pull](#method-pull)
- [push](#method-push)
- [put](#method-put)
- [random](#method-random)
- [reduce](#method-reduce)
- [reject](#method-reject)
- [replace](#method-replace)
- [reverse](#method-reverse)
- [search](#method-search)
- [select](#method-select)
- [shuffle](#method-shuffle)
- [skip](#method-skip)
- [skipUntil](#method-skipuntil)
- [skipWhile](#method-skipwhile)
- [slice](#method-slice)
- [sliding](#method-sliding)
- [sole](#method-sole)
- [some](#method-some)
- [sort](#method-sort)
- [sortBy](#method-sortby)
- [sortByDesc](#method-sortbydesc)
- [sortDesc](#method-sortdesc)
- [sortKeys](#method-sortkeys)
- [sortKeysDesc](#method-sortkeysdesc)
- [splice](#method-splice)
- [split](#method-split)
- [splitIn](#method-splitin)
- [sum](#method-sum)
- [take](#method-take)
- [takeUntil](#method-takeuntil)
- [takeWhile](#method-takewhile)
- [tap](#method-tap)
- [times](#method-times)
- [toArray](#method-toarray)
- [toJson](#method-tojson)
- [transform](#method-transform)
- [undot](#method-undot)
- [union](#method-union)
- [unique](#method-unique)
- [uniqueStrict](#method-uniquestrict)
- [values](#method-values)
- [when](#method-when)
- [whenEmpty](#method-whenempty)
- [whenNotEmpty](#method-whennotempty)
- [where](#method-where)
- [whereStrict](#method-wherestrict)
- [whereBetween](#method-wherebetween)
- [whereIn](#method-wherein)
- [whereInStrict](#method-whereinstrict)
- [whereInstanceOf](#method-whereinstanceof)
- [whereNotBetween](#method-wherenotbetween)
- [whereNotIn](#method-wherenotin)
- [whereNotInStrict](#method-wherenotinstrict)
- [whereNotNull](#method-wherenotnull)
- [whereNull](#method-wherenull)
- [wrap](#method-wrap)
- [zip](#method-zip)

### [Lazy Collection Methods](#lazy-collection-methods)

In addition to all the methods defined by the `Enumerable` contract, `LazyCollection` includes the following methods:

#### [`force()`](#method-force)

The `force` method eagerly evaluates the lazy collection, storing all the yielded items into memory, which allows the collection to be enumerated multiple times:

```php
$lazyCollection = LazyCollection::make(function () {
    yield 1;
    yield 2;
    yield 3;
});

$lazyCollection->force()->all();

// [1, 2, 3]
```

#### [`remember()`](#method-remember)

The `remember` method returns a lazy collection that will "remember" all the yielded items and return them when the collection is enumerated again. This effectively converts a lazy collection into a collection that caches all its items in memory:

```php
$lazyCollection = LazyCollection::make(function () {
    yield 1;
    yield 2;
    yield 3;
})->remember();

$lazyCollection->all();

// [1, 2, 3]

$lazyCollection->all();

// [1, 2, 3]
```

This is useful for iterators that generate data (such as database cursors) where you may need to enumerate the collection multiple times but don't want to re-run the query each time.