---
title: CSRF Protection
description: Protect your application from cross-site request forgery attacks
url: https://laravel.com/docs/13.x/csrf
tags: [framework]
cssclasses:
  - framework
  - color-blue
color: blue
---

# CSRF Protection

-   [Introduction](#csrf-introduction)
-   [Preventing CSRF Requests](#preventing-csrf-requests)
    -   [Origin Verification](#origin-verification)
    -   [Excluding URIs](#csrf-excluding-uris)
-   [X-CSRF-Token](#csrf-x-csrf-token)
-   [X-XSRF-Token](#csrf-x-xsrf-token)

## [Introduction](#csrf-introduction)

Cross-site request forgeries are a type of malicious exploit whereby unauthorized commands are performed on behalf of an authenticated user. Thankfully, Laravel makes it easy to protect your application from [cross-site request forgery](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) attacks.

#### [An Explanation of the Vulnerability](#csrf-explanation)

In case you're not familiar with cross-site request forgeries, let's discuss an example of how this vulnerability can be exploited. Imagine your application has a `/user/email` route that accepts a `POST` request to change the authenticated user's email address. Most likely, this route expects an `email` input field to contain the email address the user would like to begin using.

Without CSRF protection, a malicious website could create an HTML form that points to your application's `/user/email` route and submits the malicious user's own email address:

```
1<form action="https://your-application.com/user/email" method="POST">2    <input type="email" value="[email protected]">3</form>4 5<script>6    document.forms[0].submit();7</script>
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="[email protected]">
</form>

<script>
    document.forms[0].submit();
</script>
```

If the malicious website automatically submits the form when the page is loaded, the malicious user only needs to lure an unsuspecting user of your application to visit their website and their email address will be changed in your application.

To prevent this vulnerability, we need to inspect every incoming `POST`, `PUT`, `PATCH`, or `DELETE` request for a secret session value that the malicious application is unable to access.

## [Preventing CSRF Requests](#preventing-csrf-requests)

The `Illuminate\Foundation\Http\Middleware\PreventRequestForgery` [[04-the-basics/02-middleware.md|middleware]], which is included in the `web` middleware group by default, protects your application from cross-site request forgeries using a two-layer approach.

First, the middleware checks the browser's `Sec-Fetch-Site` header. Modern browsers automatically set this header on every request, indicating whether it originated from the same origin, the same site, or a cross-site source. If the header indicates the request came from the same origin, the request is allowed immediately without any token verification.

If origin verification does not pass — for example, because the request comes from an older browser that doesn't send the `Sec-Fetch-Site` header or because the connection is not secure — the middleware falls back to traditional CSRF token validation.

Laravel automatically generates a CSRF "token" for each active [[04-the-basics/11-http-session.md|user session]] managed by the application. This token is used to verify that the authenticated user is the person actually making the requests to the application. Since this token is stored in the user's session and changes each time the session is regenerated, a malicious application is unable to access it.

The current session's CSRF token can be accessed via the request's session or via the `csrf_token` helper function:

```
1use Illuminate\Http\Request;2 3Route::get('/token', function (Request $request) {4    $token = $request->session()->token();5 6    $token = csrf_token();7 8    // ...9});
use Illuminate\Http\Request;

Route::get('/token', function (Request $request) {
    $token = $request->session()->token();

    $token = csrf_token();

    // ...
});
```

Anytime you define a "POST", "PUT", "PATCH", or "DELETE" HTML form in your application, you should include a hidden CSRF `_token` field in the form so that the CSRF protection middleware can validate the request. For convenience, you may use the `@csrf` Blade directive to generate the hidden token input field:

```
1<form method="POST" action="/profile">2    @csrf3 4    <!-- Equivalent to... -->5    <input type="hidden" name="_token" value="{{ csrf_token() }}" />6</form>
<form method="POST" action="/profile">
    @csrf

    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

#### [CSRF Tokens & SPAs](#csrf-tokens-and-spas)

If you are building an SPA that is utilizing Laravel as an API backend, you should consult the [Laravel Sanctum documentation](/docs/13.x/sanctum) for information on authenticating with your API and protecting against CSRF vulnerabilities.

### [Origin Verification](#origin-verification)

As discussed above, Laravel's request forgery middleware first checks the `Sec-Fetch-Site` header to determine if the request is from the same origin. By default, if this check does not pass, the middleware falls back to CSRF token validation.

However, if you would like to rely solely on origin verification and disable the CSRF token fallback entirely, you may do so using the `preventRequestForgery` method in your application's `bootstrap/app.php` file:

```
1->withMiddleware(function (Middleware $middleware): void {2    $middleware->preventRequestForgery(originOnly: true);3})
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(originOnly: true);
})
```

When using origin-only mode, requests that fail origin verification will receive a `403` HTTP response instead of the `419` response typically associated with CSRF token mismatches.

The `Sec-Fetch-Site` header is only sent by browsers over secure (HTTPS) connections. If your application is not served over HTTPS, origin verification will not be available and the middleware will fall back to CSRF token validation.

If your application needs to accept requests from subdomains (for example, `dashboard.example.com` accepting requests from `example.com`), you may allow same-site requests in addition to same-origin requests:

```
1->withMiddleware(function (Middleware $middleware): void {2    $middleware->preventRequestForgery(allowSameSite: true);3})
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(allowSameSite: true);
})
```

### [Excluding URIs From CSRF Protection](#csrf-excluding-uris)

Sometimes you may wish to exclude a set of URIs from CSRF protection. For example, if you are using [Stripe](https://stripe.com) to process payments and are utilizing their webhook system, you will need to exclude your Stripe webhook handler route from CSRF protection since Stripe will not know what CSRF token to send to your routes.

Typically, you should place these kinds of routes outside of the `web` middleware group that Laravel applies to all routes in the `routes/web.php` file. However, you may also exclude specific routes by providing their URIs to the `preventRequestForgery` method in your application's `bootstrap/app.php` file:

```
1->withMiddleware(function (Middleware $middleware): void {2    $middleware->preventRequestForgery(except: [3        'stripe/*',4        'http://example.com/foo/bar',5        'http://example.com/foo/*',6    ]);7})
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(except: [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ]);
})
```

For convenience, the CSRF middleware is automatically disabled for all routes when [[10-testing/01-testing-getting-started.md|running tests]].

## [X-CSRF-TOKEN](#csrf-x-csrf-token)

In addition to checking for the CSRF token as a POST parameter, the `PreventRequestForgery` middleware will also check for the `X-CSRF-TOKEN` request header. You could, for example, store the token in an HTML `meta` tag:

```
1<meta name="csrf-token" content="{{ csrf_token() }}">
<meta name="csrf-token" content="{{ csrf_token() }}">
```

Then, you can instruct a library like jQuery to automatically add the token to all request headers. This provides simple, convenient CSRF protection for your AJAX based applications using legacy JavaScript technology:

```
1$.ajaxSetup({2    headers: {3        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')4    }5});
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

## [X-XSRF-TOKEN](#csrf-x-xsrf-token)

Laravel stores the current CSRF token in an encrypted `XSRF-TOKEN` cookie that is included with each response generated by the framework. You can use the cookie value to set the `X-XSRF-TOKEN` request header.

This cookie is primarily sent as a developer convenience since some JavaScript frameworks and libraries, like Angular and Axios, automatically place its value in the `X-XSRF-TOKEN` header on same-origin requests.

By default, the `resources/js/bootstrap.js` file includes the Axios HTTP library which will automatically send the `X-XSRF-TOKEN` header for you.