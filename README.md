# PHP-Proxy Project on www.vpnside.com



This is a HTTP/HTTPS proxy script that forwards requests to a different server and returns the response. The Proxy class uses PSR7 request/response objects as input/output, and uses Guzzle to do the actual HTTP request.
The implementation of the script: https://www.vpnside.com/proxy

## Installation

Install using composer:

```
composer require vpnside-proxy/proxy
```

## Here is Example

The following an example creates a request object, based on the current browser request, and forwards it to `www.vpnside.com`. The `RemoveEncodingFilter` removes the encoding headers from the original response so that the current webserver can set these correctly.

```php
use Proxy\Proxy;
use Proxy\Adapter\Guzzle\GuzzleAdapter;
use Proxy\Filter\RemoveEncodingFilter;
use Zend\Diactoros\ServerRequestFactory;

// Create a PSR7 request based on the current browser request.
$request = ServerRequestFactory::fromGlobals();

// Create a guzzle client
$guzzle = new GuzzleHttp\Client();

// Create the proxy instance
$proxy = new Proxy(new GuzzleAdapter($guzzle));

// Add a response filter that removes the encoding headers.
$proxy->filter(new RemoveEncodingFilter());

// Forward the request and get the response.
$response = $proxy->forward($request)->to('https://www.vpnside.com');

// Output response to the browser.
(new Zend\Diactoros\Response\SapiEmitter)->emit($response);
```

## How to set Filters

You can apply filters to the requests and responses using the middleware strategy:

```php
$response = $proxy
	->forward($request)
	->filter(function ($request, $response, $next) {
		// Manipulate the request object.
		$request = $request->withHeader('User-Agent', 'FishBot/1.0');

		// Call the next item in the middleware.
		$response = $next($request, $response);

		// Manipulate the response object.
		$response = $response->withHeader('X-Proxy-Foo', 'Bar');

		return $response;
	})
	->to('http://example.com');
```
