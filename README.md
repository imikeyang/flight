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
    // 这将匹配下列URL:
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
    // 检查一些条件
    if ($name != "Bob") {
        // 继续下一个路由
        return true;
    }
});

Flight::route('/user/*', function(){
    // 这里会被调用
});
```

## Route Info 路由信息

您匹配的回调将被传递路由的对象，你可以用它来检查
路由信息。

```php
Flight::route('/', function($route){
    // 匹配的 HTTP methods（请求方式) 数组
    $route->methods;

    // 命名参数数组
    $route->params;

    // 匹配正则表达式
    $route->regex;

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
// 映射你的方法
Flight::map('hello', function($name){
    echo "hello $name!";
});

// 调用自定义的方法
Flight::hello('Bob');
```

## Registering Classes 注册类

注册自定义的类，你可以使用`register`函数：

```php
// 注册自定义类
Flight::register('user', 'User');

// 得到一个自定义类的实例
$user = Flight::user();
```

该注册方法还允许将参数传递给类的构造函数。
所以，加载你的自定义类，它会预先初始化。
可以通过传递一个数组来定义构造函数的参数。
下面是加载数据库连接的例子：


```php
// 注册带有构造函数参数的类
Flight::register('db', 'PDO', array('mysql:host=localhost;dbname=test','user','pass'));

// 取得自定义类的一个实例
// 这将按参数创建一个对象
//
//     new PDO('mysql:host=localhost;dbname=test','user','pass');
//
$db = Flight::db();
```

如果你传递一个回调函数参数，它将会在类构造完后立即执行。
这可以让你在程序中设置刚刚构造好的对象。
回调函数接受一个参数，即新对象的实例。

```php
// 回调将被构造的对象作为参数
Flight::register('db', 'PDO', array('mysql:host=localhost;dbname=test','user','pass'), function($db){
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
});
```

默认情况下，每次加载类时，你会得到一个共享实例。
要得到一个类的新的实例，只需传递`false`参数：

```php
// 共享实例（单一实例）
$shared = Flight::db();

// 新的实例
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
    // 显示自定义的404页面
    include 'errors/404.html';
});
```

Flight 还允许你替换框架的核心组件。
例如，你可以用你自己的自定义类替换默认的路由器类：

```php
// 注册你的自定义类
Flight::register('router', 'MyRouter');

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
    // 过滤器处理的代码
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
// 映射自定义方法
Flight::map('hello', function($name){
    return "Hello, $name!";
});

// Add a before filter 添加一个前置过滤器
Flight::before('hello', function(&$params, &$output){
    // 操作参数
    $params[0] = 'Fred';
});

// Add an after filter 添加一个后置过滤器
Flight::after('hello', function(&$params, &$output){
    // 控制输出
    $output .= " Have a nice day!";
});

// 调用自定义的方法
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

    // 这里将结束调用
    return false;
});

// 这里将不会被调用
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

所有的错误和异常捕获Flight都传递给`error`方法。
默认发送 `HTTP 500 Internal Server Error` 应答以及一些错误信息。

你可以按自己的需求重写此行为：

```php
Flight::map('error', function(Exception $ex){
    // 处理错误
    echo $ex->getTraceAsString();
});
```

默认错误没有记录到服务器。
如果想记录错误日志，你可以按以下方式修改:

```php
Flight::set('flight.log_errors', true);
```

## Not Found 未找到页面

如果页面未找到，Flight调用`notFound`方法。
默认会发送 `HTTP 404 Not Found` 应答和一些简单信息。

你可以按自己的需求重写此行为：

```php
Flight::map('notFound', function(){
    // 处理未找到的错误
});
```

# Redirects 重定向

你可以通过`redirect`方法传递一个新的URL来重定向当前请求：

```php
Flight::redirect('/new/location');
```

Flight 默认发送 HTTP 303 status code。 
您也可以自己设置发送的HTTP status code：

```php
Flight::redirect('/new/location', 401);
```

# Requests 请求

Flight封装HTTP请求到一个单一的对象，
通过以下方式访问它：

```php
$request = Flight::request();
```

请求（request）对象提供了以下属性：

```
url - 被请求的URL
base - The parent subdirectory of the URL该URL的父子目录
method - 请求方式(GET, POST, PUT, DELETE)
referrer - 来源URL
ip - 客户端的ip地址
ajax - 该请求是否是一个AJAX请求
scheme - 服务器协议 (http, https)
user_agent - 浏览器信息
body - 请求 body 中的原始数据
type - 内容类型
length - 内容长度
query - 查询字符串参数
data - POST参数
cookies - Cookie参数
files - 上传的文件
secure - 是否是安全链接
accept - HTTP accept 参数
proxy_ip - 客户端的代理服务器IP地址
```

您可以以数组或是对象的方式访问 `query`, `data`, `cookies`, `files` 的属性。

例如，要获得一个查询字符串参数，你可以这样做：

