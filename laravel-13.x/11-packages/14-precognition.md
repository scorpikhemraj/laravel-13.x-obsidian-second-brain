---
title: Precognition
description: Live validation for frontend JavaScript applications using backend validation rules
url: https://laravel.com/docs/13.x/precognition
tags: [packages]
---

# Precognition

-   [Introduction](#introduction)
-   [Live Validation](#live-validation)
    -   [Using Vue](#using-vue)
    -   [Using React](#using-react)
    -   [Using Alpine and Blade](#using-alpine)
    -   [Configuring Axios](#configuring-axios)
-   [Validating Arrays](#validating-arrays)
-   [Customizing Validation Rules](#customizing-validation-rules)
-   [Handling File Uploads](#handling-file-uploads)
-   [Managing Side-Effects](#managing-side-effects)
-   [Testing](#testing)

## [Introduction](#introduction)

Laravel Precognition allows you to anticipate the outcome of a future HTTP request. One of the primary use cases of Precognition is the ability to provide "live" validation for your frontend JavaScript application without having to duplicate your application's backend validation rules.

When Laravel receives a "precognitive request", it will execute all of the route's middleware and resolve the route's controller dependencies, including validating [[04-the-basics/12-validation.md#form-request-validation|form requests]] - but it will not actually execute the route's controller method.

As of Inertia 2.3, Precognition support is built-in. Please consult the [Inertia Forms documentation](https://inertiajs.com/forms) for more information. Earlier Inertia versions require Precognition 0.x.

## [Live Validation](#live-validation)

### [Using Vue](#using-vue)

Using Laravel Precognition, you can offer live validation experiences to your users without having to duplicate your validation rules in your frontend Vue application. To illustrate how it works, let's build a form for creating new users within our application.

First, to enable Precognition for a route, the `HandlePrecognitiveRequests` middleware should be added to the route definition. You should also create a [[04-the-basics/12-validation.md#form-request-validation|form request]] to house the route's validation rules:

```
1use App\Http\Requests\StoreUserRequest;2use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;3 4Route::post('/users', function (StoreUserRequest $request) {5    // ...6})->middleware([HandlePrecognitiveRequests::class]);
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Next, you should install the Laravel Precognition frontend helpers for Vue via NPM:

```
1npm install laravel-precognition-vue
npm install laravel-precognition-vue
```

With the Laravel Precognition package installed, you can now create a form object using Precognition's `useForm` function, providing the HTTP method (`post`), the target URL (`/users`), and the initial form data.

Then, to enable live validation, invoke the form's `validate` method on each input's `change` event, providing the input's name:

```
 1<script setup> 2import { useForm } from 'laravel-precognition-vue'; 3  4const form = useForm('post', '/users', { 5    name: '', 6    email: '', 7}); 8  9const submit = () => form.submit();10</script>11 12<template>13    <form @submit.prevent="submit">14        <label for="name">Name</label>15        <input16            id="name"17            v-model="form.name"18            @change="form.validate('name')"19        />20        <div v-if="form.invalid('name')">21            {{ form.errors.name }}22        </div>23 24        <label for="email">Email</label>25        <input26            id="email"27            type="email"28            v-model="form.email"29            @change="form.validate('email')"30        />31        <div v-if="form.invalid('email')">32            {{ form.errors.email }}33        </div>34 35        <button :disabled="form.processing">36            Create User37        </button>38    </form>39</template>
<script setup>
import { useForm } from 'laravel-precognition-vue';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit();
</script>

<template>
    <form @submit.prevent="submit">
        <label for="name">Name</label>
        <input
            id="name"
            v-model="form.name"
            @change="form.validate('name')"
        />
        <div v-if="form.invalid('name')">
            {{ form.errors.name }}
        </div>

        <label for="email">Email</label>
        <input
            id="email"
            type="email"
            v-model="form.email"
            @change="form.validate('email')"
        />
        <div v-if="form.invalid('email')">
            {{ form.errors.email }}
        </div>

        <button :disabled="form.processing">
            Create User
        </button>
    </form>
</template>
```

Now, as the form is filled by the user, Precognition will provide live validation output powered by the validation rules in the route's form request. When the form's inputs are changed, a debounced "precognitive" validation request will be sent to your Laravel application. You may configure the debounce timeout by calling the form's `setValidationTimeout` function:

```
1form.setValidationTimeout(3000);
form.setValidationTimeout(3000);
```

When a validation request is in-flight, the form's `validating` property will be `true`:

```
1<div v-if="form.validating">2    Validating...3</div>
<div v-if="form.validating">
    Validating...
</div>
```

Any validation errors returned during a validation request or a form submission will automatically populate the form's `errors` object:

```
1<div v-if="form.invalid('email')">2    {{ form.errors.email }}3</div>
<div v-if="form.invalid('email')">
    {{ form.errors.email }}
</div>
```

You can determine if the form has any errors using the form's `hasErrors` property:

```
1<div v-if="form.hasErrors">2    <!-- ... -->3</div>
<div v-if="form.hasErrors">
    <!-- ... -->
</div>
```

You may also determine if an input has passed or failed validation by passing the input's name to the form's `valid` and `invalid` functions, respectively:

```
1<span v-if="form.valid('email')">2    ✅3</span>4 5<span v-else-if="form.invalid('email')">6    ❌7</span>
<span v-if="form.valid('email')">
    ✅
</span>

<span v-else-if="form.invalid('email')">
    ❌
</span>
```

A form input will only appear as valid or invalid once it has changed and a validation response has been received.

If you are validating a subset of a form's inputs with Precognition, it can be useful to manually clear errors. You may use the form's `forgetError` function to achieve this:

```
1<input2    id="avatar"3    type="file"4    @change="(e) => {5        form.avatar = e.target.files[0]6 7        form.forgetError('avatar')8    }"9>
<input
    id="avatar"
    type="file"
    @change="(e) => {
        form.avatar = e.target.files[0]

        form.forgetError('avatar')
    }"
>
```

As we have seen, you can hook into an input's `change` event and validate individual inputs as the user interacts with them; however, you may need to validate inputs that the user has not yet interacted with. This is common when building a "wizard", where you want to validate all visible inputs, whether the user has interacted with them or not, before moving to the next step.

To do this with Precognition, you should call the `validate` method passing the field names you wish to validate to the `only` configuration key. You may handle the validation result with `onSuccess` or `onValidationError` callbacks:

```
1<button2    type="button"3    @click="form.validate({4        only: ['name', 'email', 'phone'],5        onSuccess: (response) => nextStep(),6        onValidationError: (response) => /* ... */,7    })"8>Next Step</button>
<button
    type="button"
    @click="form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })"
>Next Step</button>
```

Of course, you may also execute code in reaction to the response to the form submission. The form's `submit` function returns an Axios request promise. This provides a convenient way to access the response payload, reset the form inputs on successful submission, or handle a failed request:

```
1const submit = () => form.submit()2    .then(response => {3        form.reset();4 5        alert('User created.');6    })7    .catch(error => {8        alert('An error occurred.');9    });
const submit = () => form.submit()
    .then(response => {
        form.reset();

        alert('User created.');
    })
    .catch(error => {
        alert('An error occurred.');
    });
```

You may determine if a form submission request is in-flight by inspecting the form's `processing` property:

```
1<button :disabled="form.processing">2    Submit3</button>
<button :disabled="form.processing">
    Submit
</button>
```

### [Using React](#using-react)

Using Laravel Precognition, you can offer live validation experiences to your users without having to duplicate your validation rules in your frontend React application. To illustrate how it works, let's build a form for creating new users within our application.

First, to enable Precognition for a route, the `HandlePrecognitiveRequests` middleware should be added to the route definition. You should also create a [[04-the-basics/12-validation.md#form-request-validation|form request]] to house the route's validation rules:

```
1use App\Http\Requests\StoreUserRequest;2use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;3 4Route::post('/users', function (StoreUserRequest $request) {5    // ...6})->middleware([HandlePrecognitiveRequests::class]);
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Next, you should install the Laravel Precognition frontend helpers for React via NPM:

```
1npm install laravel-precognition-react
npm install laravel-precognition-react
```

With the Laravel Precognition package installed, you can now create a form object using Precognition's `useForm` function, providing the HTTP method (`post`), the target URL (`/users`), and the initial form data.

To enable live validation, you should listen to each input's `change` and `blur` event. In the `change` event handler, you should set the form's data with the `setData` function, passing the input's name and new value. Then, in the `blur` event handler invoke the form's `validate` method, providing the input's name:

```
 1import { useForm } from 'laravel-precognition-react'; 2  3export default function Form() { 4    const form = useForm('post', '/users', { 5        name: '', 6        email: '', 7    }); 8  9    const submit = (e) => {10        e.preventDefault();11 12        form.submit();13    };14 15    return (16        <form onSubmit={submit}>17            <label htmlFor="name">Name</label>18            <input19                id="name"20                value={form.data.name}21                onChange={(e) => form.setData('name', e.target.value)}22                onBlur={() => form.validate('name')}23            />24            {form.invalid('name') && <div>{form.errors.name}</div>}25 26            <label htmlFor="email">Email</label>27            <input28                id="email"29                value={form.data.email}30                onChange={(e) => form.setData('email', e.target.value)}31                onBlur={() => form.validate('email')}32            />33            {form.invalid('email') && <div>{form.errors.email}</div>}34 35            <button disabled={form.processing}>36                Create User37            </button>38        </form>39    );40};
import { useForm } from 'laravel-precognition-react';

export default function Form() {
    const form = useForm('post', '/users', {
        name: '',
        email: '',
    });

    const submit = (e) => {
        e.preventDefault();

        form.submit();
    };

    return (
        <form onSubmit={submit}>
            <label htmlFor="name">Name</label>
            <input
                id="name"
                value={form.data.name}
                onChange={(e) => form.setData('name', e.target.value)}
                onBlur={() => form.validate('name')}
            />
            {form.invalid('name') && <div>{form.errors.name}</div>}

            <label htmlFor="email">Email</label>
            <input
                id="email"
                value={form.data.email}
                onChange={(e) => form.setData('email', e.target.value)}
                onBlur={() => form.validate('email')}
            />
            {form.invalid('email') && <div>{form.errors.email}</div>}

            <button disabled={form.processing}>
                Create User
            </button>
        </form>
    );
};
```

Now, as the form is filled by the user, Precognition will provide live validation output powered by the validation rules in the route's form request. When the form's inputs are changed, a debounced "precognitive" validation request will be sent to your Laravel application. You may configure the debounce timeout by calling the form's `setValidationTimeout` function:

```
1form.setValidationTimeout(3000);
form.setValidationTimeout(3000);
```

When a validation request is in-flight, the form's `validating` property will be `true`:

```
1{form.validating && <div>Validating...</div>}
{form.validating && <div>Validating...</div>}
```

Any validation errors returned during a validation request or a form submission will automatically populate the form's `errors` object:

```
1{form.invalid('email') && <div>{form.errors.email}</div>}
{form.invalid('email') && <div>{form.errors.email}</div>}
```

You can determine if the form has any errors using the form's `hasErrors` property:

```
1{form.hasErrors && <div><!-- ... --></div>}
{form.hasErrors && <div><!-- ... --></div>}
```

You may also determine if an input has passed or failed validation by passing the input's name to the form's `valid` and `invalid` functions, respectively:

```
1{form.valid('email') && <span>✅</span>}2 3{form.invalid('email') && <span>❌</span>}
{form.valid('email') && <span>✅</span>}

{form.invalid('email') && <span>❌</span>}
```

A form input will only appear as valid or invalid once it has changed and a validation response has been received.

If you are validating a subset of a form's inputs with Precognition, it can be useful to manually clear errors. You may use the form's `forgetError` function to achieve this:

```
1<input2    id="avatar"3    type="file"4    onChange={(e) => {5        form.setData('avatar', e.target.files[0]);6 7        form.forgetError('avatar');8    }}9>
<input
    id="avatar"
    type="file"
    onChange={(e) => {
        form.setData('avatar', e.target.files[0]);

        form.forgetError('avatar');
    }}
>
```

As we have seen, you can hook into an input's `blur` event and validate individual inputs as the user interacts with them; however, you may need to validate inputs that the user has not yet interacted with. This is common when building a "wizard", where you want to validate all visible inputs, whether the user has interacted with them or not, before moving to the next step.

To do this with Precognition, you should call the `validate` method passing the field names you wish to validate to the `only` configuration key. You may handle the validation result with `onSuccess` or `onValidationError` callbacks:

```
1<button2    type="button"3    onClick={() => form.validate({4        only: ['name', 'email', 'phone'],5        onSuccess: (response) => nextStep(),6        onValidationError: (response) => /* ... */,7    })}8>Next Step</button>
<button
    type="button"
    onClick={() => form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })}
>Next Step</button>
```

Of course, you may also execute code in reaction to the response to the form submission. The form's `submit` function returns an Axios request promise. This provides a convenient way to access the response payload, reset the form's inputs on a successful form submission, or handle a failed request:

```
 1const submit = (e) => { 2    e.preventDefault(); 3  4    form.submit() 5        .then(response => { 6            form.reset(); 7  8            alert('User created.'); 9        })10        .catch(error => {11            alert('An error occurred.');12        });13};
const submit = (e) => {
    e.preventDefault();

    form.submit()
        .then(response => {
            form.reset();

            alert('User created.');
        })
        .catch(error => {
            alert('An error occurred.');
        });
};
```

You may determine if a form submission request is in-flight by inspecting the form's `processing` property:

```
1<button disabled={form.processing}>2    Submit3</button>
<button disabled={form.processing}>
    Submit
</button>
```

### [Using Alpine and Blade](#using-alpine)

Using Laravel Precognition, you can offer live validation experiences to your users without having to duplicate your validation rules in your frontend Alpine application. To illustrate how it works, let's build a form for creating new users within our application.

First, to enable Precognition for a route, the `HandlePrecognitiveRequests` middleware should be added to the route definition. You should also create a [[04-the-basics/12-validation.md#form-request-validation|form request]] to house the route's validation rules:

```
1use App\Http\Requests\CreateUserRequest;2use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;3 4Route::post('/users', function (CreateUserRequest $request) {5    // ...6})->middleware([HandlePrecognitiveRequests::class]);
use App\Http\Requests\CreateUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (CreateUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

Next, you should install the Laravel Precognition frontend helpers for Alpine via NPM:

```
1npm install laravel-precognition-alpine
npm install laravel-precognition-alpine
```

Then, register the Precognition plugin with Alpine in your `resources/js/app.js` file:

```
1import Alpine from 'alpinejs';2import Precognition from 'laravel-precognition-alpine';3 4window.Alpine = Alpine;5 6Alpine.plugin(Precognition);7Alpine.start();
import Alpine from 'alpinejs';
import Precognition from 'laravel-precognition-alpine';

window.Alpine = Alpine;

Alpine.plugin(Precognition);
Alpine.start();
```

With the Laravel Precognition package installed and registered, you can now create a form object using Precognition's `$form` "magic", providing the HTTP method (`post`), the target URL (`/users`), and the initial form data.

To enable live validation, you should bind the form's data to its relevant input and then listen to each input's `change` event. In the `change` event handler, you should invoke the form's `validate` method, providing the input's name:

```
 1<form x-data="{ 2    form: $form('post', '/register', { 3        name: '', 4        email: '', 5    }), 6}"> 7    @csrf 8    <label for="name">Name</label> 9    <input10        id="name"11        name="name"12        x-model="form.name"13        @change="form.validate('name')"14    />15    <template x-if="form.invalid('name')">16        <div x-text="form.errors.name"></div>17    </template>18 19    <label for="email">Email</label>20    <input21        id="email"22        name="email"23        x-model="form.email"24        @change="form.validate('email')"25    />26    <template x-if="form.invalid('email')">27        <div x-text="form.errors.email"></div>28    </template>29 30    <button :disabled="form.processing">31        Create User32    </button>33</form>
<form x-data="{
    form: $form('post', '/register', {
        name: '',
        email: '',
    }),
}">
    @csrf
    <label for="name">Name</label>
    <input
        id="name"
        name="name"
        x-model="form.name"
        @change="form.validate('name')"
    />
    <template x-if="form.invalid('name')">
        <div x-text="form.errors.name"></div>
    </template>

    <label for="email">Email</label>
    <input
        id="email"
        name="email"
        x-model="form.email"
        @change="form.validate('email')"
    />
    <template x-if="form.invalid('email')">
        <div x-text="form.errors.email"></div>
    </template>

    <button :disabled="form.processing">
        Create User
    </button>
</form>
```

Now, as the form is filled by the user, Precognition will provide live validation output powered by the validation rules in the route's form request. When the form's inputs are changed, a debounced "precognitive" validation request will be sent to your Laravel application. You may configure the debounce timeout by calling the form's `setValidationTimeout` function:

```
1form.setValidationTimeout(3000);
form.setValidationTimeout(3000);
```

When a validation request is in-flight, the form's `validating` property will be `true`:

```
1<template x-if="form.validating">2    <div>Validating...</div>3</template>
<template x-if="form.validating">
    <div>Validating...</div>
</template>
```

Any validation errors returned during a validation request or a form submission will automatically populate the form's `errors` object:

```
1<template x-if="form.invalid('email')">2    <div x-text="form.errors.email"></div>3</template>
<template x-if="form.invalid('email')">
    <div x-text="form.errors.email"></div>
</template>
```

You can determine if the form has any errors using the form's `hasErrors` property:

```
1<template x-if="form.hasErrors">2    <div><!-- ... --></div>3</template>
<template x-if="form.hasErrors">
    <div><!-- ... --></div>
</template>
```

You may also determine if an input has passed or failed validation by passing the input's name to the form's `valid` and `invalid` functions, respectively:

```
1<template x-if="form.valid('email')">2    <span>✅</span>3</template>4 5<template x-if="form.invalid('email')">6    <span>❌</span>7</template>
<template x-if="form.valid('email')">
    <span>✅</span>
</template>

<template x-if="form.invalid('email')">
    <span>❌</span>
</template>
```

A form input will only appear as valid or invalid once it has changed and a validation response has been received.

As we have seen, you can hook into an input's `change` event and validate individual inputs as the user interacts with them; however, you may need to validate inputs that the user has not yet interacted with. This is common when building a "wizard", where you want to validate all visible inputs, whether the user has interacted with them or not, before moving to the next step.

To do this with Precognition, you should call the `validate` method passing the field names you wish to validate to the `only` configuration key. You may handle the validation result with `onSuccess` or `onValidationError` callbacks:

```
1<button2    type="button"3    @click="form.validate({4        only: ['name', 'email', 'phone'],5        onSuccess: (response) => nextStep(),6        onValidationError: (response) => /* ... */,7    })"8>Next Step</button>
<button
    type="button"
    @click="form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })"
>Next Step</button>
```

You may determine if a form submission request is in-flight by inspecting the form's `processing` property:

```
1<button :disabled="form.processing">2    Submit3</button>
<button :disabled="form.processing">
    Submit
</button>
```

#### [Repopulating Old Form Data](#repopulating-old-form-data)

In the user creation example discussed above, we are using Precognition to perform live validation; however, we are performing a traditional server-side form submission to submit the form. So, the form should be populated with any "old" input and validation errors returned from the server-side form submission:

```
1<form x-data="{2    form: $form('post', '/register', {3        name: '{{ old('name') }}',4        email: '{{ old('email') }}',5    }).setErrors({{ Js::from($errors->messages()) }}),6}">
<form x-data="{
    form: $form('post', '/register', {
        name: '{{ old('name') }}',
        email: '{{ old('email') }}',
    }).setErrors({{ Js::from($errors->messages()) }}),
}">
```

Alternatively, if you would like to submit the form via XHR you may use the form's `submit` function, which returns an Axios request promise:

```
 1<form 2    x-data="{ 3        form: $form('post', '/register', { 4            name: '', 5            email: '', 6        }), 7        submit() { 8            this.form.submit() 9                .then(response => {10                    this.form.reset();11 12                    alert('User created.')13                })14                .catch(error => {15                    alert('An error occurred.');16                });17        },18    }"19    @submit.prevent="submit"20>
<form
    x-data="{
        form: $form('post', '/register', {
            name: '',
            email: '',
        }),
        submit() {
            this.form.submit()
                .then(response => {
                    this.form.reset();

                    alert('User created.')
                })
                .catch(error => {
                    alert('An error occurred.');
                });
        },
    }"
    @submit.prevent="submit"
>
```

### [Configuring Axios](#configuring-axios)

The Precognition validation libraries use the [Axios](https://github.com/axios/axios) HTTP client to send requests to your application's backend. For convenience, the Axios instance may be customized if required by your application. For example, when using the `laravel-precognition-vue` library, you may add additional request headers to each outgoing request in your application's `resources/js/app.js` file:

```
1import { client } from 'laravel-precognition-vue';2 3client.axios().defaults.headers.common['Authorization'] = authToken;
import { client } from 'laravel-precognition-vue';

client.axios().defaults.headers.common['Authorization'] = authToken;
```

Or, if you already have a configured Axios instance for your application, you may tell Precognition to use that instance instead:

```
1import Axios from 'axios';2import { client } from 'laravel-precognition-vue';3 4window.axios = Axios.create()5window.axios.defaults.headers.common['Authorization'] = authToken;6 7client.use(window.axios)
import Axios from 'axios';
import { client } from 'laravel-precognition-vue';

window.axios = Axios.create()
window.axios.defaults.headers.common['Authorization'] = authToken;

client.use(window.axios)
```

## [Validating Arrays](#validating-arrays)

You may use wildcards to validate fields within arrays or nested objects. Each `*` matches a single path segment:

```
1// Validate email for all users in an array...2form.validate('users.*.email');3 4// Validate all fields in a profile object...5form.validate('profile.*');6 7// Validate all fields for all users...8form.validate('users.*.*');
// Validate email for all users in an array...
form.validate('users.*.email');

// Validate all fields in a profile object...
form.validate('profile.*');

// Validate all fields for all users...
form.validate('users.*.*');
```

## [Customizing Validation Rules](#customizing-validation-rules)

It is possible to customize the validation rules executed during a precognitive request by using the request's `isPrecognitive` method.

For example, on a user creation form, we may want to validate that a password is "uncompromised" only on the final form submission. For precognitive validation requests, we will simply validate that the password is required and has a minimum of 8 characters. Using the `isPrecognitive` method, we can customize the rules defined by our form request:

```
 1<?php 2  3namespace App\Http\Requests; 4  5use Illuminate\Foundation\Http\FormRequest; 6use Illuminate\Validation\Rules\Password; 7  8class StoreUserRequest extends FormRequest 9{10    /**11     * Get the validation rules that apply to the request.12     *13     * @return array14     */15    protected function rules()16    {17        return [18            'password' => [19                'required',20                $this->isPrecognitive()21                    ? Password::min(8)22                    : Password::min(8)->uncompromised(),23            ],24            // ...25        ];26    }27}
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class StoreUserRequest extends FormRequest
{
    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    protected function rules()
    {
        return [
            'password' => [
                'required',
                $this->isPrecognitive()
                    ? Password::min(8)
                    : Password::min(8)->uncompromised(),
            ],
            // ...
        ];
    }
}
```

## [Handling File Uploads](#handling-file-uploads)

By default, Laravel Precognition does not upload or validate files during a precognitive validation request. This ensure that large files are not unnecessarily uploaded multiple times.

Because of this behavior, you should ensure that your application [customizes the corresponding form request's validation rules](#customizing-validation-rules) to specify the field is only required for full form submissions:

```
 1/** 2 * Get the validation rules that apply to the request. 3 * 4 * @return array 5 */ 6protected function rules() 7{ 8    return [ 9        'avatar' => [10            ...$this->isPrecognitive() ? [] : ['required'],11            'image',12            'mimes:jpg,png',13            'dimensions:ratio=3/2',14        ],15        // ...16    ];17}
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
protected function rules()
{
    return [
        'avatar' => [
            ...$this->isPrecognitive() ? [] : ['required'],
            'image',
            'mimes:jpg,png',
            'dimensions:ratio=3/2',
        ],
        // ...
    ];
}
```

If you would like to include files in every validation request, you may invoke the `validateFiles` function on your client-side form instance:

```
1form.validateFiles();
form.validateFiles();
```

## [Managing Side-Effects](#managing-side-effects)

When adding the `HandlePrecognitiveRequests` middleware to a route, you should consider if there are any side-effects in *other* middleware that should be skipped during a precognitive request.

For example, you may have a middleware that increments the total number of "interactions" each user has with your application, but you may not want precognitive requests to be counted as an interaction. To accomplish this, we may check the request's `isPrecognitive` method before incrementing the interaction count:

```
 1<?php 2  3namespace App\Http\Middleware; 4  5use App\Facades\Interaction; 6use Closure; 7use Illuminate\Http\Request; 8  9class InteractionMiddleware10{11    /**12     * Handle an incoming request.13     */14    public function handle(Request $request, Closure $next): mixed15    {16        if (! $request->isPrecognitive()) {17            Interaction::incrementFor($request->user());18        }19 20        return $next($request);21    }22}
<?php

namespace App\Http\Middleware;

use App\Facades\Interaction;
use Closure;
use Illuminate\Http\Request;

class InteractionMiddleware
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): mixed
    {
        if (! $request->isPrecognitive()) {
            Interaction::incrementFor($request->user());
        }

        return $next($request);
    }
}
```

## [Testing](#testing)

If you would like to make precognitive requests in your tests, Laravel's `TestCase` includes a `withPrecognition` helper which will add the `Precognition` request header.

Additionally, if you would like to assert that a precognitive request was successful, e.g., did not return any validation errors, you may use the `assertSuccessfulPrecognition` method on the response:

Pest PHPUnit

```
 1it('validates registration form with precognition', function () { 2    $response = $this->withPrecognition() 3        ->post('/register', [ 4            'name' => 'Taylor Otwell', 5        ]); 6  7    $response->assertSuccessfulPrecognition(); 8  9    expect(User::count())->toBe(0);10});
it('validates registration form with precognition', function () {
    $response = $this->withPrecognition()
        ->post('/register', [
            'name' => 'Taylor Otwell',
        ]);

    $response->assertSuccessfulPrecognition();

    expect(User::count())->toBe(0);
});
```

```
 1public function test_it_validates_registration_form_with_precognition() 2{ 3    $response = $this->withPrecognition() 4        ->post('/register', [ 5            'name' => 'Taylor Otwell', 6        ]); 7  8    $response->assertSuccessfulPrecognition(); 9    $this->assertSame(0, User::count());10}
public function test_it_validates_registration_form_with_precognition()
{
    $response = $this->withPrecognition()
        ->post('/register', [
            'name' => 'Taylor Otwell',
        ]);

    $response->assertSuccessfulPrecognition();
    $this->assertSame(0, User::count());
}
```

Copy as markdown

### On this page

-   [Introduction](#introduction)
-   [Live Validation](#live-validation)
    -   [Using Vue](#using-vue)
    -   [Using React](#using-react)
    -   [Using Alpine and Blade](#using-alpine)
    -   [Configuring Axios](#configuring-axios)
-   [Validating Arrays](#validating-arrays)
-   [Customizing Validation Rules](#customizing-validation-rules)
-   [Handling File Uploads](#handling-file-uploads)
-   [Managing Side-Effects](#managing-side-effects)
-   [Testing](#testing)