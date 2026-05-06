---
title: Prompts
description: Beautiful and user-friendly CLI forms with browser-like features
url: https://laravel.com/docs/13.x/prompts
tags: [packages]
cssclasses:
  - ai
  - color-purple
color: purple
---

# Prompts

-   [Introduction](#introduction)
-   [Installation](#installation)
-   [Available Prompts](#available-prompts)
    -   [Text](#text)
    -   [Textarea](#textarea)
    -   [Number](#number)
    -   [Password](#password)
    -   [Confirm](#confirm)
    -   [Select](#select)
    -   [Multi-select](#multiselect)
    -   [Suggest](#suggest)
    -   [Search](#search)
    -   [Multi-search](#multisearch)
    -   [Pause](#pause)
    -   [Autocomplete](#autocomplete)
-   [Transforming Input Before Validation](#transforming-input-before-validation)
-   [Forms](#forms)
-   [Informational Messages](#informational-messages)
-   [Tables](#tables)
-   [Spin](#spin)
-   [Progress Bar](#progress)
-   [Task](#task)
-   [Stream](#stream)
-   [Terminal Title](#terminal-title)
-   [Clearing the Terminal](#clear)
-   [Terminal Considerations](#terminal-considerations)
-   [Unsupported Environments and Fallbacks](#fallbacks)
-   [Testing](#testing)

## [Introduction](#introduction)

[Laravel Prompts](https://github.com/laravel/prompts) is a PHP package for adding beautiful and user-friendly forms to your command-line applications, with browser-like features including placeholder text and validation.

![](https://laravel.com/img/docs/prompts-example.png)

Laravel Prompts is perfect for accepting user input in your [[05-digging-deeper/01-artisan-console.md#writing-commands|Artisan console commands]], but it may also be used in any command-line PHP project.

Laravel Prompts supports macOS, Linux, and Windows with WSL. For more information, please see our documentation on [unsupported environments & fallbacks](#fallbacks).

## [Installation](#installation)

Laravel Prompts is already included with the latest release of Laravel.

Laravel Prompts may also be installed in your other PHP projects by using the Composer package manager:

```
1composer require laravel/prompts
composer require laravel/prompts
```

## [Available Prompts](#available-prompts)

### [Text](#text)

The `text` function will prompt the user with the given question, accept their input, and then return it:

```
1use function Laravel\Prompts\text;2 3$name = text('What is your name?');
use function Laravel\Prompts\text;

$name = text('What is your name?');
```

You may also include placeholder text, a default value, and an informational hint:

```
1$name = text(2    label: 'What is your name?',3    placeholder: 'E.g. Taylor Otwell',4    default: $user?->name,5    hint: 'This will be displayed on your profile.'6);
$name = text(
    label: 'What is your name?',
    placeholder: 'E.g. Taylor Otwell',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

#### [Required Values](#text-required)

If you require a value to be entered, you may pass the `required` argument:

```
1$name = text(2    label: 'What is your name?',3    required: true4);
$name = text(
    label: 'What is your name?',
    required: true
);
```

If you would like to customize the validation message, you may also pass a string:

```
1$name = text(2    label: 'What is your name?',3    required: 'Your name is required.'4);
$name = text(
    label: 'What is your name?',
    required: 'Your name is required.'
);
```

#### [Additional Validation](#text-validation)

Finally, if you would like to perform additional validation logic, you may pass a closure to the `validate` argument:

```
1$name = text(2    label: 'What is your name?',3    validate: fn (string $value) => match (true) {4        strlen($value) < 3 => 'The name must be at least 3 characters.',5        strlen($value) > 255 => 'The name must not exceed 255 characters.',6        default => null7    }8);
$name = text(
    label: 'What is your name?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

The closure will receive the value that has been entered and may return an error message, or `null` if the validation passes.

Alternatively, you may leverage the power of Laravel's [[04-the-basics/12-validation.md|validator]]. To do so, provide an array containing the name of the attribute and the desired validation rules to the `validate` argument:

```
1$name = text(2    label: 'What is your name?',3    validate: ['name' => 'required|max:255|unique:users']4);
$name = text(
    label: 'What is your name?',
    validate: ['name' => 'required|max:255|unique:users']
);
```

### [Textarea](#textarea)

The `textarea` function will prompt the user with the given question, accept their input via a multi-line textarea, and then return it:

```
1use function Laravel\Prompts\textarea;2 3$story = textarea('Tell me a story.');
use function Laravel\Prompts\textarea;

$story = textarea('Tell me a story.');
```

You may also include placeholder text, a default value, and an informational hint:

```
1$story = textarea(2    label: 'Tell me a story.',3    placeholder: 'This is a story about...',4    hint: 'This will be displayed on your profile.'5);
$story = textarea(
    label: 'Tell me a story.',
    placeholder: 'This is a story about...',
    hint: 'This will be displayed on your profile.'
);
```

#### [Required Values](#textarea-required)

If you require a value to be entered, you may pass the `required` argument:

```
1$story = textarea(2    label: 'Tell me a story.',3    required: true4);
$story = textarea(
    label: 'Tell me a story.',
    required: true
);
```

If you would like to customize the validation message, you may also pass a string:

```
1$story = textarea(2    label: 'Tell me a story.',3    required: 'A story is required.'4);
$story = textarea(
    label: 'Tell me a story.',
    required: 'A story is required.'
);
```

#### [Additional Validation](#textarea-validation)

Finally, if you would like to perform additional validation logic, you may pass a closure to the `validate` argument:

```
1$story = textarea(2    label: 'Tell me a story.',3    validate: fn (string $value) => match (true) {4        strlen($value) < 250 => 'The story must be at least 250 characters.',5        strlen($value) > 10000 => 'The story must not exceed 10,000 characters.',6        default => null7    }8);
$story = textarea(
    label: 'Tell me a story.',
    validate: fn (string $value) => match (true) {
        strlen($value) < 250 => 'The story must be at least 250 characters.',
        strlen($value) > 10000 => 'The story must not exceed 10,000 characters.',
        default => null
    }
);
```

The closure will receive the value that has been entered and may return an error message, or `null` if the validation passes.

Alternatively, you may leverage the power of Laravel's [[04-the-basics/12-validation.md|validator]]. To do so, provide an array containing the name of the attribute and the desired validation rules to the `validate` argument:

```
1$story = textarea(2    label: 'Tell me a story.',3    validate: ['story' => 'required|max:10000']4);
$story = textarea(
    label: 'Tell me a story.',
    validate: ['story' => 'required|max:10000']
);
```

### [Number](#number)

The `number` function will prompt the user with the given question, accept their numeric input, and then return it. The `number` function allows the user to use the up and down arrow keys to manipulate the number:

```
1use function Laravel\Prompts\number;2 3$number = number('How many copies would you like?');
use function Laravel\Prompts\number;

$number = number('How many copies would you like?');
```

You may also include placeholder text, a default value, and an informational hint:

```
1$name = number(2    label: 'How many copies would you like?',3    placeholder: '5',4    default: 1,5    hint: 'This will be determine how many copies to create.'6);
$name = number(
    label: 'How many copies would you like?',
    placeholder: '5',
    default: 1,
    hint: 'This will be determine how many copies to create.'
);
```

#### [Required Values](#number-required)

If you require a value to be entered, you may pass the `required` argument:

```
1$copies = number(2    label: 'How many copies would you like?',3    required: true4);
$copies = number(
    label: 'How many copies would you like?',
    required: true
);
```

If you would like to customize the validation message, you may also pass a string:

```
1$copies = number(2    label: 'How many copies would you like?',3    required: 'A number of copies is required.'4);
$copies = number(
    label: 'How many copies would you like?',
    required: 'A number of copies is required.'
);
```

#### [Additional Validation](#number-validation)

Finally, if you would like to perform additional validation logic, you may pass a closure to the `validate` argument:

```
1$copies = number(2    label: 'How many copies would you like?',3    validate: fn (?int $value) => match (true) {4        $value < 1 => 'At least one copy is required.',5        $value > 100 => 'You may not create more than 100 copies.',6        default => null7    }8);
$copies = number(
    label: 'How many copies would you like?',
    validate: fn (?int $value) => match (true) {
        $value < 1 => 'At least one copy is required.',
        $value > 100 => 'You may not create more than 100 copies.',
        default => null
    }
);
```

The closure will receive the value that has been entered and may return an error message, or `null` if the validation passes.

Alternatively, you may leverage the power of Laravel's [[04-the-basics/12-validation.md|validator]]. To do so, provide an array containing the name of the attribute and the desired validation rules to the `validate` argument:

```
1$copies = number(2    label: 'How many copies would you like?',3    validate: ['copies' => 'required|integer|min:1|max:100']4);
$copies = number(
    label: 'How many copies would you like?',
    validate: ['copies' => 'required|integer|min:1|max:100']
);
```

### [Password](#password)

The `password` function is similar to the `text` function, but the user's input will be masked as they type in the console. This is useful when asking for sensitive information such as passwords:

```
1use function Laravel\Prompts\password;2 3$password = password('What is your password?');
use function Laravel\Prompts\password;

$password = password('What is your password?');
```

You may also include placeholder text and an informational hint:

```
1$password = password(2    label: 'What is your password?',3    placeholder: 'password',4    hint: 'Minimum 8 characters.'5);
$password = password(
    label: 'What is your password?',
    placeholder: 'password',
    hint: 'Minimum 8 characters.'
);
```

#### [Required Values](#password-required)

If you require a value to be entered, you may pass the `required` argument:

```
1$password = password(2    label: 'What is your password?',3    required: true4);
$password = password(
    label: 'What is your password?',
    required: true
);
```

If you would like to customize the validation message, you may also pass a string:

```
1$password = password(2    label: 'What is your password?',3    required: 'The password is required.'4);
$password = password(
    label: 'What is your password?',
    required: 'The password is required.'
);
```

#### [Additional Validation](#password-validation)

Finally, if you would like to perform additional validation logic, you may pass a closure to the `validate` argument:

```
1$password = password(2    label: 'What is your password?',3    validate: fn (string $value) => match (true) {4        strlen($value) < 8 => 'The password must be at least 8 characters.',5        default => null6    }7);
$password = password(
    label: 'What is your password?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 8 => 'The password must be at least 8 characters.',
        default => null
    }
);
```

The closure will receive the value that has been entered and may return an error message, or `null` if the validation passes.

Alternatively, you may leverage the power of Laravel's [[04-the-basics/12-validation.md|validator]]. To do so, provide an array containing the name of the attribute and the desired validation rules to the `validate` argument:

```
1$password = password(2    label: 'What is your password?',3    validate: ['password' => 'min:8']4);
$password = password(
    label: 'What is your password?',
    validate: ['password' => 'min:8']
);
```

### [Confirm](#confirm)

If you need to ask the user for a "yes or no" confirmation, you may use the `confirm` function. Users may use the arrow keys or press `y` or `n` to select their response. This function will return either `true` or `false`.

```
1use function Laravel\Prompts\confirm;2 3$confirmed = confirm('Do you accept the terms?');
use function Laravel\Prompts\confirm;

$confirmed = confirm('Do you accept the terms?');
```

You may also include a default value, customized wording for the "Yes" and "No" labels, and an informational hint:

```
1$confirmed = confirm(2    label: 'Do you accept the terms?',3    default: false,4    yes: 'I accept',5    no: 'I decline',6    hint: 'The terms must be accepted to continue.'7);
$confirmed = confirm(
    label: 'Do you accept the terms?',
    default: false,
    yes: 'I accept',
    no: 'I decline',
    hint: 'The terms must be accepted to continue.'
);
```

#### [Requiring "Yes"](#confirm-required)

If necessary, you may require your users to select "Yes" by passing the `required` argument:

```
1$confirmed = confirm(2    label: 'Do you accept the terms?',3    required: true4);
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: true
);
```

If you would like to customize the validation message, you may also pass a string:

```
1$confirmed = confirm(2    label: 'Do you accept the terms?',3    required: 'You must accept the terms to continue.'4);
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: 'You must accept the terms to continue.'
);
```

### [Select](#select)

If you need the user to select from a predefined set of choices, you may use the `select` function:

```
1use function Laravel\Prompts\select;2 3$role = select(4    label: 'What role should the user have?',5    options: ['Member', 'Contributor', 'Owner']6);
use function Laravel\Prompts\select;

$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner']
);
```

You may also specify the default choice and an informational hint:

```
1$role = select(2    label: 'What role should the user have?',3    options: ['Member', 'Contributor', 'Owner'],4    default: 'Owner',5    hint: 'The role may be changed at any time.'6);
$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner'],
    default: 'Owner',
    hint: 'The role may be changed at any time.'
);
```

You may also pass an associative array to the `options` argument to have the selected key returned instead of its value:

```
1$role = select(2    label: 'What role should the user have?',3    options: [4        'member' => 'Member',5        'contributor' => 'Contributor',6        'owner' => 'Owner',7    ],8    default: 'owner'9);
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    default: 'owner'
);
```

Up to five options will be displayed before the list begins to scroll. You may customize this by passing the `scroll` argument:

```
1$role = select(2    label: 'Which category would you like to assign?',3    options: Category::pluck('name', 'id'),4    scroll: 105);
$role = select(
    label: 'Which category would you like to assign?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

#### [Secondary Information](#select-info)

The `info` argument may be used to display additional information about the currently highlighted option. When a closure is provided, it will receive the value of the currently highlighted option and should return a string or `null`:

```
 1$role = select( 2    label: 'What role should the user have?', 3    options: [ 4        'member' => 'Member', 5        'contributor' => 'Contributor', 6        'owner' => 'Owner', 7    ], 8    info: fn (string $value) => match ($value) { 9        'member' => 'Can view and comment.',10        'contributor' => 'Can view, comment, and edit.',11        'owner' => 'Full access to all resources.',12        default => null,13    }14);
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    info: fn (string $value) => match ($value) {
        'member' => 'Can view and comment.',
        'contributor' => 'Can view, comment, and edit.',
        'owner' => 'Full access to all resources.',
        default => null,
    }
);
```

You may also pass a static string to the `info` argument if the information does not depend on the highlighted option:

```
1$role = select(2    label: 'What role should the user have?',3    options: ['Member', 'Contributor', 'Owner'],4    info: 'The role may be changed at any time.'5);
$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner'],
    info: 'The role may be changed at any time.'
);
```

#### [Additional Validation](#select-validation)

Unlike other prompt functions, the `select` function doesn't accept the `required` argument because it is not possible to select nothing. However, you may pass a closure to the `validate` argument if you need to present an option but prevent it from being selected:

```
 1$role = select( 2    label: 'What role should the user have?', 3    options: [ 4        'member' => 'Member', 5        'contributor' => 'Contributor', 6        'owner' => 'Owner', 7    ], 8    validate: fn (string $value) => 9        $value === 'owner' && User::where('role', 'owner')->exists()10            ? 'An owner already exists.'11            : null12);
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    validate: fn (string $value) =>
        $value === 'owner' && User::where('role', 'owner')->exists()
            ? 'An owner already exists.'
            : null
);
```

If the `options` argument is an associative array, then the closure will receive the selected key, otherwise it will receive the selected value. The closure may return an error message, or `null` if the validation passes.

### [Multi-select](#multiselect)

If you need the user to be able to select multiple options, you may use the `multiselect` function:

```
1use function Laravel\Prompts\multiselect;2 3$permissions = multiselect(4    label: 'What permissions should be assigned?',5    options: ['Read', 'Create', 'Update', 'Delete']6);
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete']
);
```

You may also specify default choices and an informational hint:

```
1use function Laravel\Prompts\multiselect;2 3$permissions = multiselect(4    label: 'What permissions should be assigned?',5    options: ['Read', 'Create', 'Update', 'Delete'],6    default: ['Read', 'Create'],7    hint: 'Permissions may be updated at any time.'8);
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete'],
    default: ['Read', 'Create'],
    hint: 'Permissions may be updated at any time.'
);
```

You may also pass an associative array to the `options` argument to return the selected options' keys instead of their values:

```
 1$permissions = multiselect( 2    label: 'What permissions should be assigned?', 3    options: [ 4        'read' => 'Read', 5        'create' => 'Create', 6        'update' => 'Update', 7        'delete' => 'Delete', 8    ], 9    default: ['read', 'create']10);
$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    default: ['read', 'create']
);
```

Up to five options will be displayed before the list begins to scroll. You may customize this by passing the `scroll` argument:

```
1$categories = multiselect(2    label: 'What categories should be assigned?',3    options: Category::pluck('name', 'id'),4    scroll: 105);
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

#### [Secondary Information](#multiselect-info)

The `info` argument may be used to display additional information about the currently highlighted option. When a closure is provided, it will receive the value of the currently highlighted option and should return a string or `null`:

```
 1$permissions = multiselect( 2    label: 'What permissions should be assigned?', 3    options: [ 4        'read' => 'Read', 5        'create' => 'Create', 6        'update' => 'Update', 7        'delete' => 'Delete', 8    ], 9    info: fn (string $value) => match ($value) {10        'read' => 'View resources and their properties.',11        'create' => 'Create new resources.',12        'update' => 'Modify existing resources.',13        'delete' => 'Permanently remove resources.',14        default => null,15    }16);
$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    info: fn (string $value) => match ($value) {
        'read' => 'View resources and their properties.',
        'create' => 'Create new resources.',
        'update' => 'Modify existing resources.',
        'delete' => 'Permanently remove resources.',
        default => null,
    }
);
```

#### [Requiring a Value](#multiselect-required)

By default, the user may select zero or more options. You may pass the `required` argument to enforce one or more options instead:

```
1$categories = multiselect(2    label: 'What categories should be assigned?',3    options: Category::pluck('name', 'id'),4    required: true5);
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: true
);
```

If you would like to customize the validation message, you may provide a string to the `required` argument:

```
1$categories = multiselect(2    label: 'What categories should be assigned?',3    options: Category::pluck('name', 'id'),4    required: 'You must select at least one category'5);
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: 'You must select at least one category'
);
```

#### [Additional Validation](#multiselect-validation)

You may pass a closure to the `validate` argument if you need to present an option but prevent it from being selected:

```
 1$permissions = multiselect( 2    label: 'What permissions should the user have?', 3    options: [ 4        'read' => 'Read', 5        'create' => 'Create', 6        'update' => 'Update', 7        'delete' => 'Delete', 8    ], 9    validate: fn (array $values) => ! in_array('read', $values)10        ? 'All users require the read permission.'11        : null12);
$permissions = multiselect(
    label: 'What permissions should the user have?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    validate: fn (array $values) => ! in_array('read', $values)
        ? 'All users require the read permission.'
        : null
);
```

If the `options` argument is an associative array then the closure will receive the selected keys, otherwise it will receive the selected values. The closure may return an error message, or `null` if the validation passes.

### [Suggest](#suggest)

The `suggest` function can be used to provide auto-completion for possible choices. The user can still provide any answer, regardless of the auto-completion hints:

```
1use function Laravel\Prompts\suggest;2 3$name = suggest('What is your name?', ['Taylor', 'Dayle']);
use function Laravel\Prompts\suggest;

$name = suggest('What is your name?', ['Taylor', 'Dayle']);
```

Alternatively, you may pass a closure as the second argument to the `suggest` function. The closure will be called each time the user types an input character. The closure should accept a string parameter containing the user's input so far and return an array of options for auto-completion:

```
1$name = suggest(2    label: 'What is your name?',3    options: fn ($value) => collect(['Taylor', 'Dayle'])4        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))5)
$name = suggest(
    label: 'What is your name?',
    options: fn ($value) => collect(['Taylor', 'Dayle'])
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
)
```

You may also include placeholder text, a default value, and an informational hint:

```
1$name = suggest(2    label: 'What is your name?',3    options: ['Taylor', 'Dayle'],4    placeholder: 'E.g. Taylor',5    default: $user?->name,6    hint: 'This will be displayed on your profile.'7);
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    placeholder: 'E.g. Taylor',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

#### [Secondary Information](#suggest-info)

The `info` argument may be used to display additional information about the currently highlighted option. When a closure is provided, it will receive the value of the currently highlighted option and should return a string or `null`:

```
1$name = suggest(2    label: 'What is your name?',3    options: ['Taylor', 'Dayle'],4    info: fn (string $value) => match ($value) {5        'Taylor' => 'Administrator',6        'Dayle' => 'Contributor',7        default => null,8    }9);
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    info: fn (string $value) => match ($value) {
        'Taylor' => 'Administrator',
        'Dayle' => 'Contributor',
        default => null,
    }
);
```

#### [Required Values](#suggest-required)

If you require a value to be entered, you may pass the `required` argument:

```
1$name = suggest(2    label: 'What is your name?',3    options: ['Taylor', 'Dayle'],4    required: true5);
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: true
);
```

If you would like to customize the validation message, you may also pass a string:

```
1$name = suggest(2    label: 'What is your name?',3    options: ['Taylor', 'Dayle'],4    required: 'Your name is required.'5);
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: 'Your name is required.'
);
```

#### [Additional Validation](#suggest-validation)

Finally, if you would like to perform additional validation logic, you may pass a closure to the `validate` argument:

```
1$name = suggest(2    label: 'What is your name?',3    options: ['Taylor', 'Dayle'],4    validate: fn (string $value) => match (true) {5        strlen($value) < 3 => 'The name must be at least 3 characters.',6        strlen($value) > 255 => 'The name must not exceed 255 characters.',7        default => null8    }9);
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

The closure will receive the value that has been entered and may return an error message, or `null` if the validation passes.

Alternatively, you may leverage the power of Laravel's [[04-the-basics/12-validation.md|validator]]. To do so, provide an array containing the name of the attribute and the desired validation rules to the `validate` argument:

```
1$name = suggest(2    label: 'What is your name?',3    options: ['Taylor', 'Dayle'],4    validate: ['name' => 'required|min:3|max:255']5);
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    validate: ['name' => 'required|min:3|max:255']
);
```

### [Search](#search)

If you have a lot of options for the user to select from, the `search` function allows the user to type a search query to filter the results before using the arrow keys to select an option:

```
1use function Laravel\Prompts\search;2 3$id = search(4    label: 'Search for the user that should receive the mail',5    options: fn (string $value) => strlen($value) > 06        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()7        : []8);
use function Laravel\Prompts\search;

$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

The closure will receive the text that has been typed by the user so far and must return an array of options. If you return an associative array then the selected option's key will be returned, otherwise its value will be returned instead.

When filtering an array where you intend to return the value, you should use the `array_values` function or the `values` Collection method to ensure the array doesn't become associative:

```
1$names = collect(['Taylor', 'Abigail']);2 3$selected = search(4    label: 'Search for the user that should receive the mail',5    options: fn (string $value) => $names6        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))7        ->values()8        ->all(),9);
$names = collect(['Taylor', 'Abigail']);

$selected = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

You may also include placeholder text and an informational hint:

```
1$id = search(2    label: 'Search for the user that should receive the mail',3    placeholder: 'E.g. Taylor Otwell',4    options: fn (string $value) => strlen($value) > 05        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()6        : [],7    hint: 'The user will receive an email immediately.'8);
$id = search(
    label: 'Search for the user that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

Up to five options will be displayed before the list begins to scroll. You may customize this by passing the `scroll` argument:

```
1$id = search(2    label: 'Search for the user that should receive the mail',3    options: fn (string $value) => strlen($value) > 04        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()5        : [],6    scroll: 107);
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

#### [Secondary Information](#search-info)

The `info` argument may be used to display additional information about the currently highlighted option. When a closure is provided, it will receive the value of the currently highlighted option and should return a string or `null`:

```
1$id = search(2    label: 'Search for the user that should receive the mail',3    options: fn (string $value) => strlen($value) > 04        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()5        : [],6    info: fn (int $userId) => User::find($userId)?->email7);
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    info: fn (int $userId) => User::find($userId)?->email
);
```

#### [Additional Validation](#search-validation)

If you would like to perform additional validation logic, you may pass a closure to the `validate` argument:

```
 1$id = search( 2    label: 'Search for the user that should receive the mail', 3    options: fn (string $value) => strlen($value) > 0 4        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all() 5        : [], 6    validate: function (int|string $value) { 7        $user = User::findOrFail($value); 8  9        if ($user->opted_out) {10            return 'This user has opted-out of receiving mail.';11        }12    }13);
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (int|string $value) {
        $user = User::findOrFail($value);

        if ($user->opted_out) {
            return 'This user has opted-out of receiving mail.';
        }
    }
);
```

If the `options` closure returns an associative array, then the closure will receive the selected key, otherwise, it will receive the selected value. The closure may return an error message, or `null` if the validation passes.

### [Multi-search](#multisearch)

If you have a lot of searchable options and need the user to be able to select multiple items, the `multisearch` function allows the user to type a search query to filter the results before using the arrow keys and space-bar to select options:

```
1use function Laravel\Prompts\multisearch;2 3$ids = multisearch(4    'Search for the users that should receive the mail',5    fn (string $value) => strlen($value) > 06        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()7        : []8);
use function Laravel\Prompts\multisearch;

$ids = multisearch(
    'Search for the users that should receive the mail',
    fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

The closure will receive the text that has been typed by the user so far and must return an array of options. If you return an associative array then the selected options' keys will be returned; otherwise, their values will be returned instead.

When filtering an array where you intend to return the value, you should use the `array_values` function or the `values` Collection method to ensure the array doesn't become associative:

```
1$names = collect(['Taylor', 'Abigail']);2 3$selected = multisearch(4    label: 'Search for the users that should receive the mail',5    options: fn (string $value) => $names6        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))7        ->values()8        ->all(),9);
$names = collect(['Taylor', 'Abigail']);

$selected = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

You may also include placeholder text and an informational hint:

```
1$ids = multisearch(2    label: 'Search for the users that should receive the mail',3    placeholder: 'E.g. Taylor Otwell',4    options: fn (string $value) => strlen($value) > 05        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()6        : [],7    hint: 'The user will receive an email immediately.'8);
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

Up to five options will be displayed before the list begins to scroll. You may customize this by providing the `scroll` argument:

```
1$ids = multisearch(2    label: 'Search for the users that should receive the mail',3    options: fn (string $value) => strlen($value) > 04        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()5        : [],6    scroll: 107);
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

#### [Secondary Information](#multisearch-info)

The `info` argument may be used to display additional information about the currently highlighted option. When a closure is provided, it will receive the value of the currently highlighted option and should return a string or `null`:

```
1$ids = multisearch(2    label: 'Search for the users that should receive the mail',3    options: fn (string $value) => strlen($value) > 04        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()5        : [],6    info: fn (int $userId) => User::find($userId)?->email7);
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    info: fn (int $userId) => User::find($userId)?->email
);
```

#### [Requiring a Value](#multisearch-required)

By default, the user may select zero or more options. You may pass the `required` argument to enforce one or more options instead:

```
1$ids = multisearch(2    label: 'Search for the users that should receive the mail',3    options: fn (string $value) => strlen($value) > 04        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()5        : [],6    required: true7);
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: true
);
```

If you would like to customize the validation message, you may also provide a string to the `required` argument:

```
1$ids = multisearch(2    label: 'Search for the users that should receive the mail',3    options: fn (string $value) => strlen($value) > 04        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()5        : [],6    required: 'You must select at least one user.'7);
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: 'You must select at least one user.'
);
```

#### [Additional Validation](#multisearch-validation)

If you would like to perform additional validation logic, you may pass a closure to the `validate` argument:

```
 1$ids = multisearch( 2    label: 'Search for the users that should receive the mail', 3    options: fn (string $value) => strlen($value) > 0 4        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all() 5        : [], 6    validate: function (array $values) { 7        $optedOut = User::whereLike('name', '%a%')->findMany($values); 8  9        if ($optedOut->isNotEmpty()) {10            return $optedOut->pluck('name')->join(', ', ', and ').' have opted out.';11        }12    }13);
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (array $values) {
        $optedOut = User::whereLike('name', '%a%')->findMany($values);

        if ($optedOut->isNotEmpty()) {
            return $optedOut->pluck('name')->join(', ', ', and ').' have opted out.';
        }
    }
);
```

If the `options` closure returns an associative array, then the closure will receive the selected keys; otherwise, it will receive the selected values. The closure may return an error message, or `null` if the validation passes.

### [Pause](#pause)

The `pause` function may be used to display informational text to the user and wait for them to confirm their desire to proceed by pressing the Enter / Return key:

```
1use function Laravel\Prompts\pause;2 3pause('Press ENTER to continue.');
use function Laravel\Prompts\pause;

pause('Press ENTER to continue.');
```

### [Autocomplete](#autocomplete)

The `autocomplete` function can be used to provide inline auto-completion for possible choices. As the user types, suggestions that match their input will appear as ghost text that can be accepted by pressing `Tab` or the right arrow key:

```
1use function Laravel\Prompts\autocomplete;2 3$name = autocomplete(4    label: 'What is your name?',5    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim']6);
use function Laravel\Prompts\autocomplete;

$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim']
);
```

You may also include placeholder text, a default value, and an informational hint:

```
1$name = autocomplete(2    label: 'What is your name?',3    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],4    placeholder: 'E.g. Taylor',5    default: $user?->name,6    hint: 'Use tab to accept, up/down to cycle.'7);
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    placeholder: 'E.g. Taylor',
    default: $user?->name,
    hint: 'Use tab to accept, up/down to cycle.'
);
```

#### [Dynamic Options](#autocomplete-closure)

You may also pass a closure to dynamically generate options based on the user's input. The closure will be called each time the user types a character and should return an array of options for auto-completion:

```
1$file = autocomplete(2    label: 'Which file?',3    options: fn (string $value) => collect($files)4        ->filter(fn ($file) => str_starts_with(strtolower($file), strtolower($value)))5        ->values()6        ->all(),7);
$file = autocomplete(
    label: 'Which file?',
    options: fn (string $value) => collect($files)
        ->filter(fn ($file) => str_starts_with(strtolower($file), strtolower($value)))
        ->values()
        ->all(),
);
```

#### [Required Values](#autocomplete-required)

If you require a value to be entered, you may pass the `required` argument:

```
1$name = autocomplete(2    label: 'What is your name?',3    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],4    required: true5);
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    required: true
);
```

If you would like to customize the validation message, you may also pass a string:

```
1$name = autocomplete(2    label: 'What is your name?',3    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],4    required: 'Your name is required.'5);
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    required: 'Your name is required.'
);
```

#### [Additional Validation](#autocomplete-validation)

Finally, if you would like to perform additional validation logic, you may pass a closure to the `validate` argument:

```
1$name = autocomplete(2    label: 'What is your name?',3    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],4    validate: fn (string $value) => match (true) {5        strlen($value) < 3 => 'The name must be at least 3 characters.',6        strlen($value) > 255 => 'The name must not exceed 255 characters.',7        default => null8    }9);
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

The closure will receive the value that has been entered and may return an error message, or `null` if the validation passes.

## [Transforming Input Before Validation](#transforming-input-before-validation)

Sometimes you may want to transform the prompt input before validation takes place. For example, you may wish to remove white space from any provided strings. To accomplish this, many of the prompt functions provide a `transform` argument, which accepts a closure:

```
1$name = text(2    label: 'What is your name?',3    transform: fn (string $value) => trim($value),4    validate: fn (string $value) => match (true) {5        strlen($value) < 3 => 'The name must be at least 3 characters.',6        strlen($value) > 255 => 'The name must not exceed 255 characters.',7        default => null8    }9);
$name = text(
    label: 'What is your name?',
    transform: fn (string $value) => trim($value),
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

## [Forms](#forms)

Sometimes you may want to combine several prompts together into a single "form" like experience. To accomplish this, you may use the `form` function:

```
1$results = form()2    ->text(3        'What is your name?',4        name: 'name',5        validate: fn ($value) => strlen($value) < 2 ? 'The name must be at least 2 characters.' : null6    )7    ->text(8        'What is your email address?',9        name: 'email',10        validate: fn ($value) => ! filter_var($value, FILTER_VALIDATE_EMAIL) ? 'Please enter a valid email address.' : null11    )12    ->confirm('Do you want to continue?', default: true)13    ->submit();
$results = form()
    ->text(
        'What is your name?',
        name: 'name',
        validate: fn ($value) => strlen($value) < 2 ? 'The name must be at least 2 characters.' : null
    )
    ->text(
        'What is your email address?',
        name: 'email',
        validate: fn ($value) => ! filter_var($value, FILTER_VALIDATE_EMAIL) ? 'Please enter a valid email address.' : null
    )
    ->confirm('Do you want to continue?', default: true)
    ->submit();
```

The `submit` method returns an array of values keyed by their names:

```
1// ['name' => 'Taylor', 'email' => 'taylor@laravel.com', 'confirm' => true]
// ['name' => 'Taylor', 'email' => 'taylor@laravel.com', 'confirm' => true]
```

If you'd like to access the underlying prompt objects directly, you may use the `raw` method instead of `submit`:

```
1$form = form()2    ->text('What is your name?', name: 'name');3 4$results = $form->raw();5$form->text('What is your name?', name: 'name');

$results = $form->raw();
```

## [Informational Messages](#informational-messages)

Sometimes you may simply need to display some information to the user without needing to prompt them for a response. To accomplish this, you may use the `info`, `warn`, and `error` functions:

```
1info('Application started successfully.');2warn('You are currently running in debug mode.');3error('Unable to connect to the database.');
info('Application started successfully.');
warn('You are currently running in debug mode.');
error('Unable to connect to the database.');
```

## [Tables](#tables)

The `table` function allows you to easily render a table of data:

```
1table(2    ['Name', 'Age'],3    [['Taylor', 34], ['Abigail', 28], ['Mohamed', 16]]4);
table(
    ['Name', 'Age'],
    [['Taylor', 34], ['Abigail', 28], ['Mohamed', 16]]
);
```

## [Spin](#spin)

The `spin` function displays a spinner while executing a given callback and displays a checkmark upon completion. This is useful when performing tasks that may take time:

```
1spin(fn () => sleep(3), 'Processing...');
spin(fn () => sleep(3), 'Processing...');
```

## [Progress Bar](#progress)

For longer-running tasks, you may display a progress bar using the `progress` function:

```
1$progress = progress(label: 'Installing...', steps: 10);2 3for ($i = 0; $i < 10; $i++) {4    sleep(1);5    $progress->advance();6}7 8$progress->finish();
$progress = progress(label: 'Installing...', steps: 10);

for ($i = 0; $i < 10; $i++) {
    sleep(1);
    $progress->advance();
}

$progress->finish();
```

## [Task](#task)

The `task` function displays a spinner with a message while the given callback executes. After the callback completes, the spinner is replaced with a checkmark or X depending on whether the callback returned a value indicating an error. If the callback throws an exception, an X will be displayed:

```
1task('Installing dependencies', function () {2    // ...3});
task('Installing dependencies', function () {
    // ...
});
```

You may also capture the result of the callback:

```
1$result = task('Install dependencies', function () {2    return File::get('composer.json');3});
$result = task('Install dependencies', function () {
    return File::get('composer.json');
});
```

## [Stream](#stream)

When using Prompts in a long-running process, you may stream output to the terminal in real-time using the `stream` function. The stream function accepts a label and a callback that receives a `$output` callable:

```
1stream('Building', function ($output) {2    Shell::run('npm install', $output);3    Shell::run('npm run build', $output);4});
stream('Building', function ($output) {
    Shell::run('npm install', $output);
    Shell::run('npm run build', $output);
});
```

The `$output` callable accepts a string that will be streamed to the console in real-time.

## [Terminal Title](#terminal-title)

You may update the terminal title using the `title` function:

```
1title('Installing dependencies...');
title('Installing dependencies...');
```

## [Clearing the Terminal](#clear)

You may clear the terminal using the `clear` function:

```
1clear();
clear();
```

## [Terminal Considerations](#terminal-considerations)

When using Prompts in your CLI application, there are a few things to consider regarding the terminal environment.

### Line Returns

When a prompt function returns a value, it will include any text the user entered that is visible on the line where they pressed Enter / Return. For example, if you asked "What is your name?" and the user typed "Taylor", the string "Taylor" will be returned. The newline character entered when pressing Return / Enter will not be included in the returned value.

### Non-Interactive Environments

Prompts will automatically detect if it is being run in a non-interactive environment such as CI or a piped environment. In these environments, Prompts will fall back to using default values or throw an exception if no default is available. To customize this behavior, you may use the `fallback` argument on any prompt or you may use the `fallbackFor` method to set a fallback for multiple prompts:

```
1use function Laravel\Prompts\fallbackFor;2fallbackFor('text', function () {3    return 'Default value';4});
use function Laravel\Prompts\fallbackFor;

fallbackFor('text', function () {
    return 'Default value';
});
```

Alternatively, you may register a fallback for all prompts using the `always` method:

```
1use function Laravel\Prompts\always;2 3always(fn () => 'Default value');
use function Laravel\Prompts\always;

always(fn () => 'Default value');
```

You may also throw an exception instead of using a fallback by using the `throw` method:

```
1use function Laravel\Prompts\throw;2 3throw();
throw();
```

## [Unsupported Environments and Fallbacks](#fallbacks)

Some terminals may not support the advanced features that Laravel Prompts uses to provide a rich user experience. If a terminal does not support the features used by Prompts, it will automatically fall back to simpler alternatives. If you would like to customize the fallback behavior or always use a specific terminal driver, you may use the `Terminal::use` method in your service provider's `register` method:

```
1<?php 2  3namespace App\Providers; 4  5use Illuminate\Support\ServiceProvider; 6use Laravel\Prompts\Prompt; 7use Laravel\Prompts\Terminal; 8  9class AppServiceProvider extends ServiceProvider10{11    /**12     * Register any application services.13     */14    public function register(): void15    {16        Terminal::use(BasicTerminal::class);17    }18}
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Prompts\Prompt;
use Laravel\Prompts\Terminal;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        Terminal::use(BasicTerminal::class);
    }
}
```

## [Testing](#testing)

To simplify testing when using Prompts, Laravel provides the `Laravel\Prompts\Testing\Assert` class. This class provides methods for asserting that a prompt was rendered and that certain options were available:

```
1use function Laravel\Prompts\testing\text;2 3it('asks for name', function () {4    text('What is your name?');5 6    $this->assertPrompt('What is your name?');7});
use function Laravel\Prompts\testing\text;

it('asks for name', function () {
    text('What is your name?');

    $this->assertPrompt('What is your name?');
});
```

You may use the `assertOptions` method to assert that certain options were available:

```
1use function Laravel\Prompts\testing\select;2 3it('asks for role', function () {4    select('What role should the user have?', [5        'member' => 'Member',6        'contributor' => 'Contributor',7        'owner' => 'Owner',8    ]);9 10    $this->assertPrompt('What role should the user have?')11        ->assertOptions([12            'member' => 'Member',13            'contributor' => 'Contributor',14            'owner' => 'Owner',15        ]);16});
use function Laravel\Prompts\testing\select;

it('asks for role', function () {
    select('What role should the user have?', [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ]);

    $this->assertPrompt('What role should the user have?')
        ->assertOptions([
            'member' => 'Member',
            'contributor' => 'Contributor',
            'owner' => 'Owner',
        ]);
});
```

You may also assert the prompt was rendered with a specific label:

```
1$this->assertPrompt(label: 'Email')2    ->assertLabel('Email');
$this->assertPrompt(label: 'Email')
    ->assertLabel('Email');
```

And you can assert the prompt was rendered with placeholder text:

```
1$this->assertPrompt(label: 'Email', placeholder: 'e.g. Taylor Otwell')2    ->assertPlaceholder('e.g. Taylor Otwell');
$this->assertPrompt(label: 'Email', placeholder: 'e.g. Taylor Otwell')
    ->assertPlaceholder('e.g. Taylor Otwell');
```

You can also assert the prompt was rendered with hint text:

```
1$this->assertPrompt(label: 'Email', hint: 'We will never share your email.')2    ->assertHint('We will never share your email.');
$this->assertPrompt(label: 'Email', hint: 'We will never share your email.')
    ->assertHint('We will never share your email.');
```

You may also assert that the prompt was required:

```
1$this->assertPrompt(label: 'Email', required: true)2    ->assertRequired();
$this->assertPrompt(label: 'Email', required: true)
    ->assertRequired();
```

Or assert a default value was rendered:

```
1$this->assertPrompt(label: 'Email', default: 'taylor@laravel.com')2    ->assertDefaultValue('taylor@laravel.com');
$this->assertPrompt(label: 'Email', default: 'taylor@laravel.com')
    ->assertDefaultValue('taylor@laravel.com');
```

Copy as markdown

### On this page

-   [Introduction](#introduction)
-   [Installation](#installation)
-   [Available Prompts](#available-prompts)
    -   [Text](#text)
    -   [Textarea](#textarea)
    -   [Number](#number)
    -   [Password](#password)
    -   [Confirm](#confirm)
    -   [Select](#select)
    -   [Multi-select](#multiselect)
    -   [Suggest](#suggest)
    -   [Search](#search)
    -   [Multi-search](#multisearch)
    -   [Pause](#pause)
    -   [Autocomplete](#autocomplete)
-   [Transforming Input Before Validation](#transforming-input-before-validation)
-   [Forms](#forms)
-   [Informational Messages](#informational-messages)
-   [Tables](#tables)
-   [Spin](#spin)
-   [Progress Bar](#progress)
-   [Task](#task)
-   [Stream](#stream)
-   [Terminal Title](#terminal-title)
-   [Clearing the Terminal](#clear)
-   [Terminal Considerations](#terminal-considerations)
-   [Unsupported Environments and Fallbacks](#fallbacks)
-   [Testing](#testing)