```php
$id = Flight::request()->query['id'];
```

或者你也可以这样子:

```php
$id = Flight::request()->query->id;
```

# HTTP Caching HTTP缓存

Flight提供了HTTP级缓存的内置支持。如果缓存的条件得到满足，
Flight 将返回`304 Not Modified`的HTTP响应。
下一次客户端请求相同的资源， 他们将被提示使用其本地缓存版本。

## Last-Modified 最后修改

你可以使用`lastModified` 方法并传递一个UNIX时间戳（timestamp）来设定页面的最后修改时间。
客户端将继续使用其缓存，直到最后修改的值被改变。

```php
Flight::route('/news', function(){
    Flight::lastModified(1234567890);
    echo 'This content will be cached.';
});
```

## ETag

`ETag` 缓存类似于 `Last-Modified`，
但是它可以为资源指定ID ：

```php
Flight::route('/news', function(){
    Flight::etag('my-unique-id');
    echo 'This content will be cached.';
});
```

无论调用`lastModified` 或 `etag`都将设置和检查缓存值。
如果缓存的值和请求之间是相同的，
Flight 会立即发送一个`HTTP 304` 响应，并停止处理。

# Stopping

您可以在任何时候通过调用`halt`方法停止框架：

```php
Flight::halt();
```

您也可以指定一个可选的`HTTP`状态码和消息：

```php
Flight::halt(200, 'Be right back...');
```

调用`halt`将放弃任何回应的内容。
如果你想停止的框架并且要返回当前响应，使用`stop` 方法：

```php
Flight::stop();
```

# JSON

Flight提供了一个发送JSON和JSONP响应支持。
发送一个JSON响应，传递一些数据进行JSON编码：

```php
Flight::json(array('id' => 123));
```

对于JSONP请求，
可选参数传递您用来定义回调函数的查询参数名：

```php
Flight::jsonp(array('id' => 123), 'q');
```

例如，一个GET请求使用`?q=my_func` 你应该得到的输出:

```
my_func({"id":123});
```

如果不传递查询参数的名称将默认为 `jsonp`。


# Configuration 配置

您可以通过`set`方法设置配置值，自定义Flight的某些行为。

```php
Flight::set('flight.log_errors', true);
```

以下是所有可用的配置设置的列表：

    flight.base_url - 重写请求的基本url。 (默认：null)
    flight.handle_errors - 允许Flight在内部处理所有的错误。 (default: true)
    flight.log_errors - 错误记录到Web服务器的错误日志文件。 (default: false)
    flight.views.path - 包含视图模板文件的目录路径。 (default: ./views)

# Framework Methods 框架方法

Flight 被设计为易于使用和理解。
下面是一组完整的框架的方法。 
它由核心方法，常规的静态方法，以及可扩展并且可以映射可以被过滤或覆盖的方法组成。

## Core Methods 核心方法

```php
Flight::map($name, $callback) // 创建一个自定义的框架方法。
Flight::register($name, $class, [$params], [$callback]) // 注册一个类框架的方法。
Flight::before($name, $callback) // 框架的方法之前添加一个过滤器。
Flight::after($name, $callback) // 框架的方法之后添加一个过滤器。
Flight::path($path) // 添加路径到自动加载的类路径。
Flight::get($key) // 获取一个变量。
Flight::set($key, $value) // 设置一个变量。
Flight::has($key) // 检查一个变量是否被设置。
Flight::clear([$key]) // 清除变量。
Flight::init() // 按它的默认设置，初始化框架。
```

## Extensible Methods 扩展方法

```php
Flight::start() // 启动框架。
Flight::stop() // 停止框架并发送响应。
Flight::halt([$code], [$message]) // 停止框架，可选参数状态码和消息。
Flight::route($pattern, $callback) // 映射一个URL模式到回调函数。
Flight::redirect($url, [$code]) // 重定向到另一个url。
Flight::render($file, [$data], [$key]) // 渲染一个模板文件。
Flight::error($exception) // 发送一个HTTP 500响应。
Flight::notFound() // 发送一个HTTP 404响应。
Flight::etag($id, [$type]) // 执行的ETag HTTP缓存。
Flight::lastModified($time) // 执行last modified（最后修改）HTTP缓存
Flight::json($data, [$code], [$encode]) // 发送一个JSON响应。
Flight::jsonp($data, [$param], [$code], [$encode]) // 发送一个JSONP响应。
```

`map` 和 `register` 添加的任何自定义方法都可以被过滤。


# Framework Instance 框架实例

你可以选择作为一个对象实例运行它，而不是把Flight作为一个全局的静态类运行。

```php
require 'flight/autoload.php';

use flight\Engine;

$app = new Engine();

$app->route('/', function(){
    echo 'hello world!';
});

$app->start();
```

这样，你就可以调用与引擎对象同名的实例方法，而不是调用静态方法。
