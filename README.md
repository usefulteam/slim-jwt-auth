# PSR-7 and PSR-15 JWT Authentication Middleware

> This was a fork of [tuupola/slim-jwt-auth](https://github.com/tuupola/slim-jwt-auth) by [Mika Tuupola](https://github.com/tuupola). The fork was taken from `3.x` branch (at [this state](https://github.com/tuupola/slim-jwt-auth/tree/a4d6b3857daccb393f885473a08b2ea25874ae6b)).
> Thanks to Mika Tuupola & the package's contributors for their hard work.
> We forked it because we wanted to use [firebase/php-jwt](https://github.com/firebase/php-jwt) version 6 which had not supported at that time. Related [issue](https://github.com/tuupola/slim-jwt-auth/issues/217).

This middleware implements JSON Web Token Authentication. It was originally developed for Slim but can be used with any framework using PSR-7 and PSR-15 style middlewares. It has been tested with [Slim Framework](http://www.slimframework.com/) and [Zend Expressive](https://zendframework.github.io/zend-expressive/).

[![Latest Version](https://img.shields.io/packagist/v/usefulteam/slim-jwt-auth.svg?style=flat-square)](https://packagist.org/packages/usefulteam/slim-jwt-auth)
[![Packagist](https://img.shields.io/packagist/dm/usefulteam/slim-jwt-auth.svg)](https://packagist.org/packages/usefulteam/slim-jwt-auth)
[![Software License](https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square)](LICENSE)

Middleware does **not** implement OAuth 2.0 authorization server nor does it provide ways to generate, issue or store authentication tokens. It only parses and authenticates a token when passed via header or cookie. This is useful for example when you want to use [JSON Web Tokens as API keys](https://auth0.com/blog/2014/12/02/using-json-web-tokens-as-api-keys/).

For example implementation see [Slim API Skeleton](https://github.com/usefulteam/slim-api-skeleton).

## Install

Install latest version using [composer](https://getcomposer.org/).

``` bash
$ composer require usefulteam/slim-jwt-auth
```

If using Apache add the following to the `.htaccess` file. Otherwise PHP wont have access to `Authorization: Bearer` header.

``` bash
RewriteRule .* - [env=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

## Usage

Configuration options are passed as an array. The only mandatory parameter is `secret` which is used for verifying then token signature. Note again that `secret` is not the token. It is the secret you use to sign the token.

For simplicity's sake examples show `secret` hardcoded in code. In real life you should store it somewhere else. Good option is environment variable. You can use [dotenv](https://github.com/vlucas/phpdotenv) or something similar for development. Examples assume you are using Slim Framework.

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "secret" => "supersecretkeyyoushouldnotcommittogithub"
]));
```

An example where your secret is stored as an environment variable:

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "secret" => getenv("JWT_SECRET")
]));
```

When a request is made, the middleware tries to validate and decode the token. If a token is not found or there is an error when validating and decoding it, the server will respond with `401 Unauthorized`.

Validation errors are triggered when the token has been tampered with or the token has expired. For all possible validation errors, see [JWT library](https://github.com/firebase/php-jwt/blob/master/src/JWT.php#L60-L138) source.


## Optional parameters
### Path

The optional `path` parameter allows you to specify the protected part of your website. It can be either a string or an array. You do not need to specify each URL. Instead think of `path` setting as a folder. In the example below everything starting with `/api` will be authenticated. If you do not define `path` all routes will be protected.

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "path" => "/api", /* or ["/api", "/admin"] */
    "secret" => "supersecretkeyyoushouldnotcommittogithub"
]));
```

### Ignore

With optional `ignore` parameter you can make exceptions to `path` parameter. In the example below everything starting with `/api` and `/admin`  will be authenticated with the exception of `/api/token` and `/admin/ping` which will not be authenticated.

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "path" => ["/api", "/admin"],
    "ignore" => ["/api/token", "/admin/ping"],
    "secret" => "supersecretkeyyoushouldnotcommittogithub"
]));
```

### Header

By default middleware tries to find the token from `Authorization` header. You can change header name using the `header` parameter.

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "header" => "X-Token",
    "secret" => "supersecretkeyyoushouldnotcommittogithub"
]));
```

### Regexp

By default the middleware assumes the value of the header is in `Bearer <token>` format. You can change this behaviour with `regexp` parameter. For example if you have custom header such as `X-Token: <token>` you should pass both header and regexp parameters.

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "header" => "X-Token",
    "regexp" => "/(.*)/",
    "secret" => "supersecretkeyyoushouldnotcommittogithub"
]));
```

### Cookie

If token is not found from neither environment or header, the middleware tries to find it from cookie named `token`. You can change cookie name using `cookie` parameter.

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "cookie" => "nekot",
    "secret" => "supersecretkeyyoushouldnotcommittogithub"
]));
```

### Algorithm

You can set supported algorithms via `algorithm` parameter. This can be either string or array of strings. Default value is `["HS256", "HS512", "HS384"]`. Supported algorithms are `HS256`, `HS384`, `HS512` and `RS256`. Note that enabling both `HS256` and `RS256` is a [security risk](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/).

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "secret" => "supersecretkeyyoushouldnotcommittogithub",
    "algorithm" => ["HS256", "HS384"]
]));
```

### Attribute

When the token is decoded successfully and authentication succeeds the contents of the decoded token is saved as `token` attribute to the `$request` object. You can change this with. `attribute` parameter. Set to `null` or `false` to disable this behavour

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "attribute" => "jwt",
    "secret" => "supersecretkeyyoushouldnotcommittogithub"
]));

/* ... */

$decoded = $request->getAttribute("jwt");
```

