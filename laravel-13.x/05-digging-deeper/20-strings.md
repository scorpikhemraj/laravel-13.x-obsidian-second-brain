---
title: Strings
description: Guide to Laravel's string manipulation functions and fluent strings
url: https://laravel.com/docs/13.x/strings
tags: [logic]
cssclasses:
  - logic
  - color-orange
color: orange
---

# Strings

-   [Introduction](#introduction)
-   [Available Methods](#available-methods)

## [Introduction](#introduction)

Laravel includes a variety of functions for manipulating string values. Many of these functions are used by the framework itself; however, you are free to use them in your own applications if you find them convenient.

## [Available Methods](#available-methods)

### [Strings](#strings-method-list)

[`__`](#method-__) [`class_basename`](#method-class-basename) [`e`](#method-e) [`preg_replace_array`](#method-preg-replace-array) [`Str::after`](#method-str-after) [`Str::afterLast`](#method-str-after-last) [`Str::apa`](#method-str-apa) [`Str::ascii`](#method-str-ascii) [`Str::before`](#method-str-before) [`Str::beforeLast`](#method-str-before-last) [`Str::between`](#method-str-between) [`Str::betweenFirst`](#method-str-between-first) [`Str::camel`](#method-camel-case) [`Str::charAt`](#method-char-at) [`Str::chopStart`](#method-str-chop-start) [`Str::chopEnd`](#method-str-chop-end) [`Str::contains`](#method-str-contains) [`Str::containsAll`](#method-str-contains-all) [`Str::doesntContain`](#method-str-doesnt-contain) [`Str::doesntEndWith`](#method-str-doesnt-end-with) [`Str::doesntStartWith`](#method-str-doesnt-start-with) [`Str::deduplicate`](#method-deduplicate) [`Str::endsWith`](#method-ends-with) [`Str::excerpt`](#method-excerpt) [`Str::finish`](#method-str-finish) [`Str::fromBase64`](#method-str-from-base64) [`Str::headline`](#method-str-headline) [`Str::initials`](#method-str-initials) [`Str::inlineMarkdown`](#method-str-inline-markdown) [`Str::is`](#method-str-is) [`Str::isAscii`](#method-str-is-ascii) [`Str::isJson`](#method-str-is-json) [`Str::isUlid`](#method-str-is-ulid) [`Str::isUrl`](#method-str-is-url) [`Str::isUuid`](#method-str-is-uuid) [`Str::kebab`](#method-kebab-case) [`Str::lcfirst`](#method-str-lcfirst) [`Str::length`](#method-str-length) [`Str::limit`](#method-str-limit) [`Str::lower`](#method-str-lower) [`Str::markdown`](#method-str-markdown) [`Str::mask`](#method-str-mask) [`Str::match`](#method-str-match) [`Str::matchAll`](#method-str-match-all) [`Str::isMatch`](#method-str-is-match) [`Str::orderedUuid`](#method-str-ordered-uuid) [`Str::padBoth`](#method-str-padboth) [`Str::padLeft`](#method-str-padleft) [`Str::padRight`](#method-str-padright) [`Str::password`](#method-str-password) [`Str::plural`](#method-str-plural) [`Str::pluralStudly`](#method-str-plural-studly) [`Str::position`](#method-str-position) [`Str::random`](#method-str-random) [`Str::remove`](#method-str-remove) [`Str::repeat`](#method-str-repeat) [`Str::replace`](#method-str-replace) [`Str::replaceArray`](#method-str-replace-array) [`Str::replaceFirst`](#method-str-replace-first) [`Str::replaceLast`](#method-str-replace-last) [`Str::replaceMatches`](#method-str-replace-matches) [`Str::replaceStart`](#method-str-replace-start) [`Str::replaceEnd`](#method-str-replace-end) [`Str::reverse`](#method-str-reverse) [`Str::singular`](#method-str-singular) [`Str::slug`](#method-str-slug) [`Str::snake`](#method-snake-case) [`Str::squish`](#method-str-squish) [`Str::start`](#method-str-start) [`Str::startsWith`](#method-starts-with) [`Str::studly`](#method-studly-case) [`Str::substr`](#method-str-substr) [`Str::substrCount`](#method-str-substrcount) [`Str::substrReplace`](#method-str-substrreplace) [`Str::swap`](#method-str-swap) [`Str::take`](#method-take) [`Str::title`](#method-title-case) [`Str::toBase64`](#method-str-to-base64) [`Str::transliterate`](#method-str-transliterate) [`Str::trim`](#method-str-trim) [`Str::ltrim`](#method-str-ltrim) [`Str::rtrim`](#method-str-rtrim) [`Str::ucfirst`](#method-str-ucfirst) [`Str::ucsplit`](#method-str-ucsplit) [`Str::ucwords`](#method-str-ucwords) [`Str::upper`](#method-str-upper) [`Str::ulid`](#method-str-ulid) [`Str::unwrap`](#method-str-unwrap) [`Str::uuid`](#method-str-uuid) [`Str::uuid7`](#method-str-uuid7) [`Str::wordCount`](#method-str-word-count) [`Str::wordWrap`](#method-str-word-wrap) [`Str::words`](#method-str-words) [`Str::wrap`](#method-str-wrap) [`str`](#method-str) [`trans`](#method-trans) [`trans_choice`](#method-trans-choice)

### [Fluent Strings](#fluent-strings-method-list)

[after](#method-fluent-str-after) [afterLast](#method-fluent-str-after-last) [apa](#method-fluent-str-apa) [append](#method-fluent-str-append) [ascii](#method-fluent-str-ascii) [basename](#method-fluent-str-basename) [before](#method-fluent-str-before) [beforeLast](#method-fluent-str-before-last) [between](#method-fluent-str-between) [betweenFirst](#method-fluent-str-between-first) [camel](#method-fluent-str-camel) [charAt](#method-fluent-str-char-at) [classBasename](#method-fluent-str-class-basename) [chopStart](#method-fluent-str-chop-start) [chopEnd](#method-fluent-str-chop-end) [contains](#method-fluent-str-contains) [containsAll](#method-fluent-str-contains-all) [decrypt](#method-fluent-str-decrypt) [deduplicate](#method-fluent-str-deduplicate) [dirname](#method-fluent-str-dirname) [doesntContain](#method-fluent-str-doesnt-contain) [doesntEndWith](#method-fluent-str-doesnt-end-with) [doesntStartWith](#method-fluent-str-doesnt-start-with) [encrypt](#method-fluent-str-encrypt) [endsWith](#method-fluent-str-ends-with) [exactly](#method-fluent-str-exactly) [excerpt](#method-fluent-str-excerpt) [explode](#method-fluent-str-explode) [finish](#method-fluent-str-finish) [fromBase64](#method-fluent-str-from-base64) [hash](#method-fluent-str-hash) [headline](#method-fluent-str-headline) [initials](#method-fluent-str-initials) [inlineMarkdown](#method-fluent-str-inline-markdown) [is](#method-fluent-str-is) [isAscii](#method-fluent-str-is-ascii) [isEmpty](#method-fluent-str-is-empty) [isNotEmpty](#method-fluent-str-is-not-empty) [isJson](#method-fluent-str-is-json) [isUlid](#method-fluent-str-is-ulid) [isUrl](#method-fluent-str-is-url) [isUuid](#method-fluent-str-is-uuid) [kebab](#method-fluent-str-kebab) [lcfirst](#method-fluent-str-lcfirst) [length](#method-fluent-str-length) [limit](#method-fluent-str-limit) [lower](#method-fluent-str-lower) [markdown](#method-fluent-str-markdown) [mask](#method-fluent-str-mask) [match](#method-fluent-str-match) [matchAll](#method-fluent-str-match-all) [isMatch](#method-fluent-str-is-match) [newLine](#method-fluent-str-new-line) [padBoth](#method-fluent-str-padboth) [padLeft](#method-fluent-str-padleft) [padRight](#method-fluent-str-padright) [pipe](#method-fluent-str-pipe) [plural](#method-fluent-str-plural) [position](#method-fluent-str-position) [prepend](#method-fluent-str-prepend) [remove](#method-fluent-str-remove) [repeat](#method-fluent-str-repeat) [replace](#method-fluent-str-replace) [replaceArray](#method-fluent-str-replace-array) [replaceFirst](#method-fluent-str-replace-first) [replaceLast](#method-fluent-str-replace-last) [replaceMatches](#method-fluent-str-replace-matches) [replaceStart](#method-fluent-str-replace-start) [replaceEnd](#method-fluent-str-replace-end) [scan](#method-fluent-str-scan) [singular](#method-fluent-str-singular) [slug](#method-fluent-str-slug) [snake](#method-fluent-str-snake) [split](#method-fluent-str-split) [squish](#method-fluent-str-squish) [start](#method-fluent-str-start) [startsWith](#method-fluent-str-starts-with) [stripTags](#method-fluent-str-strip-tags) [studly](#method-fluent-str-studly) [substr](#method-fluent-str-substr) [substrReplace](#method-fluent-str-substrreplace) [swap](#method-fluent-str-swap) [take](#method-fluent-str-take) [tap](#method-fluent-str-tap) [test](#method-fluent-str-test) [title](#method-fluent-str-title) [toBase64](#method-fluent-str-to-base64) [toHtmlString](#method-fluent-str-to-html-string) [toUri](#method-fluent-str-to-uri) [transliterate](#method-fluent-str-transliterate) [trim](#method-fluent-str-trim) [ltrim](#method-fluent-str-ltrim) [rtrim](#method-fluent-str-rtrim) [ucfirst](#method-fluent-str-ucfirst) [ucsplit](#method-fluent-str-ucsplit) [ucwords](#method-fluent-str-ucwords) [unwrap](#method-fluent-str-unwrap) [upper](#method-fluent-str-upper) [when](#method-fluent-str-when) [whenContains](#method-fluent-str-when-contains) [whenContainsAll](#method-fluent-str-when-contains-all) [whenDoesntEndWith](#method-fluent-str-when-doesnt-end-with) [whenDoesntStartWith](#method-fluent-str-when-doesnt-start-with) [whenEmpty](#method-fluent-str-when-empty) [whenNotEmpty](#method-fluent-str-when-not-empty) [whenStartsWith](#method-fluent-str-when-starts-with) [whenEndsWith](#method-fluent-str-when-ends-with) [whenExactly](#method-fluent-str-when-exactly) [whenNotExactly](#method-fluent-str-when-not-exactly) [whenIs](#method-fluent-str-when-is) [whenIsAscii](#method-fluent-str-when-is-ascii) [whenIsUlid](#method-fluent-str-when-is-ulid) [whenIsUuid](#method-fluent-str-when-is-uuid) [whenTest](#method-fluent-str-when-test) [wordCount](#method-fluent-str-word-count) [words](#method-fluent-str-words) [wrap](#method-fluent-str-wrap)

## [Strings](#strings)

#### [`__()`](#method-__)

The `__` function translates the given translation string or translation key using your [[05-digging-deeper/12-localization.md|language files]]:

```php
echo __('Welcome to our application');

echo __('messages.welcome');
```

If the specified translation string or key does not exist, the `__` function will return the given value. So, using the example above, the `__` function would return `messages.welcome` if that translation key does not exist.

#### [`class_basename()`](#method-class-basename)

The `class_basename` function returns the class name of the given class with the class's namespace removed:

```php
$class = class_basename('Foo\Bar\Baz');

// Baz
```

#### [`e()`](#method-e)

The `e` function runs PHP's `htmlspecialchars` function with the `double_encode` option set to `true` by default:

```php
echo e('<html>foo</html>');

// &lt;html&gt;foo&lt;/html&gt;
```

#### [`preg_replace_array()`](#method-preg-replace-array)

The `preg_replace_array` function replaces a given pattern in the string sequentially using an array:

```php
$string = 'The event will take place between :start and :end';

$replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

// The event will take place between 8:30 and 9:00
```

#### [`Str::after()`](#method-str-after)

The `Str::after` method returns everything after the given value in a string. The entire string will be returned if the value does not exist within the string:

```php
use Illuminate\Support\Str;

$slice = Str::after('This is my name', 'This is');

// ' my name'
```

#### [`Str::afterLast()`](#method-str-after-last)

The `Str::afterLast` method returns everything after the last occurrence of the given value in a string. The entire string will be returned if the value does not exist within the string:

```php
use Illuminate\Support\Str;

$slice = Str::afterLast('App\Http\Controllers\Controller', '\\');

// 'Controller'
```

#### [`Str::apa()`](#method-str-apa)

The `Str::apa` method converts the given string to title case following the [APA guidelines](https://apastyle.apa.org/style-grammar-guidelines/capitalization/title-case):

```php
use Illuminate\Support\Str;

$title = Str::apa('Creating A Project');

// 'Creating a Project'
```

#### [`Str::ascii()`](#method-str-ascii)

The `Str::ascii` method will attempt to transliterate the string into an ASCII value:

```php
use Illuminate\Support\Str;

$slice = Str::ascii('û');

// 'u'
```

#### [`Str::before()`](#method-str-before)

The `Str::before` method returns everything before the given value in a string:

```php
use Illuminate\Support\Str;

$slice = Str::before('This is my name', 'my name');

// 'This is '
```

#### [`Str::beforeLast()`](#method-str-before-last)

The `Str::beforeLast` method returns everything before the last occurrence of the given value in a string:

```php
use Illuminate\Support\Str;

$slice = Str::beforeLast('This is my name', 'is');

// 'This '
```

#### [`Str::between()`](#method-str-between)

The `Str::between` method returns the portion of a string between two values:

```php
use Illuminate\Support\Str;

$slice = Str::between('This is my name', 'This', 'name');

// ' is my '
```

#### [`Str::betweenFirst()`](#method-str-between-first)

The `Str::betweenFirst` method returns the smallest possible portion of a string between two values:

```php
use Illuminate\Support\Str;

$slice = Str::betweenFirst('[a] bc [d]', '[', ']');

// 'a'
```

#### [`Str::camel()`](#method-camel-case)

The `Str::camel` method converts the given string to `camelCase`:

```php
use Illuminate\Support\Str;

$converted = Str::camel('foo_bar');

// 'fooBar'
```

#### [`Str::charAt()`](#method-char-at)

The `Str::charAt` method returns the character at the specified index. If the index is out of bounds, `false` is returned:

```php
use Illuminate\Support\Str;

$character = Str::charAt('This is my name.', 6);

// 's'
```

#### [`Str::chopStart()`](#method-str-chop-start)

The `Str::chopStart` method removes the first occurrence of the given value only if the value appears at the start of the string:

```php
use Illuminate\Support\Str;

$url = Str::chopStart('https://laravel.com', 'https://');

// 'laravel.com'
```

You may also pass an array as the second argument. If the string starts with any of the values in the array then that value will be removed from string:

```php
use Illuminate\Support\Str;

$url = Str::chopStart('http://laravel.com', ['https://', 'http://']);

// 'laravel.com'
```

#### [`Str::chopEnd()`](#method-str-chop-end)

The `Str::chopEnd` method removes the last occurrence of the given value only if the value appears at the end of the string:

```php
use Illuminate\Support\Str;

$url = Str::chopEnd('app/Models/Photograph.php', '.php');

// 'app/Models/Photograph'
```

You may also pass an array as the second argument. If the string ends with any of the values in the array then that value will be removed from string:

```php
use Illuminate\Support\Str;

$url = Str::chopEnd('laravel.com/index.php', ['/index.html', '/index.php']);

// 'laravel.com'
```

#### [`Str::contains()`](#method-str-contains)

The `Str::contains` method determines if the given string contains the given value. By default, this method is case sensitive:

```php
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', 'my');

// true
```

You may also pass an array of values to determine if the given string contains any of the values in the array:

```php
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', ['my', 'foo']);

// true
```

You may disable case sensitivity by setting the `ignoreCase` argument to `true`:

```php
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', 'MY', ignoreCase: true);

// true
```

#### [`Str::containsAll()`](#method-str-contains-all)

The `Str::containsAll` method determines if the given string contains all of the values in a given array:

```php
use Illuminate\Support\Str;

$containsAll = Str::containsAll('This is my name', ['my', 'name']);

// true
```

You may disable case sensitivity by setting the `ignoreCase` argument to `true`:

```php
use Illuminate\Support\Str;

$containsAll = Str::containsAll('This is my name', ['MY', 'NAME'], ignoreCase: true);

// true
```

#### [`Str::doesntContain()`](#method-str-doesnt-contain)

The `Str::doesntContain` method determines if the given string doesn't contain the given value. By default, this method is case sensitive:

```php
use Illuminate\Support\Str;

$doesntContain = Str::doesntContain('This is name', 'my');

// true
```

#### [`Str::deduplicate()`](#method-deduplicate)

The `Str::deduplicate` method replaces consecutive instances of a character with a single instance of that character in the given string. By default, the method deduplicates spaces:

```php
use Illuminate\Support\Str;

$result = Str::deduplicate('The   Laravel   Framework');

// The Laravel Framework
```

#### [`Str::doesntEndWith()`](#method-str-doesnt-end-with)

The `Str::doesntEndWith` method determines if the given string doesn't end with the given value:

```php
use Illuminate\Support\Str;

$result = Str::doesntEndWith('This is my name', 'dog');

// true
```

#### [`Str::doesntStartWith()`](#method-str-doesnt-start-with)

The `Str::doesntStartWith` method determines if the given string doesn't begin with the given value:

```php
use Illuminate\Support\Str;

$result = Str::doesntStartWith('This is my name', 'That');

// true
```

#### [`Str::endsWith()`](#method-ends-with)

The `Str::endsWith` method determines if the given string ends with the given value:

```php
use Illuminate\Support\Str;

$result = Str::endsWith('This is my name', 'name');

// true
```

#### [`Str::excerpt()`](#method-excerpt)

The `Str::excerpt` method extracts an excerpt from a given string that matches the first instance of a phrase within that string:

```php
use Illuminate\Support\Str;

$excerpt = Str::excerpt('This is my name', 'my', [
    'radius' => 3
]);

// '...is my na...'
```

The `radius` option, which defaults to `100`, allows you to define the number of characters that should appear on each side of the truncated string.

In addition, you may use the `omission` option to define the string that will be prepended and appended to the truncated string:

```php
use Illuminate\Support\Str;

$excerpt = Str::excerpt('This is my name', 'name', [
    'radius' => 3,
    'omission' => '(...) '
]);

// '(...) my name'
```

#### [`Str::finish()`](#method-str-finish)

The `Str::finish` method adds a single instance of the given value to a string if it does not already end with that value:

```php
use Illuminate\Support\Str;

$adjusted = Str::finish('this/string', '/');

// this/string/

$adjusted = Str::finish('this/string/', '/');

// this/string/
```

#### [`Str::fromBase64()`](#method-str-from-base64)

The `Str::fromBase64` method decodes the given Base64 string:

```php
use Illuminate\Support\Str;

$decoded = Str::fromBase64('TGFyYXZlbA==');

// Laravel
```

#### [`Str::headline()`](#method-str-headline)

The `Str::headline` method will convert strings delimited by casing, hyphens, or underscores into a space delimited string with each word's first letter capitalized:

```php
use Illuminate\Support\Str;

$headline = Str::headline('steve_jobs');

// Steve Jobs

$headline = Str::headline('EmailNotificationSent');

// Email Notification Sent
```

#### [`Str::initials()`](#method-str-initials)

The `Str::initials` method will return the initials of a given string, optionally capitalizing them:

```php
use Illuminate\Support\Str;

$initials = Str::initials('taylor otwell');

// to

$initials = Str::initials('taylor otwell', capitalize: true);

// TO
```

#### [`Str::inlineMarkdown()`](#method-str-inline-markdown)

The `Str::inlineMarkdown` method converts GitHub flavored Markdown into inline HTML using [CommonMark](https://commonmark.thephpleague.com/). However, unlike the `markdown` method, it does not wrap all generated HTML in a block-level element:

```php
use Illuminate\Support\Str;

$html = Str::inlineMarkdown('**Laravel**');

// <strong>Laravel</strong>
```

#### Markdown Security

By default, Markdown supports raw HTML, which will expose Cross-Site Scripting (XSS) vulnerabilities when used with raw user input. As per the [CommonMark Security documentation](https://commonmark.thephpleague.com/security/), you may use the `html_input` option to either escape or strip raw HTML, and the `allow_unsafe_links` option to specify whether to allow unsafe links. If you need to allow some raw HTML, you should pass your compiled Markdown through an HTML Purifier:

```php
use Illuminate\Support\Str;

Str::inlineMarkdown('Inject: <script>alert("Hello XSS!");</script>', [
    'html_input' => 'strip',
    'allow_unsafe_links' => false,
]);

// Inject: alert(&quot;Hello XSS!&quot;);
```

#### [`Str::is()`](#method-str-is)

The `Str::is` method determines if a given string matches a given pattern. Asterisks may be used as wildcard values:

```php
use Illuminate\Support\Str;

$matches = Str::is('foo*', 'foobar');

// true

$matches = Str::is('baz*', 'foobar');

// false
```

You may disable case sensitivity by setting the `ignoreCase` argument to `true`:

```php
use Illuminate\Support\Str;

$matches = Str::is('*.jpg', 'photo.JPG', ignoreCase: true);

// true
```

#### [`Str::isAscii()`](#method-str-is-ascii)

The `Str::isAscii` method determines if a given string is 7 bit ASCII:

```php
use Illuminate\Support\Str;

$isAscii = Str::isAscii('Taylor');

// true

$isAscii = Str::isAscii('ü');

// false
```

#### [`Str::isJson()`](#method-str-is-json)

The `Str::isJson` method determines if the given string is valid JSON:

```php
use Illuminate\Support\Str;

$result = Str::isJson('[1,2,3]');

// true

$result = Str::isJson('{"first": "John", "last": "Doe"}');

// true

$result = Str::isJson('{first: "John", last: "Doe"}');

// false
```

#### [`Str::isUrl()`](#method-str-is-url)

The `Str::isUrl` method determines if the given string is a valid URL:

```php
use Illuminate\Support\Str;

$isUrl = Str::isUrl('http://example.com');

// true

$isUrl = Str::isUrl('laravel');

// false
```

The `isUrl` method considers a wide range of protocols as valid. However, you may specify the protocols that should be considered valid by providing them to the `isUrl` method:

```php
$isUrl = Str::isUrl('http://example.com', ['http', 'https']);
```

#### [`Str::isUlid()`](#method-str-is-ulid)

The `Str::isUlid` method determines if the given string is a valid ULID:

```php
use Illuminate\Support\Str;

$isUlid = Str::isUlid('01gd6r360bp37zj17nxb55yv40');

// true

$isUlid = Str::isUlid('laravel');

// false
```

#### [`Str::isUuid()`](#method-str-is-uuid)

The `Str::isUuid` method determines if the given string is a valid UUID:

```php
use Illuminate\Support\Str;

$isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de');

// true

$isUuid = Str::isUuid('laravel');

// false
```

You may also validate that the given UUID matches a UUID specification by version (1, 3, 4, 5, 6, 7, or 8):

```php
use Illuminate\Support\Str;

$isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de', version: 4);

// true

$isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de', version: 1);

// false
```

#### [`Str::kebab()`](#method-kebab-case)

The `Str::kebab` method converts the given string to `kebab-case`:

```php
use Illuminate\Support\Str;

$converted = Str::kebab('fooBar');

// foo-bar
```

#### [`Str::lcfirst()`](#method-str-lcfirst)

The `Str::lcfirst` method returns the given string with the first character lowercased:

```php
use Illuminate\Support\Str;

$string = Str::lcfirst('Foo Bar');

// foo Bar
```

#### [`Str::length()`](#method-str-length)

The `Str::length` method returns the length of the given string:

```php
use Illuminate\Support\Str;

$length = Str::length('Laravel');

// 7
```

#### [`Str::limit()`](#method-str-limit)

The `Str::limit` method truncates the given string to the specified length:

```php
use Illuminate\Support\Str;

$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

// The quick brown fox...
```

You may pass a third argument to the method to change the string that will be appended to the end of the truncated string:

```php
$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

// The quick brown fox (...)
```

If you would like to preserve complete words when truncating the string, you may utilize the `preserveWords` argument:

```php
$truncated = Str::limit('The quick brown fox', 12, preserveWords: true);

// The quick...
```

#### [`Str::lower()`](#method-str-lower)

The `Str::lower` method converts the given string to lowercase:

```php
use Illuminate\Support\Str;

$converted = Str::lower('LARAVEL');

// laravel
```

#### [`Str::markdown()`](#method-str-markdown)

The `Str::markdown` method converts GitHub flavored Markdown into HTML using [CommonMark](https://commonmark.thephpleague.com/):

```php
use Illuminate\Support\Str;

$html = Str::markdown('# Laravel');

// <h1>Laravel</h1>

$html = Str::markdown('# Taylor <b>Otwell</b>', [
    'html_input' => 'strip',
]);

// <h1>Taylor Otwell</h1>
```

#### Markdown Security

By default, Markdown supports raw HTML, which will expose Cross-Site Scripting (XSS) vulnerabilities when used with raw user input.

#### [`Str::mask()`](#method-str-mask)

The `Str::mask` method masks a portion of a string with a repeated character, and may be used to obfuscate segments of strings such as email addresses and phone numbers:

```php
use Illuminate\Support\Str;

$string = Str::mask('[email protected]', '*', 3);

// tay***************
```

If needed, you provide a negative number as the third argument to the `mask` method, which will instruct the method to begin masking at the given distance from the end of the string:

```php
$string = Str::mask('[email protected]', '*', -15, 3);

// tay***@example.com
```

#### [`Str::match()`](#method-str-match)

The `Str::match` method will return the portion of a string that matches a given regular expression pattern:

```php
use Illuminate\Support\Str;

$result = Str::match('/bar/', 'foo bar');

// 'bar'

$result = Str::match('/foo (.*)/', 'foo bar');

// 'bar'
```

#### [`Str::matchAll()`](#method-str-match-all)

The `Str::matchAll` method will return a collection containing the portions of a string that match a given regular expression pattern:

```php
use Illuminate\Support\Str;

$result = Str::matchAll('/bar/', 'bar foo bar');

// collect(['bar', 'bar'])
```

If you specify a matching group within the expression, Laravel will return a collection of the first matching group's matches:

```php
use Illuminate\Support\Str;

$result = Str::matchAll('/f(\w*)/', 'bar fun bar fly');

// collect(['un', 'ly']);
```

If no matches are found, an empty collection will be returned.

#### [`Str::isMatch()`](#method-str-is-match)

The `Str::isMatch` method will return `true` if the string matches a given regular expression:

```php
use Illuminate\Support\Str;

$result = Str::isMatch('/foo (.*)/', 'foo bar');

// true

$result = Str::isMatch('/foo (.*)/', 'laravel');

// false
```

#### [`Str::orderedUuid()`](#method-str-ordered-uuid)

The `Str::orderedUuid` method generates a "timestamp first" UUID that may be efficiently stored in an indexed database column:

```php
use Illuminate\Support\Str;

return (string) Str::orderedUuid();
```

#### [`Str::padBoth()`](#method-str-padboth)

The `Str::padBoth` method wraps PHP's `str_pad` function, padding both sides of a string with another string until the final string reaches a desired length:

```php
use Illuminate\Support\Str;

$padded = Str::padBoth('James', 10, '_');

// '__James___'

$padded = Str::padBoth('James', 10);

// '  James   '
```

#### [`Str::padLeft()`](#method-str-padleft)

The `Str::padLeft` method wraps PHP's `str_pad` function, padding the left side of a string:

```php
use Illuminate\Support\Str;

$padded = Str::padLeft('James', 10, '-=');

// '- =- =James'

$padded = Str::padLeft('James', 10);

// '     James'
```

#### [`Str::padRight()`](#method-str-padright)

The `Str::padRight` method wraps PHP's `str_pad` function, padding the right side of a string:

```php
use Illuminate\Support\Str;

$padded = Str::padRight('James', 10, '-');

// 'James-----'

$padded = Str::padRight('James', 10);

// 'James     '
```

#### [`Str::password()`](#method-str-password)

The `Str::password` method may be used to generate a secure, random password of a given length:

```php
use Illuminate\Support\Str;

$password = Str::password();

// 'EbJo2vE-AS:U,$%_gkrV4n,q~1xy/-_4'

$password = Str::password(12);

// 'qwuar>#V|i]N'
```

#### [`Str::plural()`](#method-str-plural)

The `Str::plural` method converts a singular word string to its plural form:

```php
use Illuminate\Support\Str;

$plural = Str::plural('car');

// cars

$plural = Str::plural('child');

// children
```

You may provide an integer as a second argument to the function to retrieve the singular or plural form of the string:

```php
use Illuminate\Support\Str;

$plural = Str::plural('child', 2);

// children

$singular = Str::plural('child', 1);

// child
```

#### [`Str::pluralStudly()`](#method-str-plural-studly)

The `Str::pluralStudly` method converts a singular word string formatted in studly caps case to its plural form:

```php
use Illuminate\Support\Str;

$plural = Str::pluralStudly('VerifiedHuman');

// VerifiedHumans

$plural = Str::pluralStudly('UserFeedback');

// UserFeedback
```

#### [`Str::position()`](#method-str-position)

The `Str::position` method returns the position of the first occurrence of a substring in a string:

```php
use Illuminate\Support\Str;

$position = Str::position('Hello, World!', 'Hello');

// 0

$position = Str::position('Hello, World!', 'W');

// 7
```

#### [`Str::random()`](#method-str-random)

The `Str::random` method generates a random string of the specified length:

```php
use Illuminate\Support\Str;

$random = Str::random(40);
```

During testing, it may be useful to "fake" the value that is returned by the `Str::random` method:

```php
Str::createRandomStringsUsing(function () {
    return 'fake-random-string';
});
```

To instruct the `random` method to return to generating random strings normally, you may invoke the `createRandomStringsNormally` method:

```php
Str::createRandomStringsNormally();
```

#### [`Str::remove()`](#method-str-remove)

The `Str::remove` method removes the given value or array of values from the string:

```php
use Illuminate\Support\Str;

$string = 'Peter Piper picked a peck of pickled peppers.';

$removed = Str::remove('e', $string);

// Ptr Pipr pickd a pck of pickld ppprs.
```

#### [`Str::repeat()`](#method-str-repeat)

The `Str::repeat` method repeats the given string:

```php
use Illuminate\Support\Str;

$string = 'a';

$repeat = Str::repeat($string, 5);

// aaaaa
```

#### [`Str::replace()`](#method-str-replace)

The `Str::replace` method replaces a given string within the string:

```php
use Illuminate\Support\Str;

$string = 'Laravel 11.x';

$replaced = Str::replace('11.x', '12.x', $string);

// Laravel 12.x
```

#### [`Str::replaceArray()`](#method-str-replace-array)

The `Str::replaceArray` method replaces a given value in the string sequentially using an array:

```php
use Illuminate\Support\Str;

$string = 'The event will take place between ? and ?';

$replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

// The event will take place between 8:30 and 9:00
```

#### [`Str::replaceFirst()`](#method-str-replace-first)

The `Str::replaceFirst` method replaces the first occurrence of a given value in a string:

```php
use Illuminate\Support\Str;

$replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

// a quick brown fox jumps over the lazy dog
```

#### [`Str::replaceLast()`](#method-str-replace-last)

The `Str::replaceLast` method replaces the last occurrence of a given value in a string:

```php
use Illuminate\Support\Str;

$replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

// the quick brown fox jumps over a lazy dog
```

#### [`Str::replaceMatches()`](#method-str-replace-matches)

The `Str::replaceMatches` method replaces all portions of a string matching a pattern with the given replacement string:

```php
use Illuminate\Support\Str;

$replaced = Str::replaceMatches(
    pattern: '/[^A-Za-z0-9]++/',
    replace: '',
    subject: '(+1) 501-555-1000'
);

// '15015551000'
```

#### [`Str::replaceStart()`](#method-str-replace-start)

The `Str::replaceStart` method replaces the first occurrence of the given value in a string if the value appears at the beginning of the string:

```php
use Illuminate\Support\Str;

$replaced = Str::replaceStart('prefix_', '', 'prefix_the quick brown fox');

// the quick brown fox
```

#### [`Str::replaceEnd()`](#method-str-replace-end)

The `Str::replaceEnd` method replaces the last occurrence of a given value in a string if the value appears at the end of the string:

```php
use Illuminate\Support\Str;

$replaced = Str::replaceEnd('_suffix', '', 'the quick brown fox_suffix');

// the quick brown fox
```

#### [`Str::reverse()`](#method-str-reverse)

The `Str::reverse` method reverses the given string:

```php
use Illuminate\Support\Str;

$reversed = Str::reverse('Hello World');

// dlroW olleH
```

#### [`Str::singular()`](#method-str-singular)

The `Str::singular` method converts a string to its singular form:

```php
use Illuminate\Support\Str;

$singular = Str::singular('cars');

// car
```

#### [`Str::slug()`](#method-str-slug)

The `Str::slug` method generates a URL-friendly "slug" from a given string:

```php
use Illuminate\Support\Str;

$slug = Str::slug('Laravel Framework')';

// laravel-framework
```

#### [`Str::snake()`](#method-snake-case)

The `Str::snake` method converts a string to `snake_case`:

```php
use Illuminate\Support\Str;

$converted = Str::snake('fooBar');

// foo_bar
```

#### [`Str::squish()`](#method-str-squish)

The `Str::squish` method removes all extraneous whitespace from a string:

```php
use Illuminate\Support\Str;

$string = Str::squish('    a    b   c    ');

// a b c
```

#### [`Str::start()`](#method-str-start)

The `Str::start` method adds a single instance of the given value to a string if it does not already start with that value:

```php
use Illuminate\Support\Str;

$adjusted = Str::start('this/string', '/');

// /this/string

$adjusted = Str::start('/this/string', '/');

// /this/string
```

#### [`Str::startsWith()`](#method-starts-with)

The `Str::startsWith` method determines if the given string starts with the given value:

```php
use Illuminate\Support\Str;

$result = Str::startsWith('This is my name', 'This');

// true
```

#### [`Str::studly()`](#method-studly-case)

The `Str::studly` method converts the given string to `StudlyCase`:

```php
use Illuminate\Support\Str;

$converted = Str::studly('foo_bar');

// FooBar
```

#### [`Str::substr()`](#method-str-substr)

The `Str::substr` method returns the portion of the string specified by the start and length parameters:

```php
use Illuminate\Support\Str;

$converted = Str::substr('Laravel Framework', 0, 7);

// Laravel
```

#### [`Str::substrCount()`](#method-str-substrcount)

The `Str::substrCount` method returns the number of occurrences of a substring in the given string:

```php
use Illuminate\Support\Str;

$count = Str::substrCount('Hello World', 'l');

// 3
```

#### [`Str::substrReplace()`](#method-str-substrreplace)

The `Str::substrReplace` method replaces text within a portion of a string:

```php
use Illuminate\Support\Str;

$result = Str::substrReplace('1300', '15', 0, 2);

// 1500
```

#### [`Str::swap()`](#method-str-swap)

The `Str::swap` method replaces multiple strings in a given string using an array:

```php
use Illuminate\Support\Str;

$result = Str::swap([
    'Laravel' => 'Fabric',
    'Framework' => 'Engine',
], 'Laravel Framework');

// Fabric Engine
```

#### [`Str::take()`](#method-take)

The `Str::take` method returns a substring from the beginning of the string up to the specified length:

```php
use Illuminate\Support\Str;

$substring = Str::take('Laravel Framework', 5);

// Larav
```

#### [`Str::title()`](#method-title-case)

The `Str::title` method converts the given string to `Title Case`:

```php
use Illuminate\Support\Str;

$converted = Str::title('laravel framework');

// Laravel Framework
```

#### [`Str::toBase64()`](#method-str-to-base64)

The `Str::toBase64` method encodes the given string to Base64:

```php
use Illuminate\Support\Str;

$encoded = Str::toBase64('Laravel');

// TGFyYXZlbA==
```

#### [`Str::transliterate()`](#method-str-transliterate)

The `Str::transliterate` method will convert accented characters to ASCII characters:

```php
use Illuminate\Support\Str;

$result = Str::transliterate('ü');

// u
```

#### [`Str::trim()`](#method-str-trim)

The `Str::trim` method strips whitespace from the beginning and end of the string:

```php
use Illuminate\Support\Str;

$trimmed = Str::trim('  Laravel  ');

// 'Laravel'
```

#### [`Str::ltrim()`](#method-str-ltrim)

The `Str::ltrim` method strips whitespace from the beginning of the string:

```php
use Illuminate\Support\Str;

$trimmed = Str::ltrim('  Laravel  ');

// 'Laravel  '
```

#### [`Str::rtrim()`](#method-str-rtrim)

The `Str::rtrim` method strips whitespace from the end of the string:

```php
use Illuminate\Support\Str;

$trimmed = Str::rtrim('  Laravel  ');

// '  Laravel'
```

#### [`Str::ucfirst()`](#method-str-ucfirst)

The `Str::ucfirst` method returns the given string with the first character uppercased:

```php
use Illuminate\Support\Str;

$string = Str::ucfirst('foo bar');

// Foo bar
```

#### [`Str::ucsplit()`](#method-str-ucsplit)

The `Str::ucsplit` method splits the given string into an array by uppercase characters:

```php
use Illuminate\Support\Str;

$segments = Str::ucsplit('fooBar');

// ['foo', 'Bar']
```

#### [`Str::ucwords()`](#method-str-ucwords)

The `Str::ucwords` method capitalizes the first character of each word in the string:

```php
use Illuminate\Support\Str;

$converted = Str::ucwords('foo bar foo');

// Foo Bar Foo
```

#### [`Str::upper()`](#method-str-upper)

The `Str::upper` method converts the given string to uppercase:

```php
use Illuminate\Support\Str;

$converted = Str::upper('laravel');

// LARAVEL
```

#### [`Str::ulid()`](#method-str-ulid)

The `Str::ulid` method generates a ULID:

```php
use Illuminate\Support\Str;

return (string) Str::ulid();

// 01gd6r360bp37zj17nxb55yv40
```

#### [`Str::unwrap()`](#method-str-unwrap)

The `Str::unwrap` method removes the specified wrapper from the string:

```php
use Illuminate\Support\Str;

$unwraped = Str::unwrap('[foo]', '[');

// foo
```

#### [`Str::uuid()`](#method-str-uuid)

The `Str::uuid` method generates a UUID (version 4):

```php
use Illuminate\Support\Str;

return (string) Str::uuid();
```

#### [`Str::uuid7()`](#method-str-uuid7)

The `Str::uuid7` method generates a UUID (version 7):

```php
use Illuminate\Support\Str;

return (string) Str::uuid7();
```

#### [`Str::wordCount()`](#method-str-word-count)

The `Str::wordCount` method returns the number of words in a string:

```php
use Illuminate\Support\Str;

$count = Str::wordCount('Hello World');

// 2
```

#### [`Str::wordWrap()`](#method-str-word-wrap)

The `Str::wordWrap` method wraps a string to a given number of characters using a break character:

```php
use Illuminate\Support\Str;

$wrapped = Str::wordWrap('The quick brown fox', 10);

// The quick
// brown fox
```

#### [`Str::words()`](#method-str-words)

The `Str::words` method returns the specified number of words from the string:

```php
use Illuminate\Support\Str;

$words = Str::words('Hello World', 1);

// Hello...
```

#### [`Str::wrap()`](#method-str-wrap)

The `Str::wrap` method wraps the given string with the specified wrapper:

```php
use Illuminate\Support\Str;

$wrapped = Str::wrap('foo', '"');

// "foo"
```

## Fluent Strings

Laravel's `Str::of` returns an instance of `Stringable`, which provides a fluent interface for manipulating strings. You may use `Stringable` methods to chain string operations together:

```php
use Illuminate\Support\Str;

$closure = Str::of('  hello world  ')
    ->trim()
    ->title();
```

### Additional Methods

Many other fluent string methods are available. The most commonly used are:

- `after($search)` - Returns everything after the given value
- `append($value)` - Appends the given value to the string
- `at($position)` - Returns the character at the given position
- `before($search)` - Returns everything before the given value
- `between($start, $end)` - Returns the string between two values
- `camel()` - Converts to camelCase
- `contains($needle)` - Checks if string contains value
- `endsWith($needle)` - Checks if string ends with value
- `explode($delimiter)` - Splits string into collection
- `finish($value)` - Ensures string ends with given value
- `is($pattern)` - Checks if string matches pattern
- `isEmpty()` - Checks if string is empty
- `kebab()` - Converts to kebab-case
- `limit($limit, $end)` - Limits string length
- `lower()` - Converts to lowercase
- `match($pattern)` - Extracts matching content
- `pipe($callback)` - Passes string to callback
- `plural($count)` - Converts to plural
- `replace($from, $to)` - Replaces occurrences
- `singular()` - Converts to singular
- `slug()` - Converts to URL slug
- `snake()` - Converts to snake_case
- `squish()` - Removes whitespace
- `start($value)` - Ensures string starts with value
- `startsWith($needle)` - Checks if string starts with value
- `studly()` - Converts to StudlyCase
- `substr($start, $length)` - Extracts substring
- `title()` - Converts to Title Case
- `trim()` - Removes whitespace
- `ucfirst()` - Capitalizes first character
- `upper()` - Converts to uppercase
- `when($condition, $callback)` - Executes callback if condition
- `whenContains($needle, $callback)` - Executes callback if contains
- `whenEmpty($callback)` - Executes callback if empty
- `whenNotEmpty($callback)` - Executes callback if not empty
- `wordCount()` - Returns word count
- `words($limit, $end)` - Returns words