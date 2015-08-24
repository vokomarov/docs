# Http Dispatcher Request flow
Spiral HTTP сomponent based on [PSR7](http://www.php-fig.org/psr/psr-7/) implenentation of http requests and responses, spiral utilizes [Zend implementation](https://github.com/zendframework/zend-diactoros) of such protocol as internal backbone. PSR7 makes dispatcher compatible with other frameworks,
middlewares and response generators.

## What is request
Spiral does not provide instance of "global" application request available in any place of application, instead of that it opens so called "request scope" and 
creates container binding `ServerRequestInterface` and "request" (for shortness) to access active request in controllers, request filters and services executed inside such scope. Once
request performed and response is generated scope are closed and no instance of global request available anymore.

Due http dispatcher can be created in any environment it is possible to start application or nested request with custom instance of ServerRequestInterface at any
moment. Use dispatcher method "perform" to execute given request.

Let's view example how to call http perform method in controller using altered or custom request.
```php
public function index() {
    //This is going to be instance of request previously passed into http dispatcher method, 
    //not nesessary initial request
    $request = $this->request;
    
    //Let's change request uri path
    $uri = $request->getUri()->withPath("/new-path"); 
    
    //We are emulating user request 
    return $this->http->perform($request->withUri($uri));
}
```

> You can pass any implementation of `ServerRequestInterface` into `perform` method.

## Middleware pipeline and endpoint (target)
Before request can be accessed inside controllers and other application services it will be passed thought set of [Http Middlewares](middlewares.md) which can apply custom logic to filter request/response or even halt execution if some condition met (see `CsrfFilter` middleware).

After every middeware processed incoming request, such request will be passed into so called "endpoint" which contains application logic, controllers and etc, endpoing should only be callable and accept `ServerRequestInterface` as first argument into `__invoke` method  (you can also use closures or custom methods (provided as array or class and method)). Request will be bindined (scope created) in container right before being passed into endpoint.

> If no custom endpoint provided, http dispatcher will use it's associated `Router` instance which will perform url routing to application controllers (actions).

Once endpoint (controller) generated response such responce will be converted into instance of `ResponseInterface` (for example if endpoint responded with string, `HtmlResponse` will be generated). Response again will be passed thought every associated middleware in reverse order (for example to add nesesary headers) and then returned from `HttpDispatcher->perform()` method or automatically dispatched to client. Most of listed operations beformed inside `MiddlewarePipeline` class.

### We can demonstrate request/response flow using well know image:
![middleware onion](http://stackphp.com/img/onion.png)

Where "Session" and "Authentication" treated as middlewares and "App" as endpoint.