### Logger

The optional `logger` parameter allows you to pass in a PSR-3 compatible logger to help with debugging or other application logging needs.

``` php
use Monolog\Logger;
use Monolog\Handler\RotatingFileHandler;

$app = new Slim\App;

$logger = new Logger("slim");
$rotating = new RotatingFileHandler(__DIR__ . "/logs/slim.log", 0, Logger::DEBUG);
$logger->pushHandler($rotating);

$app->add(new Usefulteam\Middleware\JwtAuth([
    "path" => "/api",
    "logger" => $logger,
    "secret" => "supersecretkeyyoushouldnotcommittogithub"
]));
```

### Before

Before function is called only when authentication succeeds but before the next incoming middleware is called. You can use this to alter the request before passing it to the next incoming middleware in the stack. If it returns anything else than `Psr\Http\Message\ServerRequestInterface` the return value will be ignored.

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "secret" => "supersecretkeyyoushouldnotcommittogithub",
    "before" => function ($request, $arguments) {
        return $request->withAttribute("test", "test");
    }
]));
```

### After

After function is called only when authentication succeeds and after the incoming middleware stack has been called. You can use this to alter the response before passing it next outgoing middleware in the stack. If it returns anything else than `Psr\Http\Message\ResponseInterface` the return value will be ignored.


``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "secret" => "supersecretkeyyoushouldnotcommittogithub",
    "after" => function ($response, $arguments) {
        return $response->withHeader("X-Brawndo", "plants crave");
    }
]));
```

> Note that both the after and before callback functions receive the raw token string as well as the decoded claims through the `$arguments` argument.

### Error

Error is called when authentication fails. It receives last error message in arguments. You can use this for example to return JSON formatted error responses.

```php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "secret" => "supersecretkeyyoushouldnotcommittogithub",
    "error" => function ($response, $arguments) {
        $data["status"] = "error";
        $data["message"] = $arguments["message"];

        $response->getBody()->write(
            json_encode($data, JSON_UNESCAPED_SLASHES | JSON_PRETTY_PRINT)
        );

        return $response->withHeader("Content-Type", "application/json")
    }
]));
```

### Rules

The optional `rules` parameter allows you to pass in rules which define whether the request should be authenticated or not. A rule is a callable which receives the request as parameter. If any of the rules returns boolean `false` the request will not be authenticated.

By default middleware configuration looks like this. All paths are authenticated with all request methods except `OPTIONS`.

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "rules" => [
        new Usefulteam\Middleware\JwtAuth\RequestPathRule([
            "path" => "/",
            "ignore" => []
        ]),
        new Usefulteam\Middleware\JwtAuth\RequestMethodRule([
            "ignore" => ["OPTIONS"]
        ])
    ]
]));
```

RequestPathRule contains both a `path` parameter and a `ignore` parameter. Latter contains paths which should not be authenticated. RequestMethodRule contains `ignore` parameter of request methods which also should not be authenticated. Think of `ignore` as a whitelist.

99% of the cases you do not need to use the `rules` parameter. It is only provided for special cases when defaults do not suffice.

## Security

JSON Web Tokens are essentially passwords. You should treat them as such and you should always use HTTPS. If the middleware detects insecure usage over HTTP it will throw a `RuntimeException`. This rule is relaxed for requests on localhost. To allow insecure usage you must enable it manually by setting `secure` to `false`.

``` php
$app->add(new Usefulteam\Middleware\JwtAuth([
    "secure" => false,
    "secret" => "supersecretkeyyoushouldnotcommittogithub"
]));
```

Alternatively you can list your development host to have relaxed security.

``` php
$app->add(new Usefulteam\Middleware\JwtAuth([
    "secure" => true,
    "relaxed" => ["localhost", "dev.example.com"],
    "secret" => "supersecretkeyyoushouldnotcommittogithub"
]));
```

## Authorization

By default middleware only authenticates. This is not very interesting. Beauty of JWT is you can pass extra data in the token. This data can include for example scope which can be used for authorization.

**It is up to you to implement how token data is stored or possible authorization implemented.**

Let assume you have token which includes data for scope. By default middleware saves the contents of the token to `token` attribute of the request.

``` php
[
    "iat" => "1428819941",
    "exp" => "1744352741",
    "scope" => ["read", "write", "delete"]
]
```

``` php
$app = new Slim\App;

$app->add(new Usefulteam\Middleware\JwtAuth([
    "secret" => "supersecretkeyyoushouldnotcommittogithub"
]));

$app->delete("/item/{id}", function ($request, $response, $arguments) {
    $token = $request->getAttribute("token");
    if (in_array("delete", $token["scope"])) {
        /* Code for deleting item */
    } else {
        /* No scope so respond with 401 Unauthorized */
        return $response->withStatus(401);
    }
});
```

## Testing

You can run tests either manually or automatically on every code change. Automatic tests require [entr](http://entrproject.org/) to work.

``` bash
$ make test
```

``` bash
$ brew install entr
$ make watch
```

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Security

If you discover any security related issues, please email bagus@usefulteam.com instead of using the issue tracker.

## License
- [tuupola/slim-jwt-auth](https://github.com/tuupola/slim-jwt-auth/) `3.x` branch license: The [MIT License](https://github.com/tuupola/slim-jwt-auth/blob/3.x/LICENSE)
- [usefulteam/slim-jwt-auth](https://github.com/usefulteam/slim-jwt-auth/) license: The [MIT License](https://oss.ninja/mit?organization=Useful%20Team&project=Slim%20Jwt%20Auth)
