#slim-jsonAPI
This is an extension to the [SLIM framework](https://github.com/codeguy/Slim) to implement json API's with great ease.

##instalation
Using composer you can add use this as your composer.json

```json
    {
        "require": {
            "slim/slim": "2.3.*",
            "entomb/slim-json-api": "dev-master"
        }
    }

```

##Usage
To include the middleware and view you just have to load them using the default _Slim_ way.

```php
    require 'vendor/autoload.php';

    $app = new \Slim\Slim();

    $app->view(new \JsonApiView());
    $app->add(new \JsonApiMiddleware());
```

###.htaccess sample
Here's an .htaccess sample, this will redirect everything except files and directories to your `index.php`
```
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteRule . /index.php [L]
</IfModule>
```

###index.php
now all your requests will be returning a JSON output.

the usage will be `$app->render( (int)$HTTP_CODE, (array)$DATA);`

####Code
```php

    $app->get('/', function() use ($app) {
        $app->render(200,array(
                'msg' => 'welcome to my API!',
            ));
    });

```


####output
```json
{
    "msg":"welcome to my API!",
    "error":false,
    "status":200
}

```

##Errors
To display an error just set the `error=>true` in your data array.
All requests have an `error` var and it default to false.
The HTTP code will also default to `200`

```php

    $app->get('/user/:id', function($id) use ($app) {

        //your code here

        $app->render(404,array(
                'error' => TRUE,
                'msg'   => 'user not found',
            ));
    });

```
```json
{
    "msg":"user not found",
    "error":true,
    "status":404
}

```

You can optionaly throw exeptions, the middleware will catch all exeptions and display error mensages.

```php

    $app->get('/user/:id', function($id) use ($app) {

        //your code here

        if(...){
            throw new Exception("Something wrong with your request!");
        }
    });

```
```json
{
    "error": true,
    "msg": "ERROR: Something wrong with your request!",
    "status": 500
}

```


##routing specific requests to the API
If your site is using regular HTML responses and you just want to expose an API point on a specific route,
you can use Slim router middlewares to define this.

```php
    function APIrequest(){
        $app = \Slim\Slim::getInstance();
        $app->view(new \JsonApiView());
        $app->add(new \JsonApiMiddleware());
    }


    $app->get('/home',function() use($app){
        //regular html response
        $app->render("template.tpl");
    });

    $app->get('/api','APIrequest',function() use($app){
        //this request will have full json responses

        $app->render(200,array(
                'msg' => 'welcome to my API!',
            ));
    });
```


##middleware
The middleware will set some static routes for default requests.
if you dont want to use it, you can copy this code into your bootstrap file.

***IMPORTANT: remember to use `$app->config('debug', false);` or errors will still be printed in HTML***

```php

    // Mirrors the API request
    $app->get('/return', function() use ($app) {
        $app->render(200,array(
            'method'    => $app->request()->getMethod(),
            'name'      => $app->request()->get('name'),
            'headers'   => $app->request()->headers(),
            'params'    => $app->request()->params(),
        ));
    });

    // Generic error handler
    $app->error(function (Exception $e) use ($app) {
        $app->render(500,array(
            'error' => TRUE,
            'msg'   => $e->getMessage(),
        ));
    });

    // Not found handler (invalid routes, invalid method types)
    $app->notFound(function() use ($app) {
        $app->render(400,array(
            'error' => TRUE,
            'msg'   => 'Invalid route',
        ));
    });

    // Handle Empty response body
    $app->hook('slim.after.router', function () use ($app) {
        if (strlen($app->response()->body()) == 0) {
            //Fix sugested by: https://github.com/bdpsoft
            //Will allow download request to flow
            if($app->response()->header('Content-Type')==='application/octet-stream'){
                return;
            }

            $app->render(500,array(
                'error' => TRUE,
                'msg'   => 'Empty response',
            ));
        }
    });


```
