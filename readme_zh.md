# Flight 是什么?

Flight 是快速的，简洁的可扩展的PHP开发框架。 Flight 使您能够快速轻松地构建REST风格的Web应用程序。

```php
require 'flight/Flight.php';

Flight::route('/', function(){
    echo 'hello world!';
});

Flight::start();
```

[Learn more](http://flightphp.com/learn)

# 必要条件

Flight 需要 `PHP 5.3` 及以上版本.

# 许可

Flight 按照 [MIT](http://flightphp.com/license) 许可发布。

# 安装

1\. [下载](https://github.com/mikecao/flight/tarball/master) 并解压
Flight framework 文件到你的web目录。

2\. 配置你的web服务器。

如果是 *Apache*, 对照下面编辑 `.htaccess` 文件:

```
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

如果是 *Nginx*, 把下面添加到你的服务器声明:

```
server {
    location / {
        try_files $uri $uri/ /index.php;
    }
}
```
3\. 建立 `index.php` 文件。

先要把框架include进来。

```php
require 'flight/Flight.php';
```

然后，定义一个路由，并指定一个函数来处理请求。

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

最后，启动框架。

```php
Flight::start();
```

# Routing 路由

Flight中路由是通过匹配RUL模式，调用回调函数进行的。

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

回调可以是任何可以调用的对象。所以，你也可以采用先定义再注册的方式：

```php
function hello(){
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

或者，定义类方法，再注册调用:

```php
class Greeting {
    public static function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', array('Greeting','hello'));
```

路由按他们定义的顺序进行匹配。第一个匹配的路由将被调用。

## Method Routing 请求方式的路由

默认情况下，路由模式匹配所有请求方式。
你可以回应具体的一个请求方式，只要在URL之前放置一个标识符。

```php
Flight::route('GET /', function(){
    echo 'I received a GET request.';
});

Flight::route('POST /', function(){
    echo 'I received a POST request.';
});
```

您也可以一个回调对应多个请求方式，使用'|'分隔符：

```php
Flight::route('GET|POST /', function(){
    echo 'I received either a GET or a POST request.';
});
```

## Regular Expressions 正则表达式

你可以在路由中使用正则表达式：

```php
Flight::route('/user/[0-9]+', function(){
    // This will match /user/1234
});
```

## Named Parameters 命名参数

你可以指定路由中的命名参数，参数将被传递
您的回调函数。

```php
Flight::route('/@name/@id', function($name, $id){
    echo "hello, $name ($id)!";
});
```

你也可以在命名参数中使用正则表达式，通过`:`分割：

```php
Flight::route('/@name/@id:[0-9]{3}', function($name, $id){
    // This will match /bob/123
    // But will not match /bob/12345
});
```

## Optional Parameters 可选参数

你可以用括号来指定哪些命名参数是可选的。

```php
Flight::route('/blog(/@year(/@month(/@day)))', function($year, $month, $day){
    // This will match the following URLS:
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
});
```

任何未匹配的可选参数都将传递NULL过去。

## Wildcards 通配符

匹配只是对应了URL部分情况。
如果你想匹配更多，可以使用`*`通配符。

```php
Flight::route('/blog/*', function(){
    // This will match /blog/2000/02/01
});
```

所有请求路由到一个回调函数，你可以这样做：

```php
Flight::route('*', function(){
    // Do something
});
```

## Passing 传递

您可以通过从返回`true`,从而传递到下一个匹配的路由执行
您的回调函数。

```php
Flight::route('/user/@name', function($name){
    // Check some condition
    if ($name != "Bob") {
        // Continue to next route
        return true;
    }
});

Flight::route('/user/*', function(){
    // This will get called
});
```

## Route Info 路由信息

您匹配的回调将被传递路由的对象，你可以用它来检查
路由信息。

```php
Flight::route('/', function($route){
    // 匹配的 HTTP methods（请求方式) 数组
    $route->methods;

    // Array of named parameters 命名参数数组
    $route->params;

    // Matching regular expression 匹配正则表达式
    $route->regex;

    // Contains the contents of any '*' used in the URL pattern
    // 包含'*'在URL模式中使用的所有内容
    $route->splat;
});
```

# Extending 扩展

Flight 被设计成可扩展的框架。框架提供了一组默认的方法和组件，
但是框架也允许你map（映射）你自己的方法，
注册您自己的类，甚至重写现有的类和方法。

## Mapping Methods 映射方法

映射自定义的方法，您可以使用`map`函数：

```php
// Map your method 映射你的方法
Flight::map('hello', function($name){
    echo "hello $name!";
});

// Call your custom method 调用自定义的方法
Flight::hello('Bob');
```

## Registering Classes 注册类

注册自定义的类，你可以使用`register`函数：

```php
// Register your class 注册自定义类
Flight::register('user', 'User');

// Get an instance of your class 得到一个自定义类的实例
$user = Flight::user();
```

该注册方法还允许将参数传递给类的构造函数。
所以，加载你的自定义类，它会预先初始化。
可以通过传递一个数组来定义构造函数的参数。
下面是加载数据库连接的例子：


```php
// Register class with constructor parameters 注册带有构造函数参数的类
Flight::register('db', 'PDO', array('mysql:host=localhost;dbname=test','user','pass'));

// Get an instance of your class 取得自定义类的一个实例
// This will create an object with the defined parameters 这将按参数创建一个对象
//
//     new PDO('mysql:host=localhost;dbname=test','user','pass');
//
$db = Flight::db();
```

如果你传递一个回调函数参数，它将会在类构造完后立即执行。
这可以让你在程序中设置刚刚构造好的对象。
回调函数接受一个参数，即新对象的实例。

```php
// The callback will be passed the object that was constructed 回调将被构造的对象作为参数
Flight::register('db', 'PDO', array('mysql:host=localhost;dbname=test','user','pass'), function($db){
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
});
```

默认情况下，每次加载类时，你会得到一个共享实例。
要得到一个类的新的实例，只需传递`false`参数：

```php
// Shared instance of the class 共享实例（单一实例）
$shared = Flight::db();

// New instance of the class 新的实例
$new = Flight::db(false);
```

记住，map映射的方法优先于register注册类。
如果您使用相同的名称声明两个，只有映射的方法将被调用。

# Overriding 重写

Flight 可以让你重写其默认的功能，以满足自己的需要，
无需修改任何代码。

例如，当Flight不能从URL匹配到路由, 他会调用 `notFound`
方法，发送一个 `HTTP 404` 应答。
你可以用 `map` 方法重写此行为:

```php
Flight::map('notFound', function(){
    // Display custom 404 page 显示自定义的404页面
    include 'errors/404.html';
});
```

Flight 还允许你替换框架的核心组件。
例如，你可以用你自己的自定义类替换默认的路由器类：

```php
// Register your custom class 注册你的自定义类
Flight::register('router', 'MyRouter');

// When Flight loads the Router instance, it will load your class
// 当Flight加载路由实例，它会加载您的类
$myrouter = Flight::router();
```

框架的 `map` 和 `register` 不可以重写。
如果你试图重写，会出现错误。

# Filtering 过滤

Flight允许你在调用方法前、后进行过滤。你不需要记忆预先定义的hooks。
您可以过滤框架的所有方法，包括你已经映射的方法以及任何自定义的方法。


过滤器函数看起来像这样:

```php
function(&$params, &$output) {
    // Filter code 过滤器处理的代码
}
```

使用传入的变量，你可以操纵的输入、输出参数。

你可以运行一个过滤器在方法调用之前：

```php
Flight::before('start', function(&$params, &$output){
    // Do something
});
```

你可以运行一个过滤器在方法调用之后：

```php
Flight::after('start', function(&$params, &$output){
    // Do something
});
```

只要你想你可以添加任意多的过滤器。
他们将按照被定义的顺序进行调用。

下面是个的过滤处理的例子：

```php
// Map a custom method 映射自定义方法
Flight::map('hello', function($name){
    return "Hello, $name!";
});

// Add a before filter 添加一个前置过滤器
Flight::before('hello', function(&$params, &$output){
    // Manipulate the parameter 操作参数
    $params[0] = 'Fred';
});

// Add an after filter 添加一个后置过滤器
Flight::after('hello', function(&$params, &$output){
    // Manipulate the output 控制输出
    $output .= " Have a nice day!";
});

// Invoke the custom method 调用自定义的方法
echo Flight::hello('Bob');
```

应该得到如下显示:

    Hello Fred! Have a nice day!

如果你定义了多个过滤器，
你可以返回`false`来打断过滤器的链式调用：

```php
Flight::before('start', function(&$params, &$output){
    echo 'one';
});

Flight::before('start', function(&$params, &$output){
    echo 'two';

    // This will end the chain 这里将结束调用
    return false;
});

// This will not get called 这里将不会被调用
Flight::before('start', function(&$params, &$output){
    echo 'three';
});
```

注意：核心方法`map` 、 `register`不能被过滤，因为他们不是动态调用的。

# Variables 变量

Flight可以让您保存变量，让他们可以在应用程序任何地方使用。

```php
// 保存你的变量
Flight::set('id', 123);

// 你应用程序的其他地方
$id = Flight::get('id');
```
要知道一个变量是否已经设置你可以这样做：

```php
if (Flight::has('id')) {
     // Do something
}
```

你可以清除一个变量:

```php
// 清除变量id
Flight::clear('id');

// 清除所有变量
Flight::clear();
```

Flight也使用变量进行配置。

```php
Flight::set('flight.log_errors', true);
```

# Views 视图

Flight 默认情况下提供一些基本的模板功能。 
要显示一个视图模板调用`render`方法，传递模板文件参数和可选的模板数据的名称：

```php
Flight::render('hello.php', array('name' => 'Bob'));
```

传递的模板数据被自动插入到模板类似于模板自己的变量。模板文件就是简单的PHP文件。  
如果`hello.php` 模板的内容如下:

```php
Hello, '<?php echo $name; ?>'!
```

输出将是：

    Hello, Bob!

您也可以手动使用set方法来设置视图的变量：

```php
Flight::view()->set('name', 'Bob');
```

变量 `name` 现在就在你的视图里，你就可以直接渲染模板:

```php
Flight::render('hello');
```

请注意，在render方法中指定模板的名称时，可以忽略`.php`的扩展名。

Flight默认会在`views`目录下查找模板文件。 
您可以通过设置，配置你的模板的路径：

```php
Flight::set('flight.views.path', '/path/to/views');
```

## Layouts 布局

一般网站都有一个提供内容替换的布局模板文件。
呈现在布局中的内容, 你可以传递一个可选参数给`render`方法。

```php
Flight::render('header', array('heading' => 'Hello'), 'header_content');
Flight::render('body', array('body' => 'World'), 'body_content');
```

现在，您的两个已经渲染的视图将保存在两个变量 `header_content` 和 `body_content`中。
然后，渲染你的布局视图：

```php
Flight::render('layout', array('title' => 'Home Page'));
```

如果模板文件看起来像这样：

`header.php`:

```php
<h1><?php echo $heading; ?></h1>
```

`body.php`:

```php
<div><?php echo $body; ?></div>
```

`layout.php`:

```php
<html>
<head>
<title><?php echo $title; ?></title>
</head>
<body>
<?php echo $header_content; ?>
<?php echo $body_content; ?>
</body>
</html>
```

输出如下：
```html
<html>
<head>
<title>Home Page</title>
</head>
<body>
<h1>Hello</h1>
<div>World</div>
</body>
</html>
```

## Custom Views 自定义视图

Flight允许你用自己的视图类替换默认的视图。
参考这里使用[Smarty](http://www.smarty.net/)模板引擎作为你的视图:

```php
// 加载Smarty library
require './Smarty/libs/Smarty.class.php';

// 注册Smarty成为视图类
// 传递一个回调函数，用于对加载Smarty的配置
Flight::register('view', 'Smarty', array(), function($smarty){
    $smarty->template_dir = './templates/';
    $smarty->compile_dir = './templates_c/';
    $smarty->config_dir = './config/';
    $smarty->cache_dir = './cache/';
});

// 指定模板变量数据
Flight::view()->assign('name', 'Bob');

// 显示模板
Flight::view()->display('hello.tpl');
```

为了完整起见，你应该重写Flight的默认渲染方法：

```php
Flight::map('render', function($template, $data){
    Flight::view()->assign($data);
    Flight::view()->display($template);
});
```
# Error Handling 错误处理

## Errors and Exceptions 错误和异常

All errors and exceptions are caught by Flight and passed to the `error` method.
The default behavior is to send a generic `HTTP 500 Internal Server Error`
response with some error information.

You can override this behavior for your own needs:

```php
Flight::map('error', function(Exception $ex){
    // Handle error
    echo $ex->getTraceAsString();
});
```

By default errors are not logged to the web server. You can enable this by
changing the config:

```php
Flight::set('flight.log_errors', true);
```

## Not Found

When a URL can't be found, Flight calls the `notFound` method. The default
behavior is to send an `HTTP 404 Not Found` response with a simple message.

You can override this behavior for your own needs:

```php
Flight::map('notFound', function(){
    // Handle not found
});
```

# Redirects

You can redirect the current request by using the `redirect` method and passing
in a new URL:

```php
Flight::redirect('/new/location');
```

By default Flight sends a HTTP 303 status code. You can optionally set a
custom code:

```php
Flight::redirect('/new/location', 401);
```

# Requests

Flight encapsulates the HTTP request into a single object, which can be
accessed by doing:

```php
$request = Flight::request();
```

The request object provides the following properties:

```
url - The URL being requested
base - The parent subdirectory of the URL
method - The request method (GET, POST, PUT, DELETE)
referrer - The referrer URL
ip - IP address of the client
ajax - Whether the request is an AJAX request
scheme - The server protocol (http, https)
user_agent - Browser information
body - Raw data from the request body
type - The content type
length - The content length
query - Query string parameters
data - Post parameters
cookies - Cookie parameters
files - Uploaded files
secure - Whether the connection is secure
accept - HTTP accept parameters
proxy_ip - Proxy IP address of the client
```

You can access the `query`, `data`, `cookies`, and `files` properties
as arrays or objects.

So, to get a query string parameter, you can do:

```php
$id = Flight::request()->query['id'];
```

Or you can do:

```php
$id = Flight::request()->query->id;
```

# HTTP Caching

Flight provides built-in support for HTTP level caching. If the caching condition
is met, Flight will return an HTTP `304 Not Modified` response. The next time the
client requests the same resource, they will be prompted to use their locally
cached version.

## Last-Modified

You can use the `lastModified` method and pass in a UNIX timestamp to set the date
and time a page was last modified. The client will continue to use their cache until
the last modified value is changed.

```php
Flight::route('/news', function(){
    Flight::lastModified(1234567890);
    echo 'This content will be cached.';
});
```

## ETag

`ETag` caching is similar to `Last-Modified`, except you can specify any id you
want for the resource:

```php
Flight::route('/news', function(){
    Flight::etag('my-unique-id');
    echo 'This content will be cached.';
});
```

Keep in mind that calling either `lastModified` or `etag` will both set and check the
cache value. If the cache value is the same between requests, Flight will immediately
send an `HTTP 304` response and stop processing.

# Stopping

You can stop the framework at any point by calling the `halt` method:

```php
Flight::halt();
```

You can also specify an optional `HTTP` status code and message:

```php
Flight::halt(200, 'Be right back...');
```

Calling `halt` will discard any response content up to that point. If you want to stop
the framework and output the current response, use the `stop` method:

```php
Flight::stop();
```

# JSON

Flight provides support for sending JSON and JSONP responses. To send a JSON response you
pass some data to be JSON encoded:

```php
Flight::json(array('id' => 123));
```

For JSONP requests you, can optionally pass in the query parameter name you are
using to define your callback function:

```php
Flight::jsonp(array('id' => 123), 'q');
```

So, when making a GET request using `?q=my_func`, you should receive the output:

```
my_func({"id":123});
```

If you don't pass in a query parameter name it will default to `jsonp`.


# Configuration

You can customize certain behaviors of Flight by setting configuration values
through the `set` method.

```php
Flight::set('flight.log_errors', true);
```

The following is a list of all the available configuration settings:

    flight.base_url - Override the base url of the request. (default: null)
    flight.handle_errors - Allow Flight to handle all errors internally. (default: true)
    flight.log_errors - Log errors to the web server's error log file. (default: false)
    flight.views.path - Directory containing view template files. (default: ./views)

# Framework Methods

Flight is designed to be easy to use and understand. The following is the complete
set of methods for the framework. It consists of core methods, which are regular
static methods, and extensible methods, which are mapped methods that can be filtered
or overridden.

## Core Methods

```php
Flight::map($name, $callback) // Creates a custom framework method.
Flight::register($name, $class, [$params], [$callback]) // Registers a class to a framework method.
Flight::before($name, $callback) // Adds a filter before a framework method.
Flight::after($name, $callback) // Adds a filter after a framework method.
Flight::path($path) // Adds a path for autoloading classes.
Flight::get($key) // Gets a variable.
Flight::set($key, $value) // Sets a variable.
Flight::has($key) // Checks if a variable is set.
Flight::clear([$key]) // Clears a variable.
Flight::init() // Initializes the framework to its default settings.
```

## Extensible Methods

```php
Flight::start() // Starts the framework.
Flight::stop() // Stops the framework and sends a response.
Flight::halt([$code], [$message]) // Stop the framework with an optional status code and message.
Flight::route($pattern, $callback) // Maps a URL pattern to a callback.
Flight::redirect($url, [$code]) // Redirects to another URL.
Flight::render($file, [$data], [$key]) // Renders a template file.
Flight::error($exception) // Sends an HTTP 500 response.
Flight::notFound() // Sends an HTTP 404 response.
Flight::etag($id, [$type]) // Performs ETag HTTP caching.
Flight::lastModified($time) // Performs last modified HTTP caching.
Flight::json($data, [$code], [$encode]) // Sends a JSON response.
Flight::jsonp($data, [$param], [$code], [$encode]) // Sends a JSONP response.
```

Any custom methods added with `map` and `register` can also be filtered.


# Framework Instance

Instead of running Flight as a global static class, you can optionally run it
as an object instance.

```php
require 'flight/autoload.php';

use flight\Engine;

$app = new Engine();

$app->route('/', function(){
    echo 'hello world!';
});

$app->start();
```

So instead of calling the static method, you would call the instance method with
the same name on the Engine object